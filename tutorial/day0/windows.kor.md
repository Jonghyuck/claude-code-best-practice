<!--
  이 문서는 tutorial/day0/windows.md의 한글 번역본입니다. 원본(영문)이 source of truth이며,
  구조·배지·링크·코드·앵커는 원본과 동일하게 보존하고 산문만 번역했습니다.
  동기화 규칙: .claude/rules/korean-sync.md
-->

# Windows Setup

[Back to Day 0](README.md)

---

**Node.js**
- [nodejs.org](https://nodejs.org)로 이동합니다
- **"Download Node.js (LTS)"** 버튼을 클릭합니다 — `.msi` 설치 파일이 다운로드됩니다
- `.msi` 파일을 실행하고 마법사에서 **Next**를 눌러 진행합니다
- 기본값을 그대로 두고 **Install**을 클릭한 뒤 완료될 때까지 기다립니다

**Verify Node.js**
- **새** 터미널(PowerShell 또는 Windows Terminal)을 열고 다음을 실행합니다:
  ```powershell
  node --version
  npm --version
  ```

**Claude Code**
- ```powershell
  npm install -g @anthropic-ai/claude-code
  ```
- 권한 오류가 발생하면 터미널을 **관리자 권한**으로 실행합니다(우클릭 > 관리자 권한으로 실행)

**Verify**
- ```powershell
  claude --version
  ```

---

이제 인증 설정을 위해 [README.md](README.md)로 다시 돌아갑니다.
