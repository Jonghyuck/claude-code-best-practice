<!--
  이 문서는 tutorial/day0/mac.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# macOS Setup

[Back to Day 0](README.md)

---

**Terminal**
- 터미널을 엽니다 (`Cmd + Space`를 누르고 "Terminal"을 입력한 뒤 Enter를 누릅니다)

**Homebrew**
- Homebrew가 이미 설치되어 있는지 확인합니다:
  ```bash
  brew --version
  ```
- "command not found"가 표시되면 먼저 Homebrew를 설치합니다:
  ```bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```

**Claude Code**
- ```bash
  brew install --cask claude-code
  ```

**Verify**
- ```bash
  claude --version
  ```

---

이제 인증 설정을 위해 [README.md](README.md)로 돌아갑니다.
