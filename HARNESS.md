# DOSKO 웹 리뉴얼 프로젝트 하네스 (Harness & Engineering Guide)

## 프로젝트 개요
- **위치**: `c:\Users\song9\OneDrive\바탕 화면\wab\dosko\dosko_new`
- **목적**: (주)도스코 공식 홈페이지 신규 반응형 웹 리뉴얼 (한/영 다국어, 제품 상세 모달, 공정도, 품질인증)
- **배포 타겟**: Cloudflare Pages / GitHub Pages

---

## 1. 세션 핵심 성과
1. **레거시(GnuBoard) 사이트의 완벽한 모던화**:
   - `dosko2` 원본 에셋, 공정 사진, 인증서 8종, 강종 7종 스펙을 100% 보존하면서 Tailwind CSS 기반 반응형 UI로 전면 재구축.
2. **다국어(i18n) 시스템 완성**:
   - `translations.ko` / `translations.en`을 통해 단 한 줄의 번역 누락 없이 한/영 전환 및 `<html lang>` / `document.title` 동기화.
3. **사용자 요청 맞춤 튜닝**:
   - 팩스 번호 우선순위 분기 (국내 0507, 해외 3237).
   - 대표이사 인사말 및 고해상도 이미지 처리.
   - 4단계 정밀 공정도 탭 (1단계: 압연·소둔·절단, 2단계: 열처리, 3단계: 연마 기계 2기 화살표 제거, 4단계: 출하).
   - 히어로 3줄 헤드라인 구조화 (특수마대강 / 고품질 / 국내).
   - 하단 푸터 로고 다크 테마 벡터화.

---

## 2. 세션 중 발생한 실수 및 교훈 (Mistakes & Lessons Learned)

### 실수 1: PowerShell 인라인 스크립트 실행 시 특수문자(`&`, `$`, `:` 등) 구문 에러
- **현상**:
  - `powershell -Command "..."` 또는 `node -e "..."` 실행 중 인라인 문자열 안에 포함된 `&` (Ampersand), `$`, 변수 콜론(`${x}:`)이 PowerShell 파서에 걸려 `AmpersandNotAllowed` 구문 오류로 중단됨.
- **해결**:
  - 복잡한 텍스트 변환이나 정규식 처리는 인라인 명령어로 실행하지 않고, 독립된 Python 스크립트(`scratch/apply_changes.py`) 파일로 생성한 뒤 실행하여 안전하게 처리함.
- **가드레일**:
  - **규칙**: 다국어 텍스트나 HTML 코드 치환 시 PowerShell 명령줄에 문자열을 직접 넣지 말고, 반드시 `.py` 또는 `.js` 스크립트 파일을 작성해 실행한다.

### 실수 2: `edit_file` 도구의 미세 공백/들여쓰기 불일치로 인한 치환 실패
- **현상**:
  - `edit_file` 도구 사용 시 원본 파일의 들여쓰기 공백이나 줄바꿈(`\r\n` vs `\n`)이 조금이라도 어긋나면 `target content not found in the file` 오류 발생.
- **해결**:
  - Python `re` 또는 정확한 파일 읽기/쓰기를 통해 안전하게 타겟을 교체함.
- **가드레일**:
  - **규칙**: 대규모 HTML 블록 수정 시 파이썬 파일 I/O 스크립트를 작성하여 안전하게 UTF-8로 저장한다.

---

## 3. GitHub 연동 및 배포 가이드

### GitHub 저장소 생성 및 푸시 방법
1. **GitHub CLI 로그인**:
   ```bash
   gh auth login
   ```
   (브라우저 인증 선택 후 코드 입력)
2. **저장소 생성 및 푸시 (자동)**:
   ```bash
   cd "c:\Users\song9\OneDrive\바탕 화면\wab\dosko\dosko_new"
   git init
   git add .
   git commit -m "feat: DOSKO renewal website release ready for Cloudflare Pages"
   gh repo create dosko-renewal --public --source=. --remote=origin --push
   ```
3. **Cloudflare Pages 연동**:
   - Cloudflare 대시보드 -> **Workers & Pages** -> **Create application** -> **Pages** -> **Connect to Git** 선택 후 생성된 `dosko-renewal` 저장소 연결.
   - Build command: (공백)
   - Build output directory: `.` (또는 루트)
