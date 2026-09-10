# 지침과 스킬

[Codex 대응표로 돌아가기](../README.md) · 확인 기준: 2026.09.10.

## 전역·프로젝트 경로

| 종류 | 전역 | 프로젝트 |
|---|---|---|
| 상시 지침 | `~/.codex/AGENTS.md` | `<repo>/AGENTS.md` |
| 스킬 | `~/.agents/skills/<name>/SKILL.md` | `<repo>/.agents/skills/<name>/SKILL.md` |

전역 지침의 `.codex`와 문서상 전역 스킬 경로의 `.agents`가 다르다. 설치 방식에 따라 `~/.codex/skills`에도 스킬이 존재할 수 있으므로 기존 설치를 옮길 때는 현재 클라이언트의 발견 경로를 확인한다. `CODEX_HOME`을 별도로 지정했다면 전역 지침은 그 디렉터리를 기준으로 한다. [Customization](https://learn.chatgpt.com/docs/customization/overview)

지침은 전역 → 프로젝트 루트 → 현재 작업 디렉터리 방향으로 합쳐지고, 더 가까운 지침이 앞선 지침보다 우선한다. 같은 디렉터리에 `AGENTS.override.md`가 있으면 `AGENTS.md` 대신 읽는다. 루트 `AGENTS.md` 초안은 `/init`으로 만들 수 있다. [AGENTS.md 탐색 규칙](https://learn.chatgpt.com/docs/agent-configuration/agents-md)

이 저장소의 Claude 자산을 옮긴다면:

- [전역 CLAUDE.md](../../claude-code/global_settings/CLAUDE.md): Codex의 경로·용어로 수정해 전역 `AGENTS.md`에 반영한다.
- [answer-only 스킬](../../claude-code/skills/answer-only/SKILL.md): `name`·`description`·본문 구조를 유지해 Codex 스킬 디렉터리에 둔다. 호출 예시는 `$answer-only`다.
- [settings.json 예시](../../claude-code/global_settings/settings.json.example): 통째로 복사하지 않는다. 인증·모델·권한 설정은 Codex에서 별도로 구성한다.

스킬은 관련 작업에서 본문을 읽는 지침 묶음이다. 스킬을 설치하는 것만으로 실행 도구·화면 제어·네트워크 권한이 생기지는 않는다. 새 세션에서 지침·스킬 발견 여부를 확인하고, 파일 저장과 실제 로드·권한 적용을 구분한다.
