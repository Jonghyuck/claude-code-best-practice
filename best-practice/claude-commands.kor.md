<!--
  이 문서는 best-practice/claude-commands.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Commands Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Jul%2011%2C%202026%2011%3A08%20AM%20PKT-white?style=flat&labelColor=555) ![Version](https://img.shields.io/badge/Claude_Code-v2.1.207-blue?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../implementation/claude-commands-implementation.md)

Claude Code 커맨드 — frontmatter 필드와 공식 내장 슬래시 커맨드.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Frontmatter Fields (16)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | 표시 이름 및 `/slash-command` 식별자. 생략하면 디렉터리 이름을 기본값으로 사용 |
| `description` | string | Recommended | 커맨드가 하는 일. 자동완성에 표시되며 Claude의 자동 발견에도 사용됨 |
| `when_to_use` | string | No | Claude가 이 스킬을 호출해야 하는 시점에 대한 추가 맥락 — 트리거 문구나 예시 요청. 목록에서 `description` 뒤에 붙으며 1,536자 제한에 포함됨 |
| `argument-hint` | string | No | 자동완성 중 표시되는 힌트(예: `[issue-number]`, `[filename]`) |
| `arguments` | string/list | No | 커맨드 내용에서 `$name` 치환에 쓰이는 명명된 위치 인자. 공백으로 구분된 문자열 또는 YAML 리스트를 허용하며 — 이름은 순서대로 인자 위치에 매핑됨 |
| `disable-model-invocation` | boolean | No | Claude가 이 커맨드를 자동으로 호출하지 못하게 하려면 `true`로 설정 |
| `user-invocable` | boolean | No | `/` 메뉴에서 숨기려면 `false`로 설정 — 커맨드가 배경 지식으로만 남음 |
| `paths` | string/list | No | 이 스킬이 활성화되는 시점을 제한하는 glob 패턴. 쉼표로 구분된 문자열 또는 YAML 리스트를 허용. 설정하면 Claude는 패턴에 일치하는 파일을 다룰 때만 자동으로 스킬을 로드함 |
| `allowed-tools` | string | No | 이 커맨드가 활성화된 동안 권한 프롬프트 없이 허용되는 도구 |
| `disallowed-tools` | string/list | No | 이 커맨드가 활성화된 동안 Claude의 사용 가능한 도구 풀에서 제거되는 도구. 다음 메시지를 보내면 해제됨. `allowed-tools`의 반대 |
| `model` | string | No | 이 커맨드 실행 시 사용할 모델(예: `haiku`, `sonnet`, `opus`) |
| `effort` | string | No | 호출 시 모델 effort 수준을 재정의(`low`, `medium`, `high`, `xhigh`, `max`) |
| `context` | string | No | 커맨드를 격리된 서브에이전트 컨텍스트에서 실행하려면 `fork`로 설정 |
| `agent` | string | No | `context: fork`가 설정되었을 때의 서브에이전트 유형(기본값: `general-purpose`) |
| `shell` | string | No | `` !`command` `` 블록에 사용할 셸 — `bash`(기본값) 또는 `powershell`을 허용. `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`이 필요함 |
| `hooks` | object | No | 이 커맨드에 스코프된 라이프사이클 훅 |

---

## ![Official](../!/tags/official.svg) **(86)**

| # | Command | Tag | Description |
|---|---------|-----|-------------|
| 1 | `/design-login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | claude.ai 계정으로 `/design-sync`의 디자인 시스템 접근을 인가 |
| 2 | `/login` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Anthropic 계정에 로그인 |
| 3 | `/logout` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | Anthropic 계정에서 로그아웃 |
| 4 | `/setup-bedrock` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 대화형 마법사를 통해 Amazon Bedrock 인증, 리전, 모델 핀을 구성. `CLAUDE_CODE_USE_BEDROCK=1`이 설정된 경우에만 표시됨. Bedrock을 처음 쓰는 사용자는 로그인 화면에서도 이 마법사에 접근할 수 있음 |
| 5 | `/setup-vertex` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 대화형 마법사를 통해 Google Cloud의 Agent Platform 인증, 프로젝트, 리전, 모델 핀을 구성. `CLAUDE_CODE_USE_VERTEX=1`이 설정된 경우에만 표시됨. 처음 쓰는 사용자는 로그인 화면에서도 이 마법사에 접근할 수 있음 |
| 6 | `/upgrade` | ![Auth](https://img.shields.io/badge/Auth-2980B9?style=flat) | 상위 요금제 등급으로 전환하는 업그레이드 페이지를 엶 |
| 7 | `/color [color\|default]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 현재 세션의 프롬프트 바 색상을 설정. 사용 가능한 색상: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan`. 초기화하려면 `default` 사용. 인자 없이 실행하면 무작위 색상을 선택 |
| 8 | `/config [key=value ...]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 테마, 모델, 출력 스타일 및 기타 환경설정을 조정하는 Settings 인터페이스를 엶. v2.1.181부터 하나 이상의 `key=value` 쌍을 전달하면 인터페이스를 열지 않고 설정을 직접 지정할 수 있음(예: `/config thinking=false`). `key=value` 형식은 비대화형(`-p`) 및 Remote Control 모드에서도 동작함. 설정할 수 있는 키 목록은 `/config help`로 확인. 별칭: `/settings` |
| 9 | `/focus` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 마지막 프롬프트, 편집 diffstat이 포함된 한 줄 도구 호출 요약, 최종 응답만 보여주는 focus 뷰를 토글. 선택은 세션 간 유지되며, 재정의하려면 설정에서 `viewMode`를 지정. 전체 화면 렌더링에서만 사용 가능 |
| 10 | `/keybindings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 키바인딩 구성 파일을 열거나 생성 |
| 11 | `/permissions` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 도구 권한의 allow, ask, deny 규칙을 관리. 스코프별로 규칙을 보고, 규칙을 추가·제거하고, 작업 디렉터리를 관리하고, 최근 auto 모드 거부를 검토할 수 있는 대화형 대화상자를 엶. 별칭: `/allowed-tools` |
| 12 | `/privacy-settings` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 개인정보 설정을 보고 업데이트. Pro 및 Max 요금제 구독자만 사용 가능 |
| 13 | `/radio` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 브라우저에서 Claude FM lo-fi 라디오를 엶. 브라우저를 사용할 수 없으면 스트림 URL을 출력. Bedrock, Vertex, Foundry에서는 사용 불가 |
| 14 | `/sandbox` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 샌드박스 모드를 토글. 지원되는 플랫폼에서만 사용 가능 |
| 15 | `/scroll-speed` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 마우스 휠 스크롤 속도를 대화형으로 조정 |
| 16 | `/statusline` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Claude Code의 상태 표시줄을 구성. 원하는 바를 설명하거나, 인자 없이 실행하면 셸 프롬프트로부터 자동 구성 |
| 17 | `/stickers` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Claude Code 스티커 주문 |
| 18 | `/terminal-setup` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | Shift+Enter 및 기타 단축키를 위한 터미널 키바인딩을 구성. VS Code, Cursor, Devin Desktop, Alacritty, Zed처럼 필요한 터미널에서만 표시됨 |
| 19 | `/theme` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 컬러 테마를 변경. 라이트 및 다크 변형, 색맹 접근성(달토나이즈) 테마, 터미널의 색상 팔레트를 사용하는 ANSI 테마, 터미널의 라이트/다크 모드를 따르는 "Auto (match terminal)" 옵션, 그리고 `~/.claude/themes/`나 플러그인에서 로드된 커스텀 테마를 포함. 직접 만들려면 "New custom theme…"를 선택 |
| 20 | `/tui [default\|fullscreen]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 터미널 UI 렌더러를 설정하고 대화를 그대로 유지한 채 재실행. `fullscreen`은 깜빡임 없는 alt-screen 렌더러를 활성화. 인자가 없으면 활성 렌더러를 출력 |
| 21 | `/voice [hold\|tap\|off]` | ![Config](https://img.shields.io/badge/Config-F39C12?style=flat) | 음성 받아쓰기를 토글하거나 특정 모드로 활성화. Claude.ai 계정이 필요함 |
| 22 | `/context [all]` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 현재 컨텍스트 사용량을 색상 그리드로 시각화. 컨텍스트를 많이 쓰는 도구, 메모리 과다, 용량 경고에 대한 최적화 제안을 표시. `all`을 전달하면 전체 분석을 펼침 |
| 23 | `/cost` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | `/usage`의 별칭 |
| 24 | `/insights` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 프로젝트 영역, 상호작용 패턴, 마찰 지점을 포함해 Claude Code 세션을 분석하는 보고서를 생성 |
| 25 | `/stats` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | `/usage`의 별칭. Stats 탭에서 열림 |
| 26 | `/status` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 버전, 모델, 계정, 연결 상태를 보여주는 Settings 인터페이스(Status 탭)를 엶. Claude가 응답하는 중에도 현재 응답이 끝나기를 기다리지 않고 동작함 |
| 27 | `/usage` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 세션 비용, 요금제 사용 한도, 활동 통계를 표시. Pro, Max, Team, Enterprise 요금제에서는 스킬, 서브에이전트, 플러그인, MCP 서버별 사용량 분석을 포함. `/cost`와 `/stats`는 별칭 |
| 28 | `/usage-credits` | ![Context](https://img.shields.io/badge/Context-8E44AD?style=flat) | 한도에 도달했을 때 계속 작업하기 위한 사용 크레딧을 구성. 이전 명칭은 `/extra-usage` |
| 29 | `/doctor` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | Claude Code 설치 및 설정을 진단하고 검증. 결과가 상태 아이콘과 함께 표시됨. `f`를 누르면 보고된 문제를 Claude가 고치게 할 수 있음 |
| 30 | `/feedback [report]` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 피드백 제출, 버그 신고, 대화 공유. 별칭: `/bug`, `/share` |
| 31 | `/heapdump` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 높은 메모리 사용을 진단하기 위해 JavaScript 힙 스냅샷과 메모리 분석을 `~/Desktop`에 기록. 메모리 증가에 대한 버그 리포트를 제출할 때 유용 |
| 32 | `/help` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 도움말과 사용 가능한 커맨드를 표시 |
| 33 | `/powerup` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 애니메이션 데모가 포함된 짧은 대화형 레슨을 통해 Claude Code 기능을 알아봄 |
| 34 | `/release-notes` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 대화형 버전 선택기에서 변경 로그를 봄. 특정 버전을 선택해 릴리스 노트를 보거나 모든 버전을 표시하도록 선택 |
| 35 | `/tasks` | ![Debug](https://img.shields.io/badge/Debug-E74C3C?style=flat) | 백그라운드에서 실행 중인 모든 것을 보고 관리. `/bashes`로도 사용 가능 |
| 36 | `/copy [N]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 마지막 어시스턴트 응답을 클립보드로 복사. 숫자 `N`을 전달하면 N번째로 최근 응답을 복사(`/copy 2`는 마지막에서 두 번째를 복사). 코드 블록이 있으면 개별 블록이나 전체 응답을 선택하는 대화형 선택기를 표시. 선택기에서 `w`를 누르면 클립보드 대신 파일로 기록하며, SSH에서 유용 |
| 37 | `/export [filename]` | ![Export](https://img.shields.io/badge/Export-7F8C8D?style=flat) | 현재 대화를 일반 텍스트로 내보냄. 파일명을 지정하면 그 파일에 직접 기록. 지정하지 않으면 클립보드로 복사하거나 파일로 저장하는 대화상자를 엶 |
| 38 | `/agents` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 에이전트 구성 관리 안내를 출력 — Claude에게 서브에이전트를 생성·관리하도록 요청하거나 `.claude/agents/` 또는 `~/.claude/agents/`를 직접 편집 |
| 39 | `/chrome` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Claude in Chrome 설정을 구성 |
| 40 | `/hooks` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 도구 이벤트에 대한 훅 구성을 봄 |
| 41 | `/ide` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | IDE 통합을 관리하고 상태를 표시 |
| 42 | `/mcp [reconnect <server>\|enable\|disable [<server>\|all]]` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | MCP 서버 연결과 OAuth 인증을 관리. 인자 없이 실행하면 대화형 목록을 열고, `reconnect <server>`를 전달하면 연결이 끊긴 서버 하나를 재연결하며, `enable`/`disable`을 서버 이름 또는 `all`과 함께 전달하면 대화상자를 열지 않고 연결 상태를 변경 |
| 43 | `/plugin [subcommand]` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | Claude Code 플러그인을 관리. 인자 없이 실행하면 플러그인 메뉴를 열고, `list`, `install`, `enable`, `disable` 같은 서브커맨드를 전달하면 직접 동작 |
| 44 | `/reload-plugins [--force]` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 재시작 없이 대기 중인 변경을 적용하기 위해 활성 플러그인을 모두 다시 로드. 재로드된 각 구성 요소의 개수를 보고하고 로드 오류를 표시. 재로드가 로드되는 MCP 도구를 바꾸고 프롬프트 캐시를 무효화하는 경우, `--force`를 전달하지 않으면 경고 후 건너뜀 |
| 45 | `/reload-skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 스킬 및 커맨드 디렉터리를 다시 스캔해 세션 중 디스크에서 추가·변경된 스킬을 재시작 없이 사용할 수 있게 함. 사용 가능한 스킬 수와 추가·제거된 수를 보고 |
| 46 | `/skills` | ![Extensions](https://img.shields.io/badge/Extensions-16A085?style=flat) | 사용 가능한 스킬을 나열. `t`를 누르면 토큰 수로 정렬. `Space`를 누르면 스킬을 Claude나 `/` 메뉴에서 숨긴 뒤 `Enter`로 저장 |
| 47 | `/memory` | ![Memory](https://img.shields.io/badge/Memory-3498DB?style=flat) | `CLAUDE.md` 메모리 파일을 편집하고, auto-memory를 켜거나 끄고, auto-memory 항목을 봄 |
| 48 | `/advisor [model\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 작업 중 주요 시점에 두 번째 모델에게 조언을 구하는 advisor 도구를 켜거나 끔. 모델 이름(`opus`, `sonnet`, `fable`)이나 전체 모델 ID를 허용하며, 인자가 없으면 선택기를 엶. 비활성화하려면 `off` 사용 |
| 49 | `/effort [low\|medium\|high\|xhigh\|max\|ultracode]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 모델 effort 수준을 설정. 사용 가능한 수준은 모델에 따라 다르며 `low`, `medium`, `high`, `xhigh`, `max`(세션 한정), `ultracode`(`xhigh` 추론과 자동 워크플로 오케스트레이션을 결합; 세션 한정)를 포함. 인자가 없으면 수준을 선택하는 대화형 슬라이더를 엶. `auto`는 모델 기본값으로 초기화. 현재 응답이 끝나기를 기다리지 않고 즉시 적용됨 |
| 50 | `/fast [on\|off]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | fast 모드를 켜거나 끔 |
| 51 | `/model [model]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | AI 모델을 전환하고 새 세션의 기본값으로 저장. 행에서 `s`를 누르면 현재 세션에만 전환. 지원하는 모델은 좌/우 화살표로 effort 수준을 조정. 이전 출력이 있는 대화 중간에 전환하면 Claude가 변경을 적용하기 전에 경고 |
| 52 | `/passes` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 친구에게 Claude Code 무료 1주일을 공유. 계정이 자격을 갖춘 경우에만 표시됨 |
| 53 | `/plan [description]` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | 프롬프트에서 바로 plan 모드로 진입. 선택적 설명을 전달하면 plan 모드로 진입해 즉시 그 작업으로 시작(예: `/plan fix the auth bug`) |
| 54 | `/ultraplan <prompt>` | ![Model](https://img.shields.io/badge/Model-E67E22?style=flat) | ultraplan 세션에서 계획을 초안하고 브라우저에서 검토한 뒤, 원격으로 실행하거나 터미널로 다시 보냄 |
| 55 | `/add-dir <path>` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 현재 세션 동안 파일 접근을 위한 작업 디렉터리를 추가. 대부분의 `.claude/` 구성은 추가된 디렉터리에서 발견되지 않음 |
| 56 | `/diff` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 커밋되지 않은 변경과 턴별 diff를 보여주는 대화형 diff 뷰어를 엶. 좌/우 화살표로 현재 git diff와 개별 Claude 턴 사이를 전환하고, 상/하 화살표로 파일을 탐색 |
| 57 | `/init` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | `CLAUDE.md` 가이드로 프로젝트를 초기화. 스킬, 훅, 개인 메모리 파일까지 안내하는 대화형 플로우를 원하면 `CLAUDE_CODE_NEW_INIT=1`을 설정 |
| 58 | `/review [PR]` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | GitHub 풀 리퀘스트를 번호로 지정해 빠른 단일 패스 리뷰. 더 깊은 리뷰는 `/code-review`를 사용. 인자가 없으면 열린 PR 목록을 표시 |
| 59 | `/security-review` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 현재 브랜치의 대기 중 변경을 보안 취약점 관점에서 분석. git diff를 검토해 인젝션, 인증 문제, 데이터 노출 같은 위험을 식별 |
| 60 | `/team-onboarding` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | Claude Code 사용 이력으로부터 팀 온보딩 가이드를 생성. Claude가 지난 30일간의 세션, 커맨드, MCP 서버 사용을 분석해 마크다운 가이드를 만듦. Pro, Max, Team, Enterprise 요금제의 claude.ai 구독자에게는 팀원이 Claude Code에서 바로 열 수 있는 공유 링크도 반환 |
| 61 | `/ultrareview [PR]` | ![Project](https://img.shields.io/badge/Project-27AE60?style=flat) | 클라우드 샌드박스에서 깊은 멀티 에이전트 코드 리뷰를 실행. 권장 호출 방식은 `/code-review ultra`이며, `/ultrareview`는 별칭으로 남아 있음. Pro와 Max에서 무료 3회를 포함하고, 이후에는 사용 크레딧이 필요 |
| 62 | `/autofix-pr [prompt]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 현재 브랜치의 PR을 감시하다가 CI가 실패하거나 리뷰어가 댓글을 남기면 수정을 푸시하는 Claude Code on the web 세션을 생성. `gh pr view`로 체크아웃된 브랜치에서 열린 PR을 감지하며, 다른 PR을 감시하려면 먼저 해당 브랜치를 체크아웃. `gh` CLI와 Claude Code on the web 접근이 필요 |
| 63 | `/desktop` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 현재 세션을 Claude Code Desktop 앱에서 이어감. macOS와 Windows 전용. 별칭: `/app` |
| 64 | `/install-github-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 저장소에 Claude GitHub App을 설치하며, GitHub Actions 워크플로와 시크릿을 설정하는 선택 단계 포함 |
| 65 | `/install-slack-app` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Claude Slack 앱을 설치. OAuth 플로우를 완료하기 위해 브라우저를 엶 |
| 66 | `/mobile` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Claude 모바일 앱을 다운로드할 QR 코드를 표시. 별칭: `/ios`, `/android` |
| 67 | `/remote-control` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | claude.ai에서 이 세션을 원격 제어할 수 있게 함. 별칭: `/rc` |
| 68 | `/remote-env` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 클라우드 에이전트의 기본 환경을 선택 |
| 69 | `/schedule [description]` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 루틴을 생성, 업데이트, 나열, 실행. Claude가 대화형으로 설정을 안내. 별칭: `/routines` |
| 70 | `/teleport` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | Claude Code on the web 세션을 이 터미널로 가져옴: 선택기를 연 뒤 브랜치와 대화를 가져옴. `/tp`로도 사용 가능. claude.ai 구독이 필요 |
| 71 | `/web-setup` | ![Remote](https://img.shields.io/badge/Remote-5D6D7E?style=flat) | 로컬 `gh` CLI 자격 증명을 사용해 GitHub 계정을 Claude Code on the web에 연결. GitHub가 연결되어 있지 않으면 `/schedule`이 자동으로 이를 요청 |
| 72 | `/background [prompt]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 현재 세션을 백그라운드 에이전트로 실행하도록 분리하고 이 터미널을 비움. 분리 전에 지시 하나를 더 보내려면 프롬프트를 전달. `claude agents`로 세션을 모니터링. 별칭: `/bg` |
| 73 | `/branch [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 이 지점에서 현재 대화의 브랜치를 생성 |
| 74 | `/btw <question>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 대화에 추가하지 않고 빠른 곁가지 질문을 함 |
| 75 | `/cd <path>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 프롬프트 캐시를 깨지 않고 세션을 새 작업 디렉터리로 이동 |
| 76 | `/clear [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 빈 컨텍스트로 새 대화를 시작. 선택적 `name`을 전달하면 이전 대화에 레이블을 붙여 `/resume`으로 쉽게 불러올 수 있음. 같은 대화를 이어가며 컨텍스트를 확보하려면 `/compact`를 대신 사용. 별칭: `/reset`, `/new` |
| 77 | `/compact [instructions]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 선택적 포커스 지시와 함께 대화를 압축 |
| 78 | `/exit` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | CLI를 종료. 연결된 백그라운드 세션에서는 분리되며 세션은 계속 실행됨. 별칭: `/quit` |
| 79 | `/fork <directive>` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 포크된 서브에이전트를 생성: 전체 대화를 상속받아 지시를 수행하는 백그라운드 서브에이전트이며 그동안 사용자는 계속 진행 가능 |
| 80 | `/goal [condition\|clear]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 목표를 설정 — 조건이 충족될 때까지 Claude가 여러 턴에 걸쳐 계속 작업. 인자가 없으면 현재 또는 가장 최근 달성한 목표를 표시. `clear`, `stop`, `off`, `reset`, `none`, `cancel`은 활성 목표를 조기에 제거 |
| 81 | `/recap` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 진행 중인 대화에 영향을 주지 않고 현재 세션의 한 줄 요약을 필요할 때 생성 |
| 82 | `/rename [name]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 현재 세션의 이름을 바꾸고 프롬프트 바에 이름을 표시. 이름이 없으면 대화 이력에서 자동 생성 |
| 83 | `/resume [session]` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | ID나 이름으로 대화를 재개하거나 세션 선택기를 엶. v2.1.144부터 백그라운드 세션이 `bg` 표시와 함께 선택기에 나타남. 별칭: `/continue` |
| 84 | `/rewind` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 대화 및/또는 코드를 이전 지점으로 되감거나, 선택한 메시지부터 요약. checkpointing 참조. 별칭: `/checkpoint`, `/undo` |
| 85 | `/stop` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 현재 백그라운드 세션을 중지. 백그라운드 세션에 연결된 동안에만 사용 가능하며, 트랜스크립트와 워크트리는 유지됨. 중지하지 않고 분리하려면 `/exit`을 사용하거나 `←`를 누름 |
| 86 | `/workflows` | ![Session](https://img.shields.io/badge/Session-4A90D9?style=flat) | 실행 중이거나 완료된 워크플로를 보고, 일시정지·재개·저장할 수 있는 워크플로 진행 뷰를 엶 |

`/debug` 같은 번들 스킬도 슬래시 커맨드 메뉴에 나타날 수 있지만, 내장 커맨드는 아님.

---

## Sources

- [Claude Code Slash Commands](https://code.claude.com/docs/en/slash-commands)
- [Claude Code Interactive Mode](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
