# 화재조사 2D 현장복원도 작성 도구

경상북도 소방서 화재조사관(사용자)을 위한 2D 현장복원도(평면도) 작성 툴.
완성 후 도내 다른 조사관들에게도 공유할 예정.

## 절대 지켜야 할 기술 제약
- **단일 파일**: `index.html` 하나가 전부. 빌드 과정 없음. 더블클릭하면 바로 실행되어야 함.
- **완전 오프라인 동작이 기본값**: 업무용 PC가 망분리 환경. 외부 CDN에 의존하는 코드를
  기본 경로에 절대 넣지 말 것. (예외: "확장 라이브러리" 기능은 URL/로컬 파일로 심볼팩을
  선택적으로 불러오는 기능인데, 실패해도 내장 아이콘으로 조용히 폴백하도록 만들어져 있음 —
  이 패턴을 벗어나는 CDN 의존은 추가하지 말 것.)
- **백엔드 없음**. 저장/불러오기는 JSON 파일 기반, 이미지 저장은 PNG/PDF 내보내기.
- 파일명은 **`index.html`** (원래 `fire-scene-2d.html`이었으나 사용자가 GitHub 웹에서
  직접 `index.html`로 이름을 바꿈 — GitHub Pages 등을 염두에 둔 것으로 보임. 계속 이 이름
  유지할 것).

## 저장소
- GitHub: https://github.com/fulva88/fire-investigation-2d-tool (public, 계정 fulva88)
- **커밋/푸시는 사용자가 명시적으로 요청할 때만.** 매 패치마다 자동으로 커밋하지 않음.
- 커밋 전 항상 `git fetch` + `git status`로 원격에 예상 못한 변경(사용자가 GitHub 웹에서
  직접 건드렸을 수 있음)이 있는지 먼저 확인할 것. 과거 한 번 파일명이 원격에서 바뀌어 있어서
  로컬 작업을 백업 후 재적용해야 했던 적이 있음.
- `v0/fulva88-...` 형태의 브랜치가 원격에 보일 수 있는데(아마 v0.dev 같은 외부 연동이 만든
  것), master와 무관하니 따로 손대지 않아도 됨.

## 버전 관리 & 패치노트 (사용자가 정한 규칙)
- 버전 문자열은 스크립트 최상단 `const APP_VERSION = '...'` 하나로 관리. 화면 좌측 상단
  타이틀 옆과 PNG/PDF 내보내기 범례 하단에 작게 표시됨.
- **버전 증가 규칙**: 기능 단위의 정식 패치는 0.1씩 (1.0 → 1.1), 그 사이에 깜빡했던 것을
  추가하거나 사소한 보완을 하는 "서브패치"는 소수점 둘째자리로 (1.1 → 1.11 → 1.12).
  사용자가 상황에 따라 직접 버전 번호를 지정하기도 하니 지시받은 번호를 그대로 쓸 것.
- **패치마다 반드시 두 가지를 함께 갱신**:
  1. `patch-notes/vX.Y.txt` — 사람이 메모장으로 읽는 간단한 텍스트 기록 (한국어, 무엇을
     왜 바꿨는지 짧게). 버전마다 새 파일.
  2. `index.html` 안의 `PatchNotes.entries` 배열 (17번 섹션, "PATCH NOTES POPUP") 끝에
     `{ version, date, items:[...] }` 항목 추가. 이게 실제로 앱 실행 시 뜨는 팝업 내용.
- 패치노트 팝업 동작: 새 버전을 열면 최소 한 번은 반드시 뜸. "오늘 하루 보지 않기"를
  체크하고 닫으면 그날 하루만 억제(localStorage 날짜 비교), 체크 안 하면 다음 로드 때 또 뜸.
  새 버전이 나오면 이전 억제 설정과 무관하게 다시 뜸. "이전 업데이트 내역 보기"로 과거
  버전 기록 누적 열람 가능.
- 날짜는 실제 오늘 날짜를 쓸 것 (시스템 reminder의 currentDate 확인).

## 코드 구조 (index.html 안, 섹션 번호 주석으로 구분되어 있음)
1. CONFIG — PX_PER_M(축척), 줌 범위
2. UTIL — escapeHtml 등 소소한 헬퍼
3. THEME — 라이트/다크 팔레트, `Theme.isDark()`. **내보내기(PNG/PDF)는 항상 라이트 고정
   팔레트(`Theme.palettes.light`)를 써서 화면 테마와 무관하게 보고서가 일관되게 나오도록
   되어 있음** — 이 분리를 깨지 말 것.
4. SYMBOL LIBRARY — `reg()`(수작업 draw 함수) / `regShape()`(rect/circle/line/path 스펙
   기반, 확장팩과 동일 포맷) 두 가지 등록 방식. 카테고리: 화재조사, 소방시설, 전기/가전
   (하위 폴더 "가전제품"), 가구(하위 폴더 "가구 프리셋"), 건축 요소, 기타(부지/외부).
   `numbered:true`인 심볼(증거위치/촬영위치/소사자)은 배치 순서대로 자동 번호, 삭제 시
   번호 재정리, 라벨 직접 수정하면 `autoLabel:false`로 바뀌어 자동 갱신 대상에서 빠짐.
   `snapToWall:true`인 심볼(문/창문)은 벽 근처로 드래그하면 벽 각도에 자동 스냅.
5. STATE — `AppState` (walls/symbols/wires/arrows/roads/machines 등 모든 도면 데이터,
   메타정보, 각종 카운터). `reset()/serialize()/loadFrom()` 셋을 항상 같이 맞춰야 함 —
   새 배열/카운터를 추가하면 이 세 곳 + History 스냅샷까지 빠짐없이 반영할 것.
6. HISTORY — undo/redo. `savedIndex`로 "저장 안 된 변경사항 있음" 추적 →
   beforeunload 경고 + 저장 버튼 점(●) 표시에 사용됨.
7. GEOMETRY UTIL — `Geo` 객체. 3점 아크(곡면 벽) 관련 원호 계산도 여기 포함
   (`circumcircle`, `angleInSweep`, `arcSweep`, `arcMidpoint`, `distToArc`).
8. NUMBERING / WALL-SNAP HELPERS
9. RENDERER — 화면 렌더(`render()`)와 내보내기 렌더(`renderExportCanvas()`)가 분리되어
   있음. 새 오브젝트 타입을 추가하면 **양쪽 다** 그려주는 코드를 넣어야 export에 반영됨.
   또한 `computeBounds()`/`fitToContent()`에도 새 타입의 좌표를 포함시켜야 화면맞춤/내보내기
   범위가 잘리지 않음.
10. CURSORS — 커서는 OS 기본이 아니라 테마색을 반영한 커스텀 SVG data URI
    (`cursorDataUri`)를 씀. 다크모드에서 흰색, 라이트모드에서 검은색.
11. TOOL MANAGER — 도구 전환, 도구별 힌트 텍스트, 커서. 화살표(연소확대방향/촬영방향)와
    도로(도로/비포장도로)처럼 "한 도구, 여러 kind"인 경우 `startArrowTool(kind)` /
    `startRoadTool(kind)` 패턴을 씀 (버튼은 `data-arrow`/`data-road` 속성으로 구분,
    `data-tool` 제네릭 바인딩과는 별도로 직접 이벤트 리스너 연결).
12. CANVAS INPUT HANDLING — pointerdown/move/up, 키보드 단축키. 벽 이어그리기, 곡면 벽
    3클릭(시작→끝→통과점), 도로/배선 체인 그리기, 기계설비/화살표 드래그 생성 등 도구별
    상태머신이 전부 여기.
13. TEMPLATES — 사각형/ㄱ자 건물 프리셋.
14. FILE IO — JSON 저장/불러오기, PNG(타임스탬프 파일명 자동 부여로 캐시 혼동 방지),
    PDF(브라우저 인쇄 기능 활용, 별도 라이브러리 없음).
15. EXTENSION LOADER — 심볼팩 URL/로컬파일 불러오기. `SymbolLibrary.registerPack()`이
    `regShape()`와 동일한 데이터 포맷을 기대함.
16. MODAL — 사각형/ㄱ자 템플릿 치수 입력용 범용 모달.
17. PATCH NOTES POPUP — 위 "버전 관리" 절 참고.
18. UI — 나머지 전부(팔레트 빌드, 툴바 바인딩, 속성 패널, 테마 토글 등)를 묶는 마지막
    모듈. `UI.init()`이 진입점, `DOMContentLoaded`에서 호출.

## 지금까지 사용자가 명확히 밝힌 취향/피드백
- **"너무 복잡하게 갈 필요 없다"**를 여러 번 강조. 곡면 벽도 스플라인 대신 3점 아크로
  단순하게 구현한 것이 그 예. 새 기능 제안 시 과설계하지 말고 실용적인 최소 구현으로.
- 심볼/기능 추가 요청은 보통 "화재조사관에게 실제로 쓸모 있는가"를 기준으로 판단해서
  진행해도 됨 (예: 소방시설 카테고리, 기계설비 사각형은 요청받지 않은 부분도 먼저 제안해서
  좋은 반응을 얻었음).
- 자극적/그래픽적인 표현은 피하고 전문적·차분한 톤 유지 (예: 소사자 마커는 빨간색 대신
  무채색 픽토그램으로).
- 매 패치 후 **반드시 브라우저에서 실제로 동작을 검증**하고 나서 완료 보고할 것 (아래 참고).

## 테스트 방법 (Claude Code 환경 — 실제 브라우저로 직접 열 수 없음)
이 프로젝트는 서버가 없는 순수 정적 파일이라, 매 패치마다 아래 절차로 실제 동작을 검증해왔음:
1. `index.html`을 스크래치패드 등 ASCII 경로로 복사 (한글 경로에서 PowerShell 스크립트가
   깨지는 문제가 있었음).
2. 최소 정적 파일 서버(PowerShell `HttpListener` 기반 스크립트, 매번 재작성해도 됨)를
   `run_in_background`로 실행.
3. `.claude/launch.json`에 `{"configurations":[{"name":"...", "url":"http://localhost:PORT"}]}`
   형태로 그 서버 주소를 등록하고, Browser 프리뷰 도구로 `/index.html` 열기.
4. `javascript_tool`로 `PointerEvent`를 직접 dispatch해서 도구 클릭→드래그→완료 흐름을
   시뮬레이션하고, `AppState`/`Renderer.render()`/`Renderer.renderExportCanvas()`가 에러
   없이 동작하는지, 계산 결과(좌표/카운터/라벨 등)가 기대값과 맞는지 확인.
5. 테스트 끝나면 서버 프로세스 종료하고 `.claude/launch.json`은 삭제 (임시 파일이라 커밋
   대상 아님).

## 기타
- `.gitignore`에 OS 잡파일(Thumbs.db 등) 정도만 있음.
- CRLF 관련 git 경고("LF will be replaced by CRLF")는 무시해도 무방 (Windows 개행 정규화,
  실제 문제 아님).
