# CLI 실행과 작업 흐름

[Codex 대응표로 돌아가기](../README.md) · 확인 기준: 2026.09.10.

## 권한 옵션과 세션 재개

```sh
# 승인·Codex 샌드박스 해제해서 시작
codex --dangerously-bypass-approvals-and-sandbox

# 같은 옵션으로 최근 세션 재개
codex resume --last --dangerously-bypass-approvals-and-sandbox

# 현재 디렉터리 필터 없이 세션 선택
codex resume --all
```

`-a never`는 승인 질문만 끄며 샌드박스를 해제하지 않는다. 따라서 막힌 명령이 질문 없이 실패할 수 있다. 전체 권한 해제는 Codex의 실행 경계를 바꾸는 옵션이지, 원격 환경에 사용자 PC를 연결하거나 OS·컨테이너·관리자 제한을 없애는 옵션은 아니다.

이 저장소의 [Claude 권한 설명](../../claude-code/reference/permissions.md)에 있는 `permissions.deny` 동작을 Codex에 그대로 대입하지 않는다. Claude의 `Bash(...)`·`Read(...)` 규칙은 Codex 설정 문법이 아니며, `AGENTS.md` 역시 기계적 차단이 아닌 모델 지침이다. [Codex 샌드박스 문서](https://learn.chatgpt.com/docs/sandboxing)

## goal과 plan

```text
/plan
/goal PLAN.md를 구현하고, 지정한 검증을 통과하면 종료해.
/goal
/goal pause
/goal resume
/goal clear
```

- `/plan`: 구현 전 계획을 만드는 모드. 결과를 `PLAN.md`로 남기는 것은 선택이며 파일명에 특별한 실행 기능은 없다.
- `/goal <목표>`: 지속할 목표를 설정한다. 목표·범위·검증 방법·종료 조건을 구체적으로 적는다.
- `/goal`: 상태 확인. `pause`·`resume`·`clear`는 목표 실행 제어다. `codex resume`의 세션 재개와 구분한다.
- goal은 권한 확대 기능이 아니다. 사용자 입력이나 외부 상태 변화 없이는 진행할 수 없는 경우 차단 상태가 될 수 있다.

`/goal`이 보이지 않으면 다음 기능 설정을 활성화한다.

```sh
codex features enable goals
```

출처: [Follow a goal](https://learn.chatgpt.com/use-cases/follow-goals), [Developer commands](https://learn.chatgpt.com/docs/developer-commands?surface=cli).

