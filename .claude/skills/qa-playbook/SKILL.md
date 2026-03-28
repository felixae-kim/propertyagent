---
name: qa-playbook
description: QA 테스트 계획, 테스트 케이스 작성 템플릿, 버그 리포트 형식, 릴리즈 판단 기준을 제공합니다.
---

# QA Playbook Skill

## Test Plan Template

```markdown
# Test Plan: [기능명]

**Tester**: [이름] | **Date**: [날짜] | **PRD**: [링크]
**Test Environment**: [URL / 브랜치명]
**Build Version**: [버전]

## Scope
**In scope**: [테스트 대상 유저 스토리 번호]
**Out of scope**: [이번 사이클에서 제외]
**Preconditions**: [테스트 전 필요한 설정, 테스트 계정 등]

## Test Cases
[아래 테스트 케이스 테이블]

## Exit Criteria
- P0 케이스 100% 통과
- Critical/Major 버그 0건
- 회귀 테스트 통과
```

## Test Case 설계 기법

### Equivalence Partitioning (동치 분할)
입력을 유효/무효 그룹으로 나눠 각 그룹에서 하나만 테스트:
```
이메일 필드:
- Valid: user@example.com (일반), user+tag@domain.co.kr (특수)
- Invalid: "" (빈값), "no-at-sign" (@없음), "@no-local.com" (로컬 없음)
```

### Boundary Value Analysis (경계값 분석)
경계 지점과 그 전후를 테스트:
```
비밀번호 (8-20자):
- 7자 (경계 아래, 무효)
- 8자 (하한 경계, 유효)
- 20자 (상한 경계, 유효)
- 21자 (경계 위, 무효)
```

### State Transition (상태 전이)
상태 변화를 추적:
```
주문 상태: 생성됨 → 결제완료 → 배송중 → 배송완료 → 반품요청
각 전이마다: 유효한 전이 + 무효한 전이 (배송중 → 생성됨은 불가)
```

## Test Case Template

| TC# | Story | Category | Scenario | Precondition | Steps | Expected | Priority | Result |
|-----|-------|----------|----------|-------------|-------|----------|----------|--------|
| TC-001 | US-1 | Happy Path | 정상 등록 | 로그인 상태 | 1. /signup 2. 입력 3. 제출 | 성공 메시지 | P0 | — |
| TC-002 | US-1 | Validation | 이메일 형식 오류 | 로그인 상태 | 1. 잘못된 이메일 입력 2. 제출 | 에러 메시지 | P0 | — |
| TC-003 | US-1 | Edge | 네트워크 끊김 | 로그인 상태, 오프라인 | 1. 제출 | 오프라인 에러 | P1 | — |

## Bug Severity Guide

| Severity | Definition | Example | Release Impact |
|----------|-----------|---------|----------------|
| **Critical** | 서비스 불가, 데이터 손실 | 결제 후 주문 사라짐, 전체 장애 | 🔴 배포 중단 |
| **Major** | 핵심 기능 사용 불가, 우회 없음 | 검색이 안 됨, 로그인 실패 | 🔴 배포 중단 |
| **Minor** | 기능 불편, 우회 가능 | 정렬이 안 됨, 레이아웃 깨짐 | 🟡 P1 수정 후 배포 |
| **Cosmetic** | 외관 이슈, 기능 문제 없음 | 1px 어긋남, 오타 | 🟢 다음 이터레이션 |

## Bug Report Template
```markdown
## BUG-[번호]: [한 줄 제목]

**Severity**: Critical / Major / Minor / Cosmetic
**Found in**: [TC# 또는 탐색적 테스트]
**PRD Ref**: [관련 요구사항 번호]

### Environment
- Browser: Chrome 120 / Safari 17
- Device: Desktop / Mobile (iPhone 15)
- OS: macOS 14.2 / iOS 17

### Steps to Reproduce
1. [정확한 URL로 이동]
2. [구체적 행동]
3. [관찰되는 문제]

### Expected Behavior
[PRD/디자인에 따른 올바른 동작]

### Actual Behavior
[실제로 발생하는 문제]

### Evidence
[스크린샷, 영상, 콘솔 로그]

### Frequency
Always / Intermittent (재현율: ~X%)
```

## Release Readiness Matrix

| Category | Criteria | Status |
|----------|---------|--------|
| P0 Test Cases | 100% Pass | ☐ |
| P1 Test Cases | ≥ 95% Pass | ☐ |
| Critical Bugs | 0 Open | ☐ |
| Major Bugs | 0 Open | ☐ |
| Minor Bugs | Documented, deferred OK | ☐ |
| Regression | Full suite pass | ☐ |
| Cross-browser | Chrome, Safari, Firefox | ☐ |
| Responsive | Mobile, Tablet, Desktop | ☐ |
| Accessibility | WCAG 2.1 AA | ☐ |
| Performance | LCP < 2.5s, FID < 100ms | ☐ |

**GO**: 모든 필수 항목 통과
**Conditional GO**: Minor 이슈 존재, 다음 이터레이션에서 해결 조건부 동의
**NO-GO**: Critical/Major 미해결 또는 P0 케이스 미통과
