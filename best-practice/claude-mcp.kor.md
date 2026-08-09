<!--
  이 문서는 best-practice/claude-mcp.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# MCP Servers Best Practice

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar%2002%2C%202026%2012%3A30%20PM%20PKT-white?style=flat&labelColor=555)<br>
[![Implemented](https://img.shields.io/badge/Implemented-2ea44f?style=flat)](../.mcp.json)

MCP(Model Context Protocol) 서버는 외부 도구, 데이터베이스, API와의 연결을 통해 Claude Code를 확장합니다. 이 가이드는 일상적으로 사용하기 좋은 추천 서버와 설정 모범 사례를 다룹니다.

<table width="100%">
<tr>
<td><a href="../">← Back to Claude Code Best Practice</a></td>
<td align="right"><img src="../!/claude-jumping.svg" alt="Claude" width="60" /></td>
</tr>
</table>

---

## MCP Servers for Daily Use

> *"더 많으면 더 좋을 것이라 생각하고 MCP 서버를 15개나 과하게 붙였습니다. 결국 매일 쓰는 건 4개뿐이었죠."* — [r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/) (682 upvotes)

| MCP Server | What It Does | Resources |
|------------|-------------|-----------|
| [**Context7**](https://github.com/upstash/context7) | 최신 라이브러리 문서를 컨텍스트로 가져옵니다. 오래된 학습 데이터에서 비롯된 API 환각을 방지합니다 | [Reddit: "by far the best MCP for coding"](https://reddit.com/r/mcp/comments/1qarjqm/) · [npm](https://www.npmjs.com/package/@upstash/context7-mcp) |
| [**Playwright**](https://github.com/microsoft/playwright-mcp) | 브라우저 자동화 — UI 기능을 자율적으로 구현·테스트·검증합니다. 스크린샷, 페이지 이동, 폼 테스트 | [Reddit: essential for frontend](https://reddit.com/r/mcp/comments/1m59pk0/) · [Docs](https://playwright.dev/) |
| [**Claude in Chrome**](https://github.com/nicobailon/claude-code-in-chrome-mcp) | Claude를 실제 Chrome 브라우저에 연결합니다 — 콘솔, 네트워크, DOM을 검사합니다. 사용자가 실제로 보는 것을 디버깅하세요 | [Reddit: "game changer" for debugging](https://reddit.com/r/mcp/comments/1qarjqm/5_mcps_that_have_genuinely_made_me_10x_faster/nza0i7t/) · [Comparison Report](../reports/claude-in-chrome-v-chrome-devtools-mcp.md) |
| [**DeepWiki**](https://github.com/devanshusemwal/deepwiki-mcp) | 모든 GitHub 저장소에 대한 구조화된 위키 형식 문서를 가져옵니다 — 아키텍처, API 표면, 관계 | [Reddit: "put it behind a gateway with Context7"](https://reddit.com/r/mcp/comments/1qarjqm/) |
| [**Excalidraw**](https://github.com/antonpk1/excalidraw-mcp-app) | 프롬프트로부터 아키텍처 다이어그램, 순서도, 시스템 설계를 손으로 그린 듯한 Excalidraw 스케치로 생성합니다 | [GitHub](https://github.com/antonpk1/excalidraw-mcp-app) |

Research (Context7/DeepWiki) -> Debug (Playwright/Chrome) -> Document (Excalidraw)

---

## Configuration

MCP 서버는 프로젝트 루트의 `.mcp.json`(프로젝트 범위) 또는 `~/.claude.json`(사용자 범위)에서 설정합니다.

### Server Types

| Type | Transport | Example |
|------|-----------|---------|
| **stdio** | 로컬 프로세스를 생성 | `npx`, `python`, binary |
| **http** | 원격 URL에 연결 | HTTP/SSE endpoint |

### Example `.mcp.json`

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    },
    "deepwiki": {
      "command": "npx",
      "args": ["-y", "deepwiki-mcp"]
    },
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp"
    }
  }
}
```

`.mcp.json`에 API 키를 직접 커밋하지 말고, 비밀 값에는 환경 변수 확장을 사용하세요:

```json
{
  "mcpServers": {
    "remote-api": {
      "type": "http",
      "url": "https://mcp.example.com/mcp?token=${MCP_API_TOKEN}"
    }
  }
}
```

### Settings for MCP Servers

`.claude/settings.json`의 다음 설정들이 MCP 서버 승인을 제어합니다:

| Key | Type | Description |
|-----|------|-------------|
| `enableAllProjectMcpServers` | boolean | 프롬프트 없이 모든 `.mcp.json` 서버를 자동 승인 |
| `enabledMcpjsonServers` | array | 자동 승인할 특정 서버 이름의 허용 목록 |
| `disabledMcpjsonServers` | array | 거부할 특정 서버 이름의 차단 목록 |

### Permission Rules for MCP Tools

MCP 도구는 권한 규칙에서 `mcp__<server>__<tool>` 명명 규칙을 따릅니다:

```json
{
  "permissions": {
    "allow": [
      "mcp__*",
      "mcp__context7__*",
      "mcp__playwright__browser_snapshot"
    ],
    "deny": [
      "mcp__dangerous-server__*"
    ]
  }
}
```

---

## MCP Scopes

MCP 서버는 세 가지 수준에서 정의할 수 있습니다:

| Scope | Location | Purpose |
|-------|----------|---------|
| **Project** | `.mcp.json` (repo root) | 팀 공유 서버, git에 커밋됨 |
| **User** | `~/.claude.json` (`mcpServers` key) | 모든 프로젝트에 걸친 개인 서버 |
| **Subagent** | Agent frontmatter (`mcpServers` field) | 특정 서브에이전트로 범위가 한정된 서버 |

우선순위: Subagent > Project > User

---

## Sources

- [MCP Servers — Claude Code Docs](https://code.claude.com/docs/en/mcp)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [5 MCPs that have genuinely made me 10x faster — r/mcp](https://reddit.com/r/mcp/comments/1qarjqm/)
- [MCP Server Overload Discussion — r/mcp](https://reddit.com/r/mcp/comments/1mj0fxs/)
