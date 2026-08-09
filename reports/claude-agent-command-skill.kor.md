<!--
  이 문서는 reports/claude-agent-command-skill.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Agents vs Commands vs Skills — When to Use What

Claude Code의 세 가지 확장 메커니즘인 서브에이전트, 커맨드, 스킬을 비교합니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

![Slash menu showing time-skill, time-command, and time-agent](assets/agent-command-skill-1.jpg)

---

## At a Glance

| | Agent | Command | Skill |
|---|---|---|---|
| **Location** | `.claude/agents/<name>.md` | `.claude/commands/<name>.md` | `.claude/skills/<name>/SKILL.md` |
| **Context** | 별도의 서브에이전트 프로세스 | 인라인(메인 대화) | 인라인(메인 대화) |
| **User-invocable** | `/` 메뉴 없음 — Claude가 호출하거나 Agent 도구로 호출 | 예 — `/command-name` | 예 — `/skill-name` (`user-invocable: false`가 아닌 경우) |
| **Auto-invoked by Claude** | 예 — `description` 필드를 통해 | 아니오 | 예 — `description` 필드를 통해 (`disable-model-invocation: true`가 아닌 경우) |
| **Accepts arguments** | `prompt` 파라미터를 통해 | `$ARGUMENTS`, `$0`, `$1` | `$ARGUMENTS`, `$0`, `$1` |
| **Dynamic context injection** | 아니오 | 예 — `` !`command` `` | 예 — `` !`command` `` |
| **Own context window** | 예 — 격리됨 | 아니오 — 메인 공유 | 아니오 — 메인 공유 (`context: fork`가 아닌 경우) |
| **Model override** | `model:` 프론트매터 | `model:` 프론트매터 | `model:` 프론트매터 |
| **Tool restrictions** | `tools:` / `disallowedTools:` | `allowed-tools:` | `allowed-tools:` |
| **Hooks** | `hooks:` 프론트매터 | — | `hooks:` 프론트매터 |
| **Memory** | `memory:` 프론트매터 (user/project/local) | — | — |
| **Can preload skills** | 예 — `skills:` 프론트매터 | — | — |
| **MCP servers** | `mcpServers:` 프론트매터 | — | — |

---

## When to Use Each

### Use an Agent when:

- 작업이 **자율적이고 여러 단계로 구성**될 때 — 에이전트가 지속적인 안내 없이 탐색하고 판단하며 실행해야 하는 경우
- **컨텍스트 격리**가 필요할 때 — 해당 작업이 메인 대화 윈도를 오염시키지 않아야 하는 경우
- 에이전트가 세션 간에 **지속적인 메모리**를 유지해야 할 때 (예: 패턴을 학습하는 코드 리뷰어)
- 메인 컨텍스트를 어지럽히지 않고 스킬을 통해 **도메인 지식을 미리 로드**하고 싶을 때
- 작업을 **백그라운드에서 실행**하거나 **git worktree**에서 실행하면 이점이 있을 때
- **도구 제한**이나 **다른 권한 모드**(예: `acceptEdits`, `plan`)가 필요할 때

**Example**: `weather-agent` — 미리 로드된 `weather-fetcher` 스킬을 사용해 날씨 데이터를 자율적으로 가져오며, 제한된 도구로 별도 컨텍스트에서 실행됩니다.

### Use a Command when:

- **사용자가 직접 시작하는 진입점**이 필요할 때 — 사용자가 명시적으로 트리거하는 워크플로
- 워크플로가 다른 에이전트나 스킬을 **오케스트레이션**하는 경우
- **컨텍스트를 가볍게 유지**하고 싶을 때 — 커맨드 내용은 사용자가 트리거하기 전까지 세션 컨텍스트에 주입되지 않습니다

**Example**: `weather-orchestrator` — 사용자가 트리거하면 C/F 선호를 묻고, 에이전트를 호출한 뒤 SVG 스킬을 호출합니다.

### Use a Skill when:

- 사용자 의도에 따라 **Claude가 자동으로 호출**하도록 하고 싶을 때 — 스킬 description은 의미 기반 매칭을 위해 세션 컨텍스트에 주입됩니다
- 작업이 여러 곳(커맨드, 에이전트, 또는 Claude 자체)에서 호출될 수 있는 **재사용 가능한 절차**일 때
- **에이전트 프리로딩**이 필요할 때 — 특정 에이전트에 시작 시점에 도메인 지식을 심어두는 경우

**Example**: `weather-svg-creator` — 사용자가 날씨 카드를 요청하면 Claude가 자동으로 호출하며, 커맨드에서도 호출할 수 있습니다.

---

## The Command → Agent → Skill Architecture

이 저장소는 계층형 오케스트레이션 패턴을 보여줍니다:

```
User triggers /command
    ↓
Command orchestrates the workflow
    ↓
Command invokes Agent (separate context, autonomous)
    ↓
Agent uses preloaded Skill (domain knowledge)
    ↓
Command invokes Skill (inline, for output generation)
```

**Concrete example** — 날씨 시스템:

```
/weather-orchestrator (command — entry point, asks C/F)
    ↓
weather-agent (agent — fetches temperature autonomously)
    ├── weather-fetcher (agent skill — preloaded API instructions)
    ↓
weather-svg-creator (skill — creates SVG inline)
```

---

## Frontmatter Comparison

### Agent Frontmatter

```yaml
---
name: my-agent
description: Use this agent PROACTIVELY when...
tools: Read, Write, Edit, Bash
model: sonnet
maxTurns: 10
permissionMode: acceptEdits
memory: user
skills:
  - my-skill
---
```

### Command Frontmatter

```yaml
---
description: Do something useful
argument-hint: [issue-number]
allowed-tools: Read, Edit, Bash(gh *)
model: sonnet
---
```

### Skill Frontmatter

```yaml
---
name: my-skill
description: Do something when the user asks for...
argument-hint: [file-path]
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Grep, Glob
model: sonnet
context: fork
agent: general-purpose
---
```

---

## Key Distinctions

### Auto-invocation

| Mechanism | Claude가 자동 호출 가능? | 방지 방법 |
|-----------|------------------------|----------------|
| Agent | 예 — `description`을 통해 ("PROACTIVELY"로 촉진) | description을 제거하거나 완화 |
| Command | 아니오 — 항상 `/`로 사용자가 시작 | 해당 없음 |
| Skill | 예 — `description`을 통해 | `disable-model-invocation: true` 설정 |

### Visibility in `/` menu

| Mechanism | `/` 메뉴에 표시됨? | 숨기는 방법 |
|-----------|---------------------|-------------|
| Agent | 아니오 | 해당 없음 |
| Command | 예 — 항상 | 숨길 수 없음 |
| Skill | 예 — 기본값 | `user-invocable: false` 설정 |

### Context isolation

| Mechanism | 자체 컨텍스트에서 실행? | 설정 방법 |
|-----------|---------------------|-----------------|
| Agent | 항상 | 기본 동작 |
| Command | 절대 아님 | 해당 없음 |
| Skill | 선택 사항 | `context: fork` 설정 |

---

## Worked Example: "What is the current time?"

이 저장소에는 동일한 작업(PKT 현재 시각 표시)에 대해 세 가지 메커니즘이 모두 정의되어 있습니다. 사용자가 어떤 `/` 커맨드도 명시적으로 호출하지 않고 **"What is the current time?"**라고 입력하면 어떤 일이 벌어지는지 살펴봅니다:

| Mechanism | 실행될까? | 이유 / 이유 아님 |
|-----------|--------------|---------------|
| `time-command` | 아니오 | 커맨드는 **절대 자동 호출되지 않습니다**. 실행하려면 사용자가 `/time-command`를 명시적으로 입력해야 합니다. 커맨드에는 자동 탐색 경로가 없으며, 엄격하게 사용자가 시작합니다. |
| `time-agent` | **예** (가능) | 에이전트의 `description`에 *"Use this agent to display the current time in Pakistan Standard Time"*라고 되어 있습니다. Claude가 이를 사용자 의도와 매칭해 Agent 도구로 스폰할 수 있습니다. 다만 에이전트는 **별도의 컨텍스트 윈도**에서 실행되므로 이런 단순 작업에는 필요 이상으로 무겁습니다. |
| `time-skill` | **예** (가장 유력) | 스킬의 `description`에 *"Display the current time in Pakistan Standard Time (PKT, UTC+5). Use when the user asks for the current time, Pakistan time, or PKT."*라고 되어 있습니다. Claude가 이를 매칭해 Skill 도구로 호출합니다. **인라인**으로 실행되어 컨텍스트 오버헤드가 없으므로 가장 효율적인 선택지입니다. |

### Resolution order

여러 메커니즘이 동일한 의도에 매칭될 때, Claude는 요청을 만족하는 **가장 가벼운 선택지**를 선호합니다:

```
1. Skill (inline, no context overhead)     ← preferred
2. Agent (separate context, autonomous)    ← used if skill is unavailable or task is complex
3. Command (never — requires explicit /)   ← only if user types /time-command
```

### What if `disable-model-invocation: true` were set on the skill?

그러면 Claude는 해당 스킬을 **자동 호출할 수 없습니다**. 에이전트가 유일한 자동 호출 가능 선택지가 되므로, Claude는 대신 `time-agent`를 스폰합니다 — 한 줄짜리 bash 커맨드를 위해 별도의 컨텍스트 윈도를 소모하는 대가를 치르면서 말입니다.

### What if both skill and agent had auto-invocation disabled?

그러면 **아무것도 자동으로 실행되지 않습니다**. Claude는 자체 일반 지식으로 되돌아가 아마도 `TZ='Asia/Karachi' date`를 직접 실행할 것입니다 — 어떤 확장 메커니즘도 관여하지 않습니다. 사용자는 하나를 사용하려면 `/time-command`나 `/time-skill`을 명시적으로 입력해야 합니다.

![Claude auto-invoking time-skill when user asks "What is the current time?"](assets/agent-command-skill-2.png)

---

## Sources

- [Claude Code Skills — Docs](https://code.claude.com/docs/en/skills)
- [Claude Code Sub-agents — Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Slash Commands — Docs](https://code.claude.com/docs/en/slash-commands)
- [Skills Best Practice](../best-practice/claude-skills.md)
- [Commands Best Practice](../best-practice/claude-commands.md)
- [Sub-agents Best Practice](../best-practice/claude-subagents.md)
