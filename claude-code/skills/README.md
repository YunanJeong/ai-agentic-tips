# Claude Code Skills

Claude Code에서 쓰는 개인 스킬 모음. 각 스킬은 `<name>/SKILL.md` 구조이며,
`~/.claude/skills/` 에 설치하면 모든 프로젝트에서 자동 로드된다.

## 수록 스킬

| 스킬 | 설명 |
|------|------|
| `answer-only` | 순수 Q&A 모드. "답만 해라 / 만들지 말고" 류 지시 시 구현·계획 없이 설명만. |
| `safety-guardrails` | 권한 확인이 꺼진 환경(`--dangerously-skip-permissions`)에서의 최소 보안 원칙. 민감정보·git/CLI·커밋·환경변수 취급. |

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
