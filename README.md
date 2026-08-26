# 🐯 TIGERMARUTCHI — 프로젝트 문서

> 싱글 HTML 파일 기반의 호랑이 다마고치 게임  
> 라이브: https://limhunjong.github.io/hello-world/

---

## 프로젝트 개요

빌드 도구 없이 `index.html` 한 파일로 동작하는 픽셀아트 타마고치 게임.  
Bandai 스타일 오렌지색 기기 디자인, SVG 픽셀아트 호랑이 캐릭터, 8단계 성장 시스템, 로그인/회원가입, 코인 경제, 미니게임, 상점을 포함.

---

## 기술 스택

| 항목 | 내용 |
|------|------|
| 언어 | HTML + CSS + Vanilla JS (빌드 없음) |
| 폰트 | Google Fonts — Press Start 2P, Jua |
| 렌더링 | SVG 픽셀아트 (`<rect>` 기반 pixel grid) |
| 저장 | `localStorage` (계정별, 세션별) |
| 배포 | GitHub Pages (`main` 브랜치 → `/index.html`) |

---

## 아키텍처

### 픽셀아트 시스템

```
const PX = 4;            // 1 논리 픽셀 = 4 SVG 단위
viewBox = "0 0 56 88"    // 14w × 22h 그리드
```

- `tigerPixelArt(mood, frame, equipped, stage)` — 서있는 호랑이 (모든 기분)
- `tigerSleepArt(frame, equipped, stage)` — 누워 자는 호랑이
- `eggPixelArt(ticks)` — 알 (stage 0)
- `accessoryLayer(id, ...)` / `sleepAccessoryLayer(id, ...)` — 아이템 오버레이

### 색상 팔레트

```js
C_O  = '#E88018'  // 밝은 오렌지 몸통
C_D  = '#2A1408'  // 짙은 갈색 줄무늬/윤곽
C_F  = '#FAF0D0'  // 크림색 얼굴/배
C_N  = '#E84858'  // 분홍 코
C_I  = '#F4B060'  // 귀 안쪽
C_SH = '#C06010'  // 그림자 음영
```

### 애니메이션

- `animFrame = 0 | 1` — 2프레임 루프
- `ANIM_SPEED` 오브젝트로 기분별 속도 제어 (idle 680ms ~ sleeping 1900ms)
- `startAnim()` — 기분/스테이지 변경 감지 후 재시작

---

## 8단계 성장 시스템

### STAGE_CFGS (핵심 설정)

```js
// [svgW, svgH, bodyStripes(0-3), headStripes(0-3), cssFilter, cane]
null,                                          // 0: 알
[24,38,  0, 0, '', ''],                        // 1: 아기 호랑이
[30,48,  1, 1, '', ''],                        // 2: 유아 호랑이
[36,57,  2, 2, '', ''],                        // 3: 어린이 호랑이
[42,66,  3, 2, '', ''],                        // 4: 청소년 호랑이
[47,74,  3, 3, '', ''],                        // 5: 대학생 호랑이
[52,82,  3, 3, '', ''],                        // 6: 청년 호랑이 (현재 성체)
[52,82,  3, 3, 'sepia(0.3) brightness(0.95)', ''],      // 7: 중년 호랑이
[50,79,  3, 3, 'grayscale(0.55) brightness(0.88)', 'cane'], // 8: 노인 호랑이
```

### 진화 타임라인

```js
const EV_TICKS = [0, 20, 200, 450, 900, 1800, 3600, 7200, Infinity];
//                   1   2    3    4     5     6     7      8
// 1tick = 1초, 예: stage 6→7은 3600초(60분)
```

### 성장 포인트

| 스테이지 | 이벤트 |
|----------|--------|
| 3 (어린이) | Branch 결정 (먹이 이력 기반) |
| 6 (청년) | 직업(Career) 확정 |

---

## 게임 스탯 & 브랜치

### 핵심 스탯

`hunger / happy / health / energy / clean` — 시간에 따라 감소, 행동으로 회복

### 특기 스탯 (직업 결정용)

`intellect / art / justice / tech / money` — 먹이·운동·시간으로 누적

### 브랜치 (어린이 단계에서 결정)

| 브랜치 | 결정 기준 | 누적 스탯 |
|--------|----------|-----------|
| Service | 고기 🍖 가장 많이 먹음 | art +0.06, justice +0.04/tick |
| Creative | 해산물 🐟 가장 많이 먹음 | justice +0.08, intellect +0.05/tick |
| Business | 분유 🍼 가장 많이 먹음 | money +0.08, tech +0.06/tick |

---

## 직업 유니버스 (Career Universe)

| 직업 | 이모지 | 브랜치 | 조건 |
|------|--------|--------|------|
| 의사 | 🩺 | Service | intellect≥25 |
| 경찰 | 👮 | Service | justice≥25 |
| 소방관 | 🚒 | Service | 기본 |
| 영화배우 | 🎬 | Service | art≥25 |
| 패션디자이너 | 👗 | Service | art≥35, 실수≤2 |
| 음악가 | 🎵 | Service | art≥30 |
| 변호사 | ⚖️ | Creative | justice≥30, intellect≥20 |
| 판사 | 🔨 | Creative | justice≥40, intellect≥30, 실수≤1 |
| CEO | 💼 | Business | money≥30, intellect≥20 |
| 투자은행가 | 📈 | Business | money≥35, intellect≥25 |
| 로켓과학자 | 🚀 | Business | tech≥35, intellect≥30 |
| 🔒 우주비행사 | 👨‍🚀 | Hidden | intellect≥40, tech≥40, 실수=0 |
| 🔒 해양탐험가 | 🌊 | Hidden | art≥35, justice≥35, 실수≤1 |
| 🔒 전설의 호랑이 | 🌟 | Hidden | 모든 스탯≥50, 실수=0 |

---

## 상점 & 코인 시스템

### 코인 획득

| 방법 | 획득량 |
|------|--------|
| 미니게임 | score × 0.8 |
| 운동 (10초 탭) | taps × 0.7 |
| 호랑이 쓰다듬기 | 20% 확률로 +1 |

### 코스튬 (인벤토리에 저장, 착용 가능)

| 아이템 | 이모지 | 가격 |
|--------|--------|------|
| 탑햇 | 🎩 | 50 |
| 선글라스 | 🕶️ | 40 |
| 리본 | 🎀 | 30 |
| 황금 왕관 | 👑 | 100 |
| 목도리 | 🧣 | 45 |
| 별 스티커 | ⭐ | 25 |
| 꽃 장식 | 🌸 | 35 |

### 프리미엄 음식 (즉시 효과)

| 음식 | 가격 | 효과 |
|------|------|------|
| 도시락 🍱 | 20 | hunger+50, happy+10 |
| 딸기 케이크 🍰 | 30 | hunger+20, happy+35 |
| 프리미엄 스테이크 🥩 | 35 | hunger+60, intellect+3 |
| 초밥 세트 🍣 | 40 | hunger+55, tech+3 |
| 특제 라면 🍜 | 15 | hunger+45, happy+15 |
| 아이스크림 🍦 | 22 | hunger+10, happy+40, art+2 |

---

## 미니게임

### 음식 캐치 게임

- 타이거 래퍼를 길게 누르면 (600ms) 실행
- 15초 동안 떨어지는 음식 아이템 탭
- 결과: happy + art 누적, 코인 획득

### 운동 게임

- 상점 탭에서 실행
- 10초 동안 탭 횟수 측정
- 결과: 코인 + happy + health + intellect 획득

---

## 계정 시스템

```js
localStorage key: 'hrtama_accounts'   // { [id]: { pw: hash, created } }
localStorage key: 'hrtama_session'    // 현재 로그인된 ID
localStorage key: 'hrtama_v8_<id>'   // 게임 세이브 데이터
```

- 비밀번호: `Math.imul` 기반 해시 (로컬 게임용)
- 자동 로그인: 세션 키 확인
- 계정별 분리 저장: 여러 계정에서 각자 다른 호랑이 키우기 가능

---

## 게임 상태 (newGame)

```js
{
  stage, age, ticks,
  hunger, happy, health, energy, clean,        // 핵심 스탯
  intellect, art, justice, tech, money,         // 특기 스탯
  feedLog: { meat, seafood, formula },          // 먹이 이력
  careMistakes, branch, careerId,              // 성장 결과
  alive, sleeping,
  name, soundOn, coins,
  collection, inventory, equipped,             // 코인/아이템
  pw, saved, lastFood
}
```

---

## UI 구조

```
[Device 외관 (오렌지 타마고치 기기)]
  ├── BANDAI NAMCO 뱃지
  ├── TIGERMARUTCHI 타이틀
  ├── Screen
  │   ├── 메시지 바 (msg1 / msg2)
  │   ├── View 0: 메인 씬 (호랑이 + 아이템 패널)
  │   ├── View 1: 스탯 바
  │   ├── View 2: Career DEX (컬렉션)
  │   ├── View 3: 상점 (코스튬/음식/착용)
  │   └── EXP 바 + 스테이지 레이블 + 코인 표시
  └── 버튼 3개: Feed / Status(순환) / Clean
```

---

## 향후 개발 아이디어

- [ ] 스테이지별 다른 배경 씬 (아기: 방, 청소년: 학교, 청년: 도시...)
- [ ] 친구 방문 기능 (같은 기기 다른 계정끼리 상호작용)
- [ ] 계절/날씨 이벤트
- [ ] 호랑이 간의 배틀 미니게임
- [ ] 스테이지별 전용 코스튬
- [ ] 직업별 직장 미니게임 (의사: 진단 퀴즈, CEO: 경영 퀴즈...)
- [ ] 서버 저장 + 랭킹 (현재는 localStorage만 사용)
- [ ] 다국어 지원 (현재 한/영 혼합)

---

## 파일 구조

```
hello-world/
└── index.html     # 전체 게임 (HTML + CSS + JS 모두 포함, ~1700줄)
```

---

## 개발 히스토리

| 버전 | 주요 변경 |
|------|----------|
| v1-v5 | 초기 타마고치 프레임 구축 |
| v6 | 픽셀아트 호랑이 (참고 이미지 기반 재디자인) |
| v7 | 로그인/회원가입 + 상점 + 코스튬 + 운동 미니게임 |
| v8 | **8단계 성장 시스템** (아기→노인, 시각적 크기/줄무늬/색상 변화) |
