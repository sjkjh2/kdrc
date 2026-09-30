# KDRC 디자인 가이드 (for Claude Code)

> 한국데이터연구소(KDRC) 사이트 리뉴얼 퍼블리싱 기준 문서.
> **UI를 만들거나 고치기 전에 이 문서를 먼저 읽고, 값은 `css/tokens.css`의 토큰으로만 쓴다.**
> 시각 기준은 디자인 캔버스(“KDRC 메인 리디자인”)의 PC(1440)·MO(390) 시안 보드와 반응형 프리뷰 보드(`Web_*`)다.

---

## 0. 핵심 규칙 10가지

1. **하드코딩 금지** — 색·폰트 크기·행간·여백·라운드는 `tokens.css` 변수로. 새 hex/px 값이 필요하면 토큰을 먼저 추가한다.
2. **모든 수치는 짝수 px** — 예외는 `1px` 선, `999.9rem` pill, 반응형 분기점(720/721)뿐. 소수점 px, 홀수 px 금지.
3. **행간은 px 고정** — `line-height: 1.6` 같은 배수 금지. 폰트 토큰과 짝인 `--lh-*`를 쓴다. (숫자 강조처럼 `line-height: 1`은 허용)
4. **분기점은 하나: 720px** — `@media (max-width: 720px)` = 모바일, 그 이상 = PC. 560/768/860/900 새로 쓰지 않는다.
5. **배경 100%, 콘텐츠 1200px 가운데** — 섹션은 전폭, 안쪽 좌우 여백은 `padding-inline: var(--gutter)`.
6. **카드 라운드는 오른쪽 위만** — `border-radius: var(--radius-card)` (PC 28 / MO 20). 입력칸·표·박스형 버튼은 각지게.
7. **브랜드 레드는 하나** — `#D82B52`(`--c-red`), hover는 `#B71F44`. 기존 `#B20028`, 청록 `--kdrc-link(#0f766e)`는 새 화면에서 쓰지 않는다.
8. **버튼은 공통 텍스트 버튼(텍스트 + 빨간 원)** — 모바일에서는 **항상 오른쪽 정렬**.
9. **폰트는 Pretendard 500/600/700/800** — 본문 500, 라벨 600, 버튼 700, 제목 800.
10. **접근성 기본값 유지** — 시맨틱 태그, `aria-current`, 포커스 표시, 44px 터치 영역, `prefers-reduced-motion` 대응.

---

## 1. 프로젝트 구조 · 코드 컨벤션 (기존 테마 기준)

WordPress 테마 `wp-content/themes/kdrc/`

```
css/reset.css     * 박스사이징 리셋, html{font-size:62.5%} → 1rem = 10px
css/tokens.css    ★ 신규 — 이 가이드의 디자인 토큰 (reset 다음, common 앞에 enqueue)
css/common.css    폰트(@font-face), 레거시 토큰(--fs-N/--sp-N), 레이아웃·공통 컴포넌트
css/header.css    헤더, GNB, 메가메뉴
css/footer.css    푸터
css/home.css      메인(front-page.php) 전용
css/sub.css       ★ 신규 권장 — 서브 공통(서브 비주얼, 서브 탭, 게시판, 폼, 카드)
style.css         테마 헤더 + 페이지별 보조 스타일
js/main.js, js/mega-menu.js, js/home.js   (jQuery 3.7, Swiper 11, odometer 로드됨)
```

**클래스 네이밍**

- 접두사 `kdrc-` 필수. 블록-요소는 하이픈으로 잇는다: `.kdrc-perf-card`, `.kdrc-perf-card-link`, `.kdrc-perf-title`.
- 변형(modifier)은 `--`: `.kdrc-home-more--dark`, `.kdrc-home-eyebrow--white`.
- 상태는 `is-*`: `.is-active`, `.is-open`, `.is-scrolled`, `.is-featured`, `.is-highlight`.
- 스크린리더 전용 텍스트는 `.kdrc-sr-only`.
- 단위는 `rem`(10px 기준). `2.4rem = 24px`. 선 두께 `1px`만 px 허용.

**enqueue 순서** (functions.php)

```php
wp_enqueue_style('kdrc-reset',  get_theme_file_uri('css/reset.css'),  [], $ver);
wp_enqueue_style('kdrc-tokens', get_theme_file_uri('css/tokens.css'), ['kdrc-reset'], $ver);
wp_enqueue_style('kdrc-common', get_theme_file_uri('css/common.css'), ['kdrc-tokens'], $ver);
// header / footer / style / (home | sub) ...
```

---

## 2. 기존 CSS와 다른 점 — 이 가이드의 결정 사항

현재 사이트 CSS(common/header/home/style.css)를 분석해서, 새 시안과 충돌하는 부분을 아래처럼 정리한다.

| 항목 | 현재 사이트 | 이 가이드 기준 | 이유 |
|---|---|---|---|
| 폰트/여백 토큰 | `--fs-N`, `--sp-N` = `clamp(최소, vw, 최대)` 유동값 | 고정 짝수값 `--fs-h1` 등, 720px에서 모바일 값으로 교체 | 유동값은 1200px 미만에서 소수점 px 발생, 720px 부근에서 본문이 10px까지 작아짐 (예: `--fs-16` → 10px) |
| 분기점 | 560 / 768 / 860 / 900 혼재 | **720px 하나** | 시안이 PC·MO 두 벌로 설계됨 |
| 행간 | 대부분 미지정(normal), 일부 `140%`, `1.4` | 폰트별 짝수 px `--lh-*`, body 기본 `--lh-body` | 소수점 줄 높이 제거 |
| 브랜드 레드 | `#D82B52` + `#B20028` 혼용 | `#D82B52` 하나 (+hover `#B71F44`) | 톤 통일 |
| 링크/주요 버튼색 | `--kdrc-link: #0f766e`(청록) | 레드 또는 `--c-text` | 시안에 청록 없음 |
| 폰트 굵기 | 400/500/700만 로드 (600·800 주석 처리) | **500/600/700/800 로드** | 시안에서 600·800을 많이 씀 (각 약 25%) |
| 카드 라운드 | `border-top-right-radius: var(--sp-36~50)` | `var(--radius-card)` (28 / 20) | 짝수·고정값 |
| 컨테이너 | `.kdrc-layout` max 1200 + 좌우 `--sp-16/18` | 전폭 섹션 + `padding-inline: var(--gutter)` | PC 최소 여백 120, MO 24 |
| 기본 버튼 | `.kdrc-btn` 박스형(radius 14) | 텍스트 버튼 `.kdrc-tbtn` | 시안 공통 버튼 |

**레거시 처리 원칙**
- `common.css`의 `--fs-N`, `--sp-N`, `--kdrc-*`는 **지우지 않는다**(기존 화면 호환). 새로 작업하거나 수정하는 화면부터 `tokens.css` 토큰으로 바꾼다.
- `common.css`에서 `Pretendard-SemiBold(600)`, `Pretendard-ExtraBold(800)` `@font-face` 주석을 해제한다. (또는 PretendardVariable 한 파일로 교체)
- `.kdrc-layout`에 `padding-inline` 뒤에 `padding: 0 var(--sp-18)`이 다시 선언돼 앞의 값이 덮어써지는 버그가 있다 → 새 컨테이너 규칙(3-4)으로 정리.
- `body`에 `line-height: var(--lh-body)`를 추가해 normal 행간(소수점)을 없앤다.

---

## 3. 디자인 토큰 요약

전체 값은 `tokens.css` 참조. 표의 값은 **PC / MO(≤720)**.

### 3-1. 컬러

| 토큰 | 값 | 용도 |
|---|---|---|
| `--c-red` | `#D82B52` | 포인트, 버튼 원, 활성 탭 밑줄, 강조 숫자, eyebrow |
| `--c-red-hover` | `#B71F44` | 레드 요소 hover |
| `--c-red-tint` | `#FDECEF` | 선택된 해시태그 배경 |
| `--c-ink` | `#0F0C0C` | 헤더·서브 비주얼·푸터 다크 배경 |
| `--c-wine` | `#451919` | 다크 그라데이션 끝색 |
| `--c-text` | `#111827` | 제목·강조 |
| `--c-text-2` | `#374151` | 진한 본문 |
| `--c-text-3` | `#4B5563` | 기본 설명 본문 |
| `--c-text-4` | `#6B7280` | 메타(날짜·라벨) — **흰 배경 텍스트 최저 명도** |
| `--c-text-5` | `#9CA3AF` | placeholder·비활성 아이콘 **전용** (대비 2.5:1 → 본문 금지) |
| `--c-on-dark` | `#D3D3D3` | 다크 배경 위 설명 |
| `--c-bg-soft` | `#F7F7F9` | 어사이드·표 헤더·첨부 영역 |
| `--c-bg-muted` | `#F3F4F6` | 해시태그 칩, 이미지 자리 |
| `--c-line` | `#E5E7EB` | 기본 구분선·카드 테두리 |
| `--c-line-2` | `#D1D5DB` | 입력칸 테두리 |
| `--c-line-soft` | `#EEF0F3` | 카드 내부 구분선 |
| `--c-line-strong` | `#111827` | 목록·표 상단 2px 라인 |

대비: `#D82B52`/흰 배경 4.8:1(AA 통과), `#6B7280`/흰 배경 4.8:1, `#D3D3D3`/`#0F0C0C` 13:1.

### 3-2. 타이포그래피

| 토큰 | PC (size/line) | MO (size/line) | 굵기 | 사용처 |
|---|---|---|---|---|
| `display` | 72 / 74 | 42 / 44 | 800 | 메인 히어로 타이틀 |
| `h1` | 56 / 64 | 34 / 42 | 800 | 서브 비주얼 타이틀 |
| `h2` | 40 / 52 | 26 / 36 | 800 | 섹션 타이틀 |
| `h3` | 22 / 30 | 20 / 28 | 700~800 | 카드·블록 타이틀 |
| `h4` | 20 / 30 | 18 / 26 | 700 | 소제목 |
| `lead` | 18 / 28 | 16 / 26 | 500 | 리드 문구 |
| `body` | 16 / 26 | 16 / 26 | 500 | 본문 |
| `small` | 14 / 22 | 14 / 22 | 500 | 카드 설명, 보조 본문 |
| `caption` | 14 / 20 | 12 / 18 | 600 | 날짜·메타·라벨 |
| `micro` | 12 / 18 | 12 / 18 | 600 | 칩·저작권 (**12px 미만 금지**) |
| `eyebrow` | 18 | 14 | 700 | 섹션 상단 영문 라벨 |
| `stat` | 46 | 32 | 800 | 통계 숫자 (`line-height: 1`) |

- 제목 자간 `letter-spacing: var(--ls-tight)` (-0.02em). 본문 자간 0.
- 한글 줄바꿈은 `word-break: keep-all`을 레이아웃 루트에 한 번.
- 두 줄 말줄임: `-webkit-line-clamp: 2` (카드 제목), 한 줄: `text-overflow: ellipsis`.

### 3-3. 여백 (이름 = px)

`--space-4 · 8 · 12 · 16 · 20 · 24 · 32 · 40 · 48 · 56 · 64 · 80 · 96 · 120`

| 용도 | PC | MO |
|---|---|---|
| 섹션 상하 `--section-py` | 120 | 64 |
| 작은 섹션 `--section-py-sm` | 96 | 48 |
| 좌우 여백 `--gutter` | max(120, (100%−1200)/2) | 24 |
| 섹션 헤드 ↔ 콘텐츠 | 48~64 | 32 |
| 카드 그리드 gap | 20~24 | 12~16 |
| 카드 내부 padding | 28 | 20 |

### 3-4. 레이아웃

- 헤더 높이 PC 84 / MO 64, 서브 비주얼 PC 360 / MO auto(상 28·하 40 패딩), 서브 탭 PC 72 / MO 56.
- 컨테이너 패턴:

```css
.kdrc-section { padding-block: var(--section-py); padding-inline: var(--gutter); }
/* 배경은 section에, 콘텐츠는 1200 안에 자동으로 들어감 */
```

- PC 그리드: 수행실적 카드 3열(서브) / 4열(메인), How We Work 3열, 파트너 로고 5~6열.
- MO: 기본 1열, 통계·숫자 2열.

### 3-5. 모양 · 그림자

| 토큰 | 값 | 비고 |
|---|---|---|
| `--radius-card` | `0 28px 0 0` / MO `0 20px 0 0` | 카드·강조 박스·통계 바 |
| `--radius-pill` | 9999 | 해시태그, 원형 버튼, 페이지 번호 |
| `--radius-none` | 0 | 입력칸, 표, 탭형 라디오 |
| `--shadow-card` | `0 10px 30px rgba(0,0,0,.08)` | 떠 있는 카드 |
| `--shadow-float` | `0 24px 60px rgba(15,12,12,.14)` | 히어로 위 통계 바 |
| `--shadow-menu` | `0 30px 60px rgba(0,0,0,.35)` | 메가메뉴 |

> 정리 필요: 사업영역(빅데이터·성과평가) 카드와 메인 MO How We Work 카드에 `0 24px 0 0`이 남아 있다 → PC 28 / MO 20으로 맞춘다.

---

## 4. 반응형 규칙

```css
/* PC가 기본, 모바일만 덮어쓴다 */
@media (max-width: 720px) { ... }
```

- **721px 이상 = PC 레이아웃.** 1440px보다 넓으면 배경은 끝까지, 콘텐츠는 1200px 중앙.
- 721~1199px(태블릿·작은 노트북)은 PC 레이아웃 유지, `--gutter`가 120px 최소값이라 좁아지면 그리드 열 수를 줄인다(4→3→2). 폰트 크기는 줄이지 않는다.
- 모바일은 390px 시안 기준이지만 **폭 고정 금지**. 360~720px에서 모두 자연스럽게 늘어나야 한다(`width:100%`, `minmax(0,1fr)`).
- PC/MO 마크업을 따로 만들지 말고 **하나의 마크업 + CSS**로 처리한다. 예외: 필터(PC 어사이드 ↔ MO 팝업), 전체 메뉴(PC 메가메뉴 ↔ MO 풀스크린 시트)는 표시 방식만 다르고 같은 데이터를 쓴다.

---

## 5. 컴포넌트

값은 **PC / MO**. 시안 보드 이름은 캔버스에서 확인.

### 5-1. 헤더 · GNB (`.kdrc-header`)
- 배경 `--c-ink`, 하단 선 `--c-line-dark`, 높이 `--header-h`. `position: sticky; top: 0; z-index: var(--z-header)`.
- 로고(좌, 높이 28) · 1depth 메뉴 5개(소개/사업영역/파트너/수행실적/인사이트, 각 136px 칸) · "문의하기" pill(높이 44, 테두리 흰 35%, 16/600) · 전체메뉴 햄버거(우).
- 메뉴 글자 16 / 600, 흰색. hover·현재 위치 = 하단 2px 레드 바 + 글자 흰색 유지.
- 메가메뉴: 헤더 hover/focus 시 전폭 패널(`--c-ink`, 높이 300, `--shadow-menu`), 좌측에 1depth 번호·설명, 우측에 2depth 목록. 3depth(정량/정성/빅데이터)는 들여쓰기 + 작은 글씨.
- 스크롤 시(`.is-scrolled`) 기존처럼 반투명 + blur 유지 가능.
- MO: 높이 64, 로고 + 햄버거만. 햄버거 → 풀스크린 시트(`position: fixed; inset: 0; height: 100dvh`), 1depth 아코디언(+/−), 하단에 문의 CTA와 연락처.

### 5-2. 서브 비주얼 (`.kdrc-subhero`)
- 배경 `--c-ink` + 오른쪽 위 레드 radial glow + 페이지별 **애니메이션 일러스트(SVG)** 우측 배치(PC 560×300 / MO 160×86, 투명도 .6).
- 순서: 브레드크럼(홈 아이콘 › 1depth › 현재) → eyebrow(레드 바 28×4 + 영문 라벨) → h1 → 설명(`--c-on-dark`, PC 최대폭 600).
- 일러스트는 `aria-hidden="true"`, 모션은 `prefers-reduced-motion`에서 정지.

### 5-3. 서브 탭 (`.kdrc-subtab`)
- 흰 배경, 하단 1px `--c-line`, 높이 72 / 56, 항목 간격 48 / 26, 글자 18 / 16.
- 현재 탭: 800 + `--c-text` + 하단 4px `--c-red`, `aria-current="page"`. 나머지: 600 + `--c-text-4`, hover 시 `--c-text`.
- MO: 가로 스크롤(`overflow-x: auto`), 스크롤바 숨김.

### 5-4. 섹션 헤드
```
[eyebrow]  ━━ About KDRC        (바 28×4 / 20×4, 글자 --fs-eyebrow, 700, --c-red)
[title]    섹션 타이틀           (--fs-h2, 800, --c-text)
[desc]     설명 문구             (--fs-lead, 500, --c-text-3)
[more →]   (선택) 오른쪽 끝 텍스트 버튼
```

### 5-5. 텍스트 버튼 (`.kdrc-tbtn`) — 공통 버튼
- 텍스트(`--tbtn-fs`, 700, `--c-text`) + 빨간 원(`--tbtn-circle` 48 / 44, `--c-red`, 흰 화살표 아이콘 16~18).
- 간격 `--tbtn-gap` 14 / 12. 다크 배경에서는 글자 흰색.
- hover/focus: 글자 `--c-red`, 원 `--c-red-hover`, 화살표가 오른쪽으로 빠졌다 들어오는 모션(0.6s).
- **정렬:** MO는 항상 오른쪽(`align-self: flex-end` / `margin-left: auto`). PC는 섹션 헤드 오른쪽 또는 콘텐츠 끝.
- 변형: 다운로드 버튼(`.kdrc-dlbtn`) = 레드 배너 안 흰 글자 + 흰 테두리 원(52, 2px) + 다운로드 아이콘. hover 시 원이 흰색으로 채워짐.
- 폼 제출도 이 버튼을 쓴다(`<button type="submit" class="kdrc-tbtn">`).

### 5-6. 수행실적 카드 (`.kdrc-perf-card`)
- 흰 배경, 1px `--c-line`, `--radius-card`, padding 28 / 20, 최소 높이 260 / 200.
- 상단: 연도(14/700 `--c-text-4`) + ↗ 아이콘 → 제목(20·30 / 18·26, 700, 2줄 말줄임) → 해시태그 목록 → 구분선(`--c-line-soft`) → 발주기관(16 / 14, 600).
- hover·포커스·`.is-featured`: 배경·테두리 `--c-red`, 모든 글자·아이콘 흰색, 해시태그 배경 흰 20%.
- 메인과 서브 카드는 **동일 컴포넌트**(열 수만 다름).

### 5-7. 해시태그 칩 (`.kdrc-hash`)
- pill, padding 4×10 / 4×8, 글자 14 / 12, 600, 배경 `--c-bg-muted`, 글자 `--c-text-3`.
- 선택된 필터와 일치하는 태그: 배경 `--c-red-tint`, 글자 `--c-red`, 700.
- 카드 안에서는 최대 3~4개 + `+N`.

### 5-8. 필터 (수행실적)
- 검색 로직: 주제1 · 주제2 · 세부분야 · 방법 중 **하나라도 포함되면 노출(OR)**.
- PC: 왼쪽 sticky 어사이드(`--c-bg-soft`, `--radius-card`), 그룹 제목 16/800, 칩 버튼 높이 36(`--chip-h`), 선택 = `--c-text` 배경 + 흰 글자, `aria-pressed`.
- 상단에 선택된 태그(×로 해제) + "검색 결과 N건 / 전체 N건" + 정렬(최신순/발주처순).
- MO: 검색창 + "필터 (N)" 버튼 → 풀스크린 팝업(`role="dialog" aria-modal="true"`), 하단 고정 바에 초기화(좌) + "N건 결과 보기" 텍스트 버튼(우).
- 목록 하단 "더보기 (8/126)" 텍스트 버튼.

### 5-9. 게시판 (인사이트: 리서치 노트 / 공지사항 / 언론보도)
- 목록 PC: `<table>` + 숨김 `<caption>`, 상단 2px `--c-line-strong`, 헤더 행 56 높이 `--c-bg-soft`, 행 높이 72, 열 = 번호(100) / 제목 / 등록일(160). 행 hover: 배경 `--c-bg-soft`, 제목 레드 + 밑줄.
- 목록 MO: 표 대신 리스트(날짜 caption + 제목 한 줄 말줄임).
- 상세: 카테고리 점+라벨 → 제목(h2) → 등록일 → 본문(CMS) → 첨부파일 박스(`--c-bg-soft`, `--radius-card`) → 이전/다음 글(상단 2px 라인) → "목록으로".
- 페이지네이션: 40×40 원형, 현재 페이지 `--c-text` 배경 + 흰 글자, `aria-current="page"`.

### 5-10. 폼 (문의)
- 라벨 16/700 + 필수 `*`(`--c-red`, `aria-hidden`) — 필수 여부는 `required`/`aria-required`로도 전달.
- 입력칸: 높이 `--input-h`(52), padding 0 16, 1px `--c-line-2`, radius 0, 글자 16(모바일 확대 방지), placeholder `--c-text-5`. focus: 테두리 `--c-text` + 2px outline.
- 문의 유형: 라디오를 탭 모양으로(각진 박스 4칸, 선택 = `--c-text` 배경).
- 개인정보 동의: 체크박스 + ">" 아코디언. 펼친 영역 흰 배경, **왼쪽 여백을 체크박스 텍스트 시작점에 맞춤**(PC 52), 수집 항목/이용 목적/보유 기간/동의 거부 권리 + "개인정보처리방침 전체 보기(새 창)".
- 어사이드(PC sticky / MO 폼 아래): Direct Contact 카드(대표번호·팩스·이메일 아이콘 행) + 하단 레드 밴드 "회사소개서 다운로드".

### 5-11. 통계 (숫자)
- 라벨 16 / 14 (600, `--c-text-4`) + 숫자 `--fs-stat`(800, `line-height: 1`) + 단위 18 / 14(700).
- 강조 항목 하나만 숫자 `--c-red`.
- 메인 히어로 하단 통계 바: 흰 배경 + `--shadow-float` + `--radius-card`, 왼쪽 레드 CTA 칸("프로젝트 문의하기").
- 숫자 카운트업(odometer) 시 `aria-live` 쓰지 말고 최종값을 텍스트로 둔다.

### 5-12. 푸터
- `--c-ink` 배경, 글자 `--c-text-5`(다크 배경이라 대비 충분), 회사명 흰색 700.
- 로고 + 맨 위로(44 원형, 흰 25% 테두리) / 연락처 / 주소(서울·부산) / 개인정보처리방침(흰색 700) + 저작권.

---

## 6. 모션

| 대상 | 스펙 |
|---|---|
| 링크·탭 색 | `color var(--dur-fast)` |
| 버튼 원·카드 배경 | `var(--dur-base)` |
| 화살표 | hover 시 오른쪽으로 빠졌다 왼쪽에서 들어옴, `.6s var(--ease-io)` |
| 햄버거 | 두 번째 선 60% → 100% `.3s var(--ease-out)` |
| 서브 비주얼 일러스트 | 막대 성장·선 그리기·태그 팝 1회 재생(무한 반복은 파형 1개만) |
| 모바일 시트 | 위에서 16px 슬라이드 + 페이드 `.3s` |

`@media (prefers-reduced-motion: reduce)`에서 모든 애니메이션 정지(`tokens.css`에서 duration 0 처리).

---

## 7. 접근성 체크리스트

- [ ] 랜드마크: `header` / `nav[aria-label]` / `main` / `footer`, 페이지당 `h1` 1개.
- [ ] 현재 위치: GNB·서브 탭·브레드크럼·페이지네이션에 `aria-current="page"`.
- [ ] 아이콘만 있는 버튼·링크에 `aria-label`(햄버거, 맨 위로, 닫기, 이전/다음).
- [ ] 장식 SVG `aria-hidden="true"`, 의미 있는 이미지는 대체텍스트(보고서 표지 = "○○ 보고서 표지").
- [ ] 텍스트 대비 4.5:1 이상 — 흰 배경 본문은 `--c-text-4`(#6B7280)보다 연하게 쓰지 않는다.
- [ ] 포커스 표시 제거 금지(`:focus-visible` 2px outline).
- [ ] 터치 영역 최소 44×44(`--touch-min`).
- [ ] 필터 칩 `aria-pressed`, 아코디언 `aria-expanded` + `aria-controls`, 팝업 `role="dialog" aria-modal="true"` + 포커스 가두기 + ESC 닫기.
- [ ] 표에 `caption`, `th[scope]`.
- [ ] 새 창 링크는 텍스트나 아이콘으로 "새 창" 안내.
- [ ] 모바일 입력칸 글자 16px 이상.

---

## 8. 작업 체크리스트 (Claude Code용)

작업 전
1. 해당 화면의 시안 보드(PC·MO)와 반응형 보드(`Web_*`)를 확인한다.
2. 이 문서의 컴포넌트(5장)에 이미 있는지 확인 → 있으면 재사용, 없으면 같은 규칙으로 새로 만든다.

작업 중
3. 값은 토큰만. 새 값이 필요하면 `tokens.css`에 추가하고 이 문서 3장 표도 갱신.
4. 클래스는 `kdrc-` 접두사, 상태는 `is-*`.
5. PC 먼저 작성 → `@media (max-width: 720px)`에서 MO 덮어쓰기.

작업 후 (셀프 점검)
6. `grep`으로 새 코드에 hex 색, 홀수 px, 배수 line-height가 없는지 확인:
   ```bash
   grep -nE '#[0-9a-fA-F]{3,6}' css/sub.css           # 토큰 파일 외 hex 금지
   grep -nE '[^0-9.][0-9]*[13579]px' css/sub.css      # 홀수 px (1px 선 제외)
   grep -nE 'line-height:\s*[0-9.]+;' css/sub.css     # 배수 행간
   ```
7. 360 / 390 / 720 / 721 / 1024 / 1440 / 1920px에서 가로 스크롤 없는지 확인.
8. 7장 접근성 체크리스트 확인.

---

## 9. 시안 보드 매핑

| 화면 | PC 보드 | MO 보드 | 반응형 |
|---|---|---|---|
| 메인 | Main (1440), Main_1920 | Mobile | Web_Main |
| 소개 | Sub_About_Overview / Vision / History / Location _PC | …_MO | Web_About_* |
| 사업영역 | Sub_Biz_Quant / Qual / Bigdata / Strategy / Evaluation _PC | …_MO | Web_Biz_* |
| 파트너 | Sub_Partner_Gov_PC | …_MO | Web_Partner_Gov |
| 수행실적 | Sub_Performance_PC, View_PC, View_NoCover_PC | …_MO, Filter_MO | Web_Performance* |
| 인사이트 | Sub_Insight_List_PC, View_PC | …_MO | Web_Insight_* |
| 문의 | Sub_Contact_PC | …_MO | Web_Contact |
| 개인정보처리방침 | Sub_Privacy_PC | …_MO | Web_Privacy |
| 스크롤 고정 규칙 | Sub_Sticky_Rules | – | – |

---

## 10. 확인이 필요한 콘텐츠 (디자인 외)

- 수행실적 건수: 사이트 523 vs 엑셀 518(업로드 O)/522(전체).
- 대표번호 끝자리 3889 / 3890.
- 개인정보처리방침: 시행일, 보유 기간 등 `[ ]` 자리표시자.
- 인사이트 게시글 수·본문은 샘플.
