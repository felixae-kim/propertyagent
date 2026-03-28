---
name: metrics-analysis
description: 성과 분석 프레임워크, 지표 설계 가이드, A/B 테스트 해석, 성과 리포트 템플릿을 제공합니다.
---

# Metrics Analysis Skill

## Metric Design Framework

### North Star Metric
```
[제품의 핵심 가치를 하나의 숫자로 표현]
예: "주간 활성 프로젝트 수" (프로젝트 관리 도구)
예: "월간 반복 매출" (SaaS)
```

### Metric Hierarchy
```
North Star Metric
├── Input Metric 1 (우리가 직접 영향 줄 수 있는 것)
│   ├── Leading Indicator 1a
│   └── Leading Indicator 1b
├── Input Metric 2
│   └── Leading Indicator 2a
└── Guardrail Metric (악화되면 안 되는 것)
```

### HEART Framework (Google)
| Dimension | Goal | Signal | Metric |
|-----------|------|--------|--------|
| **Happiness** | 사용자 만족 | 설문 응답, 리뷰 | NPS, CSAT |
| **Engagement** | 적극적 사용 | 행동 빈도, 깊이 | DAU/MAU, 세션 시간 |
| **Adoption** | 신규 사용 시작 | 신규 가입, 첫 사용 | 활성화율, 온보딩 완료율 |
| **Retention** | 지속 사용 | 재방문, 구독 유지 | D7/D30 리텐션 |
| **Task Success** | 목표 달성 | 완료율, 오류율 | 전환율, 에러율 |

## Feature-Level Metric Template
```markdown
## [기능명] Success Metrics

### Primary Metric (핵심 성공 지표)
- **Metric**: [지표명]
- **Definition**: [정확한 계산식]
- **Current Baseline**: [현재 수치]
- **Target**: [목표 수치]
- **Measurement**: [어떻게 측정? 이벤트명, 대시보드]
- **Timeline**: [언제까지 달성?]

### Secondary Metrics (보조 지표)
| Metric | Baseline | Target | Why |
|--------|----------|--------|-----|
| [지표1] | [현재] | [목표] | [이 지표를 보는 이유] |
| [지표2] | [현재] | [목표] | ... |

### Guardrail Metrics (가드레일)
| Metric | Current | Threshold | Alert |
|--------|---------|-----------|-------|
| [이탈률] | [현재] | [이 이상이면 문제] | [알림 설정] |
| [로드 타임] | [현재] | [이 이상이면 문제] | ... |
```

## A/B Test Analysis Guide

### Pre-Analysis Checklist
- [ ] 실험 기간 충분한가? (최소 1-2 full business cycles)
- [ ] 샘플 사이즈 충분한가? (statistical power ≥ 80%)
- [ ] Novelty effect 고려 기간 지났는가?
- [ ] SRM (Sample Ratio Mismatch) 확인했는가?

### Statistical Significance
```
p-value < 0.05 → 통계적으로 유의미
BUT:
- 실질적 유의미성(practical significance)도 확인
- 효과 크기(effect size)가 의미 있는 수준인지?
- 복수 비교 보정(Bonferroni)이 필요한지?
```

### Result Interpretation Matrix
| Statistical | Practical | Interpretation |
|-------------|-----------|---------------|
| Significant | Meaningful | ✅ 채택. 변경 적용. |
| Significant | Trivial | ⚠️ 수치는 변했지만 비즈니스 임팩트 미미. 비용 대비 판단. |
| Not significant | — | ❌ 효과 없음. 원래대로 유지하거나 가설 재검토. |
| — | — (짧은 기간) | ⏸️ 실험 기간 연장 필요. |

## Performance Report Template

```markdown
# Performance Report: [기능명]

**Period**: [분석 기간]
**Author**: [이름]
**Release Date**: [배포일]

## Executive Summary
[3-5문장: 무엇을 출시했고, 결과가 어떻고, 다음에 무엇을 해야 하는지]

## Results

### Primary Metric
| Metric | Before | After | Change | Target | Status |
|--------|--------|-------|--------|--------|--------|
| [지표] | [이전] | [이후] | [+/- %] | [목표] | 🟢/🟡/🔴 |

**Analysis**: [왜 이런 결과가 나왔는지 해석]

### Secondary Metrics
[같은 형식으로]

### Guardrail Metrics
[같은 형식으로 — 악화되지 않았는지 확인]

## Segment Analysis
| Segment | Primary Metric Change | Note |
|---------|----------------------|------|
| New Users | [변화] | [해석] |
| Power Users | [변화] | [해석] |
| Mobile | [변화] | [해석] |
| Desktop | [변화] | [해석] |

## Key Insights
1. **[인사이트]**: [근거] → [시사점]
2. **[인사이트]**: [근거] → [시사점]
3. **[인사이트]**: [근거] → [시사점]

## Unexpected Findings
- [예상하지 못했던 변화 + 원인 가설]

## Recommendations
| # | Action | Type | Priority | Rationale |
|---|--------|------|----------|-----------|
| 1 | [구체적 액션] | Iterate / New / Fix | High | [근거] |
| 2 | ... | ... | ... | ... |

## Feed Forward
이 분석에서 발견된 새로운 문제/기회:
- [→ problem-analyst로 전달할 인풋]
```

## Common Pitfalls
- **Survivorship Bias**: 이탈한 사용자의 데이터가 빠져 있지 않은지?
- **Simpson's Paradox**: 전체 수치는 좋은데 세그먼트별로 보면 다른지?
- **Regression to the Mean**: 일시적 변동을 효과로 착각하지 않았는지?
- **Cherry Picking**: 좋은 수치만 골라서 보고하고 있지 않은지?
