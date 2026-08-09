<!--
  이 문서는 implementation/claude-agent-teams-implementation.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Agent Teams Implementation

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar_12%2C_2026-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#time-orchestration"><img src="../!/tags/implemented-hd.svg" alt="Implemented"></a>

<p align="center">
  <img src="assets/impl-agent-teams.png" alt="Agent Teams in action — split pane mode with tmux" width="100%">
</p>

Agent Teams는 공유 태스크 목록을 통해 서로 협력하는 **여러 개의 독립적인 Claude Code 세션**을 생성합니다. (하나의 세션 안에서 격리된 컨텍스트로 분기되는) 서브에이전트와 달리, 각 팀원은 CLAUDE.md, MCP 서버, 스킬이 자동으로 로드된 자신만의 완전한 컨텍스트 윈도우를 가집니다.

---

## ![How to Use](../!/tags/how-to-use.svg)

시간 오케스트레이션 워크플로는 전적으로 에이전트 팀에 의해 구축되었습니다. 완성된 결과물을 실행하려면:

```bash
cd agent-teams
claude
/time-orchestrator
```

이는 **Command → Agent → Skill** 파이프라인을 호출합니다. 에이전트가 두바이의 현재 시간을 가져오고, 스킬이 SVG 시간 카드를 `agent-teams/output/dubai-time.svg`에 렌더링합니다.

---

## ![How to Implement](../!/tags/how-to-implement.svg)

에이전트 팀을 사용하여 날씨 오케스트레이션 워크플로의 복제본을 만들 수 있습니다 — 이 예시에서는 시간 오케스트레이션 워크플로가 전적으로 에이전트 팀에 의해 구축되었습니다.

### 1. Install [iTerm2](https://iterm2.com/) and tmux

```bash
brew install --cask iterm2
brew install tmux
```

### 2. Start iTerm2 → tmux → Claude

```bash
tmux new -s dev
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 claude
```

### 3. Prompt with team structure

<a id="time-orchestration"></a>

에이전트 팀을 사용하여 완전한 시간 오케스트레이터 워크플로를 부트스트랩하려면 이 프롬프트를 Claude에 붙여넣으세요:

Main prompt: **[agent-teams-prompt.md](../agent-teams/agent-teams-prompt.md)**

### Team Coordination Flow

```
┌──────────────────────────────────────────────────────────────┐
│                         LEAD (You)                           │
│       "Create an agent team to build time orchestration"     │
└──────────────────────────┬───────────────────────────────────┘
                           │ spawns team (all parallel)
              ┌────────────┼────────────┐
              ▼            ▼            ▼
   ┌────────────────┐ ┌──────────┐ ┌──────────────┐
   │ Command        │ │ Agent    │ │ Skill        │
   │ Architect      │ │ Engineer │ │ Designer     │
   │                │ │          │ │              │
   │ agent-teams/   │ │ agent-   │ │ agent-teams/ │
   │ .claude/       │ │ teams/   │ │ .claude/     │
   │ commands/      │ │ .claude/ │ │ skills/      │
   │ time-          │ │ agents/  │ │ time-svg-    │
   │ orchestrator.md│ │ time-    │ │ creator/     │
   │                │ │ agent.md │ │              │
   └───────┬────────┘ └────┬─────┘ └──────┬───────┘
           │               │              │
           ▼               ▼              ▼
   ┌──────────────────────────────────────────────────┐
   │            Shared Task List                      │
   │  ☐ Agree on data contract: {time, tz, formatted} │
   │  ☐ Command uses Agent tool (not bash)            │
   │  ☐ Agent preloads time-fetcher skill             │
   │  ☐ Skill reads time from context (no re-fetch)   │
   │  ☐ All files inside agent-teams/.claude/         │
   └──────────────────────────────────────────────────┘
                       │
                       ▼
          ┌──────────────────────────────┐
          │  cd agent-teams && claude    │
          │    /time-orchestrator        │
          │   Command → Agent → Skill    │
          └──────────────────────────────┘
```
