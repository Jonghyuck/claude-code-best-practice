<!--
  이 문서는 tips/claude-thariq-tips-17-mar-26.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Lessons from Building Claude Code: How We Use Skills — Thariq

Anthropic이 내부적으로 스킬을 어떻게 사용하는지에 대한 종합 가이드로, Thariq ([@trq212](https://x.com/trq212))가 2026년 3월 17일에 공유한 내용입니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Context

스킬은 Claude Code에서 가장 많이 사용되는 확장 지점 중 하나가 되었습니다. 유연하고, 만들기 쉽고, 배포하기도 간단합니다. 하지만 바로 이 유연함 때문에 무엇이 가장 잘 작동하는지 파악하기 어렵기도 합니다. Thariq는 Anthropic에서 수백 개의 스킬을 활발히 사용하며 얻은 교훈을 공유합니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/1.png" alt="Thariq intro tweet" width="50%" /></a>

---

## What are Skills?

흔한 오해 중 하나는 스킬이 "그냥 마크다운 파일"이라는 것이지만, 가장 흥미로운 점은 스킬이 스크립트, 에셋, 데이터 등을 포함할 수 있는 **폴더**라는 사실입니다. 에이전트가 이를 발견하고, 탐색하고, 조작할 수 있습니다. 스킬은 또한 동적 훅 등록을 포함해 매우 다양한 구성 옵션을 갖고 있습니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/2.png" alt="What are Skills?" width="50%" /></a>

---

## Types of Skills

팀은 모든 스킬을 목록화한 뒤, 이들이 반복적으로 나타나는 9개의 범주로 묶인다는 것을 발견했습니다. 가장 좋은 스킬은 하나의 범주에 깔끔하게 들어맞고, 더 혼란스러운 스킬은 여러 범주에 걸쳐 있습니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/3.png" alt="Types of Skills grid" width="50%" /></a>

---

### 1/ Library & API Reference

라이브러리, CLI, SDK를 올바르게 사용하는 방법을 설명하는 스킬입니다. 내부 라이브러리를 위한 것일 수도 있고, Claude Code가 종종 어려움을 겪는 일반적인 라이브러리를 위한 것일 수도 있습니다. 이러한 스킬은 흔히 참조 코드 스니펫 폴더와, 스크립트를 작성할 때 피해야 할 주의사항 목록을 포함합니다.

**Examples:** billing-lib, internal-platform-cli, frontend-design

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/4.png" alt="Library & API Reference" width="50%" /></a>

---

### 2/ Product Verification

코드가 정상적으로 작동하는지 테스트하거나 검증하는 방법을 설명하는 스킬입니다. 이런 스킬은 Playwright, tmux 같은 외부 도구와 함께 짝을 이루는 경우가 많습니다. 검증 스킬은 Claude의 출력이 올바른지 보장하는 데 매우 유용합니다. 엔지니어가 일주일 정도를 오로지 검증 스킬을 훌륭하게 만드는 데 쏟는 것도 그만한 가치가 있습니다.

**Examples:** signup-flow-driver, checkout-verifier, tmux-cli-driver

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/5.png" alt="Product Verification" width="50%" /></a>

---

### 3/ Data Fetching & Analysis

데이터 및 모니터링 스택에 연결하는 스킬입니다. 자격 증명으로 데이터를 가져오는 라이브러리, 특정 대시보드 ID 등과 함께, 일반적인 워크플로나 데이터를 얻는 방법에 대한 지침을 포함할 수 있습니다.

**Examples:** funnel-query, cohort-compare, grafana

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/6.png" alt="Data Fetching & Analysis" width="50%" /></a>

---

### 4/ Business Process & Team Automation

반복적인 워크플로를 하나의 명령으로 자동화하는 스킬입니다. 보통 상당히 단순한 지침이지만, 다른 스킬이나 MCP에 대해 더 복잡한 의존성을 가질 수도 있습니다. 이전 결과를 로그 파일에 저장해 두면 모델이 일관성을 유지하고 이전 워크플로 실행을 되돌아보는 데 도움이 됩니다.

**Examples:** standup-post, create-\<ticket-system\>-ticket, weekly-recap

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/7.png" alt="Business Process & Team Automation" width="50%" /></a>

---

### 5/ Code Scaffolding & Templates

코드베이스의 특정 기능을 위한 프레임워크 보일러플레이트를 생성하는 스킬입니다. 이런 스킬을 조합 가능한 스크립트와 결합할 수도 있습니다. 스캐폴딩에 코드만으로는 온전히 다룰 수 없는 자연어 요구사항이 있을 때 특히 유용합니다.

**Examples:** new-\<framework\>-workflow, new-migration, create-app

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/8.png" alt="Code Scaffolding & Templates" width="50%" /></a>

---

### 6/ Code Quality & Review

조직 내부의 코드 품질을 강제하고 코드 리뷰를 돕는 스킬입니다. 최대한의 견고함을 위해 결정론적 스크립트나 도구를 포함할 수 있습니다. 이런 스킬은 훅의 일부로, 또는 GitHub Action 내부에서 자동으로 실행하고 싶을 수 있습니다.

**Examples:** adversarial-review, code-style, testing-practices

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/9.png" alt="Code Quality & Review" width="50%" /></a>

---

### 7/ CI/CD & Deployment

코드베이스 내에서 코드를 가져오고, 푸시하고, 배포하는 것을 돕는 스킬입니다. 이런 스킬은 데이터를 수집하기 위해 다른 스킬을 참조할 수도 있습니다.

**Examples:** babysit-pr, deploy-\<service\>, cherry-pick-prod

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/10.png" alt="CI/CD & Deployment" width="50%" /></a>

---

### 8/ Runbooks

증상(예: Slack 스레드, 알림, 오류 시그니처)을 입력받아, 여러 도구를 활용한 조사를 단계적으로 수행하고, 구조화된 리포트를 생성하는 스킬입니다.

**Examples:** \<service\>-debugging, oncall-runner, log-correlator

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/11.png" alt="Runbooks" width="50%" /></a>

---

### 9/ Infrastructure Operations

일상적인 유지보수 및 운영 절차를 수행하는 스킬입니다. 그중 일부는 가드레일이 있으면 유익한 파괴적 작업을 포함합니다. 이런 스킬은 엔지니어가 중요한 작업에서 모범 사례를 따르기 더 쉽게 만들어 줍니다.

**Examples:** \<resource\>-orphans, dependency-management, cost-investigation

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/12.png" alt="Infrastructure Operations" width="50%" /></a>

---

## Tips for Making Skills

효과적인 스킬을 작성하기 위한 9가지 모범 사례와, 배포 및 측정에 대한 지침입니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/13.png" alt="Tips for Making Skills grid" width="50%" /></a>

---

### Tip 1: Don't State the Obvious

Claude Code는 여러분의 코드베이스에 대해 많이 알고 있고, Claude는 여러 기본적인 견해를 포함해 코딩에 대해서도 많이 알고 있습니다. 주로 지식을 다루는 스킬을 게시한다면, Claude를 평소의 사고방식에서 벗어나게 밀어붙이는 정보에 집중하도록 하세요. frontend design 스킬이 좋은 예입니다. 이 스킬은 Inter 폰트와 보라색 그라디언트 같은 전형적인 패턴을 피하면서 Claude의 디자인 감각을 개선하기 위해 고객들과 반복 작업하며 만들어졌습니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/14.png" alt="Don't State the Obvious" width="50%" /></a>

---

### Tip 2: Build a Gotchas Section

어떤 스킬에서든 가장 신호가 강한 콘텐츠는 Gotchas 섹션입니다. 이 섹션은 Claude가 여러분의 스킬을 사용할 때 부딪히는 흔한 실패 지점들로 채워 나가야 합니다. 이상적으로는, 시간이 지나며 이러한 주의사항을 담아내도록 스킬을 계속 업데이트하게 될 것입니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/15.png" alt="Build a Gotchas Section" width="50%" /></a>

---

### Tip 3: Use the File System & Progressive Disclosure

스킬은 단순한 마크다운 파일이 아니라 폴더입니다. 파일 시스템 전체를 컨텍스트 엔지니어링과 점진적 공개(progressive disclosure)의 한 형태로 생각해야 합니다. 스킬 안에 어떤 파일이 있는지 Claude에게 알려주면, Claude가 적절한 시점에 그 파일들을 읽습니다. 가장 단순한 형태는 다른 마크다운 파일을 가리키는 것입니다. 예를 들어 상세한 함수 시그니처와 사용 예시를 `references/api.md`로 분리하는 식입니다. 참조 자료, 스크립트, 예제 등의 폴더를 둘 수 있습니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/16.png" alt="Progressive Disclosure" width="50%" /></a>

---

### Tip 4: Avoid Railroading Claude

Claude는 일반적으로 여러분의 지침을 지키려고 하며, 스킬은 재사용성이 매우 높기 때문에 너무 구체적으로 만들지 않도록 주의해야 합니다. Claude에게 필요한 정보는 주되, 상황에 적응할 수 있는 유연성도 함께 부여하세요. 규범적인 단계별 지침 대신, 목표와 제약을 제시하세요.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/17.png" alt="Avoid Railroading Claude" width="50%" /></a>

---

### Tip 5: Think through the Setup

어떤 스킬은 사용자로부터 받은 컨텍스트로 설정되어야 할 수 있습니다. 좋은 패턴은 이러한 설정 정보를 스킬 디렉터리의 `config.json` 파일에 저장하는 것입니다. 설정이 되어 있지 않으면, 에이전트가 사용자에게 정보를 요청할 수 있습니다. 구조화된 객관식 질문을 위해 Claude가 AskUserQuestion 도구를 사용하도록 지시할 수 있습니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/18.png" alt="Think through the Setup" width="50%" /></a>

---

### Tip 6: The Description Field Is For the Model

Claude Code가 세션을 시작할 때, 사용 가능한 모든 스킬과 그 description으로 목록을 구성합니다. Claude는 "이 요청에 맞는 스킬이 있나?"를 판단하기 위해 이 목록을 훑어봅니다. 즉, description 필드는 요약이 아니라 이 스킬을 **언제 트리거해야 하는지**에 대한 설명입니다. 모델을 위해 작성하세요.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/19.png" alt="Description = Trigger" width="50%" /></a>

---

### Tip 7: Memory & Storing Data

어떤 스킬은 내부에 데이터를 저장함으로써 일종의 메모리를 포함할 수 있습니다. 단순히 추가 전용(append-only) 텍스트 로그 파일이나 JSON 파일처럼 간단한 것부터, SQLite 데이터베이스처럼 복잡한 것까지 무엇에든 데이터를 저장할 수 있습니다. 스킬 디렉터리에 저장된 데이터는 스킬을 업그레이드할 때 삭제될 수 있으므로, 데이터를 저장할 안정적인 플러그인별 폴더로 `${CLAUDE_PLUGIN_DATA}`를 사용하세요.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/20.png" alt="Memory & Storing Data" width="50%" /></a>

---

### Tip 8: Store Scripts & Generate Code

Claude에게 줄 수 있는 가장 강력한 도구 중 하나는 코드입니다. Claude에게 스크립트와 라이브러리를 제공하면, Claude는 보일러플레이트를 재구성하는 대신 조합에, 즉 다음에 무엇을 할지 결정하는 데 자신의 턴을 쓸 수 있습니다. 그러면 Claude는 더 고급 분석을 위해 이 기능들을 조합하는 스크립트를 즉석에서 생성할 수 있습니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/21.png" alt="Store Scripts & Generate Code" width="50%" /></a>

---

### Tip 9: On Demand Hooks

스킬은 해당 스킬이 호출될 때만 활성화되어 세션이 지속되는 동안 유지되는 훅을 포함할 수 있습니다. 항상 실행하고 싶지는 않지만 때때로 매우 유용한, 더 견해가 강한 훅에 이를 활용하세요.

**Examples:**
- `/careful` — PreToolUse matcher를 통해 Bash에서 rm -rf, DROP TABLE, force-push, kubectl delete를 차단
- `/freeze` — 특정 디렉터리에 속하지 않는 모든 Edit/Write를 차단

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/22.png" alt="On Demand Hooks" width="50%" /></a>

---

## Distributing Skills

팀과 스킬을 공유하는 두 가지 방법:
- **레포에 체크인** (`.claude/skills` 아래) — 비교적 적은 수의 레포에서 작업하는 소규모 팀에 가장 적합
- **플러그인 제작** 후, 사용자가 플러그인을 업로드하고 설치할 수 있는 Claude Code 플러그인 마켓플레이스 운영

체크인된 스킬은 각각 모델의 컨텍스트에 조금씩 더해집니다. 규모가 커질수록, 내부 플러그인 마켓플레이스를 두면 스킬을 배포하고 팀이 어떤 것을 설치할지 스스로 결정하게 할 수 있습니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/23.png" alt="Distributing Skills" width="50%" /></a>

---

## Managing a Marketplace

어떤 스킬이 마켓플레이스에 들어갈지 결정하는 중앙집중식 팀은 없습니다. 대신 가장 유용한 스킬을 유기적으로 찾아내려고 합니다. GitHub의 샌드박스 폴더에 업로드하고 Slack이나 다른 포럼에서 사람들에게 알려 주세요. 스킬이 어느 정도 호응을 얻으면(그 판단은 스킬 소유자에게 달려 있습니다), 소유자가 PR을 올려 그것을 마켓플레이스로 옮길 수 있습니다. 중복된 스킬을 피하려면 출시 전 큐레이션이 중요합니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/24.png" alt="Managing a Marketplace" width="50%" /></a>

---

## Composing Skills

서로 의존하는 스킬을 두고 싶을 수 있습니다. 예를 들어 파일을 업로드하는 파일 업로드 스킬과, CSV를 만들어 업로드하는 CSV 생성 스킬이 있습니다. 이런 식의 의존성 관리는 아직 마켓플레이스나 스킬에 기본으로 내장되어 있지 않지만, 다른 스킬을 이름으로 참조하기만 하면 됩니다. 해당 스킬이 설치되어 있다면 모델이 그것을 호출할 것입니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/25.png" alt="Composing Skills" width="50%" /></a>

---

## Measuring Skills

스킬이 어떻게 쓰이고 있는지 파악하려면, 회사 내 스킬 사용을 로깅할 수 있는 PreToolUse 훅을 사용하세요. 이렇게 하면 인기 있는 스킬이나, 기대에 비해 덜 트리거되는 스킬을 찾아낼 수 있습니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/26.png" alt="Measuring Skills" width="50%" /></a>

---

## Conclusion

스킬은 에이전트를 위한 대단히 강력하고 유연한 도구이지만, 아직 초기 단계이며 우리 모두 어떻게 하면 가장 잘 쓸 수 있을지 알아가는 중입니다. 이 글은 확정적인 가이드라기보다는, 효과가 있다고 확인된 유용한 팁 모음집 정도로 생각하세요. 스킬을 이해하는 가장 좋은 방법은 일단 시작하고, 실험하고, 무엇이 여러분에게 맞는지 지켜보는 것입니다. 우리 스킬 대부분도 몇 줄과 하나의 주의사항에서 시작했고, 사람들이 Claude가 새로운 엣지 케이스에 부딪힐 때마다 내용을 계속 추가하면서 더 나아졌습니다.

<a href="https://x.com/trq212/status/2033949937936085378"><img src="assets/thariq-26-3-17/27.png" alt="Conclusion" width="50%" /></a>

---

## Sources

- [Thariq (@trq212) on X — March 17, 2026](https://x.com/trq212/status/2033949937936085378)
- [Skilljar — Agent Skills course](https://code.claude.com/docs/en/skills)
- [Skill Creator](https://code.claude.com/docs/en/skills)
