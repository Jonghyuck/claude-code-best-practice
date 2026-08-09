<!--
  이 문서는 reports/claude-global-vs-project-settings.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Claude Code: Global vs Project-Level Features

어떤 Claude Code 기능이 전역 전용(`~/.claude/`)인지, 그리고 어떤 기능이 전역과 프로젝트 수준(`.claude/`) 양쪽에 대응물을 갖는지를 종합적으로 비교합니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## Table of Contents

1. [Overview](#overview)
2. [Global-Only Features](#global-only-features)
3. [Dual-Scope Features](#dual-scope-features)
4. [Settings Precedence](#settings-precedence)
5. [Directory Structure Comparison](#directory-structure-comparison)
6. [Tasks System](#tasks-system)
7. [Agent Teams](#agent-teams)
8. [Design Principles](#design-principles)
9. [Sources](#sources)

---

## Overview

Claude Code는 **스코프 계층(scope hierarchy)** 을 사용합니다. 일부 기능은 전역(`~/.claude/`)과 프로젝트(`.claude/`) 양쪽 수준에 모두 존재하고, 다른 일부는 오직 전역에만 존재합니다. 설계 원칙은 다음과 같습니다. *개인 상태(personal state)* 이거나 *프로젝트 간 조율(cross-project coordination)* 에 해당하는 것은 전역에 두고, *팀과 공유 가능한 프로젝트 설정(team-shareable project config)* 은 프로젝트 수준에 둘 수 있습니다.

- `~/.claude/` 는 **사용자 수준 홈**(전역, 모든 프로젝트)입니다
- 저장소 안의 `.claude/` 는 **프로젝트 수준 홈**(해당 프로젝트로 한정)입니다

---

## Global-Only Features

다음 항목들은 **오직** `~/.claude/` 아래에만 존재하며 프로젝트로 스코프를 한정할 수 없습니다:

| Feature | Location | Purpose |
|---------|----------|---------|
| **Tasks** | `~/.claude/tasks/` | 세션과 에이전트에 걸쳐 유지되는 영속적 태스크 목록 |
| **Agent Teams** | `~/.claude/teams/` | 다중 에이전트 조율 설정(실험적, 2026년 2월) |
| **Auto Memory** | `~/.claude/projects/<hash>/memory/` | 프로젝트별로 Claude가 스스로 기록한 학습 내용(개인용, 절대 공유되지 않음) |
| **Credentials & OAuth** | System keychain + `~/.claude.json` | API 키, OAuth 토큰(절대 프로젝트 파일에 저장되지 않음) |
| **Keybindings** | `~/.claude/keybindings.json` | 사용자 지정 키보드 단축키 |
| **MCP User Servers** | `~/.claude.json` (`mcpServers` key) | 모든 프로젝트에 걸친 개인 MCP 서버 |
| **Preferences/Cache** | `~/.claude.json` | 테마, 모델, 출력 스타일, 세션 상태 |

---

## Dual-Scope Features

다음 항목들은 양쪽 수준에 모두 존재하며, **프로젝트 수준이 전역보다 우선** 합니다:

| Feature | Global (`~/.claude/`) | Project (`.claude/`) | Precedence |
|---------|----------------------|---------------------|------------|
| **CLAUDE.md** | `~/.claude/CLAUDE.md` | `./CLAUDE.md` or `.claude/CLAUDE.md` | 프로젝트가 전역을 재정의 |
| **Settings** | `~/.claude/settings.json` | `.claude/settings.json` + `.claude/settings.local.json` | 프로젝트 > 전역 |
| **Rules** | `~/.claude/rules/*.md` | `.claude/rules/*.md` | 프로젝트가 재정의 |
| **Agents/Subagents** | `~/.claude/agents/*.md` | `.claude/agents/*.md` | 프로젝트가 재정의 |
| **Commands** | `~/.claude/commands/*.md` | `.claude/commands/*.md` | 양쪽 모두 사용 가능 |
| **Skills** | `~/.claude/skills/` | `.claude/skills/` | 양쪽 모두 사용 가능 |
| **Hooks** | `~/.claude/hooks/` | `.claude/hooks/` | 양쪽 모두 실행 |
| **MCP Servers** | `~/.claude.json` (user scope) | `.mcp.json` (project scope) | 세 가지 스코프: local > project > user |

---

## Settings Precedence

사용자가 수정 가능한 설정은 다음의 재정의 순서(높은 우선순위에서 낮은 순위로)로 적용됩니다:

| Priority | Location | Scope | Version Control | Purpose |
|----------|----------|-------|-----------------|---------|
| 1 | Command line flags | Session | N/A | 단일 세션 재정의 |
| 2 | `.claude/settings.local.json` | Project | No (git-ignored) | 개인용 프로젝트 전용 |
| 3 | `.claude/settings.json` | Project | Yes (committed) | 팀 공유 설정 |
| 4 | `~/.claude/settings.local.json` | User | N/A | 개인용 전역 재정의 |
| 5 | `~/.claude/settings.json` | User | N/A | 전역 개인 설정 |

정책 계층(policy layer): `managed-settings.json` 은 조직에 의해 강제되며 로컬 파일로 재정의할 수 없습니다.

**중요**: `deny` 규칙은 가장 높은 안전 우선순위를 가지며, 더 낮은 우선순위의 allow/ask 규칙으로 재정의할 수 없습니다.

---

## Directory Structure Comparison

### Global Scope (`~/.claude/`)

```
~/.claude/
├── settings.json              # User-level settings (all projects)
├── settings.local.json        # Personal overrides
├── CLAUDE.md                  # User memory (all projects)
├── agents/                    # User subagents (available to all projects)
│   └── *.md
├── rules/                     # User-level modular rules
│   └── *.md
├── commands/                  # User-level commands
│   └── *.md
├── skills/                    # User-level skills
│   └── */SKILL.md
├── tasks/                     # GLOBAL-ONLY: Task lists
│   └── {task-list-id}/
├── teams/                     # GLOBAL-ONLY: Agent team configs
│   └── {team-name}/
│       └── config.json
├── projects/                  # GLOBAL-ONLY: Per-project auto-memory
│   └── {project-hash}/
│       └── memory/
│           ├── MEMORY.md
│           └── *.md
├── keybindings.json           # GLOBAL-ONLY: Keyboard shortcuts
└── hooks/                     # User-level hooks
    ├── scripts/
    └── config/

~/.claude.json                 # GLOBAL-ONLY: MCP servers, OAuth, preferences, caches
```

### Project Scope (`.claude/`)

```
.claude/
├── settings.json              # Team-shared settings
├── settings.local.json        # Personal project overrides (git-ignored)
├── CLAUDE.md                  # Project memory (alternative to ./CLAUDE.md)
├── agents/                    # Project subagents
│   └── *.md
├── rules/                     # Project-level modular rules
│   └── *.md
├── commands/                  # Custom slash commands
│   └── *.md
├── skills/                    # Custom skills
│   └── {skill-name}/
│       ├── SKILL.md
│       └── supporting-files/
├── hooks/                     # Project-level hooks
│   ├── scripts/
│   └── config/
└── plugins/                   # Installed plugins

.mcp.json                      # Project-scoped MCP servers (repo root)
```

---

## Tasks System

**Claude Code v2.1.16**(2026년 1월 22일)에서 도입되었으며, 더 이상 사용되지 않는 TodoWrite 시스템을 대체합니다.

### Storage

태스크는 (클라우드 데이터베이스가 아닌) 로컬 파일시스템의 `~/.claude/tasks/` 에 저장됩니다. 덕분에 태스크 상태를 감사(audit)할 수 있고, 버전 관리가 가능하며, 크래시로부터 복구할 수 있습니다.

### Tools

| Tool | Purpose |
|------|---------|
| **TaskCreate** | `subject`, `description`, `activeForm` 을 가진 새 태스크 생성 |
| **TaskGet** | ID로 특정 태스크의 전체 상세 정보 조회 |
| **TaskUpdate** | 상태 변경, 소유자 설정, 의존성 추가, 또는 삭제 |
| **TaskList** | 모든 태스크와 현재 상태 목록 조회 |

### Task Lifecycle

```
pending  →  in_progress  →  completed
```

### Dependency Management

태스크는 `addBlockedBy`/`addBlocks` 를 통해 다른 태스크를 차단할 수 있으며, 이를 통해 조기 실행을 방지하는 의존성 그래프를 만듭니다.

### Multi-Session Collaboration

```bash
CLAUDE_CODE_TASK_LIST_ID=my-project-tasks claude
```

동일한 ID를 공유하는 모든 세션은 태스크 업데이트를 실시간으로 확인하여, 병렬 작업 흐름과 세션 재개를 가능하게 합니다.

### Key Differences from Old Todos

| Feature | Old Todos | New Tasks |
|---------|-----------|-----------|
| Scope | 단일 세션 | 세션 간·에이전트 간 |
| Dependencies | 없음 | 완전한 의존성 그래프 |
| Storage | 메모리 전용 | 파일 시스템 (`~/.claude/tasks/`) |
| Persistence | 세션 종료 시 소실 | 재시작과 크래시에도 유지 |
| Multi-session | 불가능 | `CLAUDE_CODE_TASK_LIST_ID` 를 통해 가능 |

---

## Agent Teams

**2026년 2월 5일** 에 실험적 기능으로 발표되었습니다. Agent Teams는 여러 Claude Code 세션이 공유 작업을 함께 조율할 수 있도록 합니다.

### Enabling

```json
// In ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### Configuration

팀 설정은 `~/.claude/teams/{team-name}/` 에 존재하며 다음 모드를 지원합니다:

| Mode | Description | Requirements |
|------|-------------|--------------|
| **In-process** (default) | 모든 팀원이 사용자의 터미널 안에서 실행됨 | 없음 |
| **Split panes** | 각 팀원이 자체 창(pane)을 가짐 | tmux 또는 iTerm2 (VS Code 터미널은 불가) |

---

## Design Principles

전역 전용 vs 이중 스코프의 구분은 명확한 패턴을 따릅니다:

| Category | Scope | Rationale |
|----------|-------|-----------|
| **조율 상태(coordination state)** (tasks, teams) | Global-only | 어떤 단일 프로젝트를 넘어 지속되어야 함 |
| **보안 상태(security state)** (credentials, OAuth) | Global-only | 버전 관리에 실수로 커밋되는 것을 방지 |
| **개인 학습(personal learning)** (auto-memory) | Global-only | 사용자별 정보이며 팀과 공유 불가 |
| **입력 환경설정(input preferences)** (keybindings) | Global-only | 사용자의 손에 익은 습관이며 프로젝트별이 아님 |
| **구성(configuration)** (settings, rules, agents) | Both levels | 팀이 프로젝트별 동작을 공유해야 함 |
| **워크플로 정의(workflow definitions)** (commands, skills) | Both levels | 개인용일 수도 팀 공유용일 수도 있음 |

Auto-memory(`~/.claude/projects/<hash>/memory/`)는 주목할 만한 하이브리드입니다. 특정 프로젝트*에 관한* 것이지만, 팀 공유 구성이 아니라 개인 학습을 나타내기 때문에 *전역으로* 저장됩니다.

---

## Sources

- [Claude Code Settings Documentation](https://code.claude.com/docs/en/settings)
- [Orchestrate Teams of Claude Code Sessions](https://code.claude.com/docs/en/agent-teams)
- [What are Tasks in Claude Code - ClaudeLog](https://claudelog.com/faqs/what-are-tasks-in-claude-code/)
- [Claude Code Task Management - ClaudeFast](https://claudefa.st/blog/guide/development/task-management)
- [Claude Code Tasks Update - VentureBeat](https://venturebeat.com/orchestration/claude-codes-tasks-update-lets-agents-work-longer-and-coordinate-across)
- [Where Are Claude Code Global Settings - ClaudeLog](https://claudelog.com/faqs/where-are-claude-code-global-settings/)
- [Claude Opus 4.6 Agent Teams - VentureBeat](https://venturebeat.com/technology/anthropics-claude-opus-4-6-brings-1m-token-context-and-agent-teams-to-take)
- [How to Set Up Claude Code Agent Teams (Full Walkthrough) - r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/comments/1qz8tyy/how_to_set_up_claude_code_agent_teams_full/)
- [Anthropic replaced Claude Code's old 'Todos' with Tasks - r/ClaudeAI](https://www.reddit.com/r/ClaudeAI/comments/1qkjznp/anthropic_replaced_claude_codes_old_todos_with/)
