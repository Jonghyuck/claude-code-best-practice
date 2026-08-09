<!--
  이 문서는 implementation/claude-scheduled-tasks-implementation.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Scheduled Tasks Implementation

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar_10%2C_2026-white?style=flat&labelColor=555)

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

<a href="#loop-demo"><img src="../!/tags/implemented-hd.svg" alt="Implemented"></a>

`/loop` 스킬은 cron 주기로 반복 작업을 스케줄링하는 데 사용됩니다. 아래는 `/loop 1m "tell current time"` 데모로, 매 분마다 실행되는 간단한 반복 작업입니다.

---

## Loop Demo

### 1. Scheduling the Task

<p align="center">
  <img src="assets/impl-loop-1.png" alt="/loop 1m tell current time — scheduling and cron setup" width="100%">
</p>

`/loop 1m "tell current time"`는 주기(`1m` → 1분마다)를 파싱하고, cron 작업을 생성한 뒤 스케줄을 확정합니다. 주요 사항:

- cron의 최소 세분성은 **1분**입니다 — `1m`은 `*/1 * * * *`로 매핑됩니다
- 반복 작업은 **3일 후 자동으로 만료**됩니다
- 작업은 **세션 범위**입니다 — 메모리에만 존재하며 Claude가 종료되면 중단됩니다
- `cron cancel <job-id>`로 언제든지 취소할 수 있습니다

---

### 2. Loop in Action

<p align="center">
  <img src="assets/impl-loop-2.png" alt="Recurring task firing every minute" width="100%">
</p>

작업은 매 분마다 실행되며, `date`를 실행하고 현재 시각을 보고합니다. 각 반복은 비동기 **UserPromptSubmit** 및 **Stop** 훅을 트리거하는데, 이는 이 저장소 전반에서 사운드 알림에 사용되는 것과 동일한 훅 시스템입니다.

---

## ![How to Use](../!/tags/how-to-use.svg)

```bash
$ claude
> /loop 1m "tell current time"
> /loop 5m /simplify
> /loop 10m "check deploy status"
```

---

## ![How to Implement](../!/tags/how-to-implement.svg)

`/loop`은 Claude Code에 내장된 스킬로, 별도의 설정이 필요 없습니다. 내부적으로 cron 도구(`CronCreate`, `CronList`, `CronDelete`)를 사용하여 반복 스케줄을 관리합니다.
