<!--
  이 문서는 reports/claude-usage-and-rate-limits.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Claude Code: Usage, Rate Limits & Extra Usage

Claude Code에서 사용량 제한이 어떻게 동작하는지, 그리고 제한에 도달했을 때 어떻게 계속 작업할 수 있는지 이해합니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Overview

구독 플랜(Pro, Max 5x, Max 20x)의 Claude Code에는 롤링 윈도우 방식으로 리셋되는 사용량 제한이 있습니다. 내장된 세 가지 슬래시 명령이 사용량을 모니터링하고 관리하는 데 도움을 줍니다:

| Command | Description | Available To |
|---------|-------------|--------------|
| `/usage` | 플랜 제한과 rate limit 상태를 확인 | Pro, Max 5x, Max 20x |
| `/extra-usage` | 제한에 도달했을 때 종량제(pay-as-you-go) 초과분을 설정 | Pro, Max 5x, Max 20x |
| `/cost` | 현재 세션의 토큰 사용량과 지출을 표시 | API key 사용자 |

---

## `/usage` — Check Your Limits

현재 플랜의 사용량 제한과 rate limit 상태를 보여줍니다. 제한에 도달하기 전에 남은 용량이 얼마나 되는지 확인하는 데 유용합니다.

---

## `/extra-usage` — Keep Working Past Limits

`/extra-usage` 명령은 **종량제(pay-as-you-go) 초과분 과금**을 설정하여, 플랜의 rate limit에 도달했을 때 작업이 차단되는 대신 Claude Code가 끊김 없이 계속 동작하도록 합니다.

### How It Works

1. 플랜의 rate limit에 도달합니다(제한은 5시간마다 리셋됩니다)
2. extra usage가 활성화되어 있고 사용 가능한 자금이 있으면 Claude Code가 중단 없이 계속됩니다
3. 초과분 토큰은 구독료와 별도로 **표준 API 요율(standard API rates)**로 과금됩니다

### Setting It Up

CLI의 `/extra-usage` 명령이 설정 과정을 안내합니다. claude.ai의 **Settings > Usage**에서 웹으로 설정할 수도 있습니다:

1. extra usage를 활성화합니다
2. 결제 수단을 추가합니다
3. **월 지출 한도(monthly spending cap)**를 설정합니다(또는 무제한 선택)
4. 선택적으로, 잔액이 임계값 아래로 떨어질 때 자동 충전되는 **선불 자금(prepaid funds)**을 추가합니다

### Key Details

| Detail | Value |
|--------|-------|
| Daily redemption limit | $2,000/day |
| Billing | 구독과 별도, 표준 API 요율로 과금 |
| Limit reset window | 5시간마다 |

### Known Issue

2026년 2월 기준으로 `/extra-usage` CLI 명령은 [undocumented](https://github.com/anthropics/claude-code/issues/12396) 상태이며, 명확한 설정 옵션 없이 로그인 창을 열 수 있습니다. 현재로서는 **claude.ai 웹 인터페이스**를 통한 설정이 더 안정적인 경로입니다.

---

## `/cost` — Session Spending (API Users)

(구독 플랜이 아니라) API key로 인증하는 사용자에게 `/cost`는 다음을 보여줍니다:

- 현재 세션의 총 비용
- API 소요 시간과 실제 경과 시간(wall time)
- 토큰 사용량 분석
- 수행된 코드 변경 내역

이 명령은 Pro/Max 구독 사용자에게는 해당되지 않습니다.

---

## Fast Mode and Extra Usage

Fast mode(`/fast`)는 Claude Opus 4.6을 더 빠른 출력으로 사용합니다. extra usage와 특별한 과금 관계를 갖습니다:

- Fast mode 사용량은 첫 토큰부터 **항상 extra usage로 과금**됩니다
- 이는 구독 플랜에 남은 사용량이 있더라도 적용됩니다
- Fast mode는 플랜에 포함된 rate limit을 소모하지 않습니다

즉, `/fast`를 사용하려면 extra usage가 활성화되고 자금이 충전되어 있어야 합니다.

---

## CLI Startup Flags

사용량 예산과 관련된 두 가지 시작 플래그가 있습니다(API key 사용자 전용, print 모드):

| Flag | Description |
|------|-------------|
| `--max-budget-usd <AMOUNT>` | 중단하기 전까지 API 호출에 사용할 최대 금액(달러) |
| `--max-turns <NUMBER>` | 에이전트 턴(agentic turn) 수 제한 |

전체 목록은 [CLI Startup Flags Reference](claude-cli-startup-flags.md)를 참고하세요.

---

## Sources

- [Extra usage for paid Claude plans — Claude Help Center](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)
- [Using Claude Code with your Pro or Max plan — Claude Help Center](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-pro-or-max-plan)
- [/extra-usage slash command is undocumented — GitHub Issue #12396](https://github.com/anthropics/claude-code/issues/12396)
- [Claude Code CLI Reference](https://code.claude.com/docs/en/cli-reference)
