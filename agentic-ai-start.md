# Agentic AI 실무 구현 정리

개념은 익숙한 상태에서, "실무로 개발을 시작하려면 뭘 어디서부터 손대나"를 정리한 문서.

---

## 0. 큰 그림

층으로 나뉜다:

```
raw HTTP  <  벤더 SDK  <  SDK + 직접 루프  <  tool_runner / Agent SDK  <  프레임워크
 (거의 안씀)  (호출·전송)     (흔한 실무)          (루프도 위임)             (복잡할 때만)
```

- **API 호출은 SDK로.** (`anthropic`, `openai` 등) raw HTTP는 원리 설명용이지 실무에서 손으로 짜지 않는다.
- 갈리는 지점은 "그 위의 루프를 내가 짜냐 vs 헬퍼/프레임워크에 맡기냐" 뿐.

---

## 1. Agentic loop 골격

```python
messages = [{"role": "user", "content": "사용자 요청"}]

while True:
    resp = llm.call(messages, tools=my_tools)

    if resp.stop_reason != "tool_use":       # 도구 안 쓰면 끝
        break

    for tool_call in resp.tool_calls:        # LLM이 "이 도구 써줘"
        result = execute(tool_call)          # 내 코드가 실행
        messages.append(tool_result(result)) # 결과를 대화에 되먹임
```

- LLM은 "이 도구를 이 인자로 써줘"를 내놓고, 실제 실행은 내 코드가 한다.
- **Agent vs Workflow**: 흐름을 코드가 하드코딩하면 workflow, 다음 행동·종료를 LLM이 정하면 agent. 루프는 그릇, 알맹이는 자율적 의사결정 + 도구 사용.

---

## 2. `messages`

- 내가 만들고 append 하는 리스트. 이름 `messages`는 API가 요구하는 파라미터 이름.
- API는 stateless → 매 호출마다 전체 히스토리를 다시 보낸다.
- 도는 동안엔 RAM 위 변수. **DB는 기본 불필요** — 지속성(세션 넘김·crash 복구), RAG(벡터DB), 감사/트레이싱이 필요해질 때만 얹는다.

---

## 3. tool use (= function calling)

- 내가 쓰던 "트리거"의 정식 이름: **tool use**(Anthropic) / **function calling**(OpenAI).
- LLM 응답의 tool_use 블록에서 `name` 보고 실제 함수를 dispatch:

```python
FUNCS = {"조회": get_x, "삭제": delete_y}   # 이름 → 함수 매핑

for block in resp.content:
    if block.type == "tool_use":
        result = FUNCS[block.name](**block.input)
```

---

## 4. `tools` 파라미터 — 소유권

```python
my_tools = [{
    "name": "조회",
    "description": "이럴 때 조회를 써라",   # LLM이 언제 쓸지 판단하는 근거
    "input_schema": {"type": "object", "properties": {"x": {"type": "string"}}, "required": ["x"]}
}]

resp = client.messages.create(model=..., tools=my_tools, messages=messages)
```

| 대상 | 누구 것 |
|---|---|
| `tools` 칸/이름 | Anthropic이 정한 고정 파라미터 이름 |
| 넣는 값(내용물) | 내가 작성 |
| 칸에 실어 보내는 배관(직렬화·전송) | SDK가 구현 |

- `tools` = Anthropic API의 구현 표면. 그 뒤 발상은 **function calling이라는 벤더 초월 설계 개념**. (OpenAI도 같은 개념을 `tools`로, 스키마 모양만 조금 다르게 노출)

---

## 5. `create()`는 1회 전송

```python
resp = client.messages.create(model=..., tools=my_tools, messages=messages)
# 요청 1번 → 응답 1번. 루프 아님 = 루프의 한 바퀴.
```

- stateless라서 `tools=`도 매 호출마다 실어 보낸다.

---

## 6. 루프: 직접 vs 위임

### 직접 (while 내가 작성)
```python
while True:
    resp = client.messages.create(model=..., tools=my_tools, messages=messages)
    if resp.stop_reason != "tool_use":
        break
    messages.append({"role": "assistant", "content": resp.content})
    tool_results = [...]   # 도구 실행 결과
    messages.append({"role": "user", "content": tool_results})
```

### 위임 (tool_runner / Agent SDK)
```python
@beta_tool
def 조회(x: str) -> str:
    return ...

runner = client.beta.messages.tool_runner(model=..., tools=[조회], messages=messages)
for message in runner:   # while를 내가 안 짬
    print(message)
```
runner가 대신하는 것: 반복 호출 + `tools` 실어보내기 + dispatch + 되먹임 + 종료 판단. 코드량이 크게 준다. 도구 설계·스키마·프롬프트는 여전히 내 몫.

### tool_runner vs Claude Agent SDK
| | tool_runner | Claude Agent SDK |
|---|---|---|
| 정체 | 벤더 SDK 안의 얇은 루프 헬퍼 | Claude Code를 라이브러리화한 별도 패키지 |
| 도구 | 내가 정의한 것만 | 내장 도구(파일·bash·grep·웹검색) + 내 것 + MCP |
| 무게 | 가벼움 | 무거움(코딩/파일 에이전트 통짜) |

---

## 7. 직접 vs 위임 판단 기준

둘 다 흔하다. 지배적 표준은 없다.

**직접 while로 가는 이유:**
- 루프 구조 자체가 특이(외부 큐/비동기 워커로 도구 실행, 특이한 다중 LLM 호출 등)
- beta 의존성 회피 (tool_runner는 beta)
- 완전한 통제 / 투명성

**예외처리·if 분기는 위임 상태에서도 된다.** tool_runner는 블랙박스가 아니라 매 스텝 개입 훅을 준다:
1. 도구 함수 내부에서 분기
   ```python
   @beta_tool
   def delete_file(path: str) -> str:
       if is_dangerous(path):
           return "거부됨: 승인 필요"   # 실행 안 하고 되먹임
       return actually_delete(path)
   ```
2. `for message in runner:` 회차마다 들여다보고 승인/거부/파라미터 변경/early stop

| 하려는 것 | 직접 while 필요? |
|---|---|
| 예외처리·승인·에러 분기·재시도·early stop | 아니오 (함수 내부/훅으로) |
| 루프 구조 자체가 특이 | 예 |
| beta 회피 / 완전 통제 | 예 |

### 프레임워크(LangGraph 등)
- 시작점이 아니다. 멀티에이전트·복잡한 분기·상태관리가 실제로 커졌을 때 얹는 선택지.
- Anthropic 입장: 대부분 SDK 직접 호출로 충분, 프레임워크는 추상화로 디버깅을 어렵게 만들 수 있다.

---

## 8. Skills ≠ tool use

- **tool use** = 실행 메커니즘(손발).
- **Skills** = 작업용 지침·문서 묶음(`SKILL.md`), 관련 작업일 때만 전체를 읽어들임(progressive disclosure).
- Skills(지침)가 tool use(실행)를 활용하는 상하 관계. Skills는 tool search(도구 스키마 지연 로딩)와 발상이 가깝다.

---

## 9. Claude Code / OpenClaw 같은 앱

- tool use 루프 + 실제 도구 세트(파일·bash·grep 등) + 에이전트 로직을 구현한 앱.
- OpenClaw는 "Claude와 도구 사이를 잇는 릴레이/미들웨어형 코딩 에이전트"로 파악됨 — 이 릴레이 구조가 곧 우리가 말한 tool use 루프.
- 전송 배관은 SDK/HTTP 계층이 담당, 그 위를 앱이 구현.

```
전송 배관 (tools 실어보내기)              → SDK
tool use 루프 + 도구들 + 에이전트 흐름   → 앱
```

---

## 10. "좋은 agent 앱 = 루프 잘 짬" 은 아니다

루프는 필요조건일 뿐, 오히려 제일 쉬운 부분. 품질은 루프 바깥에서 갈린다.

```
좋은 agent 앱 = agentic loop (그릇)
              + 도구 설계 (핵심)
              + 컨텍스트 관리 (요약·압축·잘라내기)
              + 프롬프트 / 에이전트 로직
              + 에러·가드레일·관측성·비용/지연·UX
```

---

## 11. 실무 시작 체크리스트

1. 벤더 SDK 설치. API 호출은 이걸로.
2. `create()` 1회 호출부터 — 요청→응답, tool_use 블록 형태 확인.
3. 도구 1~2개를 스키마로 정의해서 tool use 흐름 확인.
4. 루프: 배우는 단계·특이 흐름·beta 회피면 직접 while, 빠르게 굴리려면 tool_runner.
5. 여기까지가 그릇. 시간은 도구 설계·프롬프트·컨텍스트 관리·가드레일에 쓴다.
6. 멀티에이전트·복잡한 분기가 실제로 생기면 그때 프레임워크 검토.
