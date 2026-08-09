<!--
  이 문서는 implementation/claude-goal-implementation.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Goal Implementation

![Last Updated](https://img.shields.io/badge/Last_Updated-May_13%2C_2026-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#goal-tips-from-the-community"><img src="../!/tags/implemented-hd.svg" alt="Implemented"></a>

`/goal`은 조건이 충족될 때까지 여러 턴에 걸쳐 에이전트가 작업을 이어가게 해줍니다 — Claude Code, Codex, Hermes Agent 모두 이를 지원합니다. 커뮤니티는 `/goal`과 잘 어울리는 몇 가지 고효율 프롬프팅 기법으로 수렴해 가고 있습니다.

---

## Goal Tips from the Community

### 1. Ask the agent to propose its own goals

<p align="center">
  <img src="assets/impl-goal-claude.png" alt="Alex Finn tweet — /goal is the most underrated AI feature of 2026" width="50%">
</p>

> 공식 확정. Claude Code가 방금 /goal을 출시했습니다.
>
> 2026년 가장 과소평가된 단 하나의 AI 기능
>
> 이제 Claude Code, Codex, Hermes agent 모두 이 기능을 갖췄습니다.
>
> 이 기능 덕분에 에이전트가 때로는 며칠에 걸쳐 장시간 작업을 완수할 수 있습니다.
>
> 모두가 지금 당장 이 프롬프트를 실행해 봐야 합니다:
>
> '나에 대해, 내 목표와 야망, 그리고 우리가 이미 함께 만들어 온 것들에 관해 네가 아는 바를 토대로, 지금 바로 실행해서 장시간 돌아가며 최고의 결과를 낼 수 있는 /goals 3가지는 무엇일까?'
>
> 하나를 고른 다음, 그것을 위한 프롬프트를 만들어 달라고 요청하세요.
>
> 그러면 당신이 선택한 에이전트가 장시간 작업을 완수해 놀라운 결과를 안겨줄, 아주 강력한 goal 프롬프트 몇 가지 선택지를 받게 될 것입니다.
>
> 오늘 밤 15분만 시간을 내서 이걸 해보세요. 나중에 고마워하게 될 겁니다.

**Source:** [Alex Finn (@AlexFinn) on X](https://x.com/AlexFinn/status/2053976411296452887)

---

### 2. Let the agent draft the /goal prompt for you

<p align="center">
  <img src="assets/impl-goal-codex.png" alt="Meta Alchemist tweet — /goal trick for Codex" width="50%">
</p>

> Codex를 위한 최고의 /goal 요령이 궁금하신가요?
>
> 그냥 당신의 Codex에게 이렇게 말하세요:
>
> "이 세션과 레포를 읽고, 우리가 여기서 이루려는 정확한 의도와 목표를 깊이 분석한 다음, 이를 위한 /goal 프롬프트를 작성해 줘.
>
> 우리가 가진 히스토리와 문서를 반드시 파고들어 100% 명확히 해 줘"
>
> 여기에 이렇게 덧붙일 수도 있습니다:
>
> "특정 부분이 확실하지 않거나 목표를 더 명확히 하기 위해 몇 가지 질문을 하고 싶다면 주저하지 말고 물어봐"
>
> 그런 다음 Codex가 준 내용을 복사해 붙여넣고 앞부분만 /goal로 바꾸세요.
>
> 그러면 그 세션/레포에서 당신이 하려던 바로 그 작업을, 완료에 이를 때까지 멈추지 않고 정확히 수행해 줄 것입니다.

**Source:** [Meta Alchemist (@meta_alchemist) on X](https://x.com/meta_alchemist/status/2054214497443995694)

---

## ![How to Use](../!/tags/how-to-use.svg)

```bash
$ claude
> /goal <condition>
> /goal clear
```

`/goal <condition>`은 Haiku로 평가되는 조건이 충족될 때까지 Claude가 여러 턴에 걸쳐 작업을 이어가게 합니다. 이는 `/loop`(시간 기반) 및 auto 모드(툴 단위)를 보완합니다. Claude Code v2.1.139+가 필요합니다.

전체 동작은 [official docs](https://code.claude.com/docs/en/goal)를 참고하세요.
