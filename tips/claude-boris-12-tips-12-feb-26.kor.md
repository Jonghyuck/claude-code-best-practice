<!--
  이 문서는 tips/claude-boris-12-tips-12-feb-26.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# 12 Ways to Customize Claude Code — Tips from Boris Cherny

Claude Code의 제작자인 Boris Cherny ([@bcherny](https://x.com/bcherny))가 2026년 2월 12일에 공유한 커스터마이징 팁 요약입니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Context

Boris Cherny는 커스터마이징 가능성이 엔지니어들이 Claude Code에서 가장 좋아하는 요소 중 하나라고 강조했습니다 — 훅, 플러그인, LSP, MCP, 스킬, effort, 커스텀 에이전트, 상태 표시줄(status line), 출력 스타일(output style) 등이 그것입니다. 그는 개발자와 팀이 자신의 설정을 커스터마이징하는 12가지 실용적인 방법을 공유했습니다.

<a href="https://x.com/bcherny/status/2021699851499798911"><img src="assets/boris-26-2-12/0.webp" alt="Boris Cherny intro tweet" width="50%" /></a>

---

## 1/ Configure Your Terminal

최상의 Claude Code 경험을 위해 터미널을 설정하세요:

- **Theme**: `/config`를 실행하여 라이트/다크 모드를 설정합니다
- **Notifications**: iTerm2용 알림을 활성화하거나 커스텀 알림 훅을 사용합니다
- **Newlines**: IDE 터미널, Apple Terminal, Warp, 또는 Alacritty에서 Claude Code를 사용하는 경우 `/terminal-setup`을 실행하여 shift+enter로 줄바꿈을 입력할 수 있도록 설정하세요 (그러면 `\`를 입력할 필요가 없습니다)
- **Vim mode**: `/vim`을 실행합니다

<a href="https://x.com/bcherny/status/2021699859359883608"><img src="assets/boris-26-2-12/1.webp" alt="Configure your terminal" width="50%" /></a>

---

## 2/ Adjust Effort Level

`/model`을 실행하여 원하는 effort 레벨을 선택하세요:

- **Low** — 더 적은 토큰, 더 빠른 응답
- **Medium** — 균형 잡힌 동작
- **High** — 더 많은 토큰, 더 높은 지능

Boris의 선호: 모든 작업에 High.

<a href="https://x.com/bcherny/status/2021699860869902424"><img src="assets/boris-26-2-12/2.webp" alt="Adjust effort level" width="50%" /></a>

---

## 3/ Install Plugins, MCPs, and Skills

플러그인을 통해 LSP(모든 주요 언어에 제공됨), MCP, 스킬, 에이전트, 커스텀 훅을 설치할 수 있습니다.

공식 Anthropic 플러그인 마켓플레이스에서 설치하거나, 회사용으로 자체 마켓플레이스를 만드세요. `settings.json`을 코드베이스에 체크인하면 팀을 위해 마켓플레이스가 자동으로 추가됩니다.

`/plugin`을 실행하여 시작하세요.

<a href="https://x.com/bcherny/status/2021699862522364149"><img src="assets/boris-26-2-12/3.webp" alt="Install Plugins, MCPs, and Skills" width="50%" /></a>

---

## 4/ Create Custom Agents

`.claude/agents`에 `.md` 파일을 넣어 커스텀 에이전트를 만드세요. 각 에이전트는 커스텀 이름, 색상, 도구 세트, 사전 허용 및 사전 차단 도구, 권한 모드, 모델을 가질 수 있습니다.

`settings.json`의 `"agent"` 필드나 `--agent` 플래그를 사용하여 메인 대화의 기본 에이전트를 설정할 수도 있습니다.

`/agents`를 실행하여 시작하세요.

<a href="https://x.com/bcherny/status/2021700144039903699"><img src="assets/boris-26-2-12/4.webp" alt="Create custom agents" width="50%" /></a>

---

## 5/ Pre-approve Common Permissions

Claude Code는 프롬프트 인젝션 탐지, 정적 분석, 샌드박싱, 사람의 감독을 결합한 권한 시스템을 사용합니다.

기본적으로 소수의 안전한 명령이 사전 승인되어 있습니다. 더 많이 사전 승인하려면 `/permissions`를 실행하여 allow 및 block 목록에 추가하세요. 이를 팀의 `settings.json`에 체크인하세요.

전체 와일드카드 구문이 지원됩니다 — 예: `Bash(bun run *)` 또는 `Edit(/docs/**)`.

<a href="https://x.com/bcherny/status/2021700332292911228"><img src="assets/boris-26-2-12/5.webp" alt="Pre-approve common permissions" width="50%" /></a>

---

## 6/ Enable Sandboxing

Claude Code의 오픈 소스 샌드박스 런타임을 사용하여 안전성을 높이는 동시에 권한 프롬프트를 줄이세요.

`/sandbox`를 실행하여 활성화합니다. 샌드박싱은 사용자의 머신에서 실행되며 파일 및 네트워크 격리를 모두 지원합니다.

<a href="https://x.com/bcherny/status/2021700506465579443"><img src="assets/boris-26-2-12/6.webp" alt="Enable sandboxing" width="50%" /></a>

---

## 7/ Add a Status Line

커스텀 상태 표시줄은 컴포저 바로 아래에 나타나며, 모델, 디렉터리, 남은 컨텍스트, 비용, 그 밖에 작업 중에 보고 싶은 모든 것을 표시합니다.

모든 팀원이 서로 다른 상태 표시줄을 가질 수 있습니다. `/statusline`을 사용하면 Claude가 사용자의 `.bashrc`/`.zshrc`를 기반으로 상태 표시줄을 생성해 줍니다.

<a href="https://x.com/bcherny/status/2021700784019452195"><img src="assets/boris-26-2-12/7.webp" alt="Add a status line" width="50%" /></a>

---

## 8/ Customize Your Keybindings

Claude Code의 모든 키 바인딩은 커스터마이징할 수 있습니다. `/keybindings`를 실행하여 어떤 키든 다시 매핑하세요. 설정이 실시간으로 다시 로드되므로 곧바로 사용감을 확인할 수 있습니다.

<a href="https://x.com/bcherny/status/2021700883873165435"><img src="assets/boris-26-2-12/8.webp" alt="Customize your keybindings" width="50%" /></a>

---

## 9/ Set Up Hooks

훅을 사용하면 Claude의 라이프사이클에 결정론적으로 개입할 수 있습니다:

- 권한 요청을 자동으로 Slack이나 Opus로 라우팅
- Claude가 턴의 끝에 도달했을 때 계속 진행하도록 유도 (에이전트를 실행하거나 프롬프트를 사용하여 Claude가 계속 진행해야 할지 결정하게 할 수도 있습니다)
- 도구 호출을 전처리 또는 후처리 — 예: 자체 로깅 추가

Claude에게 훅을 추가해 달라고 요청하여 시작하세요.

<a href="https://x.com/bcherny/status/2021701059253874861"><img src="assets/boris-26-2-12/9.webp" alt="Set up hooks" width="50%" /></a>

---

## 10/ Customize Your Spinner Verbs

스피너 동사(spinner verb)를 커스터마이징하여 기본 목록에 자신만의 동사를 추가하거나 교체하세요. `settings.json`을 소스 관리에 체크인하면 팀과 동사를 공유할 수 있습니다.

<a href="https://x.com/bcherny/status/2021701145023197516"><img src="assets/boris-26-2-12/10.webp" alt="Customize your spinner verbs" width="50%" /></a>

---

## 11/ Use Output Styles

`/config`를 실행하고 출력 스타일을 설정하면 Claude가 다른 톤이나 형식으로 응답하도록 할 수 있습니다.

- **Explanatory** — 새로운 코드베이스에 익숙해질 때 권장되며, Claude가 작업하면서 프레임워크와 코드 패턴을 설명하도록 합니다
- **Learning** — Claude가 코드 변경을 하나씩 코칭해 주도록 합니다
- **Custom** — 커스텀 출력 스타일을 만들어 Claude의 목소리를 조정합니다

<a href="https://x.com/bcherny/status/2021701379409273093"><img src="assets/boris-26-2-12/11.webp" alt="Use output styles" width="50%" /></a>

---

## 12/ Customize All the Things!

Claude Code는 기본 상태로도 훌륭하게 작동하지만, 커스터마이징을 할 때는 `settings.json`을 git에 체크인하여 팀도 혜택을 누릴 수 있게 하세요. 구성은 여러 수준에서 지원됩니다:

- 코드베이스 단위
- 하위 폴더 단위
- 자기 자신만을 위한 단위
- 전사(全社) 정책을 통한 단위

37개의 설정과 84개의 환경 변수(래퍼 스크립트를 피하려면 `settings.json`의 `"env"` 필드를 사용하세요)가 있으므로, 원하는 거의 모든 동작을 구성할 수 있을 가능성이 높습니다.

<a href="https://x.com/bcherny/status/2021701636075458648"><img src="assets/boris-26-2-12/12.webp" alt="Customize all the things" width="50%" /></a>

---

## Sources

- [Boris Cherny (@bcherny) on X — February 12, 2026](https://x.com/bcherny)
- [Claude Code Terminal Setup Docs](https://code.claude.com/docs/en/terminal)
- [Claude Code Plugins & Discovery Docs](https://code.claude.com/docs/en/discover-plugins)
- [Claude Code Sub-agents Docs](https://code.claude.com/docs/en/sub-agents)
- [Claude Code Permissions Docs](https://code.claude.com/docs/en/permissions)
- [Claude Code Sandbox Docs](https://code.claude.com/docs/en/sandbox)
- [Claude Code Status Line Docs](https://code.claude.com/docs/en/statusline)
- [Claude Code Keyboard Shortcuts Docs](https://code.claude.com/docs/en/keybindings)
- [Claude Code Hooks Reference](https://code.claude.com/docs/en/hooks)
- [Claude Code Output Styles Docs](https://code.claude.com/docs/en/output-styles)
- [Claude Code Settings Docs](https://code.claude.com/docs/en/settings)
