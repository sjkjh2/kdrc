# KDRC 테마 — Claude Code 작업 규칙

WordPress 테마 `wp-content/themes/kdrc/`. 1rem = 10px (reset.css).

## UI 작업 전 필수
- `DESIGN.md`를 먼저 읽고 그 규칙을 따른다.
- 색·폰트·행간·여백·라운드·그림자는 `css/tokens.css` 변수만 사용한다. 새 값이 필요하면 tokens.css에 토큰을 추가하고 DESIGN.md 표도 갱신한다.
- common.css의 `--fs-N`, `--sp-N`(clamp 유동값), `--kdrc-link`(청록)는 레거시 — 새 코드에서 쓰지 않는다.

## 절대 규칙
- 모든 수치는 짝수 px(rem 환산). 예외: 1px 선, pill 999.9rem, 분기점 720/721.
- line-height는 `--lh-*` 토큰(px). 배수(1.6 등) 금지.
- 반응형 분기점은 `@media (max-width: 720px)` 하나. PC 기본 → 모바일 덮어쓰기.
- 섹션 배경 전폭, 콘텐츠는 `padding-inline: var(--gutter)`로 1200px 중앙.
- 카드 라운드는 `var(--radius-card)`(오른쪽 위만). 입력칸·표는 각지게.
- 버튼은 공통 텍스트 버튼 `.kdrc-tbtn`(텍스트 + 빨간 원). 모바일에서는 항상 오른쪽 정렬.
- 클래스는 `kdrc-` 접두사, 변형 `--`, 상태 `is-*`.
- 접근성: 시맨틱 태그, `aria-current`, `aria-expanded`, `aria-pressed`, 포커스 표시 유지, 터치 영역 44px, 흰 배경 텍스트는 #6B7280보다 연하게 쓰지 않음, reduced-motion 대응.

## 작업 후 셀프 점검
```bash
grep -nE '#[0-9a-fA-F]{3,6}' css/<수정한 파일>.css      # tokens.css 외 hex 없어야 함
grep -nE '[^0-9.][0-9]*[13579]px' css/<수정한 파일>.css # 홀수 px 없어야 함 (1px 제외)
grep -nE 'line-height:\s*[0-9.]+;' css/<수정한 파일>.css  # 배수 행간 없어야 함
```
360 / 390 / 720 / 721 / 1024 / 1440 / 1920px에서 가로 스크롤이 없는지 확인한다.
