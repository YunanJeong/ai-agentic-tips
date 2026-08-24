# Claude Code 부트스트랩 킷

새 환경의 Claude Code를 안전한 기본값으로 초기 설정할 때 쓰는 원본.
프로필 디렉터리(`$CLAUDE_CONFIG_DIR`, 미설정이면 `~/.claude`)에 파일 2개를 넣으면 끝이다.

## 새 환경에 깔 때

### 1. settings.json 의 permissions — 사람이 직접

`global_settings/settings.json.example` 의 `permissions` 블록을 프로필 디렉터리의
`settings.json` 에 손으로 옮긴다. 파일이 없으면 example을 그대로 복사한다. 끝나면 `chmod 600`.

Claude에게 시키지 않는다. 이 파일의 `env` 에 실제 API 키가 있어서 Claude가 열면 그 값이
대화에 들어온다. 그리고 차단이 없는 상태로 Claude를 붙이는 것은 순서가 뒤바뀐 것이다.

### 2. 나머지 — Claude에게

```
claude-code/ 를 전부 읽고 "적용" 절차대로 현재 세션 프로필에 반영해라.
```

## 보안 구조

```
settings.json  ← permissions.deny   기계적 차단. 프로그램이 집행, 대화로 못 풂
CLAUDE.md      ← 지시문             패턴으로 못 막는 판단. 매 세션 자동 로드
```

**`deny` 는 `--dangerously-skip-permissions` 에서도 살아 있다.** 그 옵션은 확인 프롬프트를
없애는 것이고, `deny` 는 프롬프트가 아니라 차단이다. 그래서 `deny` 만 쓴다.

패턴으로 쓸 수 없는 건 `CLAUDE.md` 가 받는다. "직접 실행 가능한 git 명령은 4개뿐"이라는
화이트리스트나 `echo $SECRET` 은 규칙으로 표현이 안 된다.

`aws` 나 `terraform apply` 가 필요하면 **Claude가 명령과 이유를 제시하고 사용자가 실행한다.**
풀어주는 스위치는 없다. 그게 보안 목표 자체다.

## 적용

이 킷 디렉터리(`claude-code/`)에서 실행한다.

```sh
PROFILE="${CLAUDE_CONFIG_DIR:-$HOME/.claude}"

cp global_settings/CLAUDE.md "$PROFILE/CLAUDE.md"
mkdir -p "$PROFILE/skills" && cp -r skills/* "$PROFILE/skills/"
```

`settings.json` 은 손대지 않는다. 열지도 않는다.

**적용한 세션에서 검증하지 마라.** 설정은 세션 시작 시 로드되므로 방금 쓴 `deny` 는 그
세션에 반영되지 않는다.

## 검증

새 세션에서 `"aws sts get-caller-identity 실행해봐"` → 차단돼야 정상.
로드 상태는 `/permissions`, `/memory`, `/skills` 로 본다.

## 한계

**`deny` 는 사고성 실수를 막는 그물이고 의도적 우회는 못 막는다.** 명령 앞부분으로
판정하므로 `python -c` 나 `git -C <경로> <명령>` 처럼 접두사를 비껴가면 안 걸린다
(`bash -c`·`sh -c`·`zsh -c` 는 막아뒀다). 그 영역은 `CLAUDE.md` 의 우회 금지 조항이 받는다.

## 환경마다 다시 정할 것

리전, 모델·추론 프로파일 ARN, Bedrock API 키 — `settings.json` 의 `env`·`model` 이다.
리전은 추론 프로파일 리전에 맞춘다(일부 최신 글로벌 모델은 `us-east-1`). 키 이름은
`AWS_BEARER_TOKEN_BEDROCK`(`AWS_BEARER_TOKEN` 아님). 저장소에 커밋하지 않는다.
Bedrock이 아니면(구독 로그인 등) `env`·`model` 은 지운다 — `CLAUDE_CODE_USE_BEDROCK` 이
남으면 인증이 깨진다.

ARN·비용 태깅은 `reference/claude-code.md`, 계정별 프로필 분리는
`reference/multi-profile.md`, `deny` 문법과 동작은 `reference/permissions.md`.
