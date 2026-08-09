<!--
  이 문서는 agent-teams/agent-teams-prompt.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

현재 두바이 시각을 시각적인 SVG 카드로 표시하는 시간 오케스트레이션
워크플로우를 구축하는 에이전트 팀을 만드세요. 이 워크플로우는
Command → Agent → Skill 아키텍처 패턴을 따릅니다:

- 커맨드는 전체 흐름을 오케스트레이션하고 사용자 상호작용을 처리합니다
- 에이전트는 미리 로드된 스킬을 사용해 두바이의 실시간 현재 시각을 가져옵니다
- 스킬은 가져온 데이터로부터 시각적인 SVG 시간 카드를 생성합니다

**중요**: 모든 파일은 `agent-teams/.claude/` 내부에 생성해야 하며 —
저장소 루트의 `.claude/` 디렉터리에 만들어서는 안 됩니다. 이렇게 하면
에이전트 팀의 산출물이 자체 완결적으로 유지되며 `cd agent-teams && claude`로
실행할 수 있습니다.
기존 weather 워크플로우를 참조하거나 복사하지 마세요 — 모든 것을 처음부터 구축하세요.

다음 팀원들을 배정하세요:

1. **Command Architect** — `agent-teams/.claude/commands/time-orchestrator.md`에
   `/time-orchestrator` 커맨드를 설계하고 구현합니다. 이 커맨드는 다음을 수행해야 합니다:
   - Agent 도구를 통해(bash가 아님) time-agent를 호출하여 두바이, UAE
     (Asia/Dubai 타임존, UTC+4)의 현재 시각을 가져옵니다
   - Skill 도구를 통해 time-svg-creator 스킬을 호출하여 가져온 시각 데이터로부터
     SVG 카드를 렌더링합니다
   - frontmatter에서 model: haiku를 사용합니다
   - 다음 핵심 요구사항을 포함합니다: 순차적 흐름, 올바른 도구 사용
     (에이전트에는 Agent 도구, 스킬에는 Skill 도구), 그리고 출력 요약
   컴포넌트 간에 전달되는 데이터 계약({time, timezone, formatted})에 합의하기
   위해 공유 태스크 리스트를 통해 다른 팀원들과 조율하세요.

2. **Agent Engineer** — `agent-teams/.claude/agents/time-agent.md`에
   `time-agent`를 설계·구현하고, `agent-teams/.claude/skills/time-fetcher/SKILL.md`에
   미리 로드되는 `time-fetcher` 스킬을 구현합니다. 이 에이전트는 다음을 수행해야 합니다:
   - `TZ='Asia/Dubai' date '+%Y-%m-%d %H:%M:%S %Z'`를 사용하는 Bash로
     두바이(Asia/Dubai, UTC+4)의 현재 시각을 가져옵니다
   - 시각 값, 타임존 이름, 그리고 포맷된 문자열을 커맨드에 반환합니다
   - frontmatter 사용: tools (Bash), model: haiku, color: blue, maxTurns: 3
   - `skills:` 필드를 통해 time-fetcher 스킬을 미리 로드합니다
   time-fetcher 스킬(`agent-teams/.claude/skills/time-fetcher/SKILL.md`)에는
   두바이 시각을 가져오는 bash 명령, 예상 출력 형식이 포함되어야 하며,
   에이전트 전용 도메인 지식이므로 user-invocable: false로 설정해야 합니다.
   합의된 데이터 계약을 공유 태스크 리스트에 게시하여 Command Architect와
   Skill Designer가 인터페이스에 대해 정렬할 수 있도록 하세요.

3. **Skill Designer** — `agent-teams/.claude/skills/time-svg-creator/SKILL.md`에
   `time-svg-creator` 스킬을 설계·구현하고, 보조 파일 `reference.md`(SVG 템플릿 +
   출력 템플릿)와 `examples.md`(입출력 예시 쌍)를 함께 구현합니다.
   이 스킬은 다음을 수행해야 합니다:
   - 호출 컨텍스트로부터 시각 값, 타임존, 포맷된 문자열을 전달받습니다
   - 현재 시각을 보여주는 자체 완결적인 두바이 SVG 시간 카드를 생성합니다
   - SVG를 `agent-teams/output/dubai-time.svg`에 작성합니다
   - 마크다운 요약을 `agent-teams/output/output.md`에 작성합니다
   - 제공된 시각을 정확히 사용합니다 — 절대 다시 가져오지 않습니다
   - 템플릿은 reference.md에(플레이스홀더가 있는 SVG 마크업, 마크다운 출력 템플릿),
     예시 쌍은 examples.md에 보관합니다
   또한 출력 파일을 위한 `agent-teams/output/` 디렉터리를 생성하세요.

세 팀원 모두 데이터 계약을 조율하기 위해 공유 태스크 리스트에 태스크를
생성해야 합니다: 에이전트는 {time, timezone, formatted}를 반환하고,
커맨드는 이를 컨텍스트를 통해 전달하며, 스킬은 이를 소비합니다.
컴포넌트들은 서로 독립적이므로 세 작업을 병렬로 시작하세요 —
서로의 구현을 기다릴 필요 없이 데이터 인터페이스에만 합의하면 됩니다.
