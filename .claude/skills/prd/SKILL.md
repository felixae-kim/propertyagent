---
name: prd
description: PRD(제품 요구사항 문서) 작성을 위한 템플릿, 유저 스토리 작성법, 수용 기준 작성법을 제공합니다.
---

# PRD Writing Skill

## PRD Template

```markdown
# [기능명] PRD

| Field | Value |
|-------|-------|
| Author | [이름] |
| Status | Draft / In Review / Approved |
| Last Updated | [날짜] |
| 2-Pager | [링크] |
| Figma | [링크] |
| Linear Project | [링크] |

---

## 1. Overview
[2-Pager에서 합의된 방향을 3-5문장으로 요약]

## 2. Goals & Non-Goals
**Goals**
- G1: [목표 — 성공 지표와 연결]
- G2: ...

**Non-Goals**
- NG1: [안 하는 것 — 이유]
- NG2: ...

## 3. User Stories

### US-1: [스토리 제목] (P0)
**As a** [사용자 유형],
**I want to** [행동],
**So that** [결과/가치].

### US-2: [스토리 제목] (P1)
...

## 4. Detailed Requirements

### US-1: [스토리 제목]

#### Functional Requirements
- FR-1.1: [시스템이 해야 하는 것]
- FR-1.2: ...

#### Acceptance Criteria
- AC-1.1: Given [조건], When [행동], Then [결과]
- AC-1.2: ...

#### Edge Cases
- EC-1.1: [상황] → [시스템 동작]
- EC-1.2: ...

#### Business Rules
- BR-1.1: [제약, 제한, 권한 규칙]

## 5. User Flows
### Flow 1: [플로우명]
1. User: [행동] → System: [응답]
2. User: [행동] → System: [응답]
3. If [조건]:
   - Path A: ...
   - Path B: ...

**States**: Default | Empty | Loading | Error | Success

## 6. Data Requirements
| Entity | Field | Type | Source | Validation |
|--------|-------|------|--------|------------|
| ... | ... | ... | ... | ... |

## 7. Technical Constraints
- [성능 요구사항]
- [호환성 요구사항]
- [의존성]
- [마이그레이션 고려사항]

## 8. Design Notes
- [핵심 UX 결정]
- [인터랙션 패턴]
- [반응형 규칙]
- [접근성 요구사항]

## 9. Release Plan
- Feature flag: [이름]
- Rollout: [단계]
- Rollback: [기준 + 절차]

## 10. Success Metrics
| Metric | Current | Target | Method |
|--------|---------|--------|--------|
| ... | ... | ... | ... |

## 11. Open Questions
| # | Question | Owner | Status | Decision |
|---|----------|-------|--------|----------|
| 1 | ... | ... | Open | — |
```

## User Story 작성 가이드

### 좋은 유저 스토리의 조건 (INVEST)
- **I**ndependent: 다른 스토리에 의존하지 않음
- **N**egotiable: 구현 방법은 유연
- **V**aluable: 사용자에게 가치 전달
- **E**stimable: 개발팀이 규모를 추정할 수 있음
- **S**mall: 1 스프린트 이내 완료 가능
- **T**estable: 검증 가능한 수용 기준

### 우선순위 기준
- **P0 (Must-have)**: 이것 없이는 출시 불가. 핵심 가치 전달에 필수.
- **P1 (Should-have)**: 출시는 가능하지만, 없으면 경험이 현저히 부족.
- **P2 (Nice-to-have)**: 있으면 좋지만, 다음 이터레이션으로 미뤄도 됨.

## Acceptance Criteria 작성 가이드

### Given-When-Then Format
```
Given [사전 조건 / 컨텍스트],
When [사용자가 수행하는 행동],
Then [시스템의 기대 동작].
```

### 반드시 포함할 시나리오
1. **Happy Path**: 정상 흐름
2. **Validation Error**: 잘못된 입력
3. **Empty State**: 데이터 없음
4. **Loading State**: 로딩 중
5. **Error State**: 서버/네트워크 오류
6. **Boundary**: 최대/최소 값
7. **Permission**: 권한 없는 사용자

## Edge Case 식별 체크리스트
- [ ] 빈 입력, null, undefined
- [ ] 매우 긴 텍스트 (max length)
- [ ] 특수 문자, 이모지, 다국어
- [ ] 동시 접근 (같은 리소스를 두 사용자가 수정)
- [ ] 네트워크 끊김 중 작업
- [ ] 이전 버전 데이터와의 호환성
- [ ] 처음 사용하는 사용자 vs 기존 사용자
- [ ] 모바일에서의 터치 인터랙션
