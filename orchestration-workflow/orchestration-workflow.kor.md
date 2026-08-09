<!--
  이 문서는 orchestration-workflow/orchestration-workflow.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Orchestration Workflow

이 문서는 날씨 데이터를 가져와 SVG로 렌더링하는 시스템을 예시로 **Command → Agent (with skill) → Skill** 오케스트레이션 워크플로우를 설명합니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## System Overview

이 날씨 시스템은 하나의 오케스트레이션 워크플로우 안에서 서로 다른 두 가지 스킬 패턴을 보여줍니다:
- **Agent Skills** (사전 로드): `weather-fetcher`는 시작 시점에 도메인 지식으로서 `weather-agent`에 주입됩니다
- **Skills** (독립 실행): `weather-svg-creator`는 command가 Skill 도구를 통해 직접 호출합니다

이는 **Command → Agent → Skill** 아키텍처 패턴을 보여주며, 그 구조는 다음과 같습니다:
- command가 워크플로우를 오케스트레이션하고 사용자 상호작용을 처리합니다
- agent가 사전 로드된 스킬을 사용해 데이터를 가져옵니다
- skill이 시각적 결과물을 독립적으로 생성합니다

## Component Summary

| Component | Role | Example |
|-----------|------|---------|
| **Command** | 진입점, 사용자 상호작용 | [`/weather-orchestrator`](../.claude/commands/weather-orchestrator.md) |
| **Agent** | 사전 로드된 스킬(agent skill)로 데이터 조회 | [`weather-agent`](../.claude/agents/weather-agent.md) with [`weather-fetcher`](../.claude/skills/weather-fetcher/SKILL.md) |
| **Skill** | 결과물을 독립적으로 생성(skill) | [`weather-svg-creator`](../.claude/skills/weather-svg-creator/SKILL.md) |

## Flow Diagram

```
╔══════════════════════════════════════════════════════════════════╗
║              ORCHESTRATION WORKFLOW                              ║
║           Command  →  Agent  →  Skill                            ║
╚══════════════════════════════════════════════════════════════════╝

                         ┌───────────────────┐
                         │  User Interaction │
                         └─────────┬─────────┘
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  /weather-orchestrator — Command (Entry Point)      │
         └─────────────────────────┬───────────────────────────┘
                                   │
                              Step 1
                                   │
                                   ▼
                      ┌────────────────────────┐
                      │  AskUser — C° or F°?   │
                      └────────────┬───────────┘
                                   │
                         Step 2 — Agent tool
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  weather-agent — Agent ● skill: weather-fetcher     │
         └─────────────────────────┬───────────────────────────┘
                                   │
                          Returns: temp + unit
                                   │
                         Step 3 — Skill tool
                                   │
                                   ▼
         ┌─────────────────────────────────────────────────────┐
         │  weather-svg-creator — Skill ● SVG card + output    │
         └─────────────────────────┬───────────────────────────┘
                                   │
                          ┌────────┴────────┐
                          │                 │
                          ▼                 ▼
                   ┌────────────┐    ┌────────────┐
                   │weather.svg │    │ output.md  │
                   └────────────┘    └────────────┘
```

## Component Details

### 1. Command

#### `/weather-orchestrator` (Command)
- **Location**: `.claude/commands/weather-orchestrator.md`
- **Purpose**: 진입점 — 워크플로우를 오케스트레이션하고 사용자 상호작용을 처리
- **Actions**:
  1. 사용자에게 선호하는 온도 단위(Celsius/Fahrenheit)를 질문
  2. Agent 도구를 통해 weather-agent 호출
  3. Skill 도구를 통해 weather-svg-creator 호출
- **Model**: haiku

### 2. Agent with Preloaded Skill (Agent Skill)

#### `weather-agent` (Agent)
- **Location**: `.claude/agents/weather-agent.md`
- **Purpose**: 사전 로드된 스킬을 사용해 날씨 데이터를 조회
- **Skills**: `weather-fetcher` (도메인 지식으로 사전 로드됨)
- **Tools Available**: Read, Skill
- **Model**: sonnet
- **Color**: green
- **Memory**: project

agent는 시작 시점에 `weather-fetcher`가 컨텍스트에 사전 로드되어 있습니다. agent는 이 스킬의 지침을 따라 온도를 가져와 그 값을 command에 반환합니다.

### 3. Skill

#### `weather-svg-creator` (Skill)
- **Location**: `.claude/skills/weather-svg-creator/SKILL.md`
- **Purpose**: 시각적인 SVG 날씨 카드를 생성하고 출력 파일을 작성
- **Invocation**: command에서 Skill 도구를 통해 호출 (어떤 agent에도 사전 로드되지 않음)
- **Outputs**:
  - `orchestration-workflow/weather.svg` — SVG 날씨 카드
  - `orchestration-workflow/output.md` — 날씨 요약

### 4. Preloaded Skill

#### `weather-fetcher` (Skill)
- **Location**: `.claude/skills/weather-fetcher/SKILL.md`
- **Purpose**: 실시간 온도 데이터를 가져오기 위한 지침
- **Data Source**: 두바이(UAE)에 대한 Open-Meteo API
- **Output**: 온도 값과 단위(Celsius 또는 Fahrenheit)
- **Note**: 이것은 agent skill입니다 — 직접 호출되지 않고 `weather-agent`에 사전 로드됩니다

## Execution Flow

1. **User Invocation**: 사용자가 `/weather-orchestrator` command를 실행
2. **User Prompt**: command가 선호하는 온도 단위(Celsius/Fahrenheit)를 사용자에게 질문
3. **Agent Invocation**: command가 Agent 도구를 통해 `weather-agent`를 호출
4. **Skill Execution** (agent 컨텍스트 내):
   - agent가 `weather-fetcher` 스킬 지침을 따라 Open-Meteo에서 온도를 가져옴
   - agent가 온도 값과 단위를 command에 반환
5. **SVG Creation**: command가 Skill 도구를 통해 `weather-svg-creator`를 호출
   - skill이 `orchestration-workflow/weather.svg`에 SVG 날씨 카드를 생성
   - skill이 `orchestration-workflow/output.md`에 요약을 작성
6. **Result Display**: 다음 내용과 함께 사용자에게 요약이 표시됨:
   - 요청한 온도 단위
   - 조회된 온도
   - SVG 카드 위치
   - 출력 파일 위치

## Example Execution

```
Input: /weather-orchestrator
├─ Step 1: Asks: Celsius or Fahrenheit?
│  └─ User: Celsius
├─ Step 2: Agent tool → weather-agent
│  ├─ Preloaded Skill:
│  │  └─ weather-fetcher (domain knowledge)
│  ├─ Fetches from Open-Meteo → 26°C
│  └─ Returns: temperature=26, unit=Celsius
├─ Step 3: Skill tool → /weather-svg-creator
│  ├─ Creates: orchestration-workflow/weather.svg
│  └─ Writes: orchestration-workflow/output.md
└─ Output:
   ├─ Unit: Celsius
   ├─ Temperature: 26°C
   ├─ SVG: orchestration-workflow/weather.svg
   └─ Summary: orchestration-workflow/output.md
```

## Key Design Principles

1. **Two Skill Patterns**: agent skill(사전 로드)과 skill(직접 호출) 두 가지를 모두 시연
2. **Command as Orchestrator**: command가 사용자 상호작용을 처리하고 워크플로우를 조율
3. **Agent for Data Fetching**: agent가 사전 로드된 스킬로 데이터를 가져와 반환
4. **Skill for Output**: SVG 생성기가 command 컨텍스트로부터 데이터를 받아 독립적으로 실행
5. **Clean Separation**: 조회(agent) → 렌더링(skill) — 각 컴포넌트가 단일 책임을 가짐

## Architecture Patterns

### Agent Skill (Preloaded)

```yaml
# In agent definition (.claude/agents/weather-agent.md)
---
name: weather-agent
skills:
  - weather-fetcher    # Preloaded into agent context at startup
---
```

- **Skills are preloaded**: 스킬의 전체 내용이 시작 시점에 agent 컨텍스트에 주입됨
- **Agent uses skill knowledge**: agent가 사전 로드된 스킬의 지침을 따름
- **No dynamic invocation**: 스킬은 참조 자료이며 별도로 호출되지 않음

### Skill (Direct Invocation)

```yaml
# In skill definition (.claude/skills/weather-svg-creator/SKILL.md)
---
name: weather-svg-creator
description: Creates an SVG weather card...
---
```

- **Invoked via Skill tool**: command가 `Skill(skill: "weather-svg-creator")`를 호출
- **Independent execution**: agent 내부가 아니라 command 컨텍스트에서 실행됨
- **Receives data from context**: 대화에 이미 존재하는 온도 데이터를 사용
