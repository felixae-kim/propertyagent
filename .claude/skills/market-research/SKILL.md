---
name: market-research
description: 시장 분석 프레임워크, 경쟁사 분석 템플릿, Opportunity Scoring, TAM/SAM/SOM 산정 가이드를 제공합니다.
user_invocable: false
---

# Market Research Frameworks

## 1. TAM/SAM/SOM Analysis

### TAM (Total Addressable Market)
전체 시장 규모. "이 문제를 겪는 모든 사람이 우리 솔루션을 쓴다면?"
- Top-down: 산업 리포트 기반 전체 시장 규모
- Bottom-up: 잠재 고객 수 × 객단가

### SAM (Serviceable Addressable Market)
우리가 실제로 도달 가능한 시장.
- 지역, 언어, 채널, 기술 접근성 필터

### SOM (Serviceable Obtainable Market)
현실적으로 초기에 획득 가능한 시장.
- 초기 타겟 세그먼트 × 예상 점유율

## 2. Competitor Analysis Template

### Feature Matrix
```
| Feature Category | Feature | Us | Competitor A | Competitor B |
|-----------------|---------|-----|-------------|-------------|
| 핵심 기능 | Feature 1 | ✅ | ✅ | ❌ |
| | Feature 2 | 🔧 | ✅ | ✅ |
| 차별화 | Feature 3 | ✅ | ❌ | ❌ |

✅ = Have  ❌ = Don't Have  🔧 = Partial/Planned
```

### Positioning Map
2x2 매트릭스로 포지셔닝 시각화:
- X축: [차원1 — 예: 간편성 ↔ 전문성]
- Y축: [차원2 — 예: 저가 ↔ 고가]

### Competitor Deep Dive
```
## [Competitor Name]
- **Founded**: 연도
- **Users**: 규모
- **Revenue Model**: 수익 모델
- **Core Value Prop**: 한 줄 요약
- **Strengths**: ...
- **Weaknesses**: ...
- **User Sentiment** (from reviews):
  - 긍정: "..."
  - 부정: "..."
- **Applicable Lesson**: 우리가 배울 점
```

## 3. Opportunity Scoring (ODI Framework)

Anthony Ulwick의 Outcome-Driven Innovation:

```
Opportunity Score = Importance + max(Importance − Satisfaction, 0)
```

- **Importance** (1-10): 사용자에게 이 니즈가 얼마나 중요한가?
- **Satisfaction** (1-10): 현재 솔루션이 이 니즈를 얼마나 만족시키는가?

**해석:**
- Score > 15: Over-served (기회 낮음)
- Score 10-15: Appropriately served
- Score < 10: Under-served (기회 높음!)

## 4. User Segmentation Framework

### Behavioral Segmentation
| Segment | Behavior Pattern | Frequency | Channel | Willingness to Pay |
|---------|-----------------|-----------|---------|-------------------|
| Power Users | ... | Daily | ... | High |
| Casual Users | ... | Weekly | ... | Low |
| Churned Users | ... | None | ... | - |

### Jobs-to-be-Done Segments
| Segment | Primary Job | Secondary Job | Current Solution | Satisfaction |
|---------|-----------|--------------|-----------------|-------------|
| ... | ... | ... | ... | Low/Med/High |

## 5. Trend Analysis Template
| Trend | Category | Impact (1-5) | Timeframe | Opportunity | Threat |
|-------|----------|-------------|-----------|------------|--------|
| ... | Tech/Social/Regulatory | ... | Near/Mid/Long | ... | ... |
