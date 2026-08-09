<!--
  이 문서는 tips/claude-boris-10-tips-01-feb-26.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# 10 Tips for Using Claude Code — From the Claude Code Team

Claude Code의 개발자인 Boris Cherny ([@bcherny](https://x.com/bcherny))가 2026년 2월 1일에 공유한 팀 팁 요약입니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## Context

Boris는 Claude Code 팀에서 직접 나온 Claude Code 사용 팁을 공유했습니다. 팀이 Claude를 쓰는 방식은 Boris 개인의 사용 방식과는 다릅니다. 기억하세요: Claude Code를 쓰는 데 정답은 하나가 아닙니다 — 사람마다 설정이 제각각입니다. 자신에게 맞는 방식을 직접 실험해 보세요!

<a href="https://x.com/bcherny/status/2017742741636321619"><img src="assets/boris-26-2-1/0.png" alt="Boris Cherny intro tweet" width="50%" /></a>

---

## 1/ Do More in Parallel

한 번에 3~5개의 git worktree를 띄우고, 각각에서 별도의 Claude 세션을 병렬로 돌리세요. 이것이 단일 요소로는 가장 큰 생산성 향상이며, 팀에서 뽑은 최고의 팁입니다. Boris 개인은 여러 개의 git 체크아웃을 사용하지만, Claude Code 팀 대부분은 worktree를 선호합니다 — `@amorisscode`가 Claude Desktop 앱에 worktree 네이티브 지원을 넣은 이유이기도 하죠!

어떤 사람들은 worktree에 이름을 붙이고 셸 별칭(`2a`, `2b`, `2c`)을 설정해서 한 번의 키 입력으로 그 사이를 오갑니다. 또 어떤 사람들은 로그를 읽고 BigQuery를 돌리는 용도로만 쓰는 전용 "analysis" worktree를 둡니다.

참고: [Worktrees Docs](https://code.claude.com/docs/en/common...)

<a href="https://x.com/bcherny/status/2017742743125299476"><img src="assets/boris-26-2-1/1.png" alt="Do more in parallel" width="50%" /></a>

---

## 2/ Start Every Complex Task in Plan Mode

계획에 에너지를 쏟아부어서 Claude가 구현을 한 방에 끝낼 수 있게 하세요.

어떤 사람은 하나의 Claude에게 계획을 작성하게 한 뒤, 두 번째 Claude를 띄워서 스태프 엔지니어처럼 그 계획을 리뷰하게 합니다.

또 다른 사람은 뭔가 어긋나는 순간 바로 plan mode로 돌아가 다시 계획을 세운다고 합니다. 계속 밀어붙이지 마세요. 이들은 빌드뿐 아니라 검증 단계에서도 plan mode로 들어가라고 Claude에게 명시적으로 지시합니다.

<a href="https://x.com/bcherny/status/2017742745365057733"><img src="assets/boris-26-2-1/2.png" alt="Start every complex task in plan mode" width="50%" /></a>

---

## 3/ Invest in Your CLAUDE.md

수정을 해줄 때마다 마지막에 이렇게 덧붙이세요: "Update your CLAUDE.md so you don't make that mistake again." Claude는 스스로를 위한 규칙을 쓰는 데 소름 끼칠 정도로 능숙합니다.

시간이 지나면서 `CLAUDE.md`를 가차 없이 다듬으세요. Claude의 실수율이 눈에 띄게 떨어질 때까지 계속 반복하세요.

한 엔지니어는 모든 작업/프로젝트에 대해 노트 디렉터리를 유지하도록 Claude에게 시키고, 매 PR마다 이를 갱신하게 합니다. 그리고 `CLAUDE.md`가 그 디렉터리를 가리키게 해둡니다.

<a href="https://x.com/bcherny/status/2017742747067945390"><img src="assets/boris-26-2-1/3.png" alt="Invest in your CLAUDE.md" width="50%" /></a>

---

## 4/ Create Your Own Skills and Commit Them to Git

모든 프로젝트에서 재사용하세요. 팀의 팁:

- 하루에 한 번 이상 하는 일이 있다면 skill이나 command로 만드세요
- `/techdebt` 슬래시 커맨드를 만들어서 매 세션 끝에 실행해 중복 코드를 찾아 없애세요
- Slack, GDrive, Asana, GitHub의 7일치를 하나의 컨텍스트 덤프로 동기화하는 슬래시 커맨드를 설정하세요
- dbt 모델을 작성하고, 코드를 리뷰하고, dev에서 변경 사항을 테스트하는 애널리틱스 엔지니어 스타일의 에이전트를 만드세요

참고: [Extend Claude with Skills — Claude Code Docs](https://code.claude.com/docs/en/skills)

<a href="https://x.com/bcherny/status/2017742748984742078"><img src="assets/boris-26-2-1/4.png" alt="Create your own skills" width="50%" /></a>

---

## 5/ Claude Fixes Most Bugs by Itself

팀은 이렇게 합니다:

Slack MCP를 활성화한 뒤, Slack의 버그 스레드를 Claude에 붙여넣고 그냥 "fix"라고만 하세요. 컨텍스트 전환이 전혀 필요 없습니다.

또는 그냥 "Go fix the failing CI tests."라고 하세요. 어떻게 하라고 세세하게 관리하지 마세요.

분산 시스템 문제를 해결하려면 Claude를 docker 로그로 가리키세요 — 이런 일에 의외로 능숙합니다.

<a href="https://x.com/bcherny/status/2017742750473720121"><img src="assets/boris-26-2-1/5.png" alt="Claude fixes most bugs by itself" width="50%" /></a>

---

## 6/ Level Up Your Prompting

a. **Claude에게 도전하세요.** "Grill me on these changes and don't make a PR until I pass your test."라고 해서 Claude를 여러분의 리뷰어로 만드세요. 또는 "Prove to me this works"라고 하고, main과 여러분의 피처 브랜치 사이의 동작 차이를 Claude가 비교하게 하세요.

b. **어중간한 수정을 받은 뒤에는** 이렇게 말하세요: "Knowing everything you know now, scrap this and implement the elegant solution."

c. **상세한 스펙을 작성**하고 작업을 넘기기 전에 모호함을 줄이세요. 구체적일수록 결과물이 좋아집니다.

<a href="https://x.com/bcherny/status/2017742752566632544"><img src="assets/boris-26-2-1/6.png" alt="Level up your prompting" width="50%" /></a>

---

## 7/ Terminal & Environment Setup

팀은 Ghostty를 정말 좋아합니다! 여러 사람이 동기화된 렌더링, 24비트 색상, 제대로 된 유니코드 지원을 마음에 들어 합니다.

Claude를 더 쉽게 다루려면 `/statusline`을 사용해 상태 바를 커스터마이즈해서 항상 컨텍스트 사용량과 현재 git 브랜치를 표시하게 하세요. 많은 이들이 터미널 탭을 색상별로 구분하고 이름을 붙이며, 때로는 tmux를 써서 작업/worktree마다 탭을 하나씩 둡니다.

음성 받아쓰기를 사용하세요. 말하는 것이 타이핑보다 3배 빠르고, 그 결과 프롬프트가 훨씬 상세해집니다. (macOS에서는 fn을 두 번 누르세요)

참고: [Terminal Setup Docs](https://code.claude.com/docs/en/termin...)

<a href="https://x.com/bcherny/status/2017742753971769626"><img src="assets/boris-26-2-1/7.png" alt="Terminal and environment setup" width="50%" /></a>

---

## 8/ Use Subagents

a. Claude가 문제에 더 많은 연산을 투입하기를 원하는 요청에는 "use subagents"를 덧붙이세요.

b. 개별 작업을 서브에이전트에 넘겨서 메인 에이전트의 컨텍스트 윈도우를 깨끗하고 집중된 상태로 유지하세요.

c. 훅을 통해 권한 요청을 Opus 4.5로 라우팅하세요 — 공격을 스캔하고 안전한 것은 자동 승인하게 하세요. 참고: [Hooks Docs](https://code.claude.com/docs/en/hooks#...)

<a href="https://x.com/bcherny/status/2017742755737555434"><img src="assets/boris-26-2-1/8.png" alt="Use subagents" width="50%" /></a>

---

## 9/ Use Claude for Data & Analytics

Claude Code에게 "bq" CLI를 사용해 즉석에서 지표를 뽑아 분석하도록 요청하세요. 팀은 코드베이스에 체크인된 BigQuery skill을 두고 있고, 모두가 Claude Code 안에서 바로 애널리틱스 쿼리에 이를 사용합니다. Boris 개인은 6개월 넘게 SQL을 한 줄도 쓰지 않았습니다.

이것은 CLI, MCP, 또는 API가 있는 모든 데이터베이스에서 동작합니다.

<a href="https://x.com/bcherny/status/2017742757666902374"><img src="assets/boris-26-2-1/9.png" alt="Use Claude for data and analytics" width="50%" /></a>

---

## 10/ Learning with Claude

Claude Code를 학습에 활용하기 위한 팀의 팁 몇 가지:

a. `/config`에서 "Explanatory" 또는 "Learning" 출력 스타일을 활성화해서 Claude가 변경 사항 뒤에 있는 "왜"를 설명하게 하세요.

b. 낯선 코드를 설명하는 시각적 HTML 프레젠테이션을 Claude가 생성하게 하세요. 의외로 슬라이드를 잘 만듭니다!

c. 새로운 프로토콜과 코드베이스를 이해하는 데 도움이 되도록 Claude에게 ASCII 다이어그램을 그려 달라고 요청하세요.

d. 간격 반복(spaced-repetition) 학습 skill을 만드세요: 여러분이 이해한 바를 설명하면, Claude가 빈틈을 메우기 위한 후속 질문을 던지고, 그 결과를 저장합니다.

<a href="https://x.com/bcherny/status/2017742759218794768"><img src="assets/boris-26-2-1/10.png" alt="Learning with Claude" width="50%" /></a>

---

## Sources

- [Boris Cherny (@bcherny) on X — February 1, 2026](https://x.com/bcherny/status/2017742741636321619)
