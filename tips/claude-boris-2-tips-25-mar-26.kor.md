<!--
  이 문서는 tips/claude-boris-2-tips-25-mar-26.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Squash Merging & PR Size Distribution — Tips from Boris Cherny

Claude Code의 개발자인 Boris Cherny ([@bcherny](https://x.com/bcherny))가 2026년 3월 25일에 공유한 인사이트 요약입니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 1/ 266 Contributions in a Single Day — Always Squash

Boris는 자신의 GitHub 기여도 그래프를 공유했는데, **3월 24일 하루에 266건의 기여**가 있었습니다 — 이는 **141개의 PR을 항상 squash**하여 만들어졌으며 PR당 중앙값은 **118줄**이었습니다.

- Squash 병합은 브랜치의 모든 커밋을 대상 브랜치의 단일 커밋으로 합쳐 히스토리를 깔끔하고 선형적으로 유지합니다
- PR 하나 = 커밋 하나이므로 기능 전체를 되돌리기 쉽고 `git bisect`가 간단해집니다
- 고속 AI 지원 워크플로(하루 141개 PR)에서 squash는 실용적인 선택입니다 — 브랜치 내의 개별 "fix lint", "try this" 같은 커밋은 노이즈일 뿐입니다

<a href="https://x.com/bcherny/status/2038552880018538749"><img src="assets/boris-26-3-25/1.png" alt="Boris Cherny — 266 contributions, always squashed" width="50%" /></a>

---

## 2/ PR Size Distribution — Keep PRs Small

Boris는 그 141개 PR의 크기 분포를 공유했으며, 총 **45,032줄이 변경**되었습니다(추가 + 삭제):

| Metric | Lines (add+del) | Meaning |
|--------|---------------:|---------|
| **p50** | **118** | PR 크기 중앙값 — 전체 PR의 절반이 118줄 이하였습니다 |
| p90 | 498 | PR의 90%가 500줄 미만이었습니다 |
| **p99** | **2,978** | 약 1개의 PR만 3천 줄가량을 초과했습니다 |
| min | 2 | 가장 작은 PR — 간단한 2줄 수정 |
| max | 10,459 | 가장 큰 단일 PR — 아마도 마이그레이션이나 생성된 코드 |

- **중앙값 118줄**이라는 것은 하루 141개 PR 속에서도 대부분의 PR이 집중적이고 리뷰하기 좋음을 의미합니다
- 분포는 오른쪽으로 크게 치우쳐 있습니다 — 가끔 나오는 큰 PR(대량 이름 변경, 마이그레이션)은 불가피하지만, 일반적인 기준은 작게 유지됩니다
- 작은 PR은 병합 충돌 위험을 줄이고 리뷰하기 쉬우며, 깔끔한 되돌리기를 위한 squash 병합과 완벽하게 어울립니다

<a href="https://x.com/bcherny/status/2038552880018538749"><img src="assets/boris-26-3-25/2.png" alt="Boris Cherny — PR size distribution table" width="50%" /></a>

---

## Sources

- [Boris Cherny (@bcherny) on X — March 25, 2026](https://x.com/bcherny)
