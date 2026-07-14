# Claude Code Skills

Claude Code에서 쓰는 개인 스킬 모음. 각 스킬은 `<name>/SKILL.md` 구조이며,
`~/.claude/skills/` 에 설치하면 모든 프로젝트에서 자동 로드된다.

## 수록 스킬

| 스킬 | 설명 |
|------|------|
| `answer-only` | 순수 Q&A 모드. "답만 해라 / 만들지 말고" 류 지시 시 구현·계획 없이 설명만. |
| `safety-guardrails` | 권한 확인이 꺼진 환경(`--dangerously-skip-permissions`)에서의 최소 보안 원칙. 민감정보·git/CLI·커밋·환경변수 취급. |

## 발동 방식

스킬은 두 가지로 트리거된다.

| 방식 | 설명 |
|------|------|
| **자동 로드** | 스킬의 `description`에 걸리는 맥락이 오면 Claude가 알아서 로드. 매번 켤 필요 없음. |
| **수동 호출** | `/skill-name` 으로 직접 실행. |

- 수록 스킬 둘 다 **자동 로드**형이다. `answer-only`는 "답만 해라" 류 발화에서,
  `safety-guardrails`는 플랫폼·인프라·git·민감정보 관련 작업에서 로드된다.
- **매 메시지마다 켤 필요 없다.** 한 번 로드되면 그 세션 내내 유지된다.

**예시 (한 세션 흐름):**
```
나: "terraform 파일 좀 봐줘"
   → (Claude가 인프라 작업 감지 → safety-guardrails 자동 로드)
나: "이거 커밋해줘"
   → 이미 로드돼 있어 "커밋은 사용자 권한" 원칙이 적용됨. 다시 안 쳐도 됨.
```
> 굳이 세션 시작에 `/safety-guardrails`를 한 번 쳐두면 명시적으로 로드돼 더 확실하다(필수는 아님).

- ⚠️ 자동 로드는 "관련 있다고 판단될 때"라 **100% 보장은 아니다.** 항상 강제하려면
  핵심을 `~/.claude/CLAUDE.md`(매 세션 상시 로드)에 함께 두는 게 안전하다.

**해제**: 스킬은 껐다 켜는 스위치가 없다. 로드된 뒤 빠지는 방법은—
- `answer-only`: "이제 만들어줘 / go ahead" 류로 말하면 모드가 풀린다(SKILL에 해제 조건 명시).
- 특정 스킬을 아예 안 뜨게: `/skills` 메뉴에서 선택 후 토글(세션 넘어 지속).
- 그 세션만 무시: "이번엔 이 원칙 빼고 진행해"처럼 직접 지시. / 새 세션 시작 시 로드 상태는 초기화.

## 설치 방법

스킬은 아래 위치 중 하나에 두면 인식된다.

| 범위 | 위치 | 적용 대상 |
|------|------|-----------|
| 개인 | `~/.claude/skills/<name>/SKILL.md` | 내 모든 프로젝트 |
| 프로젝트 | `<repo>/.claude/skills/<name>/SKILL.md` | 그 프로젝트 (git 공유) |

### 방법 1 — 심링크 (권장)

이 저장소를 원본으로 두고 링크만 건다. 저장소에서 수정하면 바로 반영됨.

```bash
ln -s "$(pwd)/answer-only"        ~/.claude/skills/answer-only
ln -s "$(pwd)/safety-guardrails"  ~/.claude/skills/safety-guardrails
```
> `claude-code/skills/` 디렉토리에서 실행. Claude Code는 심링크를 따라가 `SKILL.md`를 읽는다.

### 방법 2 — 복사

저장소와 분리해 독립 관리하고 싶을 때.

```bash
cp -r answer-only safety-guardrails ~/.claude/skills/
```

### 방법 3 — 특정 프로젝트에만

팀과 공유하거나 프로젝트 한정으로 쓸 때. 대상 프로젝트에 복사 후 커밋.

```bash
cp -r safety-guardrails <프로젝트>/.claude/skills/
```

## 반영

- `~/.claude/skills/` 의 스킬 추가·수정·삭제는 **실행 중인 세션에 자동 반영**된다.
- 단, 세션 시작 시 없던 **최상위 skills 디렉토리를 새로 만든 경우**엔 Claude Code 재시작 필요.

## 확인

```bash
/skills          # 사용 가능한 스킬 목록
/answer-only     # 직접 호출 (이름으로)
```
