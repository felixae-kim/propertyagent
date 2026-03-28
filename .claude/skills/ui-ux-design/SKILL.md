---
name: ui-ux-design
description: UI/UX 디자인 수행을 위한 프레임워크. 비주얼 스타일 데이터베이스, 타이포그래피/컬러 인텔리전스, 디자인 시스템 설계, 와이어프레임 명세, IA 설계, 인터랙션 디자인, 접근성 프레임워크, 프롭테크 특화 패턴을 제공합니다.
---

# UI/UX Design Skill

## 0. Visual Style Database

프로젝트 성격에 따라 아래에서 적절한 스타일을 선택하거나 조합한다.

### PropTech / Real Estate 추천 스타일
| Style | 특성 | 적합한 경우 |
|-------|------|------------|
| **Clean Minimal** | 넓은 여백, 선형 아이콘, 단색 강조 | 매물 리스팅, 검색 중심 |
| **Map-Centric** | 지도가 히어로, 필터 오버레이, 핀 인터랙션 | 위치 기반 탐색 |
| **Card-Rich Editorial** | 큰 이미지 카드, 매거진 레이아웃 | 매물 브라우징, 영감형 |
| **Dashboard Analytical** | 데이터 테이블, 차트, KPI 카드 | 투자/관리 도구 |
| **Spatial/3D** | 평면도 뷰, 층별 네비게이션, VR 연결 | 가상 투어, 공간 탐색 |
| **Trust-First** | 인증 배지, 리뷰 스코어, 실거래가 강조 | 중개/계약 플랫폼 |

### 범용 비주얼 스타일 참조
| Style | 설명 | 레퍼런스 |
|-------|------|----------|
| Glassmorphism | 반투명 블러 배경, 미묘한 보더 | iOS 위젯, Linear |
| Bento Grid | 다양한 크기의 카드가 조합된 그리드 | Apple, Vercel |
| Neumorphism | 소프트 그림자, 돌출/함몰 효과 | 제한적 사용 권장 (접근성 주의) |
| Brutalism | 굵은 타이포, 노출 구조, 원색 | 브랜드 차별화가 중요할 때 |
| AI-Native UI | 대화형 입력, 스트리밍 응답, 컨텍스트 패널 | AI 기능이 핵심일 때 |
| Dark Mode Premium | 어두운 배경, 네온/골드 액센트 | 프리미엄/럭셔리 포지셔닝 |

### Font Pairing Guide
| 용도 | Heading (영문) | Body (영문) | 한글 | 성격 |
|------|---------------|-------------|------|------|
| 현대적/기능적 | Satoshi, General Sans | DM Sans | Pretendard | 테크, SaaS |
| 따뜻한/친근한 | Recoleta, Fraunces | Source Sans 3 | Spoqa Han Sans | 생활 서비스, 커뮤니티 |
| 대담한/강렬한 | Clash Display, Cabinet Grotesk | Space Grotesk | Pretendard (Bold 활용) | 브랜드 차별화 |
| 신뢰/전문적 | Instrument Serif, Newsreader | IBM Plex Sans | Noto Sans KR | 금융, 법률, 계약 |
| 세련된/럭셔리 | Cormorant, Playfair Display | Outfit | Pretendard | 프리미엄 부동산 |

### PropTech Color Palette Presets
| Palette | Primary | Accent | Neutral | 적합한 프로덕트 |
|---------|---------|--------|---------|----------------|
| **Urban Trust** | Slate Blue #4A5568 | Warm Gold #D69E2E | Cool Gray | 계약/중개 플랫폼 |
| **Green Living** | Forest #2F855A | Sunrise #ED8936 | Warm Gray | 친환경/생활 서비스 |
| **Modern Space** | Deep Navy #1A365D | Electric Teal #38B2AC | Blue Gray | 공간 관리/인테리어 |
| **Warm Home** | Terracotta #C05621 | Sage #68D391 | Warm Beige | 주거/커뮤니티 |
| **Premium Estate** | Charcoal #1A202C | Champagne Gold #D4AF37 | Dark Neutral | 럭셔리/프리미엄 |
| **Smart Invest** | Royal Purple #553C9A | Lime #48BB78 | Neutral Gray | 투자/분석 도구 |

---

## 1. 디자인 프로세스

```
Exploration → Research → IA(정보 구조) → User Flow → Wireframe → Visual Design → Prototype → Usability Test
```

각 단계에서 산출물을 명확히 정의하고, 이전 단계 피드백을 반영한 후 다음 단계로 이동.
**Exploration 단계**(에이전트의 Design Exploration Ritual)를 반드시 먼저 수행.

---

## 2. Information Architecture (IA) Template

### Sitemap
```
[서비스명]
├── Home
│   ├── Hero Section
│   ├── Key Features
│   └── CTA
├── Feature A
│   ├── List View
│   ├── Detail View
│   └── Create/Edit
├── Feature B
│   └── ...
├── Settings
│   ├── Profile
│   ├── Notifications
│   └── Billing
└── Auth
    ├── Sign In
    ├── Sign Up
    └── Password Reset
```

### Navigation Model
| Level | Type | Pattern | Example |
|-------|------|---------|---------|
| Global | Persistent | Top Nav / Side Nav | Home, Dashboard, Settings |
| Local | Contextual | Tab / Breadcrumb | Feature 하위 메뉴 |
| Utility | Always Available | Header Right | Profile, Notifications, Logout |

---

## 3. User Flow Diagram Template

```
[시작점] ─── User Action ──→ [화면/상태]
                                  │
                            ┌─────┴─────┐
                     [조건 A]          [조건 B]
                        │                │
                   [화면 A]          [화면 B]
                        │                │
                   [결과 A]          [결과 B]
```

### Flow 문서화 형식
```markdown
## Flow: [플로우명]
**목적**: [사용자가 달성하려는 것]
**시작점**: [어디서 시작]
**종료점**: [성공 시 어디로]

### Steps
1. [화면명] — 사용자: [행동] → 시스템: [응답]
2. [화면명] — 사용자: [행동] → 시스템: [응답]
   - 분기: [조건] → [대안 경로]
3. ...

### Error Paths
- [에러 조건] → [에러 화면/메시지] → [복구 경로]
```

---

## 4. Wireframe Spec Template

텍스트 기반 와이어프레임 명세:

```markdown
## Screen: [화면명]
**Route**: /path
**Purpose**: [이 화면이 존재하는 이유]
**Entry Points**: [어디서 진입]

### Layout (Desktop ≥1024px)
┌─────────────────────────────────────┐
│ [Header: Logo | Nav | Profile]      │
├──────────┬──────────────────────────┤
│ Sidebar  │  Main Content            │
│          │  ┌────────────────────┐  │
│ - Nav 1  │  │ Section Title      │  │
│ - Nav 2  │  │ [Content Area]     │  │
│ - Nav 3  │  │                    │  │
│          │  └────────────────────┘  │
│          │  ┌────────────────────┐  │
│          │  │ [CTA Button]       │  │
│          │  └────────────────────┘  │
├──────────┴──────────────────────────┤
│ [Footer]                            │
└─────────────────────────────────────┘

### Layout (Mobile <768px)
┌──────────────────┐
│ [Header + ☰]     │
├──────────────────┤
│ Section Title    │
│ [Content Area]   │
│ [CTA Button]     │
├──────────────────┤
│ [Bottom Nav]     │
└──────────────────┘

### Component Inventory
| Zone | Component | Type | Content | Interaction |
|------|-----------|------|---------|-------------|
| Header | Logo | Image | 브랜드 로고 | Click → Home |
| Header | Nav | Links | 주요 메뉴 | Hover → Underline |
| Main | DataList | List | 항목 리스트 | Click → Detail |
| Main | CTAButton | Button | "시작하기" | Click → [Flow] |

### States
| State | Visual | Copy |
|-------|--------|------|
| Default | [기본 레이아웃] | — |
| Empty | [일러스트 + 텍스트] | "아직 데이터가 없어요" |
| Loading | [Skeleton UI] | — |
| Error | [에러 아이콘 + 텍스트] | "문제가 발생했어요. 다시 시도해주세요." |
```

---

## 5. Design System Foundation

### Typography Scale
| Token | Size | Weight | Line Height | Use |
|-------|------|--------|-------------|-----|
| display-lg | 48px | Bold | 1.2 | Hero Headline |
| display-md | 36px | Bold | 1.2 | Page Title |
| heading-lg | 28px | Semibold | 1.3 | Section Title |
| heading-md | 22px | Semibold | 1.3 | Card Title |
| heading-sm | 18px | Semibold | 1.4 | Subsection |
| body-lg | 16px | Regular | 1.5 | Primary Text |
| body-md | 14px | Regular | 1.5 | Secondary Text |
| body-sm | 12px | Regular | 1.5 | Caption, Helper |
| label | 14px | Medium | 1.4 | Form Label |

### Typography Anti-Patterns (AVOID)
- Inter/Roboto/Arial을 "기본"으로 선택하지 마라 — 선택하지 않은 것이다
- Heading과 Body에 같은 font-weight 사용 — 계층이 무너진다
- font-size만으로 계층 구분 — weight, letter-spacing, color를 함께 써라
- 한글과 영문 폰트의 x-height 불일치 — 나란히 놓았을 때 어색해진다

### Spacing Scale (8px grid)
| Token | Value | Use |
|-------|-------|-----|
| space-1 | 4px | Inline icon gap |
| space-2 | 8px | Tight padding |
| space-3 | 12px | Input padding |
| space-4 | 16px | Card padding, stack gap |
| space-6 | 24px | Section gap |
| space-8 | 32px | Large section gap |
| space-12 | 48px | Page section divider |
| space-16 | 64px | Hero padding |

### Color System
```
Semantic Colors (CSS 변수로 정의 필수):
--color-primary: 브랜드 메인 — CTA, 링크, 선택 상태
--color-secondary: 보조 — 이차 버튼, 배지
--color-success: 성공 — 확인, 완료
--color-warning: 경고 — 주의, 경고
--color-error: 에러 — 오류, 삭제
--color-info: 정보 — 안내, 도움말

Neutral Palette:
--color-neutral-900: Primary text
--color-neutral-700: Secondary text
--color-neutral-500: Placeholder, disabled text
--color-neutral-300: Border
--color-neutral-100: Background (subtle)
--color-neutral-50: Background (card)
--color-white: Background (page)

Dark Mode Mapping (처음부터 고려):
--color-bg-primary: light(white) / dark(neutral-900)
--color-bg-secondary: light(neutral-50) / dark(neutral-800)
--color-text-primary: light(neutral-900) / dark(neutral-50)
--color-text-secondary: light(neutral-700) / dark(neutral-300)
```

### Color Anti-Patterns (AVOID)
- 하드코딩된 hex 값 (`#3B82F6`) — 항상 시맨틱 토큰 사용
- 5-6색을 고르게 분배한 "무지개" 팔레트 — 지배적 컬러 + 날카로운 액센트가 더 강하다
- blue-500/gray-100 조합의 "기본 SaaS" 컬러 — 도메인에서 컬러를 찾아라

### Core Components Spec

#### Button
| Variant | Use Case | Style |
|---------|----------|-------|
| Primary | 주요 CTA (1 per section) | Filled, primary color |
| Secondary | 보조 액션 | Outlined, primary color |
| Tertiary | 세 번째 액션 | Text only |
| Destructive | 삭제, 취소 | Filled/Outlined, error color |
| Ghost | 최소 강조 | Transparent, hover tint |

Sizes: sm (32px), md (40px), lg (48px)
States: default, hover, active, focus, disabled, loading

#### Input
| Type | Use Case |
|------|----------|
| Text | 일반 텍스트 입력 |
| Textarea | 긴 텍스트 |
| Select | 옵션 선택 |
| Checkbox | 다중 선택 |
| Radio | 단일 선택 |
| Toggle | On/Off |
| Date Picker | 날짜 선택 |
| File Upload | 파일 첨부 |

States: default, focused, filled, error, disabled, read-only
Anatomy: Label + Input + Helper/Error text

#### Card
```
┌─────────────────────────┐
│ [Thumbnail / Image]     │  ← Optional
├─────────────────────────┤
│ [Title]                 │
│ [Description]           │
│ [Metadata]              │
├─────────────────────────┤
│ [Actions]               │  ← Optional
└─────────────────────────┘
```

#### Modal / Dialog
```
┌─────────────────────────────┐
│ Title                    ✕  │
├─────────────────────────────┤
│                             │
│  Content Area               │
│                             │
├─────────────────────────────┤
│        [Cancel] [Confirm]   │
└─────────────────────────────┘
```
- Sizes: sm (400px), md (560px), lg (720px)
- Overlay: neutral-900 @ 50% opacity
- ESC / 오버레이 클릭으로 닫기 (destructive action은 예외)

---

## 6. Interaction Design Patterns

### Micro-interactions
| Trigger | Animation | Duration | Easing |
|---------|-----------|----------|--------|
| Button hover | Scale 1.02 + shadow | 150ms | ease-out |
| Button press | Scale 0.98 | 100ms | ease-in |
| Page transition | Fade + slide-up | 300ms | ease-in-out |
| Toast appear | Slide-in from top | 250ms | ease-out |
| Toast dismiss | Fade-out | 200ms | ease-in |
| Skeleton → Content | Fade crossfade | 200ms | ease-out |
| Modal open | Fade + scale from 0.95 | 250ms | ease-out |
| Modal close | Fade + scale to 0.95 | 200ms | ease-in |

### Feedback Patterns
| Action | Feedback Type | Example |
|--------|--------------|---------|
| Form submit 성공 | Toast (success) | "저장되었습니다" |
| Form validation error | Inline error | 필드 아래 빨간 텍스트 |
| Destructive action | Confirmation dialog | "정말 삭제하시겠습니까?" |
| 긴 작업 시작 | Progress indicator | 프로그레스 바 or 스피너 |
| 데이터 업데이트 | Optimistic UI | 즉시 반영 후 서버 확인 |

### Navigation Patterns
| Pattern | When to Use | Example |
|---------|-------------|---------|
| Stack (push/pop) | 계층적 탐색 | List → Detail → Edit |
| Tab | 동일 수준 병렬 콘텐츠 | Dashboard의 여러 뷰 |
| Wizard / Stepper | 순차적 입력 | 온보딩, 결제 플로우 |
| Drawer | 보조 정보 | 필터, 상세 정보 패널 |
| Command Palette | 파워유저 빠른 탐색 | Cmd+K → 검색/액션 |

---

## 7. 사용성 원칙 (Design Heuristics)

### Jakob Nielsen's 10 Heuristics 적용 체크리스트
- [ ] **Visibility of system status**: 로딩, 진행률, 저장 상태가 항상 표시되는가?
- [ ] **Match with real world**: 사용자 언어를 사용하는가? (기술 용어 X)
- [ ] **User control & freedom**: 실행 취소, 뒤로가기가 쉬운가?
- [ ] **Consistency & standards**: 같은 동작이 같은 결과를 내는가?
- [ ] **Error prevention**: 실수를 미리 방지하는가? (확인 대화상자, 입력 제한)
- [ ] **Recognition over recall**: 옵션을 보여주는가? (기억에 의존 X)
- [ ] **Flexibility & efficiency**: 초보자와 전문가 모두를 위한 경로가 있는가?
- [ ] **Aesthetic & minimalist**: 불필요한 정보가 없는가?
- [ ] **Help users recover from errors**: 에러 메시지가 해결책을 제시하는가?
- [ ] **Help & documentation**: 필요한 곳에 도움말이 있는가?

### 혁신적 UX를 위한 추가 원칙
- **Progressive Disclosure**: 복잡한 기능은 단계적으로 노출
- **Sensible Defaults**: 대부분의 사용자에게 맞는 기본값 설정
- **Undo over Confirm**: 확인 대화상자보다 되돌리기를 제공
- **Direct Manipulation**: 드래그, 인라인 편집 등 직접 조작 우선
- **Contextual Actions**: 관련 행동을 관련 콘텐츠 가까이에 배치
- **Zero State as Onboarding**: 빈 상태를 튜토리얼/가이드로 활용

---

## 8. Responsive Design Rules

### Breakpoints
| Name | Width | Columns | Gutter | Margin |
|------|-------|---------|--------|--------|
| Mobile | < 768px | 4 | 16px | 16px |
| Tablet | 768-1023px | 8 | 24px | 24px |
| Desktop | 1024-1439px | 12 | 24px | 32px |
| Wide | ≥ 1440px | 12 | 32px | auto (max-width: 1280px) |

### Responsive Patterns
| Pattern | Description | When |
|---------|-------------|------|
| Stack | 수평 → 수직 | 2-3 column → 1 column on mobile |
| Collapse | Nav → Hamburger | Desktop nav → mobile drawer |
| Reflow | Grid → List | Card grid → card list on mobile |
| Off-canvas | Panel → Drawer | Sidebar → slide-out on mobile |
| Priority+ | 보이는 항목 줄이기 | Tab overflow → "More" menu |
| Truncate | 텍스트 줄이기 | 긴 제목 → ellipsis on small screens |

---

## 9. 디자인 산출물 체크리스트

디자인 완료 시 다음 산출물을 모두 확인:

- [ ] **IA / Sitemap**: 전체 구조와 네비게이션 모델
- [ ] **User Flows**: 주요 플로우 다이어그램
- [ ] **Wireframes**: 모든 화면의 구조 명세 (Desktop + Mobile)
- [ ] **Visual Design**: 컬러, 타이포, 스페이싱 시스템 정의
- [ ] **Component Spec**: 모든 컴포넌트의 variant, state, interaction 정의
- [ ] **Interaction Spec**: 전환, 애니메이션, 마이크로인터랙션
- [ ] **Responsive Spec**: 각 브레이크포인트별 레이아웃 변화
- [ ] **Accessibility Spec**: 대비, 포커스, 스크린리더 고려사항
- [ ] **Content Spec**: 모든 카피, 에러 메시지, 빈 상태 문구
- [ ] **Edge Case Spec**: 극단적 상황의 UI 처리

---

## 10. 혁신적 디자인 패턴 레퍼런스

### AI-Native UX Patterns
| Pattern | Description | Example |
|---------|-------------|---------|
| Proactive Suggestions | 사용자 행동 예측하여 선제 제안 | "이 데이터를 차트로 볼까요?" |
| Conversational UI | 자연어 기반 인터랙션 | AI 어시스턴트, 챗 인터페이스 |
| Adaptive Interface | 사용 패턴에 따라 UI 자동 조정 | 자주 쓰는 기능 상단 배치 |
| Ambient Intelligence | 배경에서 자동으로 작업 수행 | 자동 분류, 스마트 알림 |
| Explain & Control | AI 결과에 이유 + 수정 옵션 제공 | "이렇게 추천한 이유: ..." |

### Delightful Moments
| Moment | Technique | Example |
|--------|-----------|---------|
| First Success | Celebration animation | Confetti on first task complete |
| Milestone | Progress visualization | "100번째 기록을 달성했어요!" |
| Empty → Full | Transformation | 빈 캔버스가 점차 채워지는 느낌 |
| Speed | Instant feedback | Optimistic UI, skeleton loading |
| Discovery | Easter egg | 숨겨진 단축키, 게이미피케이션 |

---

## 11. PropTech 특화 UX 패턴

### 매물 탐색
| Pattern | 설명 | 구현 노트 |
|---------|------|----------|
| Map + List Split View | 지도와 리스트가 동기화된 분할 화면 | 리스트 호버 → 지도 핀 하이라이트, 지도 이동 → 리스트 필터 |
| Smart Filter Bar | 핵심 필터(가격, 면적, 유형)를 상단에 항상 노출 | 고급 필터는 패널/시트로 분리 |
| Quick View Card | 카드 클릭 없이 호버/롱프레스로 핵심 정보 미리보기 | 사진 캐러셀, 가격, 면적, 위치 |
| Saved Search Alert | 조건 저장 → 새 매물 알림 | "이 조건으로 알림 받기" CTA |
| Virtual Tour Entry | 매물 카드에서 3D 투어로 원클릭 진입 | 별도 페이지 아닌 모달/풀스크린 오버레이 |

### 가격/금융 정보
| Pattern | 설명 | 구현 노트 |
|---------|------|----------|
| Price Anchoring | 실거래가, 시세, 호가를 시각적으로 비교 | 수평 바 차트 or 숫자 하이라이트 |
| Affordability Calculator | 내 예산 기준 매물 필터링 | 슬라이더 + 실시간 결과 업데이트 |
| Price Trend Sparkline | 매물 카드에 가격 추이 미니 차트 | 6개월~1년 추이, 상승/하락 색상 |
| Fee Breakdown | 중개수수료, 세금 등 총 비용 투명 표시 | 접을 수 있는 상세 내역 |

### 계약/프로세스
| Pattern | 설명 | 구현 노트 |
|---------|------|----------|
| Step Progress | 계약 진행 상태를 단계별 시각화 | 수평 스테퍼, 현재 단계 강조 |
| Document Checklist | 필요 서류 체크리스트 + 업로드 | 완료율 표시, 다음 할 일 하이라이트 |
| Timeline Feed | 계약 이벤트를 시간순 피드로 | 채팅형 or 타임라인 UI |
| e-Sign Integration | 서명 필요 문서 인라인 표시 | 서명 위치 하이라이트, 원터치 서명 |

### 공간/인테리어
| Pattern | 설명 | 구현 노트 |
|---------|------|----------|
| Floor Plan Interactive | 클릭 가능한 평면도 | 방 클릭 → 상세 정보 + 사진 |
| Before/After Slider | 인테리어 전후 비교 | 드래그 슬라이더로 비교 |
| Room-by-Room Gallery | 공간별로 분류된 사진 갤러리 | 탭 or 평면도 연동 |

---

## 12. Motion & Animation Spec (강화)

### High-Impact Moments (여기에 집중)
| Moment | Animation | Duration | Easing |
|--------|-----------|----------|--------|
| Page enter | Staggered fade-up (각 요소 50ms 딜레이) | 300-500ms | cubic-bezier(0.16, 1, 0.3, 1) |
| Data load complete | Skeleton → Content crossfade | 200ms | ease-out |
| Success confirmation | Check animation + scale bounce | 400ms | spring(1, 100, 10) |
| Card hover | Subtle lift (translateY -2px + shadow) | 150ms | ease-out |
| List item add | Slide-in from right + fade | 250ms | ease-out |
| List item remove | Slide-out left + fade + height collapse | 200ms | ease-in |
| Map pin drop | Scale from 0 + bounce | 350ms | spring |
| Price change | Number counter animation | 300ms | ease-in-out |

### Low-Impact (절제하라)
- 로고 회전, 배경 파티클, 무한 로딩 애니메이션
- 스크롤마다 트리거되는 parallax — 성능과 접근성 모두 해침
- 3개 이상의 동시 애니메이션 — 시선이 분산됨

### Framer Motion 코드 패턴 (Next.js/React)

#### Apple-Style Spring Motion (가장 자연스러운 모션)
```tsx
// 기본 스프링: 부드럽고 자연스러운 감속
const appleSpring = { type: "spring", stiffness: 300, damping: 30 };

// 바운시 스프링: 완료/성공 순간
const bouncySpring = { type: "spring", stiffness: 400, damping: 15 };

// 슬로우 스프링: 페이지 전환, 큰 요소 이동
const slowSpring = { type: "spring", stiffness: 100, damping: 20 };
```

#### Staggered List Animation
```tsx
const container = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: { staggerChildren: 0.05 }
  }
};
const item = {
  hidden: { opacity: 0, y: 20 },
  show: { opacity: 1, y: 0, transition: appleSpring }
};
```

#### Page Transition (App Router)
```tsx
const pageVariants = {
  initial: { opacity: 0, y: 8 },
  animate: { opacity: 1, y: 0, transition: { duration: 0.3, ease: [0.16, 1, 0.3, 1] } },
  exit: { opacity: 0, y: -4, transition: { duration: 0.2 } }
};
```

### CSS-Only Animation Patterns (JS 불필요 시)

#### 부드러운 등장 (Intersection Observer와 결합)
```css
.fade-up {
  opacity: 0;
  transform: translateY(12px);
  transition: opacity 0.4s ease-out, transform 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}
.fade-up.visible {
  opacity: 1;
  transform: translateY(0);
}
```

#### 카드 호버 (미세한 리프트)
```css
.card {
  transition: transform 0.15s ease-out, box-shadow 0.15s ease-out;
}
.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px -5px rgba(0, 0, 0, 0.1);
}
.card:active {
  transform: translateY(0) scale(0.99);
}
```

#### 스켈레톤 로딩 펄스
```css
.skeleton {
  background: linear-gradient(90deg, var(--color-neutral-100) 25%, var(--color-neutral-50) 50%, var(--color-neutral-100) 75%);
  background-size: 200% 100%;
  animation: skeleton-pulse 1.5s ease-in-out infinite;
}
@keyframes skeleton-pulse {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

### Easing Reference (자주 쓰는 커브)
| Name | Value | 용도 |
|------|-------|------|
| Apple Ease Out | `cubic-bezier(0.16, 1, 0.3, 1)` | 요소 등장, 페이지 진입 |
| Smooth Decel | `cubic-bezier(0.0, 0.0, 0.2, 1)` | 일반적인 트랜지션 |
| Snappy | `cubic-bezier(0.2, 0, 0, 1)` | 빠른 피드백 (토글, 체크) |
| Bounce Out | `cubic-bezier(0.34, 1.56, 0.64, 1)` | 완료/성공 바운스 |
| Gentle In-Out | `cubic-bezier(0.4, 0, 0.2, 1)` | 페이지 전환 |

### `prefers-reduced-motion` 대응 (필수)
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 13. Visual Standards & Design Quality Gate

디자인 산출물의 비주얼 품질을 자체 검증하는 프레임워크.

### Gestalt 원칙 적용 체크
| 원칙 | 질문 | 위반 시 |
|------|------|---------|
| Proximity | 관련 요소가 가까이 있는가? | 무관한 요소가 같은 그룹으로 인식됨 |
| Similarity | 같은 기능의 요소가 같은 스타일인가? | 사용자가 관계를 이해 못함 |
| Continuity | 시선이 자연스럽게 흐르는가? | 시선이 튀어 핵심을 놓침 |
| Closure | 불완전한 형태를 보완하여 인식할 수 있는가? | 아이콘/형태가 혼란스러움 |
| Figure-Ground | 전경과 배경이 명확히 구분되는가? | 중요 콘텐츠가 묻힘 |

### 3-Second Test
모든 화면에 적용: **3초 안에 이 화면이 무엇을 위한 것인지, 다음에 무엇을 해야 하는지 알 수 있어야 한다.**
3초 안에 목적이 불분명하면 → 계층 구조 재설계 필요.

### Visual Weight Balance
- 화면을 4등분했을 때, 시각적 무게가 한쪽으로 심하게 쏠리지 않는가?
- 의도적 비대칭은 좋지만, 우발적 불균형은 나쁘다
- CTA는 시각적 무게의 정점(focal point)에 위치해야 한다
