<!--
  이 문서는 reports/claude-in-chrome-v-chrome-devtools-mcp.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Comprehensive Browser Automation MCP Comparison Report

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

## Executive Summary

광범위한 조사를 바탕으로, 스크린샷에 있던 두 도구와 세 번째 주요 후보를 함께 분석했습니다. 여러분의 작업을 자동으로 테스트할 때 가장 적합한 선택지를 고르는 데 도움이 되도록, 아래에 종합적인 분석을 정리했습니다.

---

## 1. The Three Contenders

### **A. Chrome DevTools MCP** (Your Screenshot #1)
- **Source:** 공식 Google Chrome 팀
- **Released:** 2025년 9월 공개 프리뷰
- **Architecture:** Chrome DevTools Protocol (CDP) + Puppeteer 기반
- **Token Usage:** 약 19.0k 토큰(컨텍스트의 9.5%)
- **Tools:** 6개 범주에 걸친 26개의 전용 도구

### **B. Claude in Chrome** (Your Screenshot #2)
- **Source:** 공식 Anthropic 확장 프로그램
- **Released:** 베타, 모든 유료 요금제(Pro, Max, Team, Enterprise)에 순차 배포 중
- **Architecture:** computer-use 기능을 갖춘 브라우저 확장 프로그램
- **Token Usage:** 약 15.4k 토큰(컨텍스트의 7.7%)
- **Tools:** computer use 기능을 포함한 16개 도구

### **C. Playwright MCP** (Strong Alternative)
- **Source:** Microsoft(공식 + 커뮤니티 구현체)
- **Architecture:** 접근성 트리(accessibility tree) 기반 자동화
- **Token Usage:** 약 13.7k 토큰(컨텍스트의 6.8%)
- **Tools:** 21개 도구

---

## 2. Detailed Feature Comparison

| Feature | Chrome DevTools MCP | Claude in Chrome | Playwright MCP |
|---------|---------------------|------------------|----------------|
| **Primary Purpose** | 디버깅 및 성능 분석 | 범용 브라우저 자동화 | UI 테스트 및 E2E |
| **Browser Support** | Chrome 전용 | Chrome 전용 | Chromium, Firefox, WebKit |
| **Token Efficiency** | 19.0k (9.5%) | 15.4k (7.7%) | 13.7k (6.8%) |
| **Element Selection** | CSS/XPath 셀렉터 | 시각 + DOM | 접근성 트리(의미 기반) |
| **Performance Traces** | ✅ 우수 | ❌ 없음 | ⚠️ 제한적 |
| **Network Inspection** | ✅ 심층 분석 | ⚠️ 기본 수준 | ⚠️ 기본 수준 |
| **Console Logs** | ✅ 전체 접근 | ✅ 전체 접근 | ⚠️ 제한적 |
| **Cross-browser** | ❌ 없음 | ❌ 없음 | ✅ 지원 |
| **CI/CD Integration** | ✅ 우수 | ❌ 미흡(로그인 필요) | ✅ 우수 |
| **Headless Mode** | ✅ 지원 | ❌ 미지원 | ✅ 지원 |
| **Authentication** | 설정 필요 | 사용자 세션 사용 | 설정 필요 |
| **Scheduled Tasks** | ❌ 없음 | ✅ 지원 | ❌ 없음 |
| **Cost** | 무료 | 유료 요금제 필요 | 무료 |
| **Local Setup** | Node.js 필요 | 브라우저 확장 프로그램 | Node.js 필요 |

---

## 3. Tool Breakdown

### Chrome DevTools MCP (26 Tools)

```
INPUT AUTOMATION (8):     click, drag, fill, fill_form, handle_dialog,
                          hover, press_key, upload_file

NAVIGATION (6):           close_page, list_pages, navigate_page,
                          new_page, select_page, wait_for

EMULATION (2):            emulate, resize_page

PERFORMANCE (3):          performance_analyze_insight,
                          performance_start_trace, performance_stop_trace

NETWORK (2):              get_network_request, list_network_requests

DEBUGGING (5):            evaluate_script, get_console_message,
                          list_console_messages, take_screenshot,
                          take_snapshot
```

### Claude in Chrome (16 Tools)

```
BROWSER CONTROL:          navigate, read_page, find, computer
                          (click, type, scroll)

FORM INTERACTION:         form_input, javascript_tool

MEDIA:                    upload_image, get_page_text, gif_creator

TAB MANAGEMENT:           tabs_context_mcp, tabs_create_mcp

DEVELOPMENT:              read_console_messages, read_network_requests

UTILITIES:                shortcuts_list, shortcuts_execute,
                          resize_window, update_plan
```

### Playwright MCP (21 Tools)

```
NAVIGATION:               navigate, goBack, goForward, reload

INTERACTION:              click, fill, select, hover, press,
                          drag, uploadFile

ELEMENT QUERIES:          getElement, getElements, waitForSelector

ASSERTIONS:               assertVisible, assertText, assertTitle

PAGE STATE:               screenshot, getAccessibilityTree,
                          evaluateScript

BROWSER MGMT:             newPage, closePage
```

---

## 4. Use Case Analysis for Automated Testing

### **Chrome DevTools MCP is BEST for:**

✅ **성능 테스트**
- Core Web Vitals와 함께 성능 트레이스 기록
- 렌더링 병목 및 레이아웃 시프트 식별
- 메모리 누수 탐지 및 CPU 프로파일링

✅ **심층 디버깅**
- 네트워크 요청 검사(헤더, 페이로드, 타이밍)
- 콘솔 오류 분석 및 스택 트레이스
- 실시간 DOM 검사

✅ **CI/CD 파이프라인**
- 헤드리스 실행 지원
- 안정적인 스크립트 기반 자동화
- 인증 상태에 대한 의존성 없음

**이상적인 워크플로:** "이 페이지가 왜 느린지 찾아줘" 또는 "이 API 호출을 디버깅해줘"

---

### **Claude in Chrome is BEST for:**

✅ **수동 테스트 지원**
- 계정에 로그인한 상태에서 테스트
- 시각적 맥락을 활용한 탐색적 테스트
- 재현할 수 있는 워크플로 기록

✅ **빠른 검증**
- 디자인 검증(Figma와 결과물 비교)
- 신규 기능 스팟 체크
- 개발 중 콘솔 오류 확인

✅ **반복적인 브라우저 작업**
- 예약된 자동 점검
- 다중 탭 워크플로 관리
- 기록된 동작으로부터 학습

**이상적인 워크플로:** "내 변경 사항이 제대로 보이는지 확인해줘" 또는 "내 로그인 상태로 이 폼을 테스트해줘"

---

### **Playwright MCP is BEST for:**

✅ **E2E 테스트 자동화**
- 크로스 브라우저 테스트(Chrome, Firefox, Safari)
- 재사용 가능한 테스트 스크립트 생성
- Page Object Model 생성

✅ **안정적인 UI 테스트**
- 접근성 트리 = 불안정한 셀렉터 없음
- 결정론적 상호작용
- UI 변경으로 인한 파손에 덜 취약

✅ **CI/CD 통합**
- 파이프라인용 헤드리스 모드
- 자연어로부터 Playwright 테스트 파일 생성
- 테스트 관리 도구와의 통합

**이상적인 워크플로:** "이 사용자 플로에 대한 E2E 테스트를 작성해줘" 또는 "여러 브라우저에서 이걸 테스트해줘"

---

## 5. Token Efficiency Analysis

| Tool | Token Usage | % of Context | Efficiency Rating |
|------|-------------|--------------|-------------------|
| Playwright MCP | ~13.7k | 6.8% | ⭐⭐⭐⭐⭐ 최고 |
| Claude in Chrome | ~15.4k | 7.7% | ⭐⭐⭐⭐ 양호 |
| Chrome DevTools MCP | ~19.0k | 9.5% | ⭐⭐⭐ 무난 |

**영향:** 200k 토큰 컨텍스트 기준:
- Playwright는 작업에 186.3k 토큰을 남깁니다
- Claude in Chrome는 184.6k 토큰을 남깁니다
- Chrome DevTools는 181k 토큰을 남깁니다

Playwright와 Chrome DevTools 간의 약 5.3k 토큰 차이는 코드 컨텍스트가 많은 복잡한 세션에서 의미가 있을 수 있습니다.

---

## 6. Security Considerations

### Chrome DevTools MCP
- ✅ 기본적으로 격리된 브라우저 프로필
- ✅ 클라우드 의존성 없음
- ✅ 완전한 로컬 제어
- ⚠️ 원격 디버깅 포트 보안(격리된 프로필 사용 권장)

### Claude in Chrome
- ⚠️ **완화 조치 없이 23.6%의 공격 성공률**(방어 조치 적용 시 11.2%로 감소)
- ⚠️ 실제 브라우저 세션 사용(쿠키 노출 위험)
- ⚠️ 금융/성인/불법 복제 사이트 차단
- ⚠️ 알려진 취약점이 있는 베타 단계

### Playwright MCP
- ✅ 격리된 브라우저 컨텍스트
- ✅ 클라우드 의존성 없음
- ✅ 성숙한 보안 모델(Microsoft 지원)
- ✅ 인증을 안전하게 처리 가능

---

## 7. Installation Commands

### Chrome DevTools MCP

```bash
claude mcp add chrome-devtools npx chrome-devtools-mcp@latest
```

### Claude in Chrome

```
Install from Chrome Web Store (requires Pro/Max/Team/Enterprise plan)
```

### Playwright MCP (Recommended)

```bash
# First, install browsers
npx playwright install

# Then add to Claude Code (user scope = all projects)
claude mcp add playwright -s user -- npx @playwright/mcp@latest
```

---

## 8. Recommendations

### **For Your Automated Testing Workflow:**

#### 🥇 **Primary Tool: Playwright MCP**

**Use for:** 일상적인 E2E 테스트, 크로스 브라우저 검증, 테스트 스크립트 생성

**Why:**
- 가장 낮은 토큰 사용량(코드에 더 많은 컨텍스트 확보)
- 크로스 브라우저 지원(Chrome, Firefox, Safari)
- 접근성 트리 방식 = 더 안정적인 셀렉터
- 우수한 CI/CD 통합
- 실제 Playwright 테스트 파일 생성 가능
- 무료, 구독 불필요

#### 🥈 **Secondary Tool: Chrome DevTools MCP**

**Use for:** 성능 디버깅, 네트워크 분석, Core Web Vitals

**Why:**
- 성능 트레이스와 디버깅에서 타의 추종을 불허함
- 심층 네트워크 요청 검사
- 장기 지원이 보장되는 공식 Google 도구
- "왜 이게 느린가?"에 답해야 할 때 필수적

#### 🥉 **Situational: Claude in Chrome**

**Use for:** 로그인 상태에서의 빠른 수동 검증, 탐색적 테스트, 디자인 검증

**Why:**
- 개발 중 빠른 시각적 확인에 적합
- 로그인된 상태를 읽을 수 있음
- "이게 제대로 보이나?" 검증에 유용
- CI/CD나 본격적인 테스트 자동화에는 부적합

---

## 9. Recommended Setup

```bash
# Install both Playwright and Chrome DevTools MCP
npx playwright install
claude mcp add playwright -s user -- npx @playwright/mcp@latest
claude mcp add chrome-devtools -s user -- npx chrome-devtools-mcp@latest
```

### Suggested Workflow

```
1. DEVELOP      → Claude Code (terminal)
2. TEST         → Playwright MCP (E2E, cross-browser)
3. DEBUG        → Chrome DevTools MCP (performance, network)
4. VERIFY       → Claude in Chrome (quick visual checks)
5. CI/CD        → Playwright MCP (headless, automated)
```

---

## 10. Final Verdict

| If You Need... | Use This |
|----------------|----------|
| 크로스 브라우저 E2E 테스트 | **Playwright MCP** |
| 성능 분석 | **Chrome DevTools MCP** |
| 네트워크 디버깅 | **Chrome DevTools MCP** |
| 빠른 시각적 검증 | **Claude in Chrome** |
| CI/CD 자동화 | **Playwright MCP** |
| 테스트 스크립트 생성 | **Playwright MCP** |
| 가장 낮은 토큰 사용량 | **Playwright MCP** |
| 로그인 세션 테스트 | **Claude in Chrome** |
| 콘솔 로그 디버깅 | **Chrome DevTools MCP** |

### **TL;DR Recommendation:**

**Playwright MCP와 Chrome DevTools MCP를 모두 설치하세요.** Playwright를 주력 테스트 도구로 사용하세요(토큰 효율이 더 좋고, 크로스 브라우저를 지원하며, E2E에 더 적합합니다). 심층 성능 분석이나 네트워크 디버깅이 필요할 때는 Chrome DevTools를 사용하세요. Claude in Chrome은 로그인 세션이 필요한 빠른 수동 검증에만 사용하세요.

---

## Sources

- [Chrome DevTools MCP - GitHub](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- [Anthropic - Piloting Claude in Chrome](https://claude.com/blog/claude-for-chrome)
- [Claude in Chrome Help Center](https://support.claude.com/en/articles/12012173-getting-started-with-claude-in-chrome)
- [Playwright MCP - GitHub](https://github.com/microsoft/playwright-mcp)
- [Simon Willison - Using Playwright MCP with Claude Code](https://til.simonwillison.net/claude-code/playwright-mcp-claude-code)
- [Testomat.io - Playwright MCP Claude Code](https://testomat.io/blog/playwright-mcp-claude-code/)
- [MCP Integration Guide - Scrapeless](https://www.scrapeless.com/en/blog/mcp-integration-guide)
- [Chrome DevTools MCP Guide - Vladimir Siedykh](https://vladimirsiedykh.com/blog/chrome-devtools-mcp-ai-browser-debugging-complete-guide-2025)
- [Addy Osmani - Give your AI eyes](https://addyosmani.com/blog/devtools-mcp/)

---

*This report was generated by Claude Code using the Opus 4.5 model on December 19, 2025.*
