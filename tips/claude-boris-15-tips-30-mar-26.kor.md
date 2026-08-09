<!--
  이 문서는 tips/claude-boris-15-tips-30-mar-26.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# 15 Hidden & Under-Utilized Features in Claude Code — From Boris Cherny

Claude Code를 만든 Boris Cherny ([@bcherny](https://x.com/bcherny))가 2026년 3월 30일에 공유한 팁 모음입니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Context

Boris가 Claude Code에서 가장 많이 사용하는, 잘 알려지지 않고 활용도가 낮은 자신의 애용 기능들을 공유했습니다.

<a href="https://x.com/bcherny/status/2038454336355999749"><img src="assets/boris-26-3-30/0.png" alt="Boris Cherny intro tweet" width="50%" /></a>

---

## 1/ Claude Code Has a Mobile App

Claude Code에 모바일 앱이 있다는 걸 알고 계셨나요? Boris는 iOS 앱에서 많은 코드를 작성합니다 — 노트북을 열지 않고도 편리하게 변경 작업을 할 수 있는 방법입니다.

- iOS/Android용 Claude 앱을 다운로드하세요
- 왼쪽의 **Code** 탭으로 이동하세요
- 변경 사항 검토, PR 승인, 코드 작성을 모두 휴대폰에서 바로 할 수 있습니다

<a href="https://x.com/bcherny/status/2038454337811386436"><img src="assets/boris-26-3-30/1.png" alt="Claude Code mobile app" width="50%" /></a>

---

## 2/ Move Sessions Between Mobile/Web/Desktop and Terminal

`claude --teleport` 또는 `/teleport`를 실행하면 클라우드 세션을 자신의 머신에서 이어서 진행할 수 있습니다. 또는 `/remote-control`을 실행하면 로컬에서 실행 중인 세션을 휴대폰/웹에서 제어할 수 있습니다.

- **Teleport**: 클라우드 세션을 로컬 터미널로 가져옵니다
- **Remote Control**: 로컬 세션을 어떤 기기에서든 제어할 수 있게 합니다
- Boris는 `/config`에서 **"Enable Remote Control for all sessions"** 를 설정해 두었습니다

<a href="https://x.com/bcherny/status/2038454339933548804"><img src="assets/boris-26-3-30/2.png" alt="Teleport and Remote Control" width="50%" /></a>

---

## 3/ /loop and /schedule — Two of the Most Powerful Features

이 기능들을 사용하면 Claude가 정해진 간격으로 최대 일주일 동안 자동으로 실행되도록 예약할 수 있습니다. Boris는 로컬에서 여러 개의 loop를 돌리고 있습니다:

- `/loop 5m /babysit` — 코드 리뷰 자동 처리, 자동 리베이스, PR을 프로덕션까지 인도
- `/loop 30m /slack-feedback` — 30분마다 Slack 피드백용 PR을 자동으로 올림
- `/loop /post-merge-sweeper` — 놓친 코드 리뷰 코멘트를 처리하는 PR을 올림
- `/loop 1h /pr-pruner` — 오래되고 더 이상 필요 없는 PR을 정리
- ...그 외에도 많습니다!

워크플로를 skill + loop로 만드는 것을 실험해 보세요. 강력합니다.

<a href="https://x.com/bcherny/status/2038454341884154269"><img src="assets/boris-26-3-30/3.png" alt="/loop and /schedule" width="50%" /></a>

---

## 4/ Use Hooks to Deterministically Run Logic

에이전트 라이프사이클의 일부로 로직을 실행하려면 hook을 사용하세요. 예를 들면:

- Claude를 시작할 때마다 컨텍스트를 **동적으로 로드**하기 (`SessionStart`)
- 모델이 실행하는 **모든 bash 명령을 로깅**하기 (`PreToolUse`)
- 권한 프롬프트를 WhatsApp으로 **라우팅**하여 승인/거부하기 (`PermissionRequest`)
- Claude가 멈출 때마다 계속 진행하도록 **재촉하기** (`Stop`)

<a href="https://x.com/bcherny/status/2038454343519932844"><img src="assets/boris-26-3-30/4.png" alt="Use hooks" width="50%" /></a>

---

## 5/ Cowork Dispatch

Boris는 Slack과 이메일을 따라잡고, 파일을 관리하고, 컴퓨터 앞에 없을 때 노트북에서 여러 작업을 하기 위해 매일 Dispatch를 사용합니다. 코딩을 하지 않을 때는 dispatch를 하고 있습니다.

- Dispatch는 Claude Desktop 앱을 위한 **보안 원격 제어**입니다
- 사용자의 허가하에 MCP, 브라우저, 컴퓨터를 사용할 수 있습니다
- 어디서든 코딩 외 작업을 Claude에게 위임하는 방법이라고 생각하면 됩니다

<a href="https://x.com/bcherny/status/2038454345419936040"><img src="assets/boris-26-3-30/5.png" alt="Cowork Dispatch" width="50%" /></a>

---

## 6/ Use the Chrome Extension for Frontend Work

Claude Code를 사용하는 가장 중요한 팁: **Claude에게 자신의 결과물을 검증할 방법을 주세요.** 그렇게 하면 Claude는 결과가 훌륭해질 때까지 반복 작업을 합니다.

- 누군가에게 웹사이트를 만들어 달라고 하면서 브라우저는 못 쓰게 하는 것과 같습니다 — 결과물이 좋아 보이지 않을 가능성이 큽니다
- Claude에게 브라우저를 주면 좋아 보일 때까지 코드를 작성하고 반복합니다
- Boris는 웹 코드를 작업할 때마다 Chrome 확장을 사용합니다 — 다른 유사한 MCP보다 더 안정적으로 동작하는 편입니다

<a href="https://x.com/bcherny/status/2038454347156398333"><img src="assets/boris-26-3-30/6.png" alt="Chrome extension for frontend" width="50%" /></a>

---

## 7/ Use the Claude Desktop App to Auto-Start and Test Web Servers

같은 맥락에서, Desktop 앱에는 Claude가 **웹 서버를 자동으로 실행하고 내장 브라우저에서 테스트까지 할 수 있는** 기능이 번들되어 있습니다.

- Chrome 확장을 사용해 CLI나 VSCode에서도 비슷하게 설정할 수 있습니다
- 아니면 통합된 경험을 위해 Desktop 앱을 그냥 사용하세요

<a href="https://x.com/bcherny/status/2038454348804714642"><img src="assets/boris-26-3-30/7.png" alt="Desktop app web server testing" width="50%" /></a>

---

## 8/ Fork Your Session

기존 세션을 어떻게 fork하는지 자주 묻는 질문입니다. 두 가지 방법이 있습니다:

1. 세션에서 `/branch`를 실행합니다
2. CLI에서 `claude --resume <session-id> --fork-session`을 실행합니다

`/branch`는 분기된 대화를 만들며 — 이제 당신은 그 분기 안에 있습니다. 원래 세션을 다시 이어가려면 `claude -r <original-session-id>`를 사용하세요.

<a href="https://x.com/bcherny/status/2038454350214041740"><img src="assets/boris-26-3-30/8.png" alt="Fork your session" width="50%" /></a>

---

## 9/ Use /btw for Side Queries

Boris는 에이전트가 작업하는 동안 간단한 질문에 답하기 위해 이 기능을 항상 사용합니다. `/btw`를 쓰면 에이전트의 현재 작업을 중단하지 않고 곁다리 질문을 할 수 있습니다.

예시:
```
/btw how do I spell dachshund?
> dachshund — German for "badger dog" (dachs + badger, hund + dog).
↑/↓ to scroll · Space, Enter, or Escape to dismiss
```

<a href="https://x.com/bcherny/status/2038454351849787485"><img src="assets/boris-26-3-30/9.png" alt="/btw for side queries" width="50%" /></a>

---

## 10/ Use Git Worktrees

Claude Code는 git worktree에 대한 깊은 지원을 기본 제공합니다. Worktree는 같은 저장소에서 대량의 병렬 작업을 하는 데 필수적입니다. Boris는 **항상 수십 개의 Claude를 동시에 실행**하는데, 바로 이 방식으로 합니다.

- `claude -w`로 worktree에서 새 세션을 시작하세요
- 또는 Claude Desktop 앱에서 **"worktree" 체크박스**를 누르세요
- git이 아닌 VCS 사용자는 `WorktreeCreate` hook을 사용해 worktree 생성에 대한 자체 로직을 추가하세요

<a href="https://x.com/bcherny/status/2038454353787519164"><img src="assets/boris-26-3-30/10.png" alt="Git worktrees" width="50%" /></a>

---

## 11/ Use /batch to Fan Out Massive Changesets

`/batch`는 당신을 인터뷰한 다음, 작업을 완수하는 데 필요한 만큼 많은 **worktree 에이전트**(수십, 수백, 심지어 수천 개)에게 Claude가 작업을 분산하도록 합니다.

- 대규모 코드 마이그레이션이나 기타 병렬화 가능한 작업에 사용하세요
- 각 worktree 에이전트는 자신만의 코드베이스 복사본에서 독립적으로 작업합니다

<a href="https://x.com/bcherny/status/2038454355469484142"><img src="assets/boris-26-3-30/11.png" alt="/batch for massive changesets" width="50%" /></a>

---

## 12/ Use --bare to Speed Up SDK Startup by Up to 10x

기본적으로 `claude -p`(또는 TypeScript나 Python SDK)를 실행하면 Claude는 로컬 CLAUDE.md, 설정, MCP를 검색합니다. 하지만 비대화형 사용에서는 대부분의 경우 `--system-prompt`, `--mcp-config`, `--settings` 등을 통해 로드할 항목을 명시적으로 지정하고 싶을 것입니다.

- 이것은 SDK를 처음 만들 때의 설계상 실수였습니다
- 향후 버전에서는 기본값을 `--bare`로 뒤집을 예정입니다
- 지금은 이 플래그로 직접 옵트인하여 **최대 10배 빠른 시작**을 얻으세요

```bash
claude -p "summarize this codebase" \
    --output-format=stream-json \
    --verbose \
    --bare
```

<a href="https://x.com/bcherny/status/2038454357088457168"><img src="assets/boris-26-3-30/12.png" alt="--bare flag for SDK startup" width="50%" /></a>

---

## 13/ Use --add-dir to Give Claude Access to More Folders

여러 저장소에 걸쳐 작업할 때, Boris는 보통 한 저장소에서 Claude를 시작한 뒤 `--add-dir`(또는 `/add-dir`)로 Claude가 다른 저장소를 볼 수 있게 합니다.

- 이는 Claude에게 해당 저장소를 알려줄 뿐만 아니라, 그 저장소에서 작업할 수 있는 **권한도 부여**합니다
- 또는 팀의 `settings.json`에 `"additionalDirectories"`를 추가하면 Claude Code를 시작할 때 항상 추가 폴더를 로드합니다

<a href="https://x.com/bcherny/status/2038454359047156203"><img src="assets/boris-26-3-30/13.png" alt="--add-dir for multiple repos" width="50%" /></a>

---

## 14/ Use --agent to Give Claude Code a Custom System Prompt & Tools

커스텀 에이전트는 자주 간과되는 강력한 기본 요소(primitive)입니다. 사용하려면 `.claude/agents/`에 새 에이전트를 정의한 다음 실행하세요:

```bash
claude --agent=<your agent's name>
```

- 에이전트는 제한된 도구, 커스텀 설명, 특정 모델을 가질 수 있습니다
- 읽기 전용 에이전트, 전문 리뷰 에이전트, 도메인 특화 도구를 만드는 데 아주 좋습니다

<a href="https://x.com/bcherny/status/2038454360418787764"><img src="assets/boris-26-3-30/14.png" alt="--agent for custom system prompts" width="50%" /></a>

---

## 15/ Use /voice to Enable Voice Input

재미있는 사실: Boris는 대부분의 코딩을 타이핑이 아니라 Claude에게 말을 걸어서 합니다.

- CLI에서 `/voice`를 실행한 뒤 스페이스바를 눌러 말하세요
- Desktop에서는 음성 버튼을 누르세요
- 또는 iOS 설정에서 받아쓰기(dictation)를 활성화하세요

<a href="https://x.com/bcherny/status/2038454362226467112"><img src="assets/boris-26-3-30/15.png" alt="/voice for voice input" width="50%" /></a>

---

## Sources

- [Boris Cherny (@bcherny) on X — March 30, 2026](https://x.com/bcherny/status/2038454336355999749)
