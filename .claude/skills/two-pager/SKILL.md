---
name: two-pager
description: 2-Pager 문서 작성을 위한 템플릿, 구조, 작성 가이드를 제공합니다.
---

# 2-Pager Skill

## 2-Pager Template

```markdown
# [프로젝트/기능명] 2-Pager

**Author**: [이름] | **Date**: [날짜] | **Status**: Draft / In Review / Approved

---

## Page 1: 문제

### Context
[3-5문장. 왜 지금 이 문제를 보고 있는가? 어떤 상황이 변했는가?]

### Problem
**Who**: [영향 받는 사용자 세그먼트]
**What**: [관찰 가능한 문제]
**Evidence**: [핵심 데이터 포인트 2-3개]
**Impact**: [사용자 + 비즈니스에 미치는 영향]

### Opportunity
- [정량 데이터: 영향받는 사용자 수, 빈도, 금액 등]
- [해결 시 기대 효과]

### Current State
[현재 사용자 경험 흐름. 어디서 문제가 발생하는지 표시]
```
[Entry] → [Step A] → ⚠️ [여기서 문제 발생] → [Drop-off]
```

---

## Page 2: 제안

### Solution Direction
**Recommended**: [추천 방향 1-2문장]
[왜 이 방향인지 근거]

**Alternative considered**: [대안 + 왜 선택하지 않았는지]

### Scope

| In Scope | Out of Scope |
|----------|-------------|
| [이번에 하는 것] | [이번에 안 하는 것] |
| ... | ... |

### Success Metrics

| Type | Metric | Current | Target |
|------|--------|---------|--------|
| Primary | [핵심 지표] | [현재값] | [목표값] |
| Secondary | [보조 지표] | [현재값] | [목표값] |
| Guardrail | [악화되면 안 되는 지표] | [현재값] | [유지] |

### Risks & Open Questions

| # | Item | Type | Owner | Status |
|---|------|------|-------|--------|
| 1 | [내용] | 🔵 Decision / ⚠️ Risk / ❓ Question | [담당] | Open |

### Rough Timeline
| Phase | Duration | Note |
|-------|----------|------|
| Design | ~Xw | [핵심 작업] |
| Development | ~Xw | [핵심 작업] |
| QA | ~Xw | |
| Rollout | ~Xw | [단계별 배포] |
```

## 작성 원칙

### DO
- 모든 문장이 존재할 이유가 있어야 한다 (fluff 금지)
- 수치로 말한다: "많은 사용자" → "월간 활성 사용자의 35%"
- 팀이 결정해야 할 사항을 명시적으로 표기한다 (🔵)
- 검증이 필요한 가정을 별도로 표기한다 (⚠️)
- 디자이너, 개발자 모두 읽는다는 전제로 쓴다

### DON'T
- 2페이지를 넘기지 않는다 (진짜로)
- 구현 디테일을 쓰지 않는다 (그건 PRD의 몫)
- 근거 없는 주장을 하지 않는다 (Problem Brief에 근거를 둘 것)
- 솔루션 하나만 제시하지 않는다 (최소한 고려한 대안을 언급)
- 리스크를 숨기지 않는다

## 2-Pager Sync Meeting Guide
문서 작성 후 팀 싱크 미팅에서:

1. **사전 공유**: 미팅 24시간 전에 2-Pager를 공유하고 사전 코멘트 요청
2. **미팅 구조** (30-45분):
   - [5분] 문제 요약 (이견 확인)
   - [10분] 솔루션 방향 논의
   - [10분] 스코프 합의
   - [10분] 리스크 & 오픈 질문 해결
   - [5분] 다음 단계 합의
3. **산출물**: 합의된 방향 + 해결된 질문들 + GO/NO-GO 결정
