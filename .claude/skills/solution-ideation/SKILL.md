---
name: solution-ideation
description: 솔루션 도출 프레임워크, 아이디에이션 기법, 가설 수립 템플릿, 사용자 경험 설계, 기능 정의 가이드를 제공합니다.
user_invocable: false
---

# Solution Ideation Frameworks

## 1. How Might We (HMW)

문제를 기회로 전환하는 질문 기법:

**공식**: "어떻게 하면 [사용자]가 [상황]에서 [원하는 결과]를 얻을 수 있을까?"

### HMW 변형
- **HMW 넓히기**: "어떻게 하면 모든 사용자가..." → 스코프 확장
- **HMW 좁히기**: "어떻게 하면 첫 방문 사용자가..." → 스코프 축소
- **HMW 뒤집기**: "어떻게 하면 이 문제가 더 심해질까?" → 반대로

## 2. SCAMPER Technique

| Letter | Meaning | Question |
|--------|---------|----------|
| S | Substitute | 무엇을 대체할 수 있을까? |
| C | Combine | 무엇을 결합할 수 있을까? |
| A | Adapt | 다른 분야에서 차용할 수 있는 것은? |
| M | Modify | 확대, 축소, 변형하면? |
| P | Put to other uses | 다른 용도로 쓸 수 있을까? |
| E | Eliminate | 제거하면 어떻게 될까? |
| R | Reverse | 순서/구조를 뒤집으면? |

## 3. Crazy 8s (Text Version)

8분 안에 8개 아이디어 — 판단 없이 양으로 승부:
```
Idea 1: [가장 명백한 해결책]
Idea 2: [반대 방향의 해결책]
Idea 3: [기술로 자동화하는 해결책]
Idea 4: [사람이 직접 하는 해결책]
Idea 5: [다른 산업에서 빌려온 해결책]
Idea 6: [극단적으로 단순화한 해결책]
Idea 7: [돈을 무한대로 쓸 수 있다면?]
Idea 8: [제약이 하나도 없다면?]
```

## 4. Idea Evaluation Matrix

### Impact-Effort Matrix
```
        High Impact
            │
    Quick   │   Big Bets
    Wins    │   (strategic)
  ──────────┼──────────
    Fill-ins│   Money
    (defer) │   Pit (avoid)
            │
        Low Impact
  Low Effort ──── High Effort
```

### Weighted Scoring
| Criteria | Weight | Idea A | Idea B | Idea C |
|----------|--------|--------|--------|--------|
| User Impact | 30% | 4 (1.2) | 3 (0.9) | 5 (1.5) |
| Feasibility | 25% | 5 (1.25) | 4 (1.0) | 2 (0.5) |
| Speed to Market | 20% | 3 (0.6) | 5 (1.0) | 2 (0.4) |
| Strategic Fit | 15% | 4 (0.6) | 3 (0.45) | 4 (0.6) |
| Innovation | 10% | 3 (0.3) | 2 (0.2) | 5 (0.5) |
| **Total** | | **3.95** | **3.55** | **3.50** |

## 5. Hypothesis Template

### Lean Hypothesis
```
우리는 [변화/솔루션]을 통해
[타겟 사용자]가 [원하는 행동]을 하게 되어
[측정 가능한 결과]를 달성할 것이라고 믿는다.

이것이 참인지 확인하기 위해,
[실험/MVP]를 실행하여
[성공 기준]을 [기간] 내에 측정할 것이다.
```

### Hypothesis Components
- **Independent Variable (인풋)**: 우리가 바꾸는 것
- **Dependent Variable (아웃풋)**: 바뀔 것으로 기대하는 것
- **Success Criteria**: 가설이 참이라고 판단하는 기준
- **Falsification Criteria**: 가설이 거짓이라고 판단하는 기준

## 6. User Experience Design Template

### Experience Map
```
Stage: 인지 → 진입 → 핵심 경험 → 반복 → 추천
Touch: [채널] [화면] [기능] [알림] [공유]
Feel:  호기심  기대감  "아하!"  습관화  만족감
```

### Aha Moment Definition
- **What**: 사용자가 솔루션의 핵심 가치를 체감하는 순간
- **When**: 사용 시작 후 [X] 이내
- **Metric**: [행동 지표]가 [기준] 이상이면 Aha 도달

## 7. Feature Prioritization (MoSCoW)

| Priority | Label | Definition |
|----------|-------|-----------|
| Must Have | 🔴 | 이것 없으면 솔루션이 아님 |
| Should Have | 🟡 | 중요하지만 workaround 가능 |
| Could Have | 🟢 | 있으면 좋지만 없어도 됨 |
| Won't Have (this time) | ⚪ | 이번 스코프에서 제외 |

## 8. Solution Benchmarking Template

### Reference Case Study
```
## [서비스/제품명]

### 해결한 문제
- 원래 문제: ...
- 타겟 사용자: ...

### 솔루션 접근
- 핵심 메커니즘: ...
- 차별화 요소: ...

### 결과
- 정량적: ...
- 정성적: ...

### 우리에게 적용 가능한 인사이트
- 차용할 점: ...
- 피할 점: ...
- 우리만의 변형: ...
```
