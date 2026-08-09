<!--
  이 문서는 implementation/claude-commands-implementation.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Commands Implementation

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar_02%2C_2026-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#weather-orchestrator"><img src="../!/tags/implemented-hd.svg" alt="Implemented"></a>

weather orchestrator 명령은 이 저장소에서 **Command → Agent → Skill** 아키텍처 패턴의 진입점으로 구현되어 있으며, 명령이 여러 단계로 이루어진 워크플로를 어떻게 오케스트레이션하는지 보여줍니다.

---

## Weather Orchestrator

**File**: [`.claude/commands/weather-orchestrator.md`](../.claude/commands/weather-orchestrator.md)

```yaml
---
description: Fetch weather data for Dubai and create an SVG weather card
model: haiku
---

# Weather Orchestrator Command

Fetch the current temperature for Dubai, UAE and create a visual SVG weather card.

## Workflow

### Step 1: Ask User Preference
Use the AskUserQuestion tool to ask the user whether they want the temperature
in Celsius or Fahrenheit.

### Step 2: Fetch Weather Data
Use the Agent tool to invoke the weather agent:
- subagent_type: weather-agent
- prompt: Fetch the current temperature for Dubai, UAE in [unit]...

### Step 3: Create SVG Weather Card
Use the Skill tool to invoke the weather-svg-creator skill:
- skill: weather-svg-creator

...
```

이 명령은 전체 워크플로를 오케스트레이션합니다. 사용자에게 온도 단위 선호를 묻고, Agent 도구를 통해 `weather-agent`를 호출한 다음, Skill 도구를 통해 `weather-svg-creator` 스킬을 호출합니다.

---

## ![How to Use](../!/tags/how-to-use.svg)

```bash
$ claude
> /weather-orchestrator
```

---

## ![How to Implement](../!/tags/how-to-implement.svg)

Claude에게 하나 만들어 달라고 요청하세요 — YAML frontmatter와 본문을 갖춘 마크다운 파일을 `.claude/commands/<name>.md` 에 생성해 줍니다.

---

<a href="https://github.com/shanraisshan/claude-code-best-practice#orchestration-workflow"><img src="../!/tags/orchestration-workflow-hd.svg" alt="Orchestration Workflow"></a>

weather orchestrator는 Command → Agent → Skill 오케스트레이션 패턴에서 **Command**에 해당합니다. 진입점 역할을 하며, 사용자 상호작용(온도 단위 선호)을 처리하고, 데이터 가져오기를 `weather-agent`에 위임하며, 시각적 출력을 위해 `weather-svg-creator` 스킬을 호출합니다.

<p align="center">
  <img src="../orchestration-workflow/orchestration-workflow.svg" alt="Command Skill Agent Architecture Flow" width="100%">
</p>

| Component | Role | This Repo |
|-----------|------|-----------|
| **Command** | 진입점, 사용자 상호작용 | [`/weather-orchestrator`](../.claude/commands/weather-orchestrator.md) |
| **Agent** | 사전 로드된 스킬(agent skill)로 데이터를 가져옴 | [`weather-agent`](../.claude/agents/weather-agent.md) with [`weather-fetcher`](../.claude/skills/weather-fetcher/SKILL.md) |
| **Skill** | 독립적으로 출력을 생성함(skill) | [`weather-svg-creator`](../.claude/skills/weather-svg-creator/SKILL.md) |
