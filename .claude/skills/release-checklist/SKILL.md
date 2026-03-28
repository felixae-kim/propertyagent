---
name: release-checklist
description: 배포 전/중/후 체크리스트, 롤아웃 전략, 롤백 절차, 인시던트 대응 템플릿을 제공합니다.
---

# Release Checklist Skill

## Pre-Release Checklist

### Code Readiness
- [ ] 모든 PR 리뷰 완료 및 머지
- [ ] CI/CD 파이프라인 통과 (lint, typecheck, unit test, build)
- [ ] DB 마이그레이션 스테이징에서 테스트 완료
- [ ] 하드코딩된 시크릿 없음 확인

### QA Sign-off
- [ ] QA 리포트 수령 (GO 판정)
- [ ] Critical/Major 버그 0건 확인
- [ ] 회귀 테스트 통과 확인

### Configuration
- [ ] Feature flag 설정 완료 (기본: OFF)
- [ ] 프로덕션 환경 변수 설정 확인
- [ ] 3rd party API 키/설정 확인
- [ ] CDN/캐시 설정 확인

### Communication
- [ ] 팀에 배포 일정 공유 (Slack)
- [ ] 배포 중 서포트팀 사전 안내 (필요 시)
- [ ] CHANGELOG 업데이트

### Rollback Preparation
- [ ] 롤백 플랜 문서화
- [ ] DB 롤백 마이그레이션 준비 (필요 시)
- [ ] 이전 버전으로 빠른 복귀 방법 확인

## Rollout Strategy Template

```markdown
# Rollout Plan: [기능명]

## Phase 1: Internal (Day 0)
- **Target**: 내부 팀만
- **Feature Flag**: team-only
- **Duration**: 최소 4시간
- **Check**: 기본 기능 정상 동작, 에러 없음

## Phase 2: Canary (Day 1)
- **Target**: 전체 사용자의 5%
- **Feature Flag**: 5%
- **Duration**: 최소 24시간
- **Check**: 에러율, 지표 이상 없음

## Phase 3: Early Rollout (Day 3)
- **Target**: 전체 사용자의 25%
- **Feature Flag**: 25%
- **Duration**: 최소 48시간
- **Check**: 주요 지표 안정, 서포트 티켓 정상 범위

## Phase 4: Full Release (Day 7+)
- **Target**: 100%
- **Feature Flag**: 100% → 2주 후 flag 제거
- **Check**: 모든 성공 지표 추적 시작

## Advancement Criteria
다음 단계로 진행하려면:
- 에러율 < baseline + 1%
- P95 latency < baseline × 1.5
- 서포트 티켓 급증 없음
- Core metrics 안정
```

## Post-Deploy Monitoring Checklist

### Immediate (0-30분)
- [ ] 배포 성공 확인 (health check)
- [ ] 에러 대시보드 확인 (Sentry / Datadog)
- [ ] 주요 API 응답 시간 확인
- [ ] Feature flag 동작 확인

### Short-term (1-4시간)
- [ ] 에러율 추이 모니터링
- [ ] 사용자 행동 정상 확인 (핵심 퍼널)
- [ ] 서포트 채널 모니터링

### Medium-term (24-48시간)
- [ ] 일간 활성 사용자 수치 정상
- [ ] 핵심 전환율 변화 없음
- [ ] 서버 리소스 (CPU, Memory) 정상

## Rollback Decision Matrix

| Signal | Threshold | Action |
|--------|-----------|--------|
| Error Rate | > baseline + 5% | 즉시 롤백 |
| P95 Latency | > baseline × 2 | 즉시 롤백 |
| Data Integrity Issue | Any | 즉시 롤백 |
| Critical Bug | Reported + Confirmed | 즉시 롤백 |
| Conversion Drop | > 10% drop | 조사 후 판단 |
| Support Ticket Spike | > 3x normal | 조사 후 판단 |

## Rollback Procedure
```
1. Feature flag → OFF (즉시 반영)
2. 팀 Slack 채널에 알림: "[기능명] 롤백 진행 중, 사유: [...]"
3. (DB 변경 있을 경우) 롤백 마이그레이션 실행
4. 모니터링 대시보드에서 정상화 확인
5. 인시던트 리포트 작성
```

## Incident Report Template
```markdown
# Incident Report: [제목]

**Date**: [발생 일시]
**Duration**: [영향 시간]
**Severity**: P1 / P2 / P3
**Owner**: [담당자]

## Summary
[1-2문장 요약]

## Timeline
| Time | Event |
|------|-------|
| HH:MM | [발생] |
| HH:MM | [감지] |
| HH:MM | [대응 시작] |
| HH:MM | [해결] |

## Root Cause
[원인 분석]

## Impact
- 영향 받은 사용자: [수]
- 기능 영향: [상세]
- 데이터 영향: [있음/없음]

## Action Items
| # | Action | Owner | Due | Status |
|---|--------|-------|-----|--------|
| 1 | [재발 방지 조치] | ... | ... | ... |
```
