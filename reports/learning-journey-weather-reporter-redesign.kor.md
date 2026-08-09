<!--
  이 문서는 reports/learning-journey-weather-reporter-redesign.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Learning Journey — Weather Reporter Redesign Plan

← Back to [README](../README.md)

## Overview

7번 슬라이드부터 이후의 모든 슬라이드를 하나의 관통 예제인 **날씨 리포터 에이전트(weather reporter agent)**를 중심으로 재설계한다. 서사의 흐름은 TOC에 보이는 순서(Agents → Skills → Context → CLAUDE.md → Commands+Workflow)와 일치한다. 청중이 먼저 날씨 리포터를 만나고, 이어서 그들이 무엇을 알고 있는지, 어떻게 사고하는지, 어떤 규칙을 따르는지를 이해한 뒤, 마지막으로 하나의 커맨드로 이들을 어떻게 트리거하는지를 배우도록 하는 구성이다.

---

## 1. Current → New Section Map

| Current section | Current slides | Action | New position |
|---|---|---|---|
| Topic 1: Context | 7-11 (section at 7) | Topic 3으로 이동 | slides 17-21 |
| Topic 2: CLAUDE.md | 12-17 (section at 12) | Topic 4로 이동 | slides 22-27 |
| Topic 3: Agents | 18-23 (section at 18) | Topic 1로 이동 | slides 7-12 |
| Topic 4: Skills | 24-29 (section at 24) | Topic 2로 이동 | slides 13-18 |
| Topic 5: Commands | 30-32 (section at 30) | Workflow와 병합하여 Topic 5로 | slides 28-32 |
| Topic 6: Workflow | 33-36 (section at 33) | Commands 섹션에 병합 | (별도 섹션 슬라이드 없음) |
| Closing slide | 37 | 유지, 부제만 갱신 | slide 33 |

**신규 총계: 33 슬라이드** (현재 37에서 Workflow 섹션 슬라이드와, Commands 섹션에 병합되는 Workflow 콘텐츠 3장을 뺀 값. Commands 섹션은 그 3장만큼 늘어난다.)

잠깐 — 다시 세어보자.

현재: 슬라이드 7-37 = 31장.
- Agents 섹션: 6장 (18-23) → Topic 1이 됨 (7-12)
- Skills 섹션: 6장 (24-29) → Topic 2가 됨 (13-18)
- Context 섹션: 5장 (7-11) → Topic 3이 됨 (19-23)
- CLAUDE.md 섹션: 6장 (12-17) → Topic 4가 됨 (24-29)
- Commands+Workflow 병합: 3 + 1 섹션 + 4 콘텐츠 = Commands (3) + Workflow (섹션 1 + 콘텐츠 3) = 7장 → Topic 5가 됨 (30-36)
- Closing: 1장 (37)

**신규 총계: 37 슬라이드.** (버려지는 슬라이드는 없다. Workflow 섹션 슬라이드는 병합된 Commands+Workflow 섹션의 일부가 된다 — 두 번째 섹션 구분자가 생기지 않도록 하위 섹션으로 유지하거나 `data-level`을 제거한다.)

**결정**: 37장 전부 유지한다. 기존 Workflow 섹션 구분자(슬라이드 33)에서 `data-level`을 제거하여, 섹션 구분자가 아니라 콘텐츠 슬라이드로 취급되게 한다. Commands 섹션이 30-36을 아우른다. Workflow 섹션 구분자는 Commands 섹션 내부의 시각적 "챕터 헤더"가 된다.

사실 더 간단하게: Workflow 섹션 구분자를 `data-level` 없는 콘텐츠 슬라이드로 유지한다. journey 바는 `commands` 레벨에 머문다. 섹션 번호 텍스트는 "Topic 6"에서 하위 제목으로 바뀐다.

---

## 2. New LEVELS Map (no change to keys or colors)

새로운 섹션 순서는 **Agents → Skills → Context → CLAUDE.md → Commands**이다. `workflow` 레벨 키는 `data-level` 용도에서 제외된다(섹션 구분자가 `data-level`을 잃는다). `LEVELS` 맵은 journey-바 이력 표시를 위해 여전히 `workflow`를 담고 있으나, 이를 트리거하는 슬라이드는 없다.

**수정된 접근**: 어떤 슬라이드도 `data-level="workflow"`를 갖지 않으므로 LEVELS 맵에서 `workflow` 레벨을 완전히 제거한다. journey 바는 `commands`(83%)에서 최댓값에 도달한다. 이는 괜찮다 — Workflow 섹션은 별도의 토픽이 아니라 Commands 섹션 *안*의 클라이맥스로 제시되기 때문이다.

사실 마무리 섹션인데 journey 바가 100%가 아니라 83%까지만 차는 것은 만족스럽지 않다. 더 나은 안: **Commands+Workflow를 "Commands & Workflow"라는 단일 섹션으로 병합**하고 `data-level="commands"`를 준다. LEVELS의 `workflow` 레벨은 100%로 유지하고, *기존* workflow 섹션 구분자 슬라이드에 `data-level="workflow"`를 부여한다 — 이 슬라이드는 Commands 섹션 내부의 시각적 전환점이 된다. 이렇게 하면 workflow 슬라이드에서 바가 100%까지 찬다.

**최종 결정**: LEVELS에 `commands`(83%)와 `workflow`(100%)를 모두 유지한다. Commands 섹션 구분자에는 `data-level="commands"`를, Workflow 하위 섹션 슬라이드에는 `data-level="workflow"`를 부여한다. journey 눈금은 현행 그대로 둔다. 이는 현재 구조와 정확히 일치하며 — 콘텐츠 슬라이드의 순서만 바뀔 뿐이다.

---

## 3. Slide-by-Slide Content Outline

### Slides 1-6 (unchanged)

슬라이드 1(제목), 2(Boris GIF), 3(Vibe→Agentic), 4(What is Vibe Coding), 5(Good vs Bad Prompts), 6(TOC — goToSlide 대상만 갱신).

**슬라이드 6의 TOC 갱신:**
- Agents 행: `goToSlide(7)` (기존 18)
- Skills 행: `goToSlide(13)` (기존 24)
- Context 행: `goToSlide(19)` (기존 7)
- CLAUDE.md 행: `goToSlide(25)` (기존 12)
- Commands 행: `goToSlide(30)` (기존 30 — 변경 없음)

---

### Section 1: Agents (slides 7-12) — "The Person"

**Slide 7** — 섹션 구분자 (`data-level="agents"`, Topic 1)
- Title: "Agents — The Weather Reporter"
- Desc: "An agent is Claude playing a specific role. Meet the weather reporter — a specialist hired to fetch and report weather data for Dubai."

**Slide 8** — "The Restaurant Kitchen" (현재 슬라이드 19)
- 내용: 동일한 비유(단순 프롬프팅 = 아무 주방에서나 소리치기; 에이전트 = 특정 전문가)
- 에이전트 예시를 전반적으로 "weather reporter" 프레이밍으로 갱신
- 단순 프롬프팅 vs weather-agent를 비교하는 2열 카드 유지

**Slide 9** — "Prompting vs. Agent — Side by Side" (현재 슬라이드 20)
- 표를 그대로 유지. 이미 날씨 예시를 잘 활용하고 있다.

**Slide 10** — "Agents Get Their Own Brain" (현재 슬라이드 21)
- Thariq의 팁 유지. 다음과 연결: "the weather reporter works in their own brain — all that web fetching stays out of yours."

**Slide 11** — "How to Create Your Own Agent" (현재 슬라이드 22)
- `/agents` how-to 패턴 유지
- 실제 `weather-agent.md` 경로를 보여주도록 코드 블록 갱신

**Slide 12** — "Agent Config Fields" (현재 슬라이드 23)
- 필드 행 표 유지. `skills: [weather-fetcher]` 필드를 맥락 속에서 보여주는 콜아웃 박스 추가.

---

### Section 2: Skills (slides 13-18) — "What the Reporter Knows"

**Slide 13** — 섹션 구분자 (`data-level="skills"`, Topic 2)
- Title: "Skills — What the Weather Reporter Knows"
- Desc: "Skills are the specific things the reporter has been trained to do. Our reporter has two: fetch the data, and render it as a card."

**Slide 14** — "The Training Manual" (현재 슬라이드 25)
- 리프레이밍: 날씨 리포터는 두 개의 스킬을 갖는다 — weather-fetcher(온도를 가져온다)와 weather-svg-creator(시각 카드를 만든다).
- "Shayan" 예시를 날씨 리포터의 두 스킬로 교체.

**Slide 15** — "When to Turn Something Into a Skill" (현재 슬라이드 26)
- Boris 팁 유지. 예시로 weather-fetcher와 weather-svg-creator 두 가지 추가.

**Slide 16** — "Why Separate Agents and Skills?" (현재 슬라이드 27)
- 2열 유지. weather-agent = 사람, weather-fetcher = 그들의 훈련이라는 점을 강조하도록 갱신.

**Slide 17** — "How to Create Your Own Skill" (현재 슬라이드 28)
- 유지. 코드 블록이 이미 실제 `weather-fetcher` SKILL.md 내용을 보여준다 — 완벽하다.

**Slide 18** — "Skill Config Fields" (현재 슬라이드 29)
- 유지. 참고 추가: weather-fetcher는 에이전트 전용이므로 `user-invocable: false`가 설정되어 있다.

---

### Section 3: Context (slides 19-23) — "The Reporter's Brain"

**Slide 19** — 섹션 구분자 (`data-level="context"`, Topic 3)
- Title: "Context — The Reporter's Brain"
- Desc: "Now that you've met the reporter and know their skills, let's understand what they can actually hold in mind at once."

**Slide 20** — "Claude's Brain" (현재 슬라이드 8)
- 유지. 날씨 리포터와 연결하는 한 문장 추가: "When the weather-agent is dispatched, it gets its own fresh brain — and weather-fetcher is pinned into it at startup."
- 두 다이어그램 모두 유지(context-window.jpeg는 여기에 남는다).

**Slide 21** — "What Loads at Session Start" (현재 슬라이드 9)
- 유지. 날씨 리포터와 연결: "At startup, Claude knows *about* weather-fetcher (description only). When the command runs, the full skill content is loaded into the agent's brain."
- context.jpg는 여기에 유지.

**Slide 22** — "Keep the Brain Clear" (현재 슬라이드 10)
- 분기점(branching-point) 표 유지.

**Slide 23** — "How to Manage Your Context" (현재 슬라이드 11)
- `/context`, `/compact`, `/clear` how-to 유지.

---

### Section 4: CLAUDE.md (slides 24-29) — "The Pocket Rulebook"

**Slide 24** — 섹션 구분자 (`data-level="claude-md"`, Topic 4)
- Title: "CLAUDE.md — The Reporter's Pocket Rulebook"
- Desc: "The reporter consults this at the start of every shift — even though their brain resets overnight."

**Slide 25** — "The Employee Handbook" (현재 슬라이드 13)
- 유지. weather-reporter 프레이밍으로 갱신: CLAUDE.md는 리포터가 방송 전에 읽는 규칙서 — "always report in Celsius unless asked, always cite the source."

**Slide 26** — "How to Create Your CLAUDE.md" (현재 슬라이드 14)
- `/init` how-to 유지.

**Slide 27** — "Grow CLAUDE.md With Every Mistake" (현재 슬라이드 15)
- Boris 팁 유지.

**Slide 28** — "What Goes in CLAUDE.md" (현재 슬라이드 16)
- 코드 블록 유지. 날씨 리포터 요소: 날씨 특화 규칙을 보여주는 주석 추가.

**Slide 29** — "How CLAUDE.md Loads" (현재 슬라이드 17)
- 유지.

---

### Section 5: Commands + Workflow (slides 30-36) — "The Trigger"

**Slide 30** — 섹션 구분자 (`data-level="commands"`, Topic 5)
- Title: "Commands — The Trigger"
- Desc: "One word kicks off the whole chain. `/weather-orchestrator` → agent → skill → SVG card."

**Slide 31** — "Commands — The Entry Point" (현재 슬라이드 31)
- 유지. 좋은 도입부. 이미 weather-orchestrator를 참조한다.

**Slide 32** — "How to Create Your Own Command" (현재 슬라이드 32)
- 유지. 코드 블록이 이미 weather-orchestrator.md를 보여준다.

**Slide 33** — Workflow 하위 섹션 (기존 슬라이드 33, `data-level="workflow"`)
- 섹션 번호 텍스트를 "Topic 6"에서 "Putting It All Together"로 변경
- 바가 100%까지 차도록 `data-level="workflow"` 유지.
- 제목을 다음으로 갱신: "Workflow — All Five Pieces Together"
- Desc: "Watch the weather reporter example run from one keystroke to SVG card output."

**Slide 34** — "Command → Agent → Skill" (현재 슬라이드 34)
- 코드 블록 흐름도 유지. 이미 완벽하다.

**Slide 35** — "Two Ways Skills Are Used" (현재 슬라이드 35)
- 사전 로드(preloaded) vs 직접 호출(direct invocation)을 비교하는 2열 유지.

**Slide 36** — "How to Wire Your Own Workflow" (현재 슬라이드 36)
- 유지. 이미 날씨 워크플로를 예시로 사용한다.

**Slide 37** — Closing (현재 슬라이드 37)
- 유지. 부제를 다음으로 갱신: "Five concepts, one running example"
- 본문 텍스트를 날씨 리포터 서사에 맞게 갱신.

---

## 4. Asset Reuse Inventory

| Asset | Current location | New location | Action |
|---|---|---|---|
| `context-window.jpeg` | Slide 8 (Claude's Brain) | Slide 20 (동일 내용, 번호만 변경) | 유지 — 변경 불필요 |
| `context.jpg` | Slide 9 (What Loads at Session Start) | Slide 21 (동일 내용, 번호만 변경) | 유지 — 변경 불필요 |
| `../../!/claude-jumping.svg` | Slides 1, header | 변경 없음 | 조치 없음 |
| `../../!/root/boris-slider.gif` | Slide 2 | 변경 없음 | 조치 없음 |

두 context 다이어그램은 있던 자리에 그대로 보존된다 — 이를 담은 슬라이드의 번호만 바뀔 뿐이다(8→20, 9→21).

---

## 5. Bookkeeping Impact

### New section-divider positions and `data-level` assignments

| Slide | Topic | `data-level` |
|---|---|---|
| 7 | Agents | `agents` |
| 13 | Skills | `skills` |
| 19 | Context | `context` |
| 25 | CLAUDE.md | `claude-md` |
| 30 | Commands | `commands` |
| 33 | Workflow sub-section | `workflow` |

### TOC `goToSlide` targets on slide 6

| Row | Topic | Old target | New target |
|---|---|---|---|
| Row 1 | Agents | 18 | 7 |
| Row 2 | Skills | 24 | 13 |
| Row 3 | Context | 7 | 19 |
| Row 4 | CLAUDE.md | 12 | 25 |
| Row 5 | Commands | 30 | 30 |

### Journey ticks (no change)

journey 눈금 레일은 이미 위→아래로 다음 순서다: Workflow, Commands, Skills, Agents, CLAUDE.md, Context. 이는 서사 순서의 *역순*이다(맨 위 = 가장 높은 레벨 = 마지막에 도달). 변경 불필요.

### LEVELS map (no change)

6개 레벨 키(`context`, `claude-md`, `agents`, `skills`, `commands`, `workflow`)가 모두 유지된다. 추가되거나 제거되는 키는 없다.

---

## 6. Implementation Approach

HTML은 하나의 커다란 파일이다. 슬라이드들은 새 서사에 맞지 않는 순서로 놓여 있다. 가장 깔끔한 구현 방법은 다음과 같다:

1. 슬라이드 div를 잘라 새 순서로 붙여넣는다(7-12 = 기존 18-23, 13-18 = 기존 24-29, 19-23 = 기존 7-11, 24-29 = 기존 12-17, 30-37 변경 없음).
2. 모든 `data-slide` 속성을 순차적으로 다시 번호 매긴다.
3. 섹션 슬라이드의 `data-level` 속성을 갱신한다.
4. 섹션 구분자의 섹션 번호 텍스트와 h1을 갱신한다.
5. 슬라이드 6의 TOC `goToSlide` 대상을 갱신한다.
6. Workflow 섹션 슬라이드(기존 33)의 섹션 번호 텍스트를 갱신한다.
7. 필요한 곳에 weather-reporter 프레이밍으로 표적 콘텐츠 편집을 가한다.

전체 슬라이드 수: **37** (변경 없음).

---

## 7. Ambiguities — None Load-Bearing

모든 모호한 점은 위에서 해결되었다. 곧바로 구현으로 진행한다.
