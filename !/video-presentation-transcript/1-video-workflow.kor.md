<!--
  이 문서는 !/video-presentation-transcript/1-video-workflow.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Video 1: From Vibe Coding to Agentic Engineering — Workflows with Claude Code

**총 길이: 약 5분**

---

## INTRO — The Problem (0:00 – 0:45)

- "Claude Code를 이제 막 시작했다면, 아마 바이브 코딩을 하고 있을 겁니다 — 프롬프트를 입력하고, 결과를 받고, 이를 반복하는 식이죠. 그것도 동작하긴 하지만, Claude Code가 할 수 있는 일의 극히 일부만 쓰고 있는 셈입니다."
- "이 저장소는 바이브 코딩에서 에이전틱 엔지니어링으로 넘어가게 해주는 베스트 프랙티스를 엄선한 모음입니다 — 여기서 Claude는 단순히 당신에게 응답하는 것을 넘어, 당신을 위해 워크플로우를 실행합니다."
- "이 첫 번째 영상에서는 기반이 되는 요소를 다룹니다: **Commands, Agents, Skills** — 그리고 이들이 어떻게 서로 이어져 반복 가능한 워크플로우가 되는지를 설명합니다."

---

## PART 1 — The Ad-Hoc Way (0:45 – 2:00)

**데모: 바이브 코딩 방식**

- 새 Claude Code 터미널을 엽니다
- 다음을 입력합니다: *"What is the weather in Dubai? Write it to an output file and create an SVG card for it."*
- 결과를 보여줍니다 — 동작은 하지만, 다음을 짚어줍니다:
  - SVG 디자인이 매번 달라집니다 (색상, 레이아웃, 폰트가 무작위)
  - 작업이 진행되는 것을 앉아서 지켜봐야 했습니다
  - 내일 다시 실행하면 완전히 다르게 생긴 카드가 나옵니다
- **두 번째 터미널을 열고, 같은 프롬프트를 다시 실행합니다**
  - SVG를 나란히 놓고 보여줍니다 — 서로 다르게 생겼습니다
- "이것이 바이브 코딩의 문제입니다. 한 번은 동작합니다. 하지만 반복 가능하지 않습니다. 신뢰할 수 있는 워크플로우가 아닙니다."

---

## PART 2 — The Workflow Way (2:00 – 3:15)

**데모: `/weather-orchestrator` command**

- "이제 같은 작업을 워크플로우로 보여드리겠습니다."
- 다음을 입력합니다: `/weather-orchestrator`
- 화면에서 벌어지는 일을 하나씩 짚어줍니다:
  1. 섭씨인지 화씨인지 **사용자에게 물어봅니다** (구조화된 사용자 상호작용)
  2. 온도를 가져오기 위해 **weather-agent를 생성합니다** (터미널에서 초록색 에이전트가 보입니다)
  3. SVG 카드를 만들기 위해 **skill을 호출합니다**
  4. 출력: `orchestration-workflow/weather.svg` + `orchestration-workflow/output.md`
- "다시 실행해보세요 — 같은 SVG 레이아웃, 같은 파일 구조, 같은 깔끔한 결과. 매번 동일합니다."
- "이것을 실행시켜 놓고 자리를 떠도 됩니다. 자율적으로 실행됩니다."

---

## PART 3 — How It Works: Command → Agent → Skill (3:15 – 4:30)

**세 가지 구성 요소 설명**

### Commands (`.claude/commands/`)

- "command는 진입점입니다 — 스크립트와 비슷하죠. Claude에게 *어떤 단계를 따라야 하는지* 알려주는 마크다운 파일입니다."
- "우리의 `weather-orchestrator`는 지휘자입니다. 사용자에게 질문을 던지고, 에이전트를 호출한 다음, skill을 호출합니다."
- command는 `.claude/commands/`에 위치하며 `/slash-commands`로 나타납니다

### Agents (`.claude/agents/`)

- "agent는 전문화된 일꾼입니다. 우리의 `weather-agent`는 단 하나의 임무를 맡습니다: 온도를 가져오는 것."
- "여기에는 `weather-fetcher`라는 **미리 로드된 skill**이 있습니다 — 이 skill은 시작 시점에 에이전트의 컨텍스트에 주입되므로, 어떤 API를 호출하고 응답을 어떻게 파싱해야 하는지 정확히 알고 있습니다."
- 에이전트는 자기만의 도구, 모델, 권한을 가집니다. 격리된 일꾼입니다.

### Skills (`.claude/skills/`)

- "skill은 재사용 가능한 지시문 모음입니다. 레시피라고 생각하면 됩니다."
- "여기에는 두 가지 skill 패턴이 있습니다:"
  - **Agent skill** (미리 로드됨): `weather-fetcher`는 에이전트에 내장되어 있습니다 — 도메인 지식입니다
  - **Invoked skill** (호출됨): `weather-svg-creator`는 Skill 도구를 통해 독립적으로 호출됩니다 — SVG 카드를 만듭니다
- skill은 배경 지식이 될 수도, 독립적인 동작이 될 수도 있습니다

### Flow Diagram (optionally show on screen)

```
/weather-orchestrator (Command)
    → AskUser: C° or F°?
    → weather-agent (Agent + weather-fetcher skill)
    → weather-svg-creator (Skill)
    → Output: weather.svg + output.md
```

---

## PART 4 — Why This Matters / Wrap-up (4:30 – 5:00)

- "바이브 코딩과 에이전틱 엔지니어링의 차이는 **구조**입니다."
  - 바이브 코딩: 입력하고, 기대하고, 무언가를 얻습니다.
  - 에이전틱 엔지니어링: 워크플로우를 한 번 정의하면, 매번 동일하게 실행됩니다.
- "Commands, Agents, Skills는 세 가지 구성 요소입니다. 이것들을 이해하고 나면, 어떤 워크플로우든 만들 수 있습니다."
- "이 저장소에는 더 많은 패턴이 있습니다 — hooks, 멀티 에이전트 팀, CLAUDE.md 구성 — 이것들은 앞으로의 영상에서 다루겠습니다."
- "저장소 링크는 설명란에 있습니다. 스타를 누르고, 클론해서, 자신만의 워크플로우를 만들기 시작하세요."

---

## Quick Reference

| 개념 | 위치 | 목적 |
|---------|----------|---------|
| Command | `.claude/commands/` | 진입점, 오케스트레이션, `/slash-command` |
| Agent | `.claude/agents/` | 자기만의 도구와 모델을 가진 전문화된 일꾼 |
| Skill | `.claude/skills/` | 재사용 가능한 지시문 (미리 로드되거나 호출됨) |
