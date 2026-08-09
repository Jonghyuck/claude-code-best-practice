> 이 문서는 [`CLAUDE.md`](CLAUDE.md)의 한글 번역본입니다. 원본(영문)이 source of truth이며,
> 번역본과 원본이 어긋날 경우 원본을 따릅니다. 동기화 규칙은 [`.claude/rules/korean-sync.md`](.claude/rules/korean-sync.md) 참고.

# CLAUDE.md

이 파일은 이 저장소에서 작업할 때 Claude Code(claude.ai/code)에 지침을 제공합니다.

## 저장소 개요

이 저장소는 Claude Code 설정을 위한 베스트 프랙티스 저장소로, skills·subagents·hooks·commands의 패턴을 시연합니다. 애플리케이션 코드베이스가 아니라 레퍼런스 구현체 역할을 합니다.

## 핵심 구성 요소

### Weather System (예제 워크플로우)
**Command → Agent → Skill** 아키텍처를 통한 두 가지 스킬 패턴의 시연:
- `/weather-orchestrator` command (`.claude/commands/weather-orchestrator.md`): 진입점 — 사용자에게 C/F를 묻고, 에이전트를 호출한 뒤, SVG 스킬을 호출
- `weather-agent` agent (`.claude/agents/weather-agent.md`): preload된 `weather-fetcher` 스킬로 기온을 가져옴 (agent skill 패턴)
- `weather-fetcher` skill (`.claude/skills/weather-fetcher/SKILL.md`): 에이전트에 preload됨 — Open-Meteo에서 기온을 가져오는 지침
- `weather-svg-creator` skill (`.claude/skills/weather-svg-creator/SKILL.md`): Skill — SVG 날씨 카드를 생성하고 `orchestration-workflow/weather.svg`와 `orchestration-workflow/output.md`를 작성

두 가지 스킬 패턴: agent skills(`skills:` 필드로 preload) vs skills(`Skill` 도구로 호출). 전체 흐름 다이어그램은 `orchestration-workflow/orchestration-workflow.md` 참고.

### Skill 정의 구조
`.claude/skills/<name>/SKILL.md`의 스킬은 YAML frontmatter를 사용합니다:
- `name`: 표시 이름 및 `/slash-command` (기본값은 디렉토리 이름)
- `description`: 언제 호출할지 (자동 발견을 위해 권장)
- `argument-hint`: 자동완성 힌트 (예: `[issue-number]`)
- `disable-model-invocation`: 자동 호출을 막으려면 `true`
- `user-invocable`: `/` 메뉴에서 숨기려면 `false` (배경 지식 전용)
- `allowed-tools`: 스킬 활성 시 권한 프롬프트 없이 허용되는 도구
- `model`: 스킬 활성 시 사용할 모델
- `context`: 격리된 서브에이전트 컨텍스트에서 실행하려면 `fork`
- `agent`: `context: fork`일 때의 서브에이전트 타입 (기본값: `general-purpose`)
- `hooks`: 이 스킬에 스코프된 라이프사이클 훅

### Presentation System
`.claude/rules/presentation.md` 참고 — 프레젠테이션 작업은 프레젠테이션별로 `presentation-vibe-coding`(`presentation/vibe-coding-to-agentic-engineering/` 담당) 또는 `presentation-claude-gemini`(`presentation/2026-04-25-gdg-kolachi-cli-claude-code-gemini/` 담당)에 위임됩니다.

### Hooks System
`.claude/hooks/`의 크로스 플랫폼 사운드 알림 시스템:
- `scripts/hooks.py`: Claude Code 훅 이벤트의 메인 핸들러
- `config/hooks-config.json`: 공유 팀 설정
- `config/hooks-config.local.json`: 개인 오버라이드 (git-ignored)
- `sounds/`: 훅 이벤트별로 정리된 오디오 파일 (ElevenLabs TTS로 생성)

훅 이벤트는 `.claude/settings.json`에 설정됨: PreToolUse, PostToolUse, UserPromptSubmit, Notification, Stop, SubagentStart, SubagentStop, PreCompact, SessionStart, SessionEnd, Setup, PermissionRequest, TeammateIdle, TaskCompleted, ConfigChange.

특별 처리: git 커밋은 `pretooluse-git-committing` 사운드를 트리거합니다.

## 핵심 패턴

### Subagent 오케스트레이션
서브에이전트는 bash 명령으로 다른 서브에이전트를 호출할 수 **없습니다**. Agent 도구를 사용하세요 (v2.1.63에서 Task에서 이름 변경됨; `Task(...)`도 별칭으로 동작):
```
Agent(subagent_type="agent-name", description="...", prompt="...", model="haiku")
```

서브에이전트 정의에서 도구 사용을 명확히 하세요. bash 명령으로 오해될 수 있는 "launch" 같은 모호한 용어는 피하세요.

### Subagent 정의 구조
`.claude/agents/*.md`의 서브에이전트는 YAML frontmatter를 사용합니다:
- `name`: 서브에이전트 식별자
- `description`: 언제 호출할지 (자동 호출엔 "PROACTIVELY" 사용)
- `tools`: 도구 허용 목록(쉼표 구분). 생략 시 전체 상속. `Agent(agent_type)` 문법 지원
- `disallowedTools`: 거부할 도구, 상속/지정 목록에서 제거됨
- `model`: 모델 별칭: `haiku`, `sonnet`, `opus`, `inherit` (기본값: `inherit`)
- `permissionMode`: 권한 모드 (예: `"acceptEdits"`, `"plan"`, `"bypassPermissions"`)
- `maxTurns`: 서브에이전트가 멈추기 전 최대 에이전트 턴 수
- `skills`: 에이전트 컨텍스트에 preload할 스킬 이름 목록
- `mcpServers`: 이 서브에이전트용 MCP 서버 (서버 이름 또는 인라인 설정)
- `hooks`: 이 서브에이전트에 스코프된 라이프사이클 훅 (모든 훅 이벤트 지원; `PreToolUse`, `PostToolUse`, `Stop`이 가장 일반적)
- `memory`: 영속 메모리 스코프 — `user`, `project`, `local` (`reports/claude-agent-memory.md` 참고)
- `background`: 항상 백그라운드 작업으로 실행하려면 `true`
- `effort`: 노력 수준 오버라이드: `low`, `medium`, `high`, `max` (기본값: 세션에서 상속)
- `isolation`: 임시 git worktree에서 실행하려면 `"worktree"`
- `color`: 시각적 구분을 위한 CLI 출력 색상

### 설정 계층 구조
1. **Managed** (`managed-settings.json` / MDM plist / Registry): 조직 강제, 오버라이드 불가
2. 명령줄 인자: 단일 세션 오버라이드
3. `.claude/settings.local.json`: 개인 프로젝트 설정 (git-ignored)
4. `.claude/settings.json`: 팀 공유 설정
5. `~/.claude/settings.json`: 전역 개인 기본값
6. `hooks-config.local.json`이 `hooks-config.json`을 오버라이드

### 훅 비활성화
`.claude/settings.local.json`에 `"disableAllHooks": true`를 설정하거나, `hooks-config.json`에서 개별 훅을 비활성화하세요.

## 베스트 프랙티스 질문에 답하기

사용자가 Claude Code 베스트 프랙티스 질문을 하면, 훈련 지식이나 외부 소스에 의존하기 전에 **항상 이 저장소를 먼저 검색**하세요 (`best-practice/`, `reports/`, `tips/`, `implementation/`, `README.md`). 이 저장소가 권위 있는 출처이며, 여기서 답을 찾지 못한 경우에만 외부 문서나 웹 검색으로 넘어갑니다.

## 워크플로우 베스트 프랙티스

이 저장소 경험에서:

- 신뢰할 수 있는 준수를 위해 CLAUDE.md는 파일당 200줄 미만으로 유지
- `paths:` YAML frontmatter가 있는 `.claude/rules/*.md`는 Claude가 매칭 파일을 만질 때만 lazy-load됨; frontmatter가 없으면 CLAUDE.md처럼 모든 세션에 로드됨
- 독립 에이전트 대신 워크플로우엔 커맨드 사용
- 범용 에이전트 대신 스킬(점진적 공개)을 가진 기능별 서브에이전트 생성
- 컨텍스트 사용량 ~50%에서 수동 `/compact` 수행
- 복잡한 작업은 plan mode로 시작
- 다단계 작업엔 human-gated 태스크 리스트 워크플로우 사용
- 서브태스크는 컨텍스트 50% 미만으로 완료할 만큼 작게 쪼개기

### 디버깅 팁

- 진단엔 `/doctor` 사용
- 장시간 실행되는 터미널 명령은 로그 가시성을 위해 백그라운드 작업으로 실행
- 브라우저 자동화 MCP(Claude in Chrome, Playwright, Chrome DevTools)로 콘솔 로그 검사
- 시각적 이슈를 보고할 때 스크린샷 제공

## Git 커밋 규칙

변경을 커밋할 때 **파일당 별도 커밋을 생성**하세요. 여러 파일 변경을 하나의 커밋에 묶지 마세요. 각 파일은 그 파일 변경에 특화된 설명적 메시지로 자체 커밋을 가집니다.

예를 들어 `README.md`, `best-practice/claude-subagents.md`, 스킬 파일이 모두 변경됐다면:
- 커밋 1: `git add README.md` → README 특화 메시지로 커밋
- 커밋 2: `git add best-practice/claude-subagents.md` → subagents 문서 특화 메시지로 커밋
- 커밋 3: `git add .claude/skills/weather-fetcher/SKILL.md` → 스킬 특화 메시지로 커밋

이렇게 하면 git 히스토리가 깔끔해지고 개별 변경을 리뷰·되돌리기·cherry-pick하기 쉬워집니다.

## 문서화

문서화 표준은 `.claude/rules/markdown-docs.md` 참고. 핵심 문서:
- `best-practice/claude-subagents.md`: 서브에이전트 frontmatter, 훅, 저장소 에이전트
- `best-practice/claude-commands.md`: 슬래시 커맨드 패턴 및 내장 커맨드 레퍼런스
- `orchestration-workflow/orchestration-workflow.md`: Weather 시스템 흐름 다이어그램
