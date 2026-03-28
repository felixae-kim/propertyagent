# Solution Architecture: PropertyAgent (부동산 매물 알림 및 탐색 플랫폼)

> 작성일: 2026-03-28
> 상태: Solution Definition Complete

---

## 1. Benchmarking (벤치마킹)

### 1.1 Direct References

| Reference | Domain | Problem Solved | Solution Approach | Result | Applicable Insight |
|-----------|--------|---------------|-------------------|--------|-------------------|
| 호갱노노 | 부동산 | 아파트 실거래가 추적 | 관심 단지 등록 시 신규 매물/실거래 알림 발송 | MAU 179만, 업계 2위 | 단지 단위 알림은 검증된 니즈. 그러나 "지정가 이하 알림"은 미지원 |
| 네이버 부동산 | 부동산 | 매물 탐색 | 압도적 매물 수 + 지도 기반 탐색 | MAU 1위권 | 매물 수가 핵심 경쟁력. 직접 매물 확보보다 기존 플랫폼 데이터 연동이 현실적 |
| 직방 | 부동산 | 매물 탐색/VR 투어 | VR 내부 투어 + 직영 매물 확보 | MAU 229만 | 시각적 경험이 차별화 요소. 단, 우리는 VR보다 "조건 매칭"에 집중 |
| 집품 | 부동산 | 아파트 종합 정보 | 학군, 교통, 환경 등 종합 정보 제공 | App Store 인기 | 복합 조건 탐색 니즈 존재. 그러나 "알림"보다 "조회" 중심 |

### 1.2 Analogous References

| Reference | Domain | Problem Solved | Solution Approach | Result | Applicable Insight |
|-----------|--------|---------------|-------------------|--------|-------------------|
| 쿠팡 가격 알림 | 이커머스 | 희망가 도달 알림 | 상품별 희망가 설정 -> 가격 하락 시 푸시 | 높은 전환율 | "지정가 알림" 패턴은 이커머스에서 검증됨. 부동산에 그대로 적용 가능 |
| Zillow (미국) | 부동산 | 매물 알림 | Saved Search + Price Alert + Zestimate | 미국 1위 부동산 플랫폼 | 복합 조건 저장 + 알림 조합이 핵심. AI 가격 예측(Zestimate)은 장기 목표 |
| 무신사/29cm | 패션 커머스 | 상품 탐색 | 큐레이션 + 인플루언서 추천 + 필터링 | MZ세대 대표 쇼핑몰 | "인플루언서 픽" + "커머스형 UX" 패턴. 부동산을 쇼핑하듯 탐색 |
| Redfin Hot Homes | 부동산 | 인기 매물 식별 | 알고리즘 기반 인기 매물 표시 + 알림 | 빠른 의사결정 유도 | "핫딜" 개념을 부동산에 적용. 시세 대비 저렴한 매물 하이라이팅 |

### 1.3 Anti-References (실패 사례)

| Reference | 실패 이유 | 교훈 |
|-----------|----------|------|
| 다수 부동산 스타트업의 자체 매물 DB 구축 시도 | 중개사 확보 비용 과다, 네이버/직방 대비 매물 수 열세 | 매물 데이터는 직접 확보하지 말고 공공 API + 기존 플랫폼 연동으로 해결 |
| 부동산 SNS/커뮤니티 앱 | 콘텐츠 생산 의존, 핵심 거래 가치 부재 | 커뮤니티보다 "알림"이라는 실용 기능에 집중해야 리텐션 확보 |
| 호갱노노 크롤링 의존 서비스들 | 크롤링 차단으로 서비스 중단 | 공공 API 기반 + 합법적 데이터 소스 확보 필수 |

---

## 2. Ideation (아이디에이션)

### 2.1 HMW (How Might We) 질문

1. 어떻게 하면 사용자가 "내 예산에 맞는 매물"을 놓치지 않게 할 수 있을까?
2. 어떻게 하면 복잡한 부동산 조건 검색을 "쇼핑처럼 쉽게" 만들 수 있을까?
3. 어떻게 하면 교통 호재 정보를 "직관적으로 이해"하게 할 수 있을까?
4. 어떻게 하면 부동산 전문가의 인사이트를 "신뢰할 수 있는 큐레이션"으로 제공할 수 있을까?
5. 어떻게 하면 부동산 탐색을 "반복적으로 하고 싶은 경험"으로 만들 수 있을까?

### 2.2 Raw Ideas (발산)

| # | Idea | HMW 연결 |
|---|------|---------|
| 1 | 지정가 알림 봇: 단지별 희망가 설정, 해당 가격 이하 매물 등록 시 즉시 푸시 | HMW-1 |
| 2 | 스마트 조건 알림: 지역+세대수+학군+금액 복합 필터 저장 -> 매칭 매물 자동 알림 | HMW-1, 2 |
| 3 | 호재 지도: GTX/신분당선 등 교통 개발 호재를 지도 위 정확한 위치에 시각화 | HMW-3 |
| 4 | 인플루언서 픽 피드: 유튜버/전문가 추천 단지를 카드형 피드로 큐레이션 | HMW-4 |
| 5 | 부동산 쇼핑몰 UX: 매물을 상품 카드처럼 표시, 장바구니(관심목록), 비교하기 | HMW-2, 5 |
| 6 | AI 가격 예측: 실거래가 트렌드 기반 3/6/12개월 후 예상 시세 | HMW-1 |
| 7 | 통근 시간 필터: "강남역 40분 이내" 조건으로 매물 필터링 | HMW-2, 3 |
| 8 | 학군 스코어카드: 초중고 학업성취도, 학원가 밀집도 등 학군 점수화 | HMW-2 |
| 9 | 매물 타임라인: 특정 단지의 매물 등록/삭제/가격 변동 히스토리 타임라인 | HMW-1, 5 |
| 10 | 주간 리포트: 관심 지역의 시세 변동, 신규 매물, 호재 뉴스를 주간 요약 이메일 | HMW-5 |
| 11 | 실거래가 알림: 관심 단지에서 실거래 발생 시 즉시 알림 (가격/층/면적 포함) | HMW-1 |
| 12 | 갭투자 계산기: 매매가-전세가 갭 자동 계산 + 갭 줄어드는 단지 알림 | HMW-1 |

### 2.3 Convergence (수렴) - Impact-Feasibility 평가

| Idea | User Impact (1-5) | Feasibility (1-5) | Effort (1-5, 낮을수록 쉬움) | Innovation (1-5) | Total |
|------|-------------------|-------------------|---------------------------|------------------|-------|
| 1. 지정가 알림 봇 | 5 | 4 | 4 | 4 | **17** |
| 2. 스마트 조건 알림 | 5 | 3 | 3 | 4 | **15** |
| 3. 호재 지도 | 4 | 4 | 4 | 5 | **17** |
| 5. 부동산 쇼핑몰 UX | 4 | 4 | 3 | 4 | **15** |
| 4. 인플루언서 픽 | 3 | 3 | 3 | 4 | **13** |
| 11. 실거래가 알림 | 4 | 5 | 4 | 3 | **16** |
| 7. 통근 시간 필터 | 4 | 3 | 2 | 4 | **13** |
| 9. 매물 타임라인 | 3 | 4 | 4 | 3 | **14** |
| 10. 주간 리포트 | 3 | 4 | 4 | 3 | **14** |

### 2.4 Top 3 Ideas (정교화)

#### Top 1: 지정가 매물 알림 시스템 (Score: 17)

- **핵심 메커니즘**: 사용자가 특정 단지 + 희망가를 설정하면, 주기적으로 매물 데이터를 수집하여 조건 매칭 시 푸시 알림 발송. 이커머스의 "가격 알림"을 부동산에 적용한 것.
- **차별점**: 호갱노노는 "새 매물 알림"만 제공. 우리는 "지정가 이하" 조건 알림을 제공하여, 사용자가 직접 매물을 체크할 필요 없이 원하는 가격대 매물만 받아볼 수 있음.
- **리스크**: 매물 데이터 수집 주기에 따라 알림 지연 발생 가능. 매물 정보의 정확성(허위매물) 이슈.
- **최소 검증 방법**: 특정 10개 단지에 대해 수동으로 매물 모니터링 -> 조건 매칭 시 카카오톡 알림 발송하는 프로토타입 (1주)

#### Top 2: 교통 호재 지도 (Score: 17)

- **핵심 메커니즘**: 확정/예정 교통 개발 사업(GTX, 신분당선 연장, 신규 지하철 등)의 역 위치를 지도 위에 정확히 표시. 각 호재별 진행 상태(계획/착공/완공), 예상 개통일, 영향 반경을 시각화.
- **차별점**: 기존 서비스는 뉴스 기사 수준. 우리는 "정확한 좌표 기반 시각화 + 주변 아파트 연결"로 투자 판단 지원.
- **리스크**: 교통 호재 데이터의 수동 관리 필요 (자동화 어려움). 계획 변경 시 업데이트 지연.
- **최소 검증 방법**: 수도권 주요 5개 호재(GTX-A/B/C, 신분당선 연장, 9호선 연장)만 지도에 표시하는 정적 페이지 (3일)

#### Top 3: 커머스형 부동산 탐색 UX (Score: 15)

- **핵심 메커니즘**: 매물을 쇼핑몰 상품 카드처럼 표시. 썸네일(단지 사진/평면도) + 핵심 정보(가격/평수/층/세대수) + 한줄 태그(초품아/역세권/신축). 관심 목록 저장, 매물 비교 기능.
- **차별점**: 기존 부동산 앱은 "목록형" or "지도형"만 제공. 커머스형 카드 UI + 태그 시스템으로 직관적 탐색.
- **리스크**: 매물 사진 확보 어려움. UX가 좋아도 매물 수가 적으면 가치 없음.
- **최소 검증 방법**: 실거래가 데이터 기반으로 10개 단지의 카드형 UI 프로토타입 제작 (5일)

---

## 3. Solution Definition (솔루션 정의)

### 3.1 Hypothesis (가설)

```
우리는 [아파트 매수를 고려하는 30-40대 실수요자/투자자]에게
[지정가 매물 알림 + 교통 호재 지도 + 커머스형 탐색 UX]를 제공하면,

[매일 여러 부동산 앱을 돌아다니며 매물을 체크하는 행동]이
[알림을 기다리다가 조건에 맞는 매물이 오면 즉시 확인하는 행동]으로 변화하여,

[주간 활성 사용자의 70%가 알림을 통해 매물을 확인하고,
 월 리텐션 40% 이상을 달성]할 것이다.

이것을 [랜딩 페이지 + MVP 앱 베타 테스트 (100명)]으로
[8주] 내에 검증할 수 있다.
```

### 3.2 Input/Output Definition

```
[Input]
  - 사용자 입력: 관심 단지, 희망가, 복합 조건(지역/세대수/학군/금액)
  - 공공 데이터: 국토교통부 실거래가 API, 학교 정보 API
  - 매물 데이터: 네이버 부동산 내부 API (주기적 수집)
  - 교통 호재: 수동 큐레이션 데이터 (GTX, 신분당선 등 좌표/상태/일정)
  - 인플루언서 콘텐츠: YouTube API 기반 부동산 유튜버 추천 단지 수집

[Transformation]
  - 매물 데이터 수집 -> 사용자 조건 매칭 엔진 -> 알림 발송
  - 교통 호재 + 아파트 위치 -> 지도 위 시각적 연결
  - 매물 정보 -> 커머스형 카드 UI 변환 (태그 자동 생성)

[Output]
  - 지정가/조건 매칭 매물 푸시 알림
  - 교통 호재가 표시된 인터랙티브 지도
  - 커머스형 매물 탐색 인터페이스
  - 인플루언서 추천 단지 큐레이션 피드
```

### 3.3 User Experience Design

```
사용자 여정:

1. [진입점] 온보딩
   - "어떤 아파트를 찾고 계세요?" -> 관심 지역 선택
   - "예산은 얼마인가요?" -> 희망가 범위 설정
   - "중요한 조건은?" -> 학군/교통/신축/세대수 등 태그 선택
   -> 3단계 온보딩으로 첫 알림 조건 자동 생성

2. [핵심 경험] "아하 모먼트"
   - 설정 후 첫 알림 수신: "래미안 OO, 9억 이하 매물이 등록되었습니다"
   - 알림 탭 -> 매물 카드 확인 -> 상세 정보 -> 중개사 연결
   -> "내가 원하는 조건의 매물을 자동으로 찾아주는구나!"

3. [반복 경험] 일상적 사용
   - 출퇴근 시 푸시 알림 확인 (하루 0-3건)
   - 주말에 호재 지도 탐색 -> 관심 지역 확대
   - 주간 리포트 이메일 -> 시세 트렌드 파악
   -> 매일 부동산 앱을 순회할 필요 없는 편안함

4. [성공 경험] 문제 해결 순간
   - 알림으로 받은 매물로 실제 임장/계약 진행
   - "이 앱 아니었으면 이 매물 놓쳤을 거야"
   -> 주변에 추천 (바이럴)
```

### 3.4 Feature Definition

#### Phase 1 - MVP (8주)

| # | Feature | User Story | Priority | Complexity | Dependency |
|---|---------|-----------|----------|------------|------------|
| F1 | 회원가입/로그인 | 사용자로서, 소셜 로그인으로 빠르게 가입하여 내 알림 설정을 저장하고 싶다 | Must | S | - |
| F2 | 관심 단지 등록 | 사용자로서, 특정 아파트 단지를 관심 등록하여 매물 변동을 추적하고 싶다 | Must | S | F1 |
| F3 | 지정가 알림 설정 | 사용자로서, 관심 단지에 희망가를 설정하여 해당 가격 이하 매물이 나오면 알림받고 싶다 | Must | M | F2 |
| F4 | 복합 조건 알림 | 사용자로서, 지역+세대수+학군+금액 조건을 조합하여 매칭 매물 알림을 받고 싶다 | Must | L | F1 |
| F5 | 매물 카드 UI | 사용자로서, 매물 정보를 쇼핑몰 상품처럼 카드 형태로 보고 싶다 | Must | M | - |
| F6 | 매물 상세 페이지 | 사용자로서, 매물의 실거래가 이력, 단지 정보, 주변 시설을 한 화면에서 보고 싶다 | Must | M | F5 |
| F7 | 교통 호재 지도 | 사용자로서, GTX/신분당선 등 교통 호재를 지도에서 정확한 위치로 보고 싶다 | Should | M | - |
| F8 | 관심 목록 (찜) | 사용자로서, 마음에 드는 매물을 저장하고 나중에 다시 보고 싶다 | Must | S | F1, F5 |
| F9 | 푸시 알림 (Web Push) | 사용자로서, 브라우저를 닫아도 매칭 매물 알림을 받고 싶다 | Must | M | F3, F4 |
| F10 | 온보딩 플로우 | 사용자로서, 가입 직후 3단계로 관심 조건을 빠르게 설정하고 싶다 | Should | S | F1 |

#### Phase 2 (이후)

| # | Feature | Priority | Complexity |
|---|---------|----------|------------|
| F11 | 인플루언서 픽 피드 | Should | L |
| F12 | 매물 비교하기 (2-3개 매물 나란히 비교) | Should | M |
| F13 | 주간 리포트 이메일 | Could | M |
| F14 | 통근 시간 필터 | Could | L |
| F15 | 실거래가 알림 | Should | S |
| F16 | 네이티브 앱 (PWA -> React Native) | Could | XL |
| F17 | AI 가격 예측 | Could | XL |

### 3.5 Data Source Strategy (데이터 소스 전략)

| 데이터 | 소스 | 수집 방법 | 주기 | 비고 |
|--------|------|----------|------|------|
| 아파트 실거래가 | 국토교통부 공공데이터 API | REST API 호출 | 일 1회 | 무료, 인증키 필요. data.go.kr 에서 신청 |
| 아파트 매물 정보 | 네이버 부동산 내부 API | 서버사이드 HTTP 요청 | 30분~1시간 | 비공식 API, 호출량 제한 필요. 차단 리스크 존재 |
| 아파트 단지 정보 | 국토교통부 공동주택 단지 API | REST API 호출 | 월 1회 | 세대수, 준공일, 주소 등 |
| 학교 정보 | 학교알리미(schoolinfo.go.kr) API | REST API 호출 | 분기 1회 | 학업성취도, 학교 위치 등 |
| 교통 호재 데이터 | 수동 큐레이션 + 뉴스 크롤링 | 관리자 입력 + 반자동 | 주 1회 | 노선도, 역 좌표, 진행상태, 예상 개통일 |
| 인플루언서 콘텐츠 | YouTube Data API v3 | 키워드 검색 + 채널 구독 | 일 1회 | 부동산 유튜버 채널 목록 수동 관리 |
| 지도 타일/좌표 | 네이버 지도 API 또는 Kakao 지도 API | JavaScript SDK | 실시간 | 무료 티어로 시작 가능 |

**핵심 리스크: 매물 데이터 수집**

네이버 부동산은 공식 API를 제공하지 않으며, 내부 AJAX API를 통해 데이터를 수집해야 합니다. 이는 다음 리스크를 수반합니다:

1. **차단 리스크**: 과도한 호출 시 IP 차단 가능
2. **법적 리스크**: 이용약관 위반 가능성
3. **데이터 안정성**: API 구조 변경 시 수집 실패

**완화 전략**:
- MVP 단계에서는 실거래가 공공 API 중심으로 시작
- 매물 데이터는 제한된 수의 관심 단지에 대해서만 수집 (사용자 요청 기반)
- 장기적으로 중개사 직접 매물 등록 또는 직방/다방 등 공식 제휴 추진
- 호출 간격 조절 (분당 요청 수 제한) + 프록시 로테이션

---

## 4. Technical Direction (시스템 아키텍처)

### 4.1 High-Level Architecture

```
[Client Layer]
  Next.js (Vercel)
    - SSR/SSG 페이지
    - 지도 컴포넌트 (Kakao Maps SDK)
    - PWA + Web Push (Service Worker)
    - Tailwind CSS

[API Layer]
  Supabase Edge Functions (Deno)
    - /api/alerts        : 알림 조건 CRUD
    - /api/listings      : 매물 조회/검색
    - /api/favorites     : 관심 목록 관리
    - /api/transport-map : 교통 호재 데이터
    - /api/notifications : 푸시 알림 발송

[Data Layer]
  Supabase (PostgreSQL)
    - 사용자, 알림 조건, 관심 목록
    - 매물 데이터 (캐시)
    - 실거래가 이력
    - 교통 호재 마스터 데이터
  Supabase Realtime
    - 새 매물 등록 시 실시간 이벤트
  Supabase Storage
    - 단지 이미지, 호재 지도 오버레이

[Background Jobs]
  Supabase Edge Functions (Cron)
    - 매물 수집기: 30분 간격으로 관심 단지 매물 수집
    - 실거래가 수집기: 일 1회 국토부 API 호출
    - 알림 매칭 엔진: 신규 매물 vs 사용자 조건 매칭
    - 알림 발송기: Web Push / Email 발송

[External Services]
    - 국토교통부 실거래가 API
    - 네이버 부동산 내부 API
    - Kakao Maps JavaScript SDK
    - Web Push (VAPID)
    - Resend (이메일 발송)
```

### 4.2 Data Model (핵심 엔티티)

```sql
-- 사용자
users (
  id UUID PK,           -- Supabase Auth UID
  email TEXT,
  nickname TEXT,
  push_subscription JSONB,  -- Web Push subscription
  created_at TIMESTAMPTZ
)

-- 아파트 단지
complexes (
  id SERIAL PK,
  name TEXT,             -- "래미안 블레스티지"
  address TEXT,
  lat DECIMAL,
  lng DECIMAL,
  total_units INT,       -- 세대수
  built_year INT,        -- 준공연도
  nearby_schools JSONB,  -- 인근 학교 목록
  tags TEXT[],           -- ['초품아', '역세권', '대단지']
  updated_at TIMESTAMPTZ
)

-- 매물 (수집된 데이터 캐시)
listings (
  id SERIAL PK,
  complex_id INT FK,
  source TEXT,           -- 'naver', 'zigbang', etc.
  source_id TEXT,        -- 원본 매물 ID
  price BIGINT,          -- 매매가 (만원)
  deposit BIGINT,        -- 전세/보증금 (만원)
  monthly_rent INT,      -- 월세 (만원)
  deal_type TEXT,        -- 'sale', 'jeonse', 'monthly'
  area_m2 DECIMAL,       -- 전용면적 (m2)
  floor INT,
  description TEXT,
  source_url TEXT,
  is_active BOOLEAN DEFAULT true,
  first_seen_at TIMESTAMPTZ,
  last_seen_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ
)

-- 실거래가 이력
transactions (
  id SERIAL PK,
  complex_id INT FK,
  deal_date DATE,
  price BIGINT,          -- 거래가 (만원)
  area_m2 DECIMAL,
  floor INT,
  deal_type TEXT,
  created_at TIMESTAMPTZ
)

-- 알림 조건
alert_rules (
  id SERIAL PK,
  user_id UUID FK,
  rule_type TEXT,        -- 'price_watch' | 'condition_match'
  -- price_watch 전용
  complex_id INT FK NULL,
  target_price BIGINT,   -- 희망가 (만원)
  -- condition_match 전용
  conditions JSONB,      -- {"region": "...", "min_units": 500, "school_nearby": true, "max_price": 90000}
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ
)

-- 알림 발송 이력
notifications (
  id SERIAL PK,
  user_id UUID FK,
  alert_rule_id INT FK,
  listing_id INT FK,
  channel TEXT,          -- 'web_push', 'email'
  title TEXT,
  body TEXT,
  is_read BOOLEAN DEFAULT false,
  sent_at TIMESTAMPTZ
)

-- 관심 목록
favorites (
  id SERIAL PK,
  user_id UUID FK,
  listing_id INT FK NULL,
  complex_id INT FK NULL,
  created_at TIMESTAMPTZ
)

-- 교통 호재
transport_projects (
  id SERIAL PK,
  name TEXT,             -- "GTX-A"
  line_type TEXT,        -- 'gtx', 'subway', 'bus_rapid'
  status TEXT,           -- 'planned', 'approved', 'under_construction', 'completed'
  expected_completion DATE,
  description TEXT,
  route_geojson JSONB,   -- 노선 GeoJSON
  updated_at TIMESTAMPTZ
)

-- 교통 호재 역
transport_stations (
  id SERIAL PK,
  project_id INT FK,
  name TEXT,             -- "동탄역"
  lat DECIMAL,
  lng DECIMAL,
  status TEXT,
  updated_at TIMESTAMPTZ
)
```

### 4.3 Tech Stack (확정)

| Layer | Technology | 선택 이유 |
|-------|-----------|----------|
| Frontend | Next.js 14 (App Router) + Tailwind CSS | SSR/SSG 지원, SEO 유리, 빠른 개발 |
| Backend | Supabase Edge Functions (Deno) | 서버리스, PostgreSQL 직접 접근, Cron 지원 |
| Database | Supabase PostgreSQL | PostGIS 확장 가능 (지리 쿼리), Realtime 지원 |
| Auth | Supabase Auth | 소셜 로그인 (Google, Kakao), Row Level Security |
| Storage | Supabase Storage | 이미지 저장, CDN 제공 |
| Map | Kakao Maps JavaScript SDK | 국내 지도 정확도, 무료 티어 충분 |
| Push | Web Push API (VAPID) | 브라우저 네이티브, 무료 |
| Email | Resend | 개발자 친화적, 무료 티어 3000건/월 |
| Deployment | Vercel | Next.js 최적화, 자동 배포, 무료 티어 |
| Monitoring | Vercel Analytics + Supabase Dashboard | 기본 모니터링 무료 |

### 4.4 Background Job Architecture (알림 엔진)

```
[매물 수집 Cron - 30분 간격]
  1. alert_rules 테이블에서 활성 조건의 관심 단지 목록 추출
  2. 각 단지별 네이버 부동산 API 호출 (rate limit 준수)
  3. 신규/변경 매물을 listings 테이블에 upsert
  4. Supabase Realtime으로 "new_listing" 이벤트 발행

[알림 매칭 Cron - 신규 매물 발생 시 (또는 5분 간격)]
  1. 최근 수집된 신규 매물 조회
  2. 각 매물에 대해 매칭되는 alert_rules 검색
     - price_watch: listing.price <= alert_rule.target_price
     - condition_match: listing 속성 vs conditions JSONB 매칭
  3. 매칭된 (user, listing) 쌍으로 notification 생성
  4. Web Push / Email 발송
```

---

## 5. Technical Constraints and Risks (기술적 제약사항 및 리스크)

### 5.1 Critical Risks

| # | Risk | Impact | Probability | Mitigation |
|---|------|--------|-------------|------------|
| R1 | 네이버 부동산 API 차단 | 매물 데이터 수집 불가 (서비스 핵심 기능 마비) | 높음 | 호출 간격 조절, 사용자 요청 기반 최소 수집, 장기적으로 공식 제휴 추진 |
| R2 | 매물 데이터 정확성 | 허위매물, 이미 거래 완료된 매물 알림 -> 사용자 신뢰 하락 | 중간 | 매물 활성 상태 주기적 검증, 사용자 신고 기능, 실거래가 데이터와 교차 검증 |
| R3 | 법적 리스크 (크롤링) | 서비스 중단 명령 | 중간 | 공공 API 중심 설계, 크롤링은 최소화, 법률 검토 필수 |
| R4 | Supabase Edge Functions 한계 | Cron 실행 시간 제한 (150초), 동시 실행 제한 | 낮음 | 배치 크기 조절, 필요시 외부 Cron 서비스(Upstash QStash) 활용 |
| R5 | Web Push 제한 | iOS Safari에서 PWA로만 동작, 도달률 불확실 | 중간 | 이메일 알림 병행, 장기적으로 네이티브 앱 전환 |

### 5.2 Technical Constraints

1. **Supabase Free Tier 제한**: DB 500MB, Edge Functions 500K 호출/월, Storage 1GB. MVP 단계에서는 충분하나, 사용자 증가 시 Pro 플랜($25/월) 전환 필요.
2. **Vercel Free Tier 제한**: Serverless Function 실행 시간 10초, 대역폭 100GB. SSG 활용으로 최적화 가능.
3. **Kakao Maps API**: 일 300,000 호출 무료. 초기에는 충분하나, 지도 중심 서비스 특성상 사용량 모니터링 필요.
4. **실거래가 API 호출 제한**: 일 1,000건 (기본). 대량 수집 시 트래픽 제한 해제 신청 필요.

### 5.3 MVP Scope Decision

**Phase 1 MVP에 포함 (8주)**:
- F1 회원가입/로그인 (Supabase Auth)
- F2 관심 단지 등록
- F3 지정가 알림 설정
- F5 매물 카드 UI
- F6 매물 상세 페이지
- F7 교통 호재 지도 (수도권 주요 5개 노선)
- F8 관심 목록
- F9 Web Push 알림
- F10 온보딩 플로우

**Phase 1에서 제외 (Phase 2로 연기)**:
- F4 복합 조건 알림 (조건 매칭 엔진 복잡도 높음, MVP에서는 단지+가격 조건만)
- F11 인플루언서 픽 (콘텐츠 큐레이션 운영 비용)
- F12-F17 기타 고급 기능

### 5.4 MVP Timeline (8주)

| Week | Sprint | Deliverable |
|------|--------|-------------|
| 1-2 | Sprint 1 | 프로젝트 셋업, Supabase 스키마, Auth, 단지 데이터 수집 파이프라인 |
| 3-4 | Sprint 2 | 매물 카드 UI, 매물 상세 페이지, 관심 단지/목록 기능 |
| 5-6 | Sprint 3 | 지정가 알림 설정 UI, 매물 수집 Cron, 알림 매칭 엔진, Web Push |
| 7-8 | Sprint 4 | 교통 호재 지도, 온보딩 플로우, QA, 베타 출시 |

---

## 6. Summary

PropertyAgent는 **"부동산 매물을 쇼핑하듯 탐색하고, 내 조건에 맞는 매물은 자동으로 알림받는"** 서비스입니다.

**핵심 차별화**:
1. 기존 서비스가 "새 매물 알림"만 제공하는 반면, 우리는 **"지정가 이하 매물 알림"**을 제공
2. 교통 호재를 뉴스가 아닌 **"정확한 좌표 기반 지도 시각화"**로 제공
3. 부동산을 **"커머스형 카드 UI"**로 탐색하는 직관적 경험

**MVP 핵심 성공 지표**:
- 알림 설정 완료율 > 60% (온보딩 완료 기준)
- 알림 클릭률 > 30%
- 주간 리텐션 > 50% (WAU/MAU)
- 베타 테스터 100명 중 NPS > 40

---

> 솔루션 정의가 완료되었습니다. 2-Pager 작성을 위해 two-pager-writer 에이전트에 전달하세요.

---

### Sources
- [호갱노노 - 아파트 실거래가 1등 앱](https://hogangnono.com/)
- [직방, 호갱노노에 전국 아파트 매물 정보 서비스](https://www.newspim.com/news/view/20241018000466)
- [부동산 어플 추천 - 네이버/직방/호갱노노/타운](https://weolbu.com/community/3399015/)
- [국토교통부 아파트 매매 실거래가 API](https://www.data.go.kr/data/15126469/openapi.do)
- [국토교통부 실거래가 공개시스템](https://rt.molit.go.kr/)
- [PublicDataReader - 부동산 실거래가 조회](https://wooiljeong.github.io/python/public_data_reader_01/)
- [네이버 부동산 매물 정보 수집 자동화](https://blog.hashscraper.com/how-to-automate-collecting-real-estate-listings/)
- [10 Proptech Trends in 2025](https://www.netguru.com/blog/proptech-trends-digital-acceleration)
- [신분당선 연장 - 나무위키](https://namu.wiki/w/%EC%8B%A0%EB%B6%84%EB%8B%B9%EC%84%A0/%EC%97%B0%EC%9E%A5)
- [수도권 광역급행철도 - 나무위키](https://namu.wiki/w/%EC%88%98%EB%8F%84%EA%B6%8C%20%EA%B4%91%EC%97%AD%EA%B8%89%ED%96%89%EC%B2%A0%EB%8F%84)
