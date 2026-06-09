# 대전 한화생명 볼파크 — AV 시스템 설계 프로젝트

## 프로젝트 위치
`/Users/gisancityboy/Downloads/hanwha-ballpark-project/index.html`

## 작업 개요
한화생명 볼파크 AV 시스템 설계 포트폴리오 HTML 단일 파일. 편집 기능 내장.

---

## 현재 파일 상태
- **index.html** = 항상 최신 소스. 사용자가 다운받은 최신 `hanwha-ballpark(N).html`을 복사해서 덮어씌우는 방식으로 최신화함.
- 파일 크기 약 170MB (영상 base64 포함)
- 현재 삽입된 이미지: 음향 배너(w-audio), 영상 배너(w-video), 공연인프라 배너(w-perf), 통합제어 갤러리 c1~c5
- 현재 삽입된 영상: 히어로 영상(hero-video, Enscape_2026_converted.mp4 28.6MB→38MB base64), 인프라 시스템 영상(infra-video, 한화_converted.mp4 42.9MB→57MB base64)

---

## 주요 기능 구현 현황

### 영상 (히어로 배너)
- `<video id="hero-video">` 로컬 파일 업로드 지원
- **IndexedDB**에 base64로 저장 → 브라우저 재열어도 복원
- 재생은 blob URL로 (DOM에 data URL 직접 저장 안 함)
- **💾 HTML 저장** 버튼: `showSaveFilePicker` (File System Access API) 사용
  - 저장 시 IndexedDB에서 영상 꺼내 base64로 HTML에 포함
  - Chrome 전용 (Safari/Firefox 미지원)
- **⚠️ USB 다른 PC 이전 시**: IndexedDB 없어도 src에 data URL 있으면 자동 재생되도록 픽스 적용됨
  - `dbLoad` else 분기에서 `vid.getAttribute('src').startsWith('data:')` 체크 후 `vid.load()` + `vid.play()`
- **저장 시 중복 src 방지**: saveHtml()에서 정규식으로 기존 src 제거 후 새 src 삽입

### 이미지
- 이미지 슬롯 클릭 시 `<label for="img-picker">` 방식으로 파일탐색기 직접 오픈 (onclick 방식 제거)
- `<input id="img-picker">` 가 `</body>` 직전에 고정 존재
- HTML 저장 시 base64로 포함됨 → 다른 PC에서도 보임
- **이미지 직접 삽입**: 사용자가 이미지 파일 주면 Python으로 base64 변환 후 해당 슬롯 img src에 직접 삽입 가능
  ```python
  import base64, re
  with open('이미지.png','rb') as f: b64 = base64.b64encode(f.read()).decode()
  # content에서 id="슬롯-img" src="" 찾아서 교체
  ```

### 갤러리 프레임
- 음향: a1~a4
- 영상: v1~v4
- 네트워크: n1~n4
- **통합제어: c1~c7** (기본 4개 + 3개 추가함)
- **공연인프라: p1~p4** (2026-06-05 추가, 탭 ID: p-perf, 배너: w-perf/perf-img)
- **인프라 시스템: i1~i4** (2026-06-06 추가, 탭 ID: p-infra, 영상: infra-video)

---

## 토폴로지 다이어그램

### 색상 기준 (전체 공통)
- 🔵 파란선 `rgba(96,165,250,.75)` → **광케이블**
- 🟣 보라선 `rgba(167,139,250,.6~.8)` → **랜선**

### Audio Network (링 토폴로지)
- 노드: 1st AR ~ 5th AR + Main Control Room (MCR)
- 엣지: AR끼리 링 연결 → 파란선 (광케이블)
- MCR ↔ 3rd AR → 보라선 (랜선)

### Control Network (메쉬)
- 노드: 1st AR ~ 5th AR + Main Control Room (MCR)
- 엣지: 전부 파란선 (광케이블)

### Video Network (스타 토폴로지)
노드 및 연결:
- MCR → 1st AR, 3rd AR, 5th AR (파란선, 광케이블)
- 3rd AR → 1F Ribbon LED, 3F Ribbon LED (보라선, 랜선)
- 3rd AR → Relay Box (파란선, 광케이블)
- 5th AR → Dugout (파란선, 광케이블)
- 5th AR → Main LED (보라선, 랜선)
- 1st AR → 응원석 Infra (보라선, 랜선)

---

## 영상 변환 (코덱 이슈)
브라우저는 H.264 mp4만 안정 지원. 변환 명령:
```bash
avconvert --source "원본.mp4" --preset Preset1920x1080 --output "변환.mp4" --replace --progress
```
- 변환된 파일: `Downloads/Enscape_converted.mp4`, `Downloads/탈코_converted.mp4`

---

## index.html 최신화 방법
사용자가 브라우저에서 💾 저장 후 다운로드한 파일을 index로 복사:
```bash
cp "/Users/gisancityboy/Downloads/hanwha-ballpark(N).html" "/Users/gisancityboy/Downloads/hanwha-ballpark-project/index.html"
```

---

## 자주 쓰는 작업
- **토폴로지 노드/엣지 수정**: index.html에서 `TW_POS`, `TW_EDGES`, tw-diagram div 수정
- **갤러리 프레임 추가**: `IM` 맵 업데이트 + HTML에 `.gi` div 추가
- **대용량 파일 수정**: Python으로 문자열 치환 (grep/sed는 출력 용량 초과 위험)
- **최신화**: `ls -t Downloads/hanwha-ballpark*.html | head -1` 로 최신 파일 확인 후 cp
- **이미지 삽입**: 사용자가 파일 주면 Python base64 변환 후 img src에 직접 삽입
- **이미지 위치 조정**: img 태그에 `object-fit:cover; object-position:center XX%;` 로 조절

---

## 버그 수정 이력 (2026-06-05)
- **영상 USB 이전 후 재생 안 됨**: IndexedDB 없는 PC에서 data URL src 감지 후 자동 재생 로직 추가
- **저장 시 영상 데이터 중복**: saveHtml() 정규식으로 기존 src 제거 후 삽입, 중복 158MB 제거
- **이미지 클릭 파일탐색기 안 열림**: onclick→label 방식으로 전환 (opacity 트랜지션 중 클릭 차단 문제)
- **탭 전환 시 화면 아래로 스크롤**: `scrollIntoView` 제거
- **패널 열릴 때 클릭 불가**: `.dpw` opacity 트랜지션 제거, max-height만 사용 (9000px으로 확장)
- **공연인프라 탭 추가**: p-perf 패널, w-perf 배너, p1~p4 갤러리, 탭 그리드 5열로 확장
- **토폴로지 애니메이션**: twDrawLine에 flowDash 애니메이션 추가 (베이스 반투명선 + 흐르는 dash 오버레이)
- **각 탭 텍스트 수정**: 음향(대주제/주석), 영상(대주제/주석), 공연인프라(대주제/주석) 내용 교체

## 텍스트/UI 수정 이력 (2026-06-05 세션)
- **편집모드 배너 제거**: `<div class="edit-banner">` 삭제
- **회사 로고 위치·크기 조정**: 우상단→좌상단, 로고 32→48px, 회사명 11→16px, 서브텍스트 9→11px
- **네트워크 인프라 탭 텍스트 전면 교체**: h3·p 문구 변경, STP/스타 토폴로지 설명 추가
- **전체 탭 텍스트 가독성 개선**: 음향·영상·통합제어·공연인프라 모든 dt 블록 `<br>`·공백줄 추가
- **통합제어 본문 교체**: Q-SYS 플랫폼 중심 3문단으로 재작성
- **히어로 인트로 문구 변경**: "현장에서만 느낄 수 있는 그 순간 / 우리는 그 순간을 디자인합니다."
- **히어로 인트로 주석 교체**: 볼파크 소개 + Q-SYS·Dante·광케이블 통합 설명으로 교체
- **히어로 인트로 폰트 변경**: DM Serif Display italic → Inter/Pretendard 600
- **히어로 태그 연도**: `2025` → `2026`
- **히어로 메타 글자색**: `.hero-meta .hmi .lb/vl` 흰색(불투명도 조정)·굵기·크기 조정
- **공연인프라 탭 버튼 설명**: `SR 시스템 · 무대 조명 · 공연 연출` → `SR 시스템 · 공연 연출`
- **blockquote 폰트 변경**: DM Serif Display italic → Pretendard/Inter 산세리프
- **히어로 영상 교체**: `Enscape_2026-06-04-20-14-21.mp4` → H.264 변환 후 삽입 (138MB)
  - ffmpeg H.264/High/yuv420p/1080p 변환, 350MB→28.6MB
  - video src 직접 삽입 방식 → `</video>` 직후 즉시실행 fetch→blob 스크립트로 재생
- **음향 시뮬레이션 서브탭 추가**: p-audio 패널 내 7종 (Direct 1k/2k/4k/500~4k/BroadBand, Total BroadBand, STI) — 이미지 포함
- **히어로 영상 교체**: Enscape_2026-06-04 → H.264 변환본 (38.1MB base64)
- **현재 파일 크기**: 365MB

## 구조 수정 이력 (2026-06-06 세션)
- **음향 시뮬레이션 섹션 재구성**: 인용구(pq) 바로 아래 `sim-toggle-btn` 버튼 → 클릭 시 `.sim-body` 토글
  - 7개 pill 탭 버튼: Direct 1k/2k/4k/500-4k/Direct BB/Total BB/STI
  - 이미지: `C:\Users\apdlc\OneDrive\문서\카카오톡 받은 파일\매핑` 폴더 KakaoTalk_20260604_202324460(_base~_06).jpg 7장
  - CSS: `.sim-toggle-btn`, `.sim-body`, `.sim-tabs`, `.sim-tab`, `.sim-panel`, `.sim-img-wrap`
  - JS: `simToggle(btn)`, `simTab(btn, pid)` — `</script>` 바로 앞에 삽입
  - `grid-column:1/-1` 추가로 2열 그리드에서 전체 폭 차지
- **빈 갤러리 프레임 삭제**: wa1~4, wv1~4, wn1~4, wc6~7, wp1~4 (이미지 없는 슬롯) 제거
  - 보존: wc1~wc5 (통합제어, 이미지 있음)
  - 비어있는 dg 섹션(음향/영상/네트워크/공연인프라)도 함께 제거
- **현재 파일 크기**: 366MB

## 구조 수정 이력 (2026-06-06 세션 2차)
- **히어로 영상 교체**: `Enscape_2026_converted.mp4` (H.264, 28.6MB → base64 38.1MB) 재삽입
  - video 태그 src 속성 직접 교체 방식
- **음향 시뮬레이션 결과 버튼 크기 확대**: font-size 11→14px, padding 13px 28px→18px 36px, 배경 반투명, 글자 흰색, 전체폭+양쪽정렬

## 구조 수정 이력 (2026-06-06 세션 추가)
- **시뮬레이션 온도 2종 추가**: 각 주파수 탭에 10°C / 30°C 이미지 2장씩 (총 14장)
  - 10°C: 매핑/_00~_06, 30°C: 매핑/_07~_13
  - 파란 뱃지(t10) / 주황 뱃지(t30) 온도 레이블
- **국내 야구장 음향 설계 기준 표 추가**: sim-body 상단, sim-tabs 위
  - STI ≥0.55 / SPL +10dB / 편차 ±3dB 이내 / 에코 억제
  - CSS: `.sim-std-table`, caption 오렌지, th 오렌지 배경

## 구조 수정 이력 (2026-06-06 세션 3차)
- **탭 이름 변경**: 네트워크 인프라 → 네트워크 시스템, 공연 인프라 → 공연 시스템 (탭 버튼·패널 타이틀·갤러리 라벨)
- **인프라 시스템 탭 추가 (06번)**: 탭 ID `p-infra`, sc-desc: 중계 카메라 패널·방송중계실 패널·중계외함 패널·덕아웃 패널·응원단상 패널
  - `.sn` 그리드 5열 → 6열
  - 갤러리 슬롯 wi1~wi4, IM 맵 i1~i4 등록
- **인프라 시스템 영상 삽입**: `한화.mp4` (316MB) → ffmpeg CRF 26 / 30fps H.264 변환 → `한화_converted.mp4` (42.9MB)
  - base64 57MB, `id="infra-video"` autoplay/muted/loop, 16:9 비율, 패널 상단 배치
- **현재 파일 크기**: 423MB

## 버그 수정 이력 (2026-06-06 세션 3차)
- **다른 PC 히어로 영상 오작동**: video 태그 내부에 구 영상(150MB × 2 = 300MB) 텍스트/스크립트가 박혀 페이지 로드 시 Enscape 영상을 덮어씌우던 문제 → 301MB 쓰레기 데이터 제거
  - 파일 크기 423MB → 121MB (302MB 감소)
  - 히어로 영상 src: Enscape_2026_converted.mp4 (38MB base64) 그대로 유지 — 이미 올바른 영상이 있었음
- **현재 파일 크기**: 121MB

## 텍스트/UI 수정 이력 (2026-06-06 세션 4차)
- **기본 활성 탭 변경**: 통합제어 시스템 → 음향 시스템 (sc active, dpw open)
- **히어로 인트로 주석 줄바꿈 정리**: ov-t 두 문단 br 위치 조정
- **전체 패널 주석 줄바꿈 정리**: 음향·영상·네트워크·통합제어·공연·인프라 모든 p 태그 br 과잉 제거, 문단당 2줄 기준으로 통일
- **word-break:keep-all 추가**: `.dt p` CSS — 한글 어절 중간 줄바꿈("다." 혼자 잘리는 현상) 방지
- **네트워크 인용구 수직 위치 조정**: 2열 그리드 align-items: start → center

## 수정 이력 (2026-06-06 세션 5차)
- **음향 시뮬레이션 기준표 항목 수정**: 4항목 → 5항목 (외부 노출 소음 65dB 이하 추가, 에코 유발 억제 맨 아래로 이동)
- **음향 시뮬레이션 외부 노출 소음 탭 추가**: `sim-ext` 패널, Ease0001.bmp / Ease0002.bmp 좌우 배치 (align-items:center)
- **탭 이름**: "외부 노출 음압" → "외부 노출 소음"
- **버그 수정 — 다른 탭 내용 안 보임**: p-audio dpw 닫는 태그 2개 누락으로 p-video~p-infra가 p-audio 안에 중첩 → </div> 2개 삽입으로 수정
- **버그 수정 — sim-ext 탭 전환 시 이미지 잔류**: sim-ext 패널이 .sim-body 밖에 있어 simTab() 범위 밖 → .sim-body 안으로 이동
- **탭 전환 스크롤**: toggle() 함수에 card.scrollIntoView({behavior:'smooth',block:'start'}) 추가
- **영상 시스템 탭 콘텐츠 추가**:
  - 인용구(pq) 아래 메인 전광판 사진(화면 캡처 2026-06-06 162137.png) + 우측 텍스트 (2fr 1fr 그리드, grid-column:1/-1)
  - 메인 전광판 텍스트: 3개 별도 p 태그 (가로 60m·세로 23m / 컨트롤 유닛 / NDI·SDI)
  - 보조 디스플레이 사진(스케치업 캡쳐본/화면 캡처 2026-06-06 151732.png) + 우측 텍스트 (동일 레이아웃)
- **버그 수정 — p-video div 중첩**: 삽입 작업 중 </div> 2개 누락 + 유령 텍스트 블록 p-infra에 혼입 → 모두 정리
- **현재 파일 크기**: 132.2MB

## 수정 이력 (2026-06-06 세션 6차)
- **영상 시스템 탭 — 이미지+텍스트 비율 조정**: 메인 전광판·보조 디스플레이 두 블록 모두 `2fr 1fr` → `1.2fr 1fr` (사진 축소, 텍스트 영역 확대)
- **영상 시스템 탭 — 리본보드 블록 추가**: 화면 캡처 155401.png / 155008.png 좌우 2열 배치 + 우측 텍스트 (3층 리본보드 267개 모듈·132m×1.28m·팬 메시지 실시간 송출)
- **현재 파일 크기**: 141.2MB

## 수정 이력 (2026-06-06 세션 8차)
- **영상 시스템 탭 — PTZ 카메라 블록 추가**: 스케치업 캡쳐본 155931/155951/160019.png 3장 전체폭 3열 배치 + 텍스트 아래 배치
  - NDI BirdDog A200 PTZ 4대 / 30배 광학 줌 / 경기 이벤트·관중 참여형 콘텐츠
- **영상 시스템 탭 — 이미지 비율 조정**: 메인전광판·보조디스플레이·리본보드 블록 `1.2fr 1fr` 유지, PTZ 블록은 전체폭 단독 레이아웃
- **현재 파일 크기**: 152.3MB

## 수정 이력 (2026-06-06 세션 9차)
- **라이트박스 기능 추가**: 이미지 클릭 시 확대, 오버레이 클릭 시 닫힘
  - 전역 이벤트 위임 방식 — `data:` URL을 가진 모든 img 자동 적용 (현재+미래 이미지)
  - CSS: `img[src^="data:"]:not(#lb-img){cursor:zoom-in}`, `#lb-overlay` 오버레이
  - JS: `document.addEventListener('click',...)` + `lbClose()`
  - 추가 이미지 삽입 시 별도 설정 불필요
- **현재 파일 크기**: 152.3MB

## 수정 이력 (2026-06-06 세션 7차)
- **오디오 토폴로지 MCR 이중화**: MCR → 3rd AR 보라선을 Primary/Secondary 미러 구조로 변경
  - 기존 MCR(우측 x:.790,y:.880) → PRIMARY 라벨 (보라색)
  - 신규 MCR2(좌측 x:.210,y:.880) → SECONDARY 라벨 (연보라), 3rd AR에 보라선 연결
  - TW_POS.audio에 mcr2 추가, TW_EDGES.audio에 mcr2→n3 엣지 추가, ta-mcr2 HTML 노드 추가
- **현재 파일 크기**: 141.2MB

## 수정 이력 (2026-06-06 세션 10차)
- **공연시스템 탭 콘텐츠 추가**:
  - SR 시스템 4장 이미지 2×2 그리드 (160457/160517/160624/160714.png) + 설명 텍스트 (K1·K2·X8·KS28)
  - 좌석 배치도 (KakaoTalk_Photo_2026-06-06-18-26-15.png) + 우측 텍스트 (A1~A6·B1·B2·C 스탠딩, FOH)
  - **공연 음향 시뮬레이션 토글** 추가: Direct 1k/2k/4k/500~4k/Total BB 5탭, 10°C 이미지 (매핑 016~020.jpeg)
  - 패널 ID: `perf-sim-d1k` ~ `perf-sim-tbb` (기존 sim-* ID와 충돌 없음)
- **br 태그 정리**: 공연시스템 탭 dt 섹션 + 새 삽입 블록 과도한 중간 줄바꿈 제거
- **현재 파일 크기**: 170.0MB

## 수정 이력 (2026-06-06 세션 11차)
- **영상시스템 — 타자 시야각도 분석 다이어그램 추가**: 메인전광판↔보조디스플레이 사이, `타자_전광판_시야각도.png` 전체폭 + 6.6° 여유각 설명 텍스트
- **음향시스템 외부노출소음 탭 이미지 교체**: BMP 2장 → `KakaoTalk_Photo_2026-06-06-19-05-09.png` 단일 이미지
- **공연시스템 음향 시뮬 버튼 이름 변경**: "공연 음향 시뮬레이션 결과" → "음향 시뮬레이션 결과"
- **공연시스템 음향 시뮬 Direct 63Hz 탭 추가**: `Response/Slide23.PNG` (10도 공연 Low Frequency), 서브우퍼 시스템 구분 레이블
  - 탭 구조: [메인 시스템] Direct 1k/2k/4k/500~4k/Total BB | [서브우퍼 시스템] Direct 63Hz
  - 63Hz 레이블+버튼을 flex 묶음으로 감싸 항상 같은 줄 유지
  - 패널 ID: `perf-sim-d63`
- **현재 파일 크기**: 167.6MB

## 수정 이력 (2026-06-07 세션 1차)
- **UI 수정**: 히어로 인트로 "그 순간" 뒤 쉼표 추가, `.ov` align-items:center 적용
- **네트워크 탭**: sc-desc에서 "VLAN" 제거, 토폴로지 탭 줄바꿈 방지(flex-wrap:nowrap)
- **탭 버튼 sc-desc**: white-space:nowrap + font-size:10px, 각 탭 설명 텍스트 단축
- **sc-title**: white-space:nowrap 추가 ("통합 제어" 줄바꿈 방지)
- **인프라 시스템 탭 콘텐츠 추가**:
  - 홈 응원석 패널 (`응원단상판낼.png`) + 텍스트 (XLR 입력, NDI 송수신)
  - 홈 응원단 인프라 (`스케치업 캡쳐본/화면 캡처 155105.png`) + 텍스트
  - 원정 응원석 패널 (`판넬얼빡/스크린샷 214249.png`) + 텍스트
  - 3루 덕아웃 패널 (`판넬얼빡/스크린샷 214106.png`) + 텍스트
  - 1루 덕아웃 패널 (`판넬얼빡/스크린샷 214027.png`) + 텍스트
  - 모든 레이블 font-size 13px 통일
- **현재 파일 크기**: 180.8MB

## 수정 이력 (2026-06-07 세션 2차)
- **인프라 시스템 탭 콘텐츠 추가**:
  - 중계 내함 (`스케치업 캡쳐본/화면 캡처 2026-06-06 151636.png`) + 텍스트 (총 12개, 이중화 회선)
  - 중계 외함 (`판넬얼빡/스크린샷 2026-06-04 214515.png`) + 텍스트 (12채널 영상·오디오 신호 통합)
  - 중계실 (`스케치업 캡쳐본/화면 캡처 2026-06-06 155737.png` / `155652.png`) 이미지 2장 + 텍스트 (총 5개, IO Expander)
  - 중계실 레이아웃: 텍스트 위, 이미지 2장(1:1) 아래
- **현재 파일 크기**: 193.6MB

## 수정 이력 (2026-06-07 세션 3차)
- **통합제어 갤러리 프레임 제거**: wc1~wc5 이미지 포함 dg 섹션 전체 삭제 (6MB 감소)
  - 삭제 후 div 2개 초과 문제 → 다른 탭 좌우 너비 이상 → 초과 `</div>` 2개 제거로 수정
- **전체 탭 갤러리 프레임 제거**: p-net·p-perf·p-infra의 dg 섹션 삭제
  - 모든 패널 div 균형 확인 (dpw 기준 ✅)
- **현재 파일 크기**: 187.5MB

## 수정 이력 (2026-06-07 세션 4차)
- **통합제어 탭 콘텐츠 추가** (p-ctrl, blockquote 아래):
  - 오디오 제어 시스템: `오디오컨트롤_converted.mp4` (46MB base64), 영상 좌+텍스트 우, aspect-ratio:16/8.6 크롭
  - 구역별 음압 제어 시스템: `구역제어_converted.mp4` (32MB base64), 동일 레이아웃
  - 점수판 제어 시스템: `전광판컨트롤_converted.mp4` (41MB base64), 레이블+영상 전체폭+텍스트 아래
  - 문자 응원 시스템: `응원메시지_trimmed.mp4` (앞 2초 제거, 41MB base64), 전체폭+텍스트+링크 3개
    - 링크: fanboard-dkti.onrender.com / /admin / /preview
  - 통합 조정실: 이미지 2장(155200/155242.png) 나란히 + 텍스트 아래
- **현재 파일 크기**: 358.2MB

## 수정 이력 (2026-06-07 세션 5차)
- **음향 탭 콘텐츠 추가** (blockquote↔시뮬레이션 버튼 사이):
  - JBL CWT128 블록: 텍스트(줄바꿈 2곳) + 스피커사진(그림1.jpg) 좌·커버리지다이어그램(그림2.jpg) 우 1:2.5 그리드
  - Crown 앰프 블록: Crown(images.png)+JBL(다운로드.png) 로고 세로배치 좌(200px) + 텍스트 우, align-items:center
- **현재 파일 크기**: 358.3MB

## 수정 이력 (2026-06-07 세션 6차)
- **공연 시스템 탭 KS28 텍스트 교체**: Rear-Front-Front 카디오이드 배열 내용 추가, 문장 재구성 + 줄바꿈 적용
- **현재 파일 크기**: 358.3MB

## 수정 이력 (2026-06-07 세션 7차)
- **전체 텍스트 교열**: 맞춤법·문체·일관성 수정
  - 영상탭: `메인 스코어보드` → `메인 전광판` (2곳, 전체 통일)
  - 통합제어탭: 통합 조정실 중복 표현 정리
  - 공연탭: "관중석 중앙부에" 표현 수정
  - 인프라탭: `마련하였습니다`·`하였습니다` → `했습니다`, `확장이 가능합니다` → `확장할 수 있습니다`
- **페이지 하단 "적용 시스템" 태그 섹션 삭제**
- **현재 파일 크기**: 358.3MB

## GitHub Pages 배포 이력 (2026-06-07 세션)
- **레포**: https://github.com/Gisancityboy/hanwha-ballpark
- **URL**: https://gisancityboy.github.io/hanwha-ballpark/
- **구조**:
  - `index.html` (91.7MB) — 데스크탑 버전, 모바일 감지 시 mobile.html 자동 전환
  - `mobile.html` (91.7MB) — 모바일 전용 버전, 데스크탑 감지 시 index.html 전환
  - `videos/` — 영상 6개 + audio-banner.png (GitHub Pages 직접 서빙, Content-Type: video/mp4)
  - GitHub Releases v1.0 — 영상 백업 (단, Content-Type: application/octet-stream으로 모바일 재생 불가)
- **영상 URL 방식**: `https://gisancityboy.github.io/hanwha-ballpark/videos/*.mp4`
- **gh CLI 설치**: `brew install gh`, `gh auth login` 완료

## 모바일 버전 주요 수정 내역 (2026-06-07)
- **자동 기기 감지**: index.html → 모바일이면 mobile.html로 redirect, mobile.html → PC면 index.html로 redirect
- **영상 재생 해결**: GitHub Releases URL이 `Content-Type: application/octet-stream`으로 모바일 Safari 재생 불가 → `videos/` 폴더에 직접 커밋하여 `video/mp4` 타입으로 서빙
- **토폴로지 모바일 최적화**:
  - 실제 클래스명 `.tw-diagram` (`.tw-wrap` 아님), `height:420px` 고정
  - 오디오/컨트롤: 375×420px 기준 r=120px 원형 배치 (인접 노드 141px 균등)
  - 비디오: 375×375 1:1 캔버스, 75px 간격 직접 설계
  - TW_EDGES bend 조정 (-26 → -20)
- **sig-grid (네트워크 사진)**: 3열 → 2열, scale transform 제거로 짤림 방지
- **스크롤 끊김**: scrollIntoView 제거, GPU 가속 CSS 추가
- **JBL 레이아웃**: display:flex row로 이미지 나란히, Crown 로고 소형화

## 인용구 수정 이력 (2026-06-07)
- **음향 탭**: "경기장의 모든 구역에서 균일한 음향이 들리도록 설계하였습니다." / 도움사운드 엔지니어 이상현
- **영상 탭**: "야구장에서만 느낄 수 있는 보는 즐거움을 충족 시켜주는 영상 시스템입니다." / 도움사운드 엔지니어 홍성민
- **현재 파일 크기**: 91.7MB (영상 외부 호스팅 후 크게 감소)

## 수정 이력 (2026-06-08 세션)

### 인용구 전면 수정
- **네트워크 탭**: "전체 시스템의 신뢰성을 보장하는 이중화 구성." / 도움사운드 엔지니어 곽현민
- **통합제어 탭**: "전체를 움직이는 단 한번의 클릭." / 도움사운드 엔지니어 이상현
- **공연 탭**: 인용구 유지, 엔지니어 → 도움사운드 엔지니어 남윤형
- **개요 인용구**: 두 번째 줄 "우리는" 첫 글자 가로축 정렬 (flex span 분리)
- **개요 인용구 출처**: "— 도움사운드 일동" 추가

### 텍스트 수정
- **음향 탭 JBL 레퍼런스**: 영국 토트넘/미국 Crypto.com Arena/국내 SSG랜더스필드로 교체, Crypto.com Arena 뒤 줄바꿈
- **영상 탭**: 메인 전광판·리본보드 "모듈" → "캐비넷" 수정
- **음향 설계기준 음압편차**: ±3.5dB → ±3dB

### 성능 최적화
- **이미지 56개 lazy loading**: `loading="lazy" decoding="async"` 일괄 적용
- **영상 5개 on-demand 로드**: 히어로 외 영상 data-src + toggle() lazy load 로직
- **통합제어 탭 hover prefetch**: 탭 버튼 hover/touchstart 시 영상 4개 선행 버퍼링
- **preconnect 힌트**: github.com 도메인 연결 선제 처리
- **스크롤탑 스마트 버튼**: 탭 아래→탭 위치로, 탭 위→맨 위로 이동

### 새 기능 추가
- **클라이언트 카드 모달**: 한화이글스 로고(HH_300300.png) + 볼파크 기본정보 팝업
- **수용규모 카드 모달**: 좌석배치도(img_eagles_park_map_mo.png) + 편의시설 탭 (1~4층)
- **설계완료 카드 모달**: 설계도서 탭(구성도·배치도·배관배선도·실장도·사업내역서) + 설계기준 탭(FIFA·IAAF)
- **프로젝트규모 카드 모달**: 도넛 파이차트(카테고리별 비용 분포) + 원가계산서 탭 (엑셀 데이터 기반)
- **스크롤탑 플로팅 버튼**: 우측 하단 고정, 300px 이후 등장
- **라이트박스**: 이미지 클릭 확대 (이미 적용)

### 콘텐츠 추가
- **공연시스템 음향시뮬레이션**: 설계기준표 추가 (최대음압·편차·서브우퍼), 주파수 편차 탭 (Slide16.PNG 크롭)
- **음향시뮬레이션 외부노출소음**: 30°C 온도 뱃지 추가
- **통합제어 탭**: 서버 설치영상(server-install.mp4) + 운영 설명 블록 (점수판↔문자응원 사이)
- **문자응원 시스템**: 관리자 페이지 비밀번호 1234 표시
- **모바일 PC버전 버튼 제거**

### 버그 수정
- **Safari simTab 스크롤 점프**: scrollY 저장 + btn.blur() + rAF 복원
- **Safari 히어로 영상 자동재생 안됨**: `muted=""` → `muted` 정규화 + JS play() fallback (canplay)
- **맥북 Safari 영상 재생 안됨**: index.html 영상 URL Releases → GitHub Pages videos/ 로 교체
- **클라이언트 모달 안 열림**: 인라인 style display:none vs CSS 우선순위 충돌 수정

### videos/ 추가
- `server-install.mp4` (서버 설치 영상, 9.8MB, H.264 변환)

### 현재 파일 크기: 92.4MB

---

## 계활 명령어

사용자가 **"계활"** 이라고 말하면 아래 **네 가지**를 순서대로 실행:

1. **index.html 최신화 확인** — Downloads 폴더에서 가장 최근 `hanwha-ballpark*.html` 파일을 찾아 index.html과 비교:
   - Windows 경로: `C:\Users\apdlc\Downloads\hanwha-ballpark*.html`
   - index.html 수정시각 vs 최신 hanwha-ballpark*.html 수정시각 비교
   - **최신 파일이 index보다 새것이면** → 해당 파일을 index.html로 복사 후 알림
   - index.html이 이미 최신이면 → "✅ index.html이 최신 상태입니다" 출력

2. **USB 이전 체크** — index.html을 Python으로 읽어 아래 항목 확인:
   - 영상: fetch→blob 스크립트 또는 `data:video` src 여부, `muted/loop`, 다른PC 재생 픽스
   - 저장 로직: 중복 src 방지 코드 존재 여부
   - 배너 이미지: audio/video/net/ctrl/perf-img 각각 data URL 임베딩 여부 + KB 크기
   - 갤러리: wc1~wc5 이미지 여부 (나머지 빈 프레임은 삭제된 상태)
   - 음향 시뮬레이션: sim-toggle-btn 존재 여부, simToggle JS 함수, 시뮬레이션 이미지 래퍼 7개 이상
   - 패널: p-audio/p-video/p-net/p-ctrl/p-perf/p-infra 6개 모두 존재 여부
   - 공연시스템 추가 콘텐츠: SR 4장 이미지, 좌석배치도, 공연 음향 시뮬레이션 토글 존재 여부
   - 파일 크기(MB)
   - 결과를 ✅/⬜ 로 출력

3. **백업 생성** — index.html을 날짜+시간 파일명으로 새 파일 복사 (기존 백업 덮어쓰지 않음):
   - 파일명 형식: `index.backup.YYYYMMDD_HHMMSS.html`
   - PowerShell: `$ts = Get-Date -Format 'yyyyMMdd_HHmmss'; Copy-Item index.html "index.backup.$ts.html"`
   - 백업 완료 후 파일명·크기 출력
   - 오래된 백업이 5개 초과 시 가장 오래된 것부터 삭제하여 최대 5개 유지

4. **CLAUDE.md 업데이트** — 오늘 대화에서 변경된 내용(추가 탭, 삽입 이미지, 수정 텍스트 등)을 CLAUDE.md에 반영

## 버그 수정 이력 (2026-06-09 세션)
- **공연시스템 sim 주파수편차 탭 잔류**: `perf-sim-freq` 패널 `sim-body` 밖→안으로 이동 (index/mobile 모두)
- **Safari 시뮬레이션 탭 스크롤 점프**: `simToggle` 열릴 때 모든 img `loading=eager` + `img.decode()` 프리디코딩, `simTab` Promise.then 내부에 rAF 추가
- **맥 Safari 히어로 영상 자동재생**: `played` 가드 레이스 컨디션 제거, `vid.load()` 명시 호출, `preload="metadata"` 추가, 다중 이벤트 리스너 (loadedmetadata/canplay/loadeddata/canplaythrough)
- **dbOpen 에러 처리**: `onerror` + `try-catch` 추가, `dbLoad` null db 처리

## 수정 이력 (2026-06-09 세션)
- **영상시스템 탭 — 컨트롤 유닛 텍스트 삭제**: "컨트롤 유닛 1포트당 53개 모듈을 제어하며 총 35포트로 운영됩니다." 제거 (index/mobile)
- **현재 파일 크기**: 92.4MB

## 수정 이력 (2026-06-09 세션 2차 — 최적화 및 콘텐츠 추가)

### 성능 최적화
- **이미지 외부 분리**: 61개 base64 이미지(69.6MB) → `images/` 폴더 호스팅 (91% 감소)
  - PNG→JPEG 변환(quality 78), 도움사운드 로고는 PNG 유지 (filter:invert(1) 호환)
  - index.html: 92.9MB → 135KB / mobile.html: 92.9MB → 140KB
  - `saveHtml()` 업데이트: 저장 시 `inlineExternalImages()` fetch→base64 재인라인
  - 이미지 URL: `https://gisancityboy.github.io/hanwha-ballpark/images/img001.png~img061.jpg`
- **백업 태그**: `git tag v1.0-embedded-images` (원본 93MB 버전 복원 가능)

### 버그 수정
- **라이트박스 data: 조건 제거**: 이미지 외부 URL 이동 후 확대 안되던 문제 (index/mobile)
- **모바일 중복 lb-overlay 제거**: img029 14번 중복 등 이미지 매핑 오류 → SHA256 해시 매칭으로 재처리
- **index.html 중복 lb-overlay 제거**: 계활 점검 중 발견·수정
- **도움사운드 로고 PNG 복원**: JPEG 변환 시 검정 배경 → filter:invert(1) 깨짐 → 원본 PNG 200x200 리사이즈(569KB→10KB)
- **모바일 라이트박스 .open→.on CSS 클래스 불일치 수정**
- **Safari 스크롤 점프 — 새 토글 함수 4개**: perfSumToggle/netSumToggle/perfSumTab/netSumTab 모두 scrollY 복원 적용

### 콘텐츠 추가 (음향시스템 탭)
- **시스템 배치도면 삽입**: PDF p40 (지상 4층 Zone A 배치도, JBL AW159×16EA) — "설계했습니다." 뒤
- **배관배선도면 삽입**: PDF p62 (지상 4층 Zone A 배관배선도) — Crown 앰프 설명 아래
- **음향 시뮬레이션 기준표**: "에코 유발 억제" 항목 제거

### 콘텐츠 추가 (네트워크 시스템 탭)
- **시스템 신호 Summary 토글**: 하단 추가
  - 오디오·컨트롤 신호 탭: PDF p1 (Summary 음향 구성도)
  - 비디오 신호 탭: PDF p2 (Summary 영상 구성도)
  - 캡션: "구성도면 중 일부 발췌"

### 콘텐츠 추가 (공연시스템 탭)
- **공연 티켓 가격표**: A구역(7000석)~스카이박스(24실) 구역별 금액·좌석수·총액, 합계 약 15억
- **레이아웃 재구성**: 표+텍스트 왼쪽(1.3fr), 공연객석 이미지 오른쪽(1fr)
- **BEP 분석 다이어그램** (전체폭 2카드):
  - 좌: 대관료 구성 바 차트 (시설8천/음향8천/LED6천/전기1천 = 2.3억/회)
  - 우: 3년 투자회수 시뮬레이션 (연 5회 기준, 34.5억 BEP)
- **공연시스템 Summary 토글**: 하단 추가
  - 오디오·컨트롤 신호 탭: PDF p3 (공연용 Summary 음향 구성도)
- **서브우퍼 63Hz 버튼**: flex-basis:100% 추가 (항상 새 줄 왼쪽 고정)
- **BEP 문구 추가**: "대관 측은 만석 시 회당 수익 약 15억 원으로 예상되며, 구장 측의 BEP는 15회 공연 시로 예상하여 시스템을 설계하였습니다."

### 콘텐츠 수정 (인프라 시스템 탭)
- **중계외함 텍스트**: "12개 카메라의 영상 소스" → "12개의 카메라 포인트의 영상 소스"

### 현재 파일 크기: 151KB (이미지 외부 분리 후)

## 포트폴리오 산출물 (2026-06-09)
- 저장 경로: `/Users/gisancityboy/Downloads/hanwha-ballpark-project/portfolio_assets/`
- **이력서_포트폴리오_한화볼파크.docx** (2페이지): 담당업무 6개·기술태그·성과요약 개인 이력서용
- **한화볼파크_AV설계_포트폴리오.docx** (9챕터 풀버전)
- **한화볼파크_AV설계_포트폴리오_2p.docx** (2페이지 회사용)
- **이미지 32장**: 00_company_logo ~ 61_seating_map
- **음향 시뮬 합성 이미지**:
  - `sim_composite_10C.png` — Direct BB·Total BB·STI (10°C) 3장 가로 배치
  - `sim_composite_30C.png` — Direct BB·Total BB·STI·외부노출소음 (30°C) 2×2 그리드
- **공연 시뮬 개별 이미지 7장**: perf_sim_01~07 (매핑 016~020 + Response/Slide23·16)
- **도면 PNG**: 대전한화생명볼파크_도면.pdf 1~2페이지 → `도면_page01.png` / `도면_page02.png` (400DPI, 6617×4678)
