<!--
  이 문서는 implementation/claude-skills-implementation.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Skills Implementation

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar_02%2C_2026-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#weather-svg-creator"><img src="../!/tags/implemented-hd.svg" alt="Implemented"></a>

이 저장소에는 **Command → Agent → Skill** 아키텍처 패턴의 일부로 두 개의 스킬이 구현되어 있으며, 서로 다른 두 가지 스킬 호출 패턴인 **agent skills**(사전 로드)와 **skills**(직접 호출)을 보여줍니다.

---

## Weather SVG Creator (Skill)

**File**: [`.claude/skills/weather-svg-creator/SKILL.md`](../.claude/skills/weather-svg-creator/SKILL.md)

```yaml
---
name: weather-svg-creator
description: Creates an SVG weather card showing the current temperature for
  Dubai. Writes the SVG to orchestration-workflow/weather.svg and updates
  orchestration-workflow/output.md.
---

# Weather SVG Creator Skill

This skill creates a visual SVG weather card and writes the output files.

## Task
Create an SVG weather card displaying the temperature for Dubai, UAE,
and write it along with a summary to output files.

## Instructions
You will receive the temperature value and unit (Celsius or Fahrenheit)
from the calling context.

### 1. Create SVG Weather Card
Generate a clean SVG weather card...

### 2. Write SVG File
Write the SVG content to `orchestration-workflow/weather.svg`.

### 3. Write Output Summary
Write to `orchestration-workflow/output.md`...

...
```

이것은 **skill** 로, 커맨드가 Skill 툴을 통해 직접 호출합니다. 대화 컨텍스트에서 온도 데이터를 전달받아 SVG 날씨 카드와 출력 요약을 생성합니다.

---

## Weather Fetcher (Agent Skill)

**File**: [`.claude/skills/weather-fetcher/SKILL.md`](../.claude/skills/weather-fetcher/SKILL.md)

```yaml
---
name: weather-fetcher
description: Instructions for fetching current weather temperature data
  for Dubai, UAE from Open-Meteo API
user-invocable: false
---

# Weather Fetcher Skill

This skill provides instructions for fetching current weather data.

## Task
Fetch the current temperature for Dubai, UAE in the requested unit
(Celsius or Fahrenheit).

## Instructions
1. Fetch Weather Data: Use the WebFetch tool to get current weather data
   - Celsius URL: https://api.open-meteo.com/v1/forecast?latitude=25.2048&longitude=55.2708&current=temperature_2m&temperature_unit=celsius
   - Fahrenheit URL: https://api.open-meteo.com/v1/forecast?latitude=25.2048&longitude=55.2708&current=temperature_2m&temperature_unit=fahrenheit
2. Extract Temperature: From the JSON response, extract `current.temperature_2m`
3. Return Result: Return the temperature value and unit clearly.

...
```

이것은 **agent skill** 로, `skills:` 프론트매터 필드를 통해 시작 시점에 `weather-agent`로 사전 로드됩니다. 직접 호출되지 않으며, 대신 에이전트의 컨텍스트에 주입되는 도메인 지식으로 사용됩니다. `user-invocable: false`가 지정되어 `/` 커맨드 메뉴에서 숨겨진다는 점에 유의하세요.

---

## Two Skill Patterns

| Pattern | Invocation | Example | Key Difference |
|---------|-----------|---------|----------------|
| **Skill** | `Skill(skill: "name")` | `weather-svg-creator` | Skill 툴을 통해 직접 호출됨 |
| **Agent Skill** | Preloaded via `skills:` field | `weather-fetcher` | 시작 시점에 에이전트 컨텍스트로 주입됨 |

---

## ![How to Use](../!/tags/how-to-use.svg)

**Skill** — 슬래시 커맨드로 직접 호출:
```bash
$ claude
> /weather-svg-creator
```

---

## ![How to Implement](../!/tags/how-to-implement.svg)

Claude에게 하나 만들어 달라고 요청하세요 — YAML 프론트매터와 본문을 갖춘 마크다운 파일을 `.claude/skills/my-skill/SKILL.md`에 생성해 줍니다.

# My Skill

Instructions for what the skill does.
```
