<!--
  이 문서는 reports/llm-day-to-day-degradation.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# LLM Day-to-Day Degradation: Myth vs Reality

배포된 LLM의 성능이 모델 가중치가 고정되어 있음에도 날마다 달라질 수 있을까? 입증된 원인, 인프라 버그, 심리적 요인을 깊이 있게 파헤친다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<table width="100%">
<tr>
<td width="50%"><a href="https://x.com/nicksdot/status/2029520949176049704"><img src="assets/llm-degradation.png" alt="Twitter users reporting day-to-day Claude quality degradation" width="100%" /></a></td>
<td width="50%"><a href="https://x.com/levelsio/status/2029369159893569680"><img src="assets/llm-degradation-2.png" alt="Twitter users reporting day-to-day Claude quality degradation" width="100%" /></a></td>
</tr>
</table>

---
---

# 🔥 Claude Code Ops 4.6 Analysis. High Reasoning

Anthropic이 Opus 4.6 같은 모델을 출시하면 **모델 가중치** — 학습된 수십억 개의 파라미터 — 는 고정된다. 학습에는 막대한 비용이 든다(수백만 달러, 수 주의 연산). 밤사이에 모델을 재학습하는 사람은 없다.

하지만 가중치는 훨씬 더 큰 시스템의 한 계층일 뿐이다. 연구에 따르면 모델 가중치가 고정된 상태에서도 실제 또는 체감 품질 변화를 일으킬 수 있는 **최소 7가지 별개의 메커니즘**이 존재한다.

| Question | Answer |
|----------|--------|
| 출시 후 모델 가중치가 바뀌는가? | **아니오** — 모든 공급자가 확인함 |
| 모델이 날마다 다르게 동작할 수 있는가? | **예** — ±8-14% 편차로 입증됨 |
| 의도적인 "너프(nerfing)"인가? | **아니오** — 고의적 성능 저하의 증거 없음 |
| 인프라 버그는 실제인가? | **예** — Anthropic이 요청의 최대 16%에 영향을 준 버그 3개를 확인함 |
| 일부는 심리적인 것인가? | **예** — 확증 편향과 허니문 효과는 실재함 |
| 시스템 프롬프트/사후 학습이 바뀔 수 있는가? | **예** — 여러 공급자에서 문서화됨 |
| 사용자는 자신의 체감을 믿어야 하는가? | **부분적으로** — 실제 원인은 존재하지만, 체감이 이를 증폭함 |

---

## The Full Inference Stack

모델 가중치는 고정되어 있지만, **그 위의 아홉 개 계층**이 각각 독립적으로 사용자가 경험하는 결과에 영향을 줄 수 있다.

```
┌──────────────────────────────────────────────┐
│  YOUR SESSION CONTEXT                        │  ← Degrades within session
│  (accumulated errors, long conversations)    │
├──────────────────────────────────────────────┤
│  SYSTEM PROMPT                               │  ← Updated regularly
│  (safety rules, behavior instructions)       │
├──────────────────────────────────────────────┤
│  POST-TRAINING (RLHF / Fine-tuning)         │  ← Can be updated quietly
│  (instruction following, safety alignment)   │
├──────────────────────────────────────────────┤
│  SAMPLING PARAMETERS                         │  ← Can be tuned server-side
│  (temperature, top-p, top-k)                 │
├──────────────────────────────────────────────┤
│  SPECULATIVE DECODING                        │  ← Draft model quality varies
│  (draft model predictions + verification)    │
├──────────────────────────────────────────────┤
│  MoE ROUTING / BATCH COMPOSITION             │  ← ±8-14% variance proven
│  (which experts activate per request)        │
├──────────────────────────────────────────────┤
│  HARDWARE ROUTING                            │  ← TPU vs GPU vs Trainium
│  (which cluster serves your request)         │
├──────────────────────────────────────────────┤
│  QUANTIZATION LEVEL                          │  ← May vary under load
│  (FP16 vs INT8 vs INT4 precision)            │
├──────────────────────────────────────────────┤
│  COMPILER & RUNTIME                          │  ← XLA bugs proven real
│  (XLA:TPU, CUDA, hardware-specific code)     │
├──────────────────────────────────────────────┤
│  MODEL WEIGHTS (FROZEN)                      │  ← These DON'T change
│  (billions of learned parameters)            │
└──────────────────────────────────────────────┘
```

핵심 멘탈 모델은 이것이다. **고정된 가중치 ≠ 고정된 동작**. 이는 "엔진이 같으면 주행 경험도 같다"고 말하면서 타이어, 노면 상태, 연료 품질, 운전자의 피로를 무시하는 것과 같다.

---

## Proven Causes: Infrastructure Bugs

### Anthropic's September 2025 Postmortem

2025년 9월, Anthropic은 2025년 8월부터 9월 사이에 Claude의 품질을 저하시킨 **세 가지 별개의 인프라 버그**를 밝힌 상세 포스트모템을 발표했다. 공식 성명은 다음과 같다.

> "We never reduce model quality due to demand, time of day, or server load. The problems our users reported were due to infrastructure bugs alone."

### Bug #1 — Context Window Routing Error

Sonnet 4 요청이 실수로 표준 서버 대신 1M 토큰 컨텍스트 윈도우용으로 구성된 서버로 라우팅되었다.

- **Timeline**: 8월 5일에 도입, 로드 밸런싱 변경 이후 8월 29일에 악화
- **Peak impact**: 최악의 시간대(8월 31일)에 Sonnet 4 요청의 16%가 영향받음
- **User impact**: Claude Code 사용자의 약 30%가 최소 한 개의 저하된 메시지를 겪음
- **Insidious detail**: 라우팅이 "sticky"했다 — 한 번 문제 서버에 걸리면 이후 요청도 계속 그곳으로 갔다
- **Fixed**: 9월 4–18일(플랫폼 전반에 걸쳐 롤아웃)

### Bug #2 — TPU Output Corruption

TPU 서버의 잘못된 구성으로 인해 토큰 생성 중 오류가 발생하여, 거의 나타나지 않아야 할 토큰에 높은 확률이 할당되었다.

- **Symptoms**: 영어 응답 중간에 태국어나 중국어 문자가 등장, 명백한 코드 문법 오류
- **Affected**: Opus 4.1 및 Opus 4(8월 25–28일), Sonnet 4(8월 25일–9월 2일)
- **Scope**: Claude API에만 해당; 서드파티 플랫폼은 영향받지 않음
- **Fixed**: 9월 2일에 롤백

### Bug #3 — XLA:TPU Compiler Miscompilation (the nastiest)

정밀도 문제를 고치기 위한 코드 변경이 실수로 Google XLA:TPU의 **잠복 컴파일러 버그**를 노출시켰다.

- **Root cause**: 근사(approximate) top-k 연산(가장 가능성 높은 다음 토큰을 고르는 데 사용)이 "특정 배치 크기와 모델 구성에서만 때때로 완전히 잘못된 결과를 반환했다"
- **Why it was hard to find**: 그 앞뒤에 어떤 연산이 실행되는지, 그리고 디버깅 도구가 활성화되어 있는지에 따라 동작이 달라졌다
- **Hidden for months**: 2024년 12월의 이전 우회책이 실수로 이 더 깊은 버그를 가리고 있었다
- **Affected**: Haiku 3.5 확인됨; Sonnet 4 및 Opus 3의 일부는 의심됨
- **Resolution**: 근사 top-k에서 정확(exact) top-k로 전환; "모델 품질은 타협 불가"이므로 "약간의 효율 손실"을 감수함

### Why Detection Was Difficult

Anthropic 자체 자동 평가는 사용자가 보고한 품질 저하를 잡아내지 못했는데, "부분적으로는 Claude가 고립된 실수에서 잘 회복하는 경우가 많기 때문"이었다. 각 버그는 서로 다른 플랫폼에서 서로 다른 비율로 서로 다른 증상을 일으켜 "어느 하나의 원인도 가리키지 않는 혼란스러운 보고의 뒤섞임"을 만들었다.

핵심 맥락: Claude는 **세 가지 서로 다른 하드웨어 플랫폼**(AWS Trainium, NVIDIA GPU, Google TPU)에서 실행되며, 각각 실패 모드, 컴파일러, 정밀도 동작이 다르다. 당신의 요청은 날마다 다른 하드웨어에 걸릴 수 있다.

---

## Proven Causes: MoE Routing Variance

현대의 대형 모델은 흔히 **Mixture-of-Experts (MoE)** 아키텍처를 사용하는데, 여기서는 각 입력마다 모델 파라미터의 일부("experts")만 활성화된다. 학습된 라우터가 어떤 expert를 사용할지 결정한다.

Scale AI의 연구는 중요한 발견을 밝혔다.

> "The combination of Sparse MoE and batched inference creates unpredictable results because the composition of a batch can determine which expert your query gets routed to, and the mix of queries from other users in the same batch is not deterministic."

### Measured Day-to-Day Variance Across Providers

| Provider | Day-to-Day Score Variance |
|----------|--------------------------|
| OpenAI (GPT-4 variants) | ±10–12% |
| Anthropic (Claude variants) | ±8–11% |
| Google (Gemini variants) | ±9–14% |

구체적인 예: 같은 모델이 **어느 날은 탈옥 저항성에서 77%, 다음 날은 63%**를 기록했다. 같은 모델, 같은 가중치, 같은 테스트에서 인프라만으로 14퍼센트포인트의 편차가 났다.

이는 버그가 전혀 없고 변경이 전혀 없어도, 요청이 어떻게 배치되고 라우팅되는지에 따라 순수하게 같은 모델이 날마다 눈에 띄게 다른 품질의 출력을 낼 수 있음을 의미한다. 날마다의 잡음이 10–15%일 때 A/B 테스트로는 5%의 품질 신호를 신뢰성 있게 검출할 수 없다.

---

## Proven Causes: System Prompt & Post-Training Updates

### System Prompt Changes

모델 가중치는 바뀌지 않지만, 그 가중치를 감싸는 **시스템 프롬프트**는 언제든지 업데이트될 수 있다. Claude 시스템 프롬프트의 변천을 분석하면 수십 차례의 반복이 드러나며, 원치 않는 동작을 패치하기 위해 추가되는 짧은 지시인 "핫픽스(hot-fix)"가 정기적으로 추가되고 제거된다.

Claude 3.7의 시스템 프롬프트에는 흔한 LLM "함정(gotcha)"을 겨냥한 여러 핫픽스 지시가 담겨 있었다. Claude 4.0의 시스템 프롬프트는 이를 모두 제거하고, 대신 강화 학습을 통한 사후 학습에서 해당 동작들을 다루었다.

### The Post-Training Theory

설명되지 않는 품질 변화에 대한 가장 설득력 있는 이론: 회사는 기반 모델 가중치를 바꾸지 않고도 **파인튜닝과 RLHF**(인간 피드백 기반 강화 학습)를 업데이트할 수 있다. 이렇게 하면 갱신된 안전 가드레일과 지시 이행 조정을 통해 동작을 바꾸면서도 "모델은 바뀌지 않았다"고 말하는 것이 기술적으로 사실일 수 있다.

---

## Proven Causes: Silent Model Swaps

OpenAI는 사용자가 상호작용하는 모델을 조용히 바꾼 사례가 여러 차례 문서화되었다.

- 모델 선택기를 하룻밤 사이에 제거하여 사용자를 GPT-4o에서 GPT-5로 강제 이동
- GPT-4o를 설정에서 수동 토글이 필요한 숨겨진 "레거시 모델"로 만들고 앱 내 알림은 없음
- 사용자를 잘못된 모델로 라우팅한 "autoswitcher" 버그
- 더불어 구독자들은 동의 없이 모델이 "제한된 버전"으로 전환되었다고 보고함

Sam Altman은 롤아웃이 "기대했던 것보다 조금 더 삐걱거렸다"고 인정했다. Reddit 스레드는 새 모델을 "재앙(disaster)"이자 "다운그레이드"라고 부르며 수천 개의 추천을 받았다.

이는 모델 교체가 업계에서 **실제로 일어난다**는 것을 보여준다 — 때로는 의도적으로(제품 결정), 때로는 우발적으로(라우팅 버그).

---

## Contributing Factors

### Quantization Under Load

수백만 사용자를 비용 효율적으로 서비스하기 위해, 회사는 정밀도를 FP16에서 INT8 또는 INT4로 낮춘 **양자화(quantized)** 버전의 모델을 제공할 수 있다. 이는 메모리 사용량을 2–4배 줄이고 추론을 가속할 수 있지만, 미묘한 품질 손실을 유발한다. 공급자가 부하에 따라 양자화 수준을 동적으로 전환하는지는 논쟁적이지만, 기술적 역량은 존재하며 vLLM 및 TensorRT 같은 서빙 프레임워크에 잘 문서화되어 있다.

### Speculative Decoding

현대의 서빙 스택은 더 작은 "draft" 모델을 사용해 여러 토큰을 앞서 예측한 뒤, 실제 모델이 이를 검증하게 한다. 이론상으로는 동일한 출력 분포를 보존하지만, 실제로는 수락률이 도메인과 맥락에 따라 달라진다. 기본 제공 draft 모델은 일부 경우에는 잘 작동하지만 도메인 특화 작업이나 매우 긴 컨텍스트에서는 종종 어려움을 겪는다.

### Context Window Pollution

긴 코딩 세션에서는 앞선 실수가 컨텍스트에 쌓인다. 모델은 자신의 오류를 보고 이를 반복할 수 있다. 이것이 단일 세션 내에서 "Claude가 멍청해졌다"는 가장 흔한 원인이다 — 모델이 저하되는 것이 아니라 컨텍스트 오염이다.

**Practical tip**: 품질이 이상하게 느껴지면 `/compact`를 쓰거나 새 세션을 시작하라. 이것이 당신이 할 수 있는 가장 실질적인 조치다.

---

## The Stanford Study — And Why It's Complicated

Stanford와 UC Berkeley(Chen, Zaharia, Zou)의 2023년 획기적 연구 — "How is ChatGPT's Behavior Changing Over Time?" — 는 LLM이 저하된다는 증거로 자주 인용된다. 대표적 발견은 다음과 같다.

> GPT-4's accuracy on "Is this number prime? Think step by step" fell from **97.6% to 2.4%** between March and June 2023.

### What the Study Proved

- "같은" LLM 서비스의 동작이 짧은 기간에 **상당히 바뀔 수 있다**
- 서로 다른 능력이 반대 방향으로 움직일 수 있다(GPT-4는 수학에서 나빠지고, GPT-3.5는 좋아졌다)
- 코드 생성 품질이 떨어졌다(GPT-4 실행 가능 코드: 52% → 10%)
- 이 연구가 **"LLM drift"**라는 용어를 만들었다

### Methodological Critiques

- 3월 버전은 **temperature 0.0**을 사용한 반면 6월 버전은 **temperature 1.0**을 사용했다 — 무작위성을 높이는 근본적인 교란 변수다
- 작업당 **쿼리 500개**뿐이었다 — 확정적 통계 주장을 하기에는 너무 적다
- "수학 문제"는 실제로는 예/아니오 질문이었고, 바뀐 것은 모델의 수학 능력이 아니라 추측 패턴이었다
- 변화는 품질 저하가 아니라 의도적인 **사후 학습 안전 업데이트**를 반영했을 가능성이 높다

이 연구는 중요한 것을 입증했다 — LLM 동작이 시간에 따라 변한다는 것 — 하지만 그 메커니즘은 비의도적 저하가 아니라 의도적 업데이트였을 가능성이 높다.

---

## The Psychology

### Confirmation Bias

누군가 "Claude is dumb today"라고 트윗하면, 당신은 모든 실수를 알아채기 시작한다. 아무도 불평하지 않는 날에는 같은 오류를 대수롭지 않게 넘긴다. 소셜 미디어는 이 효과를 증폭한다.

### The Honeymoon Effect

사용자는 새 모델에 대해 초기 허니문 기간을 경험한 뒤 점차 한계를 발견한다. 모델이 바뀐 것이 아니라 — 기대가 능력이 뒷받침하는 것보다 더 빨리 상향 조정된 것이다.

### Task Difficulty Variance

당신의 작업은 날마다 다르다. 어려운 문제가 많은 날에는, 모델이 그대로임에도 마치 나빠진 것처럼 느껴진다.

### The "Weekend Claude" Myth

많은 사용자가 요일별 패턴을 믿음에도 불구하고, 엄밀한 분석 결과 요일별 품질 패턴에 대한 **일관된 증거는 발견되지 않았다**. "AI is Dumber on Mondays"라는 제목의 한 분석도 성과가 없었다.

### Stochastic Nature of LLMs

LLM은 확률적이다. 같은 프롬프트가 매번 다른 출력을 낼 수 있다. 운이 나쁜 연속에서는 형편없는 응답을 여러 번 연달아 받을 수 있다 — 이는 순수한 무작위성이지 저하가 아니다.

---

## Bottom Line

사용자가 묘사하는 현상은 **실재하지만 잘못 귀인된다**.

- **Correct**: 특정 날에 그들의 경험이 저하되었다
- **Incorrect**: 모델이 의도적으로 "너프"되었다

실제 원인은 다음의 조합이다.

1. **Infrastructure bugs** — Anthropic의 포스트모템으로 입증됨(요청의 최대 16% 영향)
2. **MoE routing variance** — 변경이 전혀 없어도 Scale AI가 측정한 ±8-14% 품질 편차
3. **System prompt and post-training updates** — 여러 공급자에서 문서화됨
4. **Hardware heterogeneity** — TPU vs GPU vs Trainium, 각각 실패 모드가 다름
5. **Context pollution** — 긴 세션은 세션 내 품질을 저하시킴
6. **Confirmation bias** — 소셜 미디어가 체감 패턴을 증폭함
7. **Stochastic variance** — 같은 모델, 같은 프롬프트, 매번 다른 출력

측정 문제는 심각하다. ±8-14%의 날마다 편차는 실제 5% 품질 변화를 잡음과 구별할 수 없음을 뜻한다. 이것이 "다 네 머릿속 착각"이라는 진영과 "그들이 너프했다"는 진영이 모두 확신하는 이유다 — 신호 대 잡음비 때문에 개별 경험만으로는 구별이 불가능하다.

---

## Sources

- [Anthropic: A Postmortem of Three Recent Issues](https://www.anthropic.com/engineering/a-postmortem-of-three-recent-issues) — 세 가지 인프라 버그를 상세히 다룬 공식 포스트모템 (2025년 9월)
- [Anthropic Reveals Three Infrastructure Bugs — InfoQ](https://www.infoq.com/news/2025/10/anthropic-infrastructure-bugs/) — 포스트모템에 대한 기술 분석
- [How is ChatGPT's Behavior Changing Over Time? — Stanford/UC Berkeley](https://arxiv.org/abs/2307.09009) — LLM drift에 관한 획기적 연구 (2023)
- [The Truth About ChatGPT's Degrading Capabilities — TechTalks](https://bdtechtalks.com/2023/07/24/chatgpt-capabilities-degrading-study/) — Stanford 연구에 대한 방법론적 비판
- [LLMs Are Getting Dumber and We Have No Idea Why — Ignorance.ai](https://www.ignorance.ai/p/llms-are-getting-dumber-and-we-have) — 체감 저하에 대한 다섯 가지 이론
- [When Claude Forgets How to Code — Robert Matsuoka](https://hyperdev.matsuoka.com/p/when-claude-forgets-how-to-code) — Claude 품질 변동과 인프라 원인 분석
- [Smoothing Out LLM Variance — Scale AI](https://scale.com/blog/smoothing-out-llm-variance) — 공급자 전반에서 측정된 ±8-14%의 날마다 편차
- [What We Can Learn from Anthropic's System Prompt Updates — PromptLayer](https://blog.promptlayer.com/what-we-can-learn-from-anthropics-system-prompt-updates/) — 시스템 프롬프트 변천 분석
- [Claude's System Prompt Changes Reveal Anthropic's Priorities — Drew Breunig](https://www.dbreunig.com/2025/06/03/comparing-system-prompts-across-claude-versions.html) — 시스템 프롬프트의 핫픽스 패턴
- [Complaints About Secretly Switching Models — OpenAI Forum](https://community.openai.com/t/complaints-about-secretly-switching-models/1360150) — 문서화된 조용한 모델 교체
- [Speculative Decoding — BentoML LLM Inference Handbook](https://bentoml.com/llm/inference-optimization/speculative-decoding) — draft 모델이 서빙에 미치는 영향
- [A Visual Guide to Mixture of Experts — Maarten Grootendorst](https://newsletter.maartengrootendorst.com/p/a-visual-guide-to-mixture-of-experts) — MoE 아키텍처와 라우팅 해설

---
---

# 🔥 Codex 5.3 High Reason and Finding

### Report Scope

이 섹션은 Claude 출력 품질이 떨어지는 짧은 구간에서 왜 사용자가 Codex 5.3을 코딩 작업에서 안정적이거나 더 강하다고 느낄 수 있는지를 설명한다. 초점은 영구적인 모델 품질 순위가 아니다. 초점은 실제 서빙 조건에서의 단기 프로덕션 동작이다.

Report date: March 5, 2026.

### Observed Pattern

보고된 패턴은 다음과 같다.

1. 일정 기간 동안 모델 품질이 수용 가능하다.
2. 며칠간 품질이 저하되는 것처럼 보인다.
3. 품질이 이전 기준선 가까이로 돌아온다.

이 형태는 대개 영구적인 기반 모델 능력 변화가 아니라 서빙 스택 또는 롤아웃 패턴이다. 영구적 능력 저하라면 명시적 롤백이나 수정 없이 이렇게 빨리 회복하는 것이 정상적이지 않다.

### High Reason: Why Codex 5.3 Can Look Better in a Bad Window

Codex 5.3은 다른 공급자의 저하 기간 동안 명백히 더 강해 보일 수 있는데, 이는 동시에 발생할 수 있는 몇 가지 기술적 이유 때문이다.

1. 제품 목적 적합성. Codex 5.3은 코드 생성 및 에이전트형 코딩 워크플로에 최적화되어 있어, 원(raw) 모델 강도가 동등하더라도 도구 오케스트레이션, 저장소 추론, 코드 중심 지시 튜닝 덕분에 더 나은 코딩 결과를 낼 수 있다.
2. 추론 정책 차이. 공급자는 지연, 추론 깊이, 디코딩 기본값을 각자 독립적으로 조정한다. 한 공급자의 더 보수적인 정책이, 같은 날 다른 공급자의 공격적인 속도 최적화 정책보다 "더 똑똑해" 보일 수 있다.
3. 서빙 경로 분리. 두 공급자가 최신(SOTA) 모델을 호스팅하더라도, 서로 다른 라우팅 계층, 컴파일러/런타임 스택, 롤아웃 파이프라인을 운영한다. 한 스택의 사건이 다른 쪽의 상관된 저하를 의미하지는 않는다.
4. 롤아웃과 롤백 타이밍. 한 공급자가 롤아웃 중이고 다른 공급자가 안정적이라면, 모델 가중치에 근본적인 장기 변화가 없어도 사용자는 큰 일시적 품질 격차를 볼 수 있다.
5. 세션 수준 오염 효과. 긴 코딩 채팅에서는 오류 누적이 체감 저하를 증폭할 수 있다. 실패한 세션이 리셋되었거나 도구 루프가 더 빨리 회복되었다는 이유만으로 경쟁 어시스턴트가 더 나아 보일 수 있다.

### Detailed Finding

"Claude가 나흘 정도 매우 약하게 느껴지다가 돌아왔다" 같은 보고에 대한 가장 개연성 있는 설명은 다음과 같다.

1. 공급자 측 사건, 라우팅 문제, 디코딩/런타임 버그, 또는 롤아웃 회귀가 요청의 일부에 영향을 주었다.
2. 그 문제가 실제 워크플로에서 반복적으로 인지될 만큼 오래 지속되었다.
3. 문제가 수정되거나 롤백되었다.
4. 체감 품질이 빠르게 돌아왔다.

같은 기간 동안 Codex 5.3은 동일한 사건 경로를 공유하지 않았고 코딩 작업 최적화가 실질적 결과에서 격차를 확대했기 때문에 상당히 더 나아 보일 수 있었다.

### Hypothesis Ranking for This Pattern

| Hypothesis | Likelihood | Rationale |
|------------|------------|-----------|
| 공급자 사건 후 롤백 | High | 며칠간의 하락 후 빠른 회복에 가장 잘 부합 |
| 서빙 구성 변경 (샘플링/지연/추론 예산) | High | 모델 재학습 없이 갑작스러운 동작 변화를 일으키는 흔한 원인 |
| 조용한 별칭 또는 스냅샷 이동 | Medium-High | 눈에 보이는 사용자 동작 없이 동작이 바뀔 수 있음 |
| 프롬프트 드리프트와 컨텍스트 오염만 | Medium | 세션을 저하시킬 수 있으나, 광범위한 다일간 보고를 단독으로 설명하기는 어려움 |
| 영구적 기반 모델 저하 | Low | 이전 품질로의 빠른 복귀와 부합하지 않음 |

### What Would Confirm or Falsify This Finding

이를 높은 신뢰의 추론에서 확실한 증거로 바꾸려면, 여러 날에 걸쳐 동일한 작업 세트에 대한 요청 수준 텔레메트리를 수집하라.

1. 요청 시점의 정확한 모델 식별자와 스냅샷/별칭.
2. 공급자가 노출하는 백엔드 지문이나 릴리스 마커.
3. 디코딩 파라미터(temperature, top_p, top_k, max tokens).
4. 지연, 타임아웃, 오류율 추적.
5. 고정된 코딩 벤치마크 프롬프트 세트에 대한 구조화된 품질 점수.
6. 실패 지점에서의 세션 길이와 토큰 컨텍스트 깊이.

품질 하락이 사건 구간, 구성 변경, 또는 백엔드 지문 변화와 상관된다면 사건/구성 가설이 확인된다. 그런 변화가 없고 저하가 긴 세션에서만 나타난다면 컨텍스트 오염이 주된 설명이 된다.

### Practical Engineering Guidance

프로덕션에서 날마다의 편차를 줄이려면 다음을 하라.

1. 가능하면 부동(floating) 별칭 대신 모델 스냅샷을 고정하라.
2. 요청 메타데이터(모델 ID, 파라미터, 지연, 오류, 응답 품질 라벨)를 저장하라.
3. 코딩 작업에 대한 고정된 일일 카나리 스위트를 실행하고 회귀 시 알림을 보내라.
4. 여러 번 실패한 턴 이후에는 장기 실행 세션을 리셋하거나 컴팩트하라.
5. 사건 구간을 위한 대체 공급자/모델 경로를 유지하라.
6. 내부 대시보드에서 "모델 품질"과 "서빙 신뢰성"을 분리하라.

### Final Conclusion

짧은 Claude 저하 구간 동안 Codex 5.3이 더 나아 보이는 것은 현대 LLM 운영에서 기술적으로 개연성 있고 예상되는 결과다. 가장 강력한 설명은 영구적 모델 붕괴가 아니다. 가장 강력한 설명은 한 공급자에서의 일시적 서빙 경로 저하가, 같은 기간 동안 다른 공급자의 코딩 특화 최적화 및 안정적 운영과 결합된 것이다.
