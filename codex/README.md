# Codex

Claude Code에 익숙한 사용자를 위한 로컬 Codex CLI 전환 가이드. 명령·지침 파일·스킬의 대응 관계와 차이를 정리한다.

| Claude Code에서 쓰던 것 / 개념 | Codex 대응 | 사용법 |
|---|---|---|
| `--dangerously-skip-permissions` | `--dangerously-bypass-approvals-and-sandbox` | 승인 질문과 Codex 샌드박스를 모두 해제. 권한 체계가 달라 완전히 동일한 의미는 아님 |
| `--resume` | `codex resume` | 플래그가 아니라 서브커맨드. 저장된 세션 선택 |
| 최근 세션 이어가기 | `codex resume --last` | 선택창 없이 최근 세션 재개 |
| 특정 세션 이어가기 | `codex resume <SESSION_ID>` | 세션 ID 지정. 실행 중에는 `/resume` 사용 |
| 목표까지 계속 작업하는 goal 개념 | `/goal <목표>` | 턴을 넘어 검증 가능한 완료 조건까지 작업을 지속 |
| plan mode | `/plan` | 조사·설계·계획 작성 모드 |
| skill | `<스킬이름>/SKILL.md` | `$스킬이름`으로 호출하거나 description에 맞는 작업에서 자동 선택 |
| 전역 `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` | 프로젝트 공통 개인 지침 |
| 프로젝트 `CLAUDE.md` | 프로젝트 루트 `AGENTS.md` | 프로젝트 지침. 하위 디렉터리에도 배치 가능 |

명령 출처: [Developer commands](https://learn.chatgpt.com/docs/developer-commands?surface=cli), [권한과 승인](https://learn.chatgpt.com/docs/agent-approvals-security). `resume`과 전체 권한 해제 옵션은 설치된 CLI의 `codex --help`, `codex resume --help`로도 확인했다.

## 문서 구성

| 문서 | 내용 |
|---|---|
| [CLI 실행과 작업 흐름](reference/cli-workflows.md) | 권한 옵션, 세션 재개, goal, plan |
| [지침과 스킬](reference/instructions-and-skills.md) | 전역·프로젝트 경로, 우선순위, Claude 자산 이식 시 차이 |
| [초기 사용 기록](../tools/codex-cli.md) | 2026.02 시점의 사용 경험·평가 |

## 이 디렉터리의 범위

`tools/`의 짧은 도구 사용기와 구분해 Codex의 지속적으로 참고할 사용법·전환 지식을 모은다. 확인 기준일은 2026.09.10.이며, 버전·클라이언트에 따라 달라지는 동작은 각 문서의 공식 출처와 현재 CLI 도움말로 확인한다.

[claude-code/](../claude-code/README.md)는 설정·스킬 원본까지 갖춘 부트스트랩 킷이다. 이 디렉터리는 현재 설명 문서만 제공한다. 실제로 검증된 재사용 설정·스킬 원본이 생기면 그때 `global_settings/`, `skills/`를 추가하고 적용·검증 절차를 문서화한다. 설명 문서를 읽는 것과 로컬 설정을 설치하는 것은 별도 작업이다.
