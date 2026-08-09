<!--
  이 문서는 best-practice/claude-memory.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Claude Memory

CLAUDE.md 파일을 통한 영속적 컨텍스트 — 작성 방법과 모노레포에서의 로딩 방식.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## 1. Writing a Good CLAUDE.md

잘 구조화된 CLAUDE.md는 프로젝트에서 Claude Code의 출력을 개선하는 가장 효과적인 단 하나의 방법입니다. Humanlayer는 무엇을 담아야 하는지, 어떻게 구조화해야 하는지, 그리고 흔히 저지르는 실수를 다룬 훌륭한 가이드를 제공합니다.

- [Humanlayer - Writing a good Claude.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)

---

## 2. CLAUDE.md in Large Monorepos

모노레포에서 Claude Code를 사용할 때, 프로젝트 지침을 효과적으로 구성하려면 CLAUDE.md 파일이 어떻게 컨텍스트로 로드되는지 이해하는 것이 매우 중요합니다.

<p align="center">
  <a href="https://x.com/bcherny/status/2016339448863355206"><img src="assets/claude-memory/claude-memory-monorepo.jpg" alt="CLAUDE.md loading in monorepos" width="600"></a>
</p>

### The Two Loading Mechanisms

Claude Code는 CLAUDE.md 파일을 로드하는 데 두 가지 서로 다른 메커니즘을 사용합니다:

#### Ancestor Loading (UP the directory tree)

Claude Code를 시작하면, 현재 작업 디렉터리에서 파일시스템 루트를 향해 **위쪽으로** 이동하면서 경로상에서 발견하는 모든 CLAUDE.md를 로드합니다. 이 파일들은 **시작 시점에 즉시** 로드됩니다.

#### Descendant Loading (DOWN the directory tree)

현재 작업 디렉터리 아래의 하위 디렉터리에 있는 CLAUDE.md 파일들은 **실행 시점에 로드되지 않습니다**. 이 파일들은 세션 도중 Claude가 해당 하위 디렉터리의 파일을 읽을 때에만 포함됩니다. 이를 **지연 로딩(lazy loading)**이라고 합니다.

### Example Monorepo Structure

서로 다른 컴포넌트를 위한 별도 디렉터리를 가진 전형적인 모노레포를 생각해 봅시다:

```
/mymonorepo/
├── CLAUDE.md          # Root-level instructions (shared across all components)
├── frontend/
│   └── CLAUDE.md      # Frontend-specific instructions
├── backend/
│   └── CLAUDE.md      # Backend-specific instructions
└── api/
    └── CLAUDE.md      # API-specific instructions
```

### Scenario 1: Running Claude Code from the Root Directory

`/mymonorepo/`에서 Claude Code를 실행하는 경우:

```bash
cd /mymonorepo
claude
```

| File | Loaded at Launch? | Reason |
|------|-------------------|--------|
| `/mymonorepo/CLAUDE.md` | Yes | 현재 작업 디렉터리이기 때문 |
| `/mymonorepo/frontend/CLAUDE.md` | No | `frontend/`의 파일을 읽거나 편집할 때에만 로드됨 |
| `/mymonorepo/backend/CLAUDE.md` | No | `backend/`의 파일을 읽거나 편집할 때에만 로드됨 |
| `/mymonorepo/api/CLAUDE.md` | No | `api/`의 파일을 읽거나 편집할 때에만 로드됨 |

### Scenario 2: Running Claude Code from a Component Directory

`/mymonorepo/frontend/`에서 Claude Code를 실행하는 경우:

```bash
cd /mymonorepo/frontend
claude
```

| File | Loaded at Launch? | Reason |
|------|-------------------|--------|
| `/mymonorepo/CLAUDE.md` | Yes | 상위(조상) 디렉터리이기 때문 |
| `/mymonorepo/frontend/CLAUDE.md` | Yes | 현재 작업 디렉터리이기 때문 |
| `/mymonorepo/backend/CLAUDE.md` | No | 디렉터리 트리의 다른 가지에 있음 |
| `/mymonorepo/api/CLAUDE.md` | No | 디렉터리 트리의 다른 가지에 있음 |

### Key Takeaways

1. **조상은 항상 시작 시점에 로드됩니다** — Claude는 디렉터리 트리를 위로 거슬러 올라가며 발견하는 모든 CLAUDE.md를 로드합니다. 이를 통해 루트 수준의 저장소 전역 지침에 항상 접근할 수 있습니다.

2. **자손은 지연 로딩됩니다** — 하위 디렉터리의 CLAUDE.md 파일은 해당 하위 디렉터리의 파일과 상호작용할 때에만 로드됩니다. 이로써 관련 없는 컨텍스트가 세션을 비대하게 만드는 것을 방지합니다.

3. **형제(sibling)는 절대 로드되지 않습니다** — `frontend/`에서 작업 중이라면 `backend/CLAUDE.md`나 `api/CLAUDE.md`는 컨텍스트로 로드되지 않습니다.

4. **전역 CLAUDE.md** — 홈 폴더의 `~/.claude/CLAUDE.md`에도 CLAUDE.md를 둘 수 있으며, 이는 프로젝트와 무관하게 모든 Claude Code 세션에 적용됩니다.

### Why This Design Works for Monorepos

- **공유 지침이 아래로 전파됩니다** — 루트 수준의 CLAUDE.md는 어디에나 적용되는 저장소 전역의 규약, 코딩 표준, 공통 패턴을 담습니다.

- **컴포넌트별 지침은 격리된 상태로 유지됩니다** — 프런트엔드 개발자는 백엔드 전용 지침이 컨텍스트를 어지럽힐 필요가 없고, 그 반대도 마찬가지입니다.

- **컨텍스트가 최적화됩니다** — 자손 CLAUDE.md 파일을 지연 로딩함으로써 Claude Code는 시작 시점에 수백 킬로바이트에 달할 수 있는 관련 없는 지침을 로드하지 않습니다.

### Best Practices

1. **공유 규약은 루트 CLAUDE.md에 두세요** — 코딩 표준, 커밋 메시지 형식, PR 템플릿, 기타 저장소 전역 가이드라인.

2. **컴포넌트별 지침은 컴포넌트 CLAUDE.md에 두세요** — 프레임워크별 패턴, 컴포넌트 아키텍처, 해당 컴포넌트에 고유한 테스트 규약.

3. **개인 취향은 CLAUDE.local.md에 두세요** — 팀과 공유하지 않아야 할 지침이라면 `.gitignore`에 추가하세요.

---

## Sources

- [Claude Code Documentation - How Claude Looks Up Memories](https://code.claude.com/docs/en/memory#how-claude-looks-up-memories)
- [Boris Cherny on X - Clarification on CLAUDE.md Loading](https://x.com/bcherny/status/2016339448863355206)
- [Humanlayer - Writing a good Claude.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
