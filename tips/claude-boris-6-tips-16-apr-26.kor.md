<!--
  이 문서는 tips/claude-boris-6-tips-16-apr-26.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# 6 Tips for Getting More Out of Opus 4.7 — From Boris Cherny

Claude Code를 만든 Boris Cherny([@bcherny](https://x.com/bcherny))가 2026년 4월 16일에 공유한 팁 스레드입니다 — 지난 몇 주 동안 Opus 4.7을 직접 사용(dogfooding)해 본 뒤 정리한 내용입니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Context

몇 주 동안 Opus 4.7을 직접 써 본 Boris는 "엄청나게 생산적"이라고 느꼈고, 새 모델을 더 잘 활용하는 여섯 가지 방법을 공유했습니다 — 권한 자동화부터 effort 조정, 검증 패턴까지 아우릅니다.

<a href="https://x.com/bcherny"><img src="assets/boris-26-4-16/0.png" alt="Boris Cherny intro tweet — dogfooding Opus 4.7" width="50%" /></a>

---

## 1/ Auto Mode — No More Permission Prompts

Opus 4.7은 복잡하고 오래 걸리는 작업을 즐겨 수행합니다: 심층 리서치, 코드 리팩터링, 복잡한 기능 구현, 성능 벤치마크를 달성할 때까지 반복하기 등입니다. 예전에는 이런 장기 작업을 모델이 수행하는 동안 곁에서 지켜보거나, `--dangerously-skip-permissions`를 사용해야 했습니다.

Anthropic은 최근 더 안전한 대안으로 **auto mode**를 출시했습니다. 이 모드에서는 권한 프롬프트가 모델 기반 분류기로 전달되어, 해당 명령이 실행해도 안전한지 판단합니다:

- 안전하면 자동 승인
- 위험하면 일시 정지하고 사용자에게 질문

이는 모델이 실행되는 동안 곁에서 지켜볼 필요가 없다는 뜻입니다. 나아가 여러 Claude를 병렬로 실행할 수 있게 됩니다 — 안전하다고 판단되면 다음 Claude로 초점을 옮기면 됩니다.

Auto mode는 이제 Max, Teams, Enterprise 사용자를 대상으로 Opus 4.7에서 사용할 수 있습니다. CLI에서는 **Shift+Tab**으로 `Ask permissions` → `Plan mode` → `Auto mode`를 순환하거나, Desktop 또는 VS Code에서는 드롭다운에서 선택하면 됩니다.

<a href="https://x.com/bcherny"><img src="assets/boris-26-4-16/1.png" alt="Boris Cherny on auto mode" width="50%" /></a>

---

## 2/ The New /fewer-permission-prompts Skill

Anthropic은 새로운 `/fewer-permission-prompts` 스킬을 출시했습니다. 이 스킬은 세션 기록을 훑어보며, 안전하지만 반복적으로 권한을 묻는 흔한 bash 및 MCP 명령을 찾아냅니다. 그런 다음 권한 허용 목록(allowlist)에 추가할 명령 목록을 추천해 줍니다.

이를 활용해 권한 설정을 정비하고 불필요한 권한 프롬프트를 줄이세요. 특히 auto mode를 쓰지 않는 경우에 유용합니다.

<a href="https://x.com/bcherny"><img src="assets/boris-26-4-16/2.png" alt="Boris Cherny on /fewer-permission-prompts skill" width="50%" /></a>

---

## 3/ Recaps

Anthropic은 Opus 4.7을 준비하며 이번 주 초 **recaps**를 출시했습니다. recap은 에이전트가 무엇을 했고 다음에 무엇을 할지 요약해 주는 짧은 요약입니다.

몇 분 또는 몇 시간 뒤에 오래 실행되던 세션으로 돌아올 때 매우 유용합니다:

```
* Cogitated for 6m 27s

* recap: Fixing the post-submit transcript shift bug. The styling-flash
  part is shipped as PR #29869 (auto-merge on, posted to stamps). Next:
  I need a screen recording of the remaining horizontal rewrap on `cc -c`
  to target that separate cause. (disable recaps in /config)
```

recap을 원하지 않으면 `/config`에서 비활성화하세요.

<a href="https://x.com/bcherny"><img src="assets/boris-26-4-16/3.png" alt="Boris Cherny on recaps" width="50%" /></a>

---

## 4/ Focus Mode

Boris는 CLI의 새로운 **focus mode**를 매우 마음에 들어 하고 있습니다. 이 모드는 중간 작업을 모두 숨기고 최종 결과에만 집중하게 해 줍니다. 모델은 이제 올바른 명령을 실행하고 올바른 편집을 수행하리라고 그가 대체로 신뢰할 만한 수준에 도달했습니다. 그는 최종 결과만 확인합니다.

`/focus`로 켜고 끌 수 있습니다.

<a href="https://x.com/bcherny"><img src="assets/boris-26-4-16/4.png" alt="Boris Cherny on focus mode" width="50%" /></a>

---

## 5/ Configure Your Effort Level

Opus 4.7은 thinking budget 대신 **adaptive thinking**을 사용합니다. 모델이 더 많이 또는 더 적게 사고하도록 조정하려면 effort를 조정하세요.

- **낮은 effort** — 더 빠른 응답과 더 적은 토큰 사용
- **높은 effort** — 최고의 지능과 역량

슬라이더는 다섯 단계를 제공합니다: `low` · `medium` · `high` · `xhigh` · `max` — 왼쪽이 속도(Speed), 오른쪽이 지능(Intelligence)입니다.

<a href="https://x.com/bcherny"><img src="assets/boris-26-4-16/5.png" alt="Boris Cherny on effort levels" width="50%" /></a>

---

## 6/ Give Claude a Way to Verify Its Work

마지막으로, Claude가 자신의 작업을 검증할 수단을 반드시 마련해 주세요. 이는 늘 중요했지만 — 이제 4.7은 Claude에서 얻을 수 있는 것을 2~3배로 끌어올리므로, 그 어느 때보다 중요합니다.

검증 방식은 작업 종류에 따라 달라집니다:

- **백엔드 작업** — Claude가 서버/서비스를 실행해 엔드투엔드로 테스트하게 하세요
- **프론트엔드 작업** — [Claude Chromium extension](https://code.claude.com/docs/en/chrome)을 사용해 Claude가 브라우저를 제어할 수 있게 하세요
- **데스크톱 앱** — Computer Use를 사용하세요

요즘 Boris의 프롬프트는 `Claude do blah blah /go`와 같은 형태인데, 여기서 `/go`는 다음을 수행하는 스킬입니다:

1. bash, 브라우저, 또는 computer use를 사용해 스스로 엔드투엔드로 테스트
2. `/simplify` 실행
3. PR 올리기

오래 걸리는 작업일수록 검증은 더욱 중요합니다 — 작업으로 돌아왔을 때 코드가 제대로 동작한다는 것을 알 수 있기 때문입니다.

<a href="https://x.com/bcherny"><img src="assets/boris-26-4-16/6.png" alt="Boris Cherny on verification" width="50%" /></a>

---

## Sources

- [Boris Cherny (@bcherny) on X — April 16, 2026](https://x.com/bcherny)
