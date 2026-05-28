# Thyroid K-TIRADS Tool — Session Handoff

## 프로젝트 개요
- **파일**: `/home/user/hello-world/Thyroid_K-TIRADS.html` (단일 파일 웹앱, ~2300줄)
- **브랜치**: `claude/website-to-local-code-xJ18U`
- **라이브 URL**: https://limhunjong.github.io/hello-world/Thyroid_K-TIRADS.html
- **배포**: `git push origin claude/website-to-local-code-xJ18U:gh-pages` 로 gh-pages에 자동 반영
- **localStorage 키**: `thyroidTool_v5`

---

## 기술 스택
- Vanilla HTML/CSS/JS 단일 파일 (프레임워크 없음)
- iOS Dark mode 기본, CSS custom properties로 Light mode 지원 (`body.light`)
- SF Symbols 스타일 SVG 아이콘 (테마 토글 버튼)
- `touch-action: manipulation` + `inputmode="decimal"` for iOS

---

## CSS 디자인 토큰 (`:root`)
```css
--bg0: #000  --bg1: #1C1C1E  --bg2: #2C2C2E  --bg3: #3A3A3C
--label1~4, --fill1~4, --sep, --blue, --red, --green, --orange
--r-xs~xxl (border-radius), --shadow-card, --shadow-btn, --shadow-seg
```
Light mode: `body.light { --bg0: #F2F2F7; --bg1: #FFF; --label1: #000; ... }`

---

## 주요 상태 구조 (`defaultNodule()`)
```javascript
{
  locationUpper: false, locationMiddle: false, locationLower: false,  // 다중선택
  diamAP: '', diamT: '', diamL: '', diamUnit: 'mm',                   // 기본 mm
  composition: '', echogenicity: '', orientation: '', margin: '',
  calcification_micro: false, calcification_rim: false,              // 다중선택
  calcification_macro: false, calcification_entire: false,           // entire는 단독
  vascularity_none: false, vascularity_peri: false,                   // 다중선택
  vascularity_mild: false, vascularity_marked: false,                 // mild↔marked 배타
  spongiform: '', cometTailArtifact: '',
  biopsyDone: false, biopsyFNA: false, biopsyCNB: false,             // 독립 선택
  fnaGauge: '', cnbSize: '',
  compNo: true, compYes: false, compIntra: false, compPeri: false, compOthers: false,
  recommendation: ''
}
```

---

## 완료된 작업 목록

### UI/UX
- [x] 모든 라디오 그룹 — 다시 클릭 시 선택 해제 (toggle)
- [x] `setupStaticRadioToggle` — double-fire 버그 수정 (`e.target.tagName === 'INPUT'` guard + state 비교)
- [x] Apple HIG 전체 리디자인 (CSS token system, glass morphism nav, 44pt touch targets)
- [x] 다크/라이트 모드 토글 (우상단 SVG 아이콘 버튼, localStorage 저장)
- [x] 하단 푸터: "Hunjong Lim, MD · Thyroid Ultrasound Report Tool · K-TIRADS 2021 · © 2025"

### 기능
- [x] **Calcification** → 다중선택 체크박스 (`calcification_micro/rim/macro/entire`)
  - `entirely calcified` ↔ 나머지 상호 배타
- [x] **Vascularity** → 다중선택 체크박스
  - `none` ↔ 나머지 상호 배타
  - `mild intranodular` ↔ `marked intranodular` 상호 배타
- [x] **Location** → 다중선택 (`locationUpper/Middle/Lower`)
  - 판독문: Upper+Middle→"Upper to middle", Middle+Lower→"Middle to lower", Upper+Lower→"Upper to lower"
- [x] **Diameter** 기본 단위 → mm (cm 선택 가능)
- [x] **Biopsy** → FNA/CNB 독립 선택; 입력창 뒤에 단위(G/cm) 표시; 여백 개선
- [x] **Spongiform / Comet tail / Normal Thyroid** → radio-group 버튼 스타일 (체크박스 → 텍스트 하이라이트)
- [x] **Complications** No/Yes 및 하위 항목 → radio-group 버튼 스타일
- [x] **Normal Thyroid** 활성화 시 선택 항목 시각적 해제, 비활성화 시 상태 복원; `opacity:0.38` 그레이아웃
- [x] **Date select** 미래 연도 제거 (현재 연도~2006 동적 생성)
- [x] **Tab 키** 직경 입력(AP→T→L) 간 이동 시 re-render 방지 (`relatedTarget` 체크)
- [x] **Nodule/LN 테이블** 동적 생성 요소 → CSS class 사용 (하드코딩 색상 제거, 테마 대응)
- [x] **판독문 다이얼로그** 라이트 모드 글자 보임 (`color: var(--label1)`)

### 마이그레이션 로직 (`loadState()`)
- 기존 `calcification: 'micro'` 문자열 → `calcification_micro: true` 등 boolean 변환
- 기존 `vascularity: 'none'` 문자열 → boolean 변환
- 기존 `location: 'Upper'` 문자열 → `locationUpper: true` 변환
- 기존 `biopsyType: 'FNA'` 문자열 → `biopsyFNA: true` 변환

---

## 핵심 함수 위치 (대략적 줄 번호)
| 함수 | 위치 |
|---|---|
| `defaultNodule()` | ~870 |
| `loadState()` + migration | ~900 |
| `renderAll()` | ~910 |
| `applyNormalThyroidDisable()` | ~990 |
| `buildNoduleTable()` | ~1090 |
| Location UI | ~1200 |
| Diameter UI | ~1215 |
| Calcification UI | ~1370 |
| Vascularity UI | ~1402 |
| Spongiform/Comet UI | ~1425 |
| Biopsy UI | ~1532 |
| `hasSuspiciousFeature()` | ~1650 |
| `getKTIRADS()` | ~1668 |
| `getLocationText()` | ~2175 |
| `buildBiopsyLines()` | ~2185 |
| `setupStaticRadioToggle()` | ~2210 |
| `toggleTheme()` + ICON_MOON/ICON_SUN | ~2260 |

---

## 알려진 미해결 이슈 / 다음 세션 후보 작업
- (없음 — 현재까지 보고된 모든 버그 수정 완료)
- 사용자가 새로 요청하는 기능에 따라 진행

---

## 빠른 시작 명령
```bash
# 작업 위치
cd /home/user/hello-world

# 현재 브랜치 확인
git status

# 수정 후 배포
git add Thyroid_K-TIRADS.html
git commit -m "..."
git push -u origin claude/website-to-local-code-xJ18U
git push origin claude/website-to-local-code-xJ18U:gh-pages
```
