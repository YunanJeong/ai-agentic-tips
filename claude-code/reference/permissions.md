# permissions 동작

## allow / ask / deny

`settings.json` 의 `permissions` 아래에 있는 **나란한 세 개의 키**다. 각각 규칙 문자열
배열이고, 필요한 것만 써도 된다.

```json
{
  "permissions": {
    "allow": ["Bash(npm run test:*)"],
    "ask":   ["Bash(git push:*)"],
    "deny":  ["Read(**/.env)"]
  }
}
```

| 키 | 뜻 |
|----|-----|
| `allow` | 확인 없이 자동 실행 |
| `ask` | 매번 확인 프롬프트를 띄운다 |
| `deny` | 무조건 차단. 프롬프트도 안 뜬다 |

우선순위는 **`deny` > `ask` > `allow`**. `deny` 에 걸린 것은 `allow` 에 같이 적혀 있어도,
더 구체적으로 적어도 차단된다.

## 왜 이 킷은 deny만 쓰나

`--dangerously-skip-permissions` 는 "확인 프롬프트를 띄우지 않는다"는 뜻이다. 그래서
프롬프트에 의존하는 `allow`(자동 승인할 게 없어짐)와 `ask`(안 띄움)는 의미를 잃고
**`deny` 만 계속 차단한다.** 공식 문서 표현: *"Deny rules block in every mode, including
bypassPermissions"*.

`deny` 는 **대화로도 못 푼다.** 집행 주체가 모델이 아니라 Claude Code 프로그램이라
"이번만 허용해줘"라고 해도 Claude가 스스로 풀 수 없다. 사용자가 `/permissions` 로 지우거나
`settings.json` 을 고치는 것만 가능하다.

## 규칙 문법

`도구이름(대상)` 형태다.

```
Bash(git commit:*)     git commit 으로 시작하는 모든 명령
Bash(env)              정확히 env 라는 명령 (인자 없는 형태는 따로 써야 걸린다)
Read(**/.env)          어느 경로든 .env 파일 읽기
Read(**/.aws/**)       .aws 디렉터리 안의 모든 것
```

`Bash(...)` 는 명령의 **앞부분 일치**로, `Read(...)` 는 경로 패턴으로 판정한다.
`Read` 규칙은 Read 도구뿐 아니라 `cat` 같은 Bash 명령으로 읽으려는 경우도 잡는다.

## 못 막는 것

앞부분 일치라서 앞부분을 바꾸면 비껴간다.

```
bash -c 'cat .env'        따로 막지 않으면 통과
git -C /경로 commit       Bash(git commit:*) 에 안 걸림
echo $SECRET              애초에 패턴으로 표현 불가
```

첫 줄은 `bash -c`·`sh -c`·`zsh -c` 자체를 막아 대응했지만 `python -c`, `perl -e` 등이
남아 **패턴만으로 완전 봉쇄는 불가능하다.** 그래서 막을 수 있는 것은 `deny` 에, 못 막는
것은 상시 로드되는 `CLAUDE.md` 에 나눠 둔다.
