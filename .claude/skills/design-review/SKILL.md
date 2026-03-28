---
name: design-review
description: 디자인 리뷰 체크리스트, 디자인-개발 핸드오프 가이드, 상태 인벤토리 작성법을 제공합니다.
---

# Design Review Skill

## State Inventory Template
모든 화면/컴포넌트에 대해 아래 상태를 디자인에서 확인:

| State | Description | Designed? |
|-------|-------------|-----------|
| Default | 일반 상태 | ☐ |
| Empty | 데이터 없음 | ☐ |
| Loading | 로딩 중 (skeleton / spinner) | ☐ |
| Loaded | 데이터 있음 | ☐ |
| Error | 오류 발생 | ☐ |
| Partial Error | 일부 데이터만 로드 | ☐ |
| Disabled | 비활성 상태 | ☐ |
| Hover | 마우스 오버 | ☐ |
| Focus | 키보드 포커스 | ☐ |
| Active / Pressed | 클릭/탭 중 | ☐ |
| Selected | 선택됨 | ☐ |
| First-time User | 온보딩, 코치마크 | ☐ |
| Mobile | 모바일 뷰포트 | ☐ |
| Tablet | 태블릿 뷰포트 | ☐ |

## Design Review Checklist

### Completeness (완전성)
- [ ] PRD의 모든 유저 스토리에 해당하는 화면이 있는가?
- [ ] 모든 상태(empty, loading, error, success)가 디자인되었는가?
- [ ] 모든 엣지 케이스가 반영되었는가?
- [ ] 반응형 디자인(모바일, 태블릿, 데스크톱)이 정의되었는가?
- [ ] 에러 메시지 카피가 작성되었는가?
- [ ] 빈 상태 카피가 작성되었는가?

### Consistency (일관성)
- [ ] 기존 디자인 시스템 컴포넌트를 사용하고 있는가?
- [ ] 타이포그래피, 간격, 색상이 기존 패턴과 일치하는가?
- [ ] 인터랙션 패턴이 제품의 다른 부분과 일관적인가?
- [ ] 용어가 제품 전체에서 통일되어 있는가?

### Accessibility (접근성)
- [ ] 색상 대비 비율 ≥ 4.5:1 (텍스트) / ≥ 3:1 (큰 텍스트)
- [ ] 터치 타겟 ≥ 44x44px
- [ ] 색상만으로 정보를 전달하지 않는가? (색각 이상 고려)
- [ ] 포커스 순서가 논리적인가?
- [ ] 스크린 리더용 대체 텍스트가 정의되었는가?

### Feasibility (실현 가능성)
- [ ] PRD의 기술 제약사항이 반영되었는가?
- [ ] 현재 API로 제공 가능한 데이터만 사용하는가?
- [ ] 애니메이션이 성능에 영향을 주지 않는가?
- [ ] 이미지/에셋 사이즈가 합리적인가?

## Design-Dev Handoff Document Template

```markdown
# Design Handoff: [기능명]

## Figma Link
[Figma 프로젝트 링크]

## Screen Inventory

### Screen 1: [화면명]
**Route**: /path/to/page
**Components**:
| Component | New/Existing | Notes |
|-----------|-------------|-------|
| Header | Existing | variant: compact |
| DataTable | New | sortable, paginated |
| EmptyState | Existing | custom illustration |

**Interactions**:
- [요소] 클릭 → [동작]
- [요소] 호버 → [스타일 변화]

**Responsive Rules**:
- Desktop (≥1024px): [레이아웃]
- Tablet (768-1023px): [레이아웃]
- Mobile (<768px): [레이아웃]

**Edge Cases**:
- 데이터 0건: [Empty State 디자인 참조]
- 데이터 1000건+: [페이지네이션 동작]
- 긴 텍스트: [truncation 규칙]

### Screen 2: ...
```

## 디자인 피드백 작성 가이드

피드백 유형별 태그:
- 🔴 **Blocker**: 이대로 개발 진행 불가
- 🟡 **Revision**: 수정이 필요하지만 전체를 막지는 않음
- 🔵 **Question**: 의도가 명확하지 않아 확인 필요
- 🟢 **Suggestion**: 선택적 개선 제안

피드백 형식:
```
[태그] [화면명] — [구체적 위치]
현재: [지금 디자인의 상태]
문제: [왜 변경이 필요한지]
제안: [권장하는 변경] (또는 질문)
PRD 참조: [관련 요구사항 번호]
```
