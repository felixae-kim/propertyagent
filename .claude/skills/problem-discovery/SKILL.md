---
name: problem-discovery
description: 사용자 문제 발견과 우선순위 결정에 필요한 프레임워크, 템플릿, 분석 기법을 제공합니다.
---

# Problem Discovery Skill

## Problem Statement Template
```
[사용자 세그먼트]는 [컨텍스트/상황]에서 [행동/작업]을 하려고 할 때,
[관찰 가능한 문제/고통점]을 경험하고 있다.
이로 인해 [사용자에게 미치는 영향]이 발생하며,
비즈니스 관점에서 [비즈니스 임팩트]로 이어진다.
```

## Jobs-to-Be-Done (JTBD) Framework
```
When [상황/트리거],
I want to [동기/목표],
So I can [기대하는 결과].
```

현재 이 Job을 수행하기 위해 사용자는:
- **현재 해결책**: [어떻게 하고 있는가]
- **불만족 요소**: [뭐가 불편한가]
- **전환 장벽**: [왜 아직 바꾸지 못했는가]

## Opportunity Scoring (Ulwick)
```
Opportunity Score = Importance + max(Importance - Satisfaction, 0)
```

| Job / Need | Importance (1-10) | Satisfaction (1-10) | Opportunity Score |
|------------|------------------|--------------------|--------------------|
| ... | ... | ... | ... |

- Score > 15: Over-served (가치 없음)
- Score 10-15: Appropriately served
- Score > 15 (Importance high, Satisfaction low): **Under-served (기회!)**

## RICE Scoring
```
RICE = (Reach × Impact × Confidence) ÷ Effort
```

| Factor | How to Estimate |
|--------|----------------|
| **Reach** | 분기당 영향받는 사용자 수 |
| **Impact** | 0.25 (최소) / 0.5 (낮음) / 1 (중간) / 2 (높음) / 3 (매우 높음) |
| **Confidence** | 100% (높음) / 80% (중간) / 50% (낮음) |
| **Effort** | person-weeks 단위 |

## ICE Scoring (빠른 우선순위용)
```
ICE = Impact × Confidence × Ease
```
각 항목 1-10 스케일. 빠른 판단에 적합하지만 RICE보다 주관적.

## Evidence Quality Matrix

| Level | Source | Confidence |
|-------|--------|------------|
| L1 (직접 관찰) | 사용자 인터뷰, 세션 녹화, 유저빌리티 테스트 | 높음 |
| L2 (정량 데이터) | 퍼널 분석, A/B 테스트 결과, 이탈 데이터 | 높음 |
| L3 (간접 피드백) | 서포트 티켓, 설문, NPS 코멘트 | 중간 |
| L4 (이해관계자 의견) | 내부 요청, 영업 피드백, 경쟁사 벤치마크 | 낮음 |
| L5 (직감) | "사용자가 원할 것 같다" | 매우 낮음 |

## Problem Brief Template
```markdown
# Problem Brief: [제목]

## 1. Problem Statement
[위의 Problem Statement Template 사용]

## 2. Evidence
### Quantitative
- [데이터 포인트 + 출처]
### Qualitative  
- [사용자 인용문 + 컨텍스트]
### Confidence Level: [High / Medium / Low]

## 3. Opportunity Sizing
- Affected users: [수치]
- Frequency: [얼마나 자주]
- Severity: [Blocker / Major / Minor]
- Business impact: [retention / revenue / NPS 영향]

## 4. Priority Score
[RICE 또는 Opportunity Score 계산 과정]

## 5. Assumptions to Validate
- [ ] [가정 1] — 검증 방법: [...]
- [ ] [가정 2] — 검증 방법: [...]

## 6. Recommendation
[이 문제를 풀어야 하는 이유와 다음 단계]
```

## 유의사항
- 하나의 문제에 여러 원인이 있을 수 있다. Root cause를 찾아라.
- "사용자가 X 기능을 원한다"는 문제가 아니라 해결책이다. 그 뒤의 문제를 파라.
- 데이터가 없으면 만들어라 (quick survey, 5-user test, log 분석).
