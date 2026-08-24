# 클로드 멀티 프로필(다계정, 다인증 사용시 로컬에서 적용법)

```sh
# 1. 계정별 디렉터리 생성
mkdir -p ~/.claude-a ~/.claude-b

# 2. 셸 설정 파일에 alias 추가 (~/.zshrc 기준, bash면 ~/.bashrc)
cat << 'EOF' >> ~/.bashrc

# Claude Code Multi-Account Setup
alias claude-a="CLAUDE_CONFIG_DIR=~/.claude-a CLAUDE_USE_KEYCHAIN=false claude"
alias claude-b="CLAUDE_CONFIG_DIR=~/.claude-b CLAUDE_USE_KEYCHAIN=false claude"
EOF

# 3. 변경 사항 즉시 반영
source ~/.bashrc

# 4. 터미널 1에서 'claude-a' 실행 후 A 계정 로그인
# 5. 터미널 2에서 'claude-b' 실행 후 B 계정 로그인 (시크릿 브라우저 권장)
```

## `CLAUDE_USE_KEYCHAIN=false` 적용 이유

* 기본 상태: OS 암호화 금고(키체인)에 토큰을 저장함. 보안은 높지만 OS 전체에서 토큰 1개만 공유하므로 다른 계정으로 로그인 시 기존 토큰이 덮어씌워져 세션이 꼬임.
* `false` 설정: OS 키체인을 끄고 지정한 폴더(`~/.claude-a/` 등)에 토큰을 평문 파일로 직접 저장함.
* 결과: 보안 수준은 OS 암호화보다 낮아지지만, 계정별로 토큰 저장 파일 경로가 완전히 쪼개져 터미널 간 충돌 없이 동시 실행이 가능해짐.


## `~/.claude-X/claude-settings.json` 환경변수(env) 격리 요약

* 계정 A: `~/.claude-a/claude-settings.json` (또는 `~/.claude-a/settings.json`)
* 계정 B: `~/.claude-b/claude-settings.json` (또는 `~/.claude-b/settings.json`)

### 1. 격리 여부
* 완벽 격리됨 (타 터미널 및 OS 시스템 환경변수에 영향 없음).

### 2. 격리 원리
* `CLAUDE_CONFIG_DIR`로 지정된 디렉터리의 설정 파일만 각각 독립적으로 읽음.
* 설정 파일 내 `"env": { ... }` 블록은 해당 Claude 프로세스 및 하위 실행 도구(Subshell)에만 프로세스 레벨로 인라인 주입됨.

### 3. 계정별 적용 흐름
* `claude-a` 실행 -> `~/.claude-a/` 내 설정의 `env`만 주입 (A 세션에만 적용)
* `claude-b` 실행 -> `~/.claude-b/` 내 설정의 `env`만 주입 (B 세션에만 적용)


## `~/.claude.json` 역할 및 멀티계정 영향 요약

* **역할**: 신뢰할 프로젝트 폴더 목록(`trustedDirectories`), CLI 온보딩 여부, 전역 공지 플래그 등을 담는 메타데이터 파일
* **멀티계정 영향**: 계정 인증 토큰, 세션, `env` 환경변수는 `CLAUDE_CONFIG_DIR` 디렉터리에 격리되므로, `~/.claude.json` 공유로 인한 **계정 충돌 및 간섭은 없음**

## 각 환경별 settings.json 예시
  - model은 켰을 때 초기 default 선택 모델을 의미
  - 특정 모델 등록은 `ANTHROPIC_MODEL`환경변수에 의해 적용됨
  - 프로필 별로 theme를 다르게 두면 편하다.
  - 리전은 현 지역이 아닌, us-east-1으로 두는 것이 여러 글로벌모델활용에 좋다.
```json
{
  "theme": "dark",
  "model": "arn:aws:bedrock:us-east-1:123456789012:application-inference-profile/your-profile-id",
  "env": {
    "CLAUDE_CODE_USE_BEDROCK": "1",
    "AWS_REGION": "us-east-1",
    "AWS_BEARER_TOKEN": "your-bedrock-bearer-token-here",
    "ANTHROPIC_MODEL": "arn:aws:bedrock:us-east-1:123456789012:application-inference-profile/your-profile-id"
  }
}
```