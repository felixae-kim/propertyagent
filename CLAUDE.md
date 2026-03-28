# SeongjaeCorp — AI Product Squad

## Entry Point: 비서실장 (Chief of Staff)
모든 명령은 `chief-of-staff` 에이전트를 통해 처리합니다.
자연어로 지시하면, 비서실장이 의도를 파악하고 적절한 에이전트에게 위임합니다.

```
사용자 → chief-of-staff → [적절한 에이전트(들)] → 결과 보고
```

## Team
- **Product Lead (나)**: 문제 정의, 우선순위 결정, 전체 프로세스 오너
- **Backend Developer**: 서버, API, DB (Supabase), 인프라
- **Frontend Developer**: 웹/앱 UI (Next.js), 인터랙션, 클라이언트 로직
- **Product Designer**: UX 리서치, UI 디자인, 프로토타이핑 (토스 원칙 기반)

## Product Development Lifecycle
```
시장분석 → 문제발견 → 원인분석 → 솔루션도출 → 솔루션정의 → 2-Pager → PRD → Design → Prototype → Dev → QA → Release → Analysis
```

## Tech Stack
- **Frontend**: Next.js (React) + Tailwind CSS
- **Backend/DB**: Supabase (PostgreSQL, Auth, Storage, Realtime, Edge Functions)
- **Deployment**: Vercel
- **Design**: Figma (토스 제품 원칙 기반)

## Workspace
- 문서: Notion (2-Pager, PRD, 회의록)
- 디자인: Figma
- 프로젝트 관리: Linear
- 커뮤니케이션: Slack
- 코드: GitHub
- 분석: Amplitude / Mixpanel

## Agents
`.claude/agents/` 디렉토리에 워크플로우 단계별 에이전트가 정의되어 있음.
- **비서실장**: `chief-of-staff` — 모든 명령의 진입점
- **프로세스 관리**: `squad-orchestrator` — 단계 관리, 게이트 리뷰
- 개별 에이전트: market-researcher, problem-analyst, solution-architect, two-pager-writer, prd-writer, ui-ux-designer, design-collaborator, frontend-engineer, backend-engineer, qa-engineer, release-manager, metrics-analyst

## Agent Teams (Parallel Execution)
`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` 활성화됨.
독립적인 작업은 teammate를 생성하여 병렬 실행. 특히 Phase 8(개발)에서 frontend + backend 동시 진행.

## Skills
`.claude/skills/` 디렉토리에 각 에이전트가 참조하는 템플릿과 프레임워크가 정의되어 있음.

## Design Principles
토스(Toss)의 제품 원칙을 적극 반영:
- 한 화면, 한 목적
- 말하듯이 쓰기 (전문 용어 금지)
- 결정 피로 최소화
- 즉각적 피드백
- 심플하고 직관적이고 쉬워서 자꾸 쓰고 싶은 서비스
