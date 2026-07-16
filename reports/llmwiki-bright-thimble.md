# 계획: 업로드한 LLMWIKI Obsidian vault를 새 GitHub 저장소로 구성

## Context (왜 하는가)

사용자가 개인 지식관리 Obsidian vault(`_00.LLMWIKI.zip`, 6.9MB)를 업로드하고, 이를
**새로운 GitHub 저장소로 구성**해 달라고 요청했다. 목적은 이 개인 위키를 git으로
버전 관리·백업·기기 동기화하기 위함이다.

vault에는 **민감한 개인정보**가 포함되어 있어(개인신상·재무·건강 관련 대화 아카이브,
`민감·개인정보.md`, 백업 zip), 공개 범위와 업로드 범위를 먼저 확정했다.

### 확정된 사용자 결정
- **저장소 이름**: `LLMWIKI`
- **소유 계정**: `Jonghyuck` (인증된 GitHub 계정, get_me로 확인)
- **공개 범위**: **Private (비공개)**
- **업로드 범위**: **전체 그대로** (민감 아카이브·백업 zip·`.obsidian` 포함, 제외 없음)

## 대상 데이터 (확인 완료)
- 총 59개 파일, 약 12MB (압축 해제 기준)
- 구성: `.md` 47, `.json` 5(Obsidian 설정), `.py` 2(변환 스크립트), `.zip` 2(백업),
  `.jsonl` 1, `.base` 1(Obsidian Bases), `.ttxfolder` 1
- 최대 파일: `06_아카이브/클로드-대화/...git·GitHub·MCP 학습 [33aa553d].jsonl` (4.75MB)
- **모든 파일이 GitHub 한도(50MB 경고/100MB 차단) 미만** → 일반 git push로 안전, LFS 불필요
- 압축 해제 위치(작업용):
  `<scratchpad>/llmwiki/_00.LLMWIKI/` — 이 폴더의 **내용물**이 저장소 루트가 됨
  (Obsidian이 저장소 폴더를 vault 루트로 바로 열 수 있도록 `_00.LLMWIKI` 래퍼는 벗김)

## 접근 방식 (선택 이유 포함)

바이너리 `.zip` 2개가 포함되어, 텍스트 content 문자열만 받는 GitHub API 방식
(`mcp__github__push_files`)은 zip을 손상시킬 위험이 있다. 따라서 **실제 git clone →
파일 복사 → commit → push** 방식을 사용한다(바이너리 무손실).

### 실행 단계
1. **저장소 생성**: `mcp__github__create_repository`
   - `name: "LLMWIKI"`, `private: true`, `autoInit: false`
   - (autoInit=false로 두어 첫 push가 깨끗한 초기 커밋이 되도록)
2. **세션 스코프 편입 + 프록시 URL 확보**: `add_repo(owner="Jonghyuck", repo="LLMWIKI")`
   - 반환되는 clone 명령의 프록시 remote(`http://local_proxy@127.0.0.1:.../git/Jonghyuck/LLMWIKI`) 사용
   - clone 후 `register_repo_root`로 등록
3. **로컬 구성** (scratchpad 내 작업, 사용자 기존 저장소 `claude-code-best-practice`는 절대 건드리지 않음):
   - 빈 LLMWIKI를 scratchpad로 clone
   - `_00.LLMWIKI/`의 **모든 내용물**(숨김 파일 `.obsidian/`, `.ttxfolder`, `.base` 포함)을
     clone된 저장소 루트로 복사
4. **커밋**: 외부 vault 초기 임포트이므로 **단일 커밋** "Initial import of LLMWIKI vault"로 스냅샷.
   - 참고: 이 프로젝트 CLAUDE.md의 "파일별 개별 커밋" 규칙은 *best-practice 저장소*의
     변경분에 대한 규칙이며, 별도 저장소로의 59개 파일 일괄 초기 임포트에는 단일 커밋이 관례적·합리적.
5. **push**: `git push -u origin main` (기본 브랜치 `main`).
   - 네트워크 오류 시에만 지수 백오프(2s·4s·8s·16s)로 최대 4회 재시도.
6. **PR은 만들지 않음** (사용자가 요청하지 않았고, 새 저장소 초기 임포트라 불필요).

### 다루지 않는 것 / 유지 결정
- 파일 제외 없음(사용자 "전체 그대로"). `.gitignore` 강제 추가 안 함.
  - (선택) 향후 `.obsidian/workspace*.json`(기기별 UI 상태)은 gitignore 후보임을 안내만 함.
- 파일명에 한글·공백·특수문자(·, [], 중점) 포함 → git UTF-8로 정상 처리, 별도 조치 불요.

## 남는 리스크 / 주의
- **민감정보**: Private이지만 `06_아카이브/클로드-대화/`의 사적·건강·재무 대화와
  `민감·개인정보.md`가 저장소에 포함된다(사용자가 "전체 그대로" 명시 선택). 한 번 push되면
  git 히스토리에 남으므로, 추후 특정 파일 제외를 원하면 히스토리 재작성이 필요함을 인지.
- **프록시 스코프**: 새 저장소 push는 2단계 `add_repo`로 세션 스코프에 편입해 해결.
  만약 프록시가 새 저장소 push를 거부하면, 텍스트 57개는 `push_files`로 올리고 zip 2개만
  별도 처리하는 대안으로 전환(마지막 수단).

## 검증 (Verification)
1. `mcp__github__search_repositories` 또는 저장소 조회로 `Jonghyuck/LLMWIKI`가
   **private=true**로 생성되었는지 확인.
2. `git ls-remote origin` 또는 `mcp__github__get_file_contents`로 대표 파일 존재 확인:
   - `CLAUDE.md`, `00_타임라인.md`, `card/원칙노트/P001 카드 시스템 선언.md`,
     `06_아카이브/backup_2026-07-08_세션마무리_전체.zip`(바이너리 무손실 확인용)
3. 파일 개수 대조: 로컬 59개 == 원격 트리 blob 개수(±Obsidian 설정 파일).
4. push 결과(커밋 SHA, 원격 브랜치 `main`)를 사용자에게 보고하고 저장소 URL 제공:
   `https://github.com/Jonghyuck/LLMWIKI`
