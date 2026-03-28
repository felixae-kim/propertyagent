# PropertyAgent 데이터 전략 v2 -- 크롤링 기반 재설계

> 작성일: 2026-03-28
> 전제: 비상업 개인 프로젝트 (소규모 사용, 법적 리스크 최소)
> 목적: 네이버부동산/직방 비공식 API + 국토부 공공 API를 조합한 실전 데이터 파이프라인 설계

---

## 1. 네이버부동산 비공식 API 상세 분석

네이버부동산은 공식 외부 API를 제공하지 않지만, 웹/모바일 프론트엔드가 내부적으로 호출하는 REST API가 존재한다. 크게 두 가지 도메인으로 나뉜다.

### 1.1 new.land.naver.com/api -- 지역/단지/매물 조회

이 API는 네이버부동산 웹 프론트엔드가 사용하는 핵심 API이다.

#### 공통 요청 헤더 (필수)

```
Accept-Encoding: gzip
Host: new.land.naver.com
Referer: https://new.land.naver.com/complexes/{complexNo}?ms=37.5,127.0,16&a=APT&b=A1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
```

> 핵심: `User-Agent`와 `Referer` 헤더가 없으면 403 또는 빈 응답이 반환된다.

#### API 1: 지역 목록 조회 (시도 > 시군구 > 읍면동 계층 탐색)

```
GET https://new.land.naver.com/api/regions/list?cortarNo={cortarNo}
```

| 파라미터 | 설명 | 예시 |
|---------|------|------|
| cortarNo | 법정동 코드 (10자리). 0000000000이면 시도 목록 반환 | 1100000000 (서울) |

응답 예시:
```json
{
  "regionList": [
    {
      "cortarNo": "1100000000",
      "cortarNm": "서울시",
      "cortarType": "city",
      "centerLat": 37.566,
      "centerLon": 126.978
    },
    {
      "cortarNo": "1168000000",
      "cortarNm": "강남구",
      "cortarType": "gungu",
      "centerLat": 37.517,
      "centerLon": 127.047
    }
  ]
}
```

계층 탐색 흐름:
1. `cortarNo=0000000000` -- 시도 목록 (서울, 경기, ...)
2. `cortarNo=1100000000` -- 서울시 시군구 목록 (강남구, 서초구, ...)
3. `cortarNo=1168000000` -- 강남구 읍면동 목록 (대치동, 역삼동, ...)

#### API 2: 특정 동의 아파트 단지 목록

```
GET https://new.land.naver.com/api/regions/complexes?cortarNo={dongCode}&realEstateType=APT&order=
```

| 파라미터 | 설명 | 예시 |
|---------|------|------|
| cortarNo | 읍면동 코드 (10자리) | 1168010600 (대치동) |
| realEstateType | 부동산 유형 | APT, OPST, ABYG, VL 등 |
| order | 정렬 기준 | (빈 값 = 기본 정렬) |

응답 예시:
```json
{
  "complexList": [
    {
      "complexNo": "19204",
      "complexName": "은마아파트",
      "cortarNo": "1168010600",
      "realEstateTypeName": "아파트",
      "totalHouseholdCount": 4424,
      "totalBuildingCount": 28,
      "highFloor": 14,
      "lowFloor": 1,
      "useApproveYmd": "19790901",
      "dealCount": 5,
      "leaseCount": 12,
      "rentCount": 3,
      "latitude": 37.5018,
      "longitude": 127.0565
    }
  ]
}
```

#### API 3: 아파트 단지 상세 정보

```
GET https://new.land.naver.com/api/complexes/{complexNo}
```

응답 예시:
```json
{
  "complexDetail": {
    "complexNo": "19204",
    "complexName": "은마아파트",
    "address": "서울시 강남구 대치동 955",
    "roadAddress": "서울시 강남구 삼성로 212",
    "totalHouseholdCount": 4424,
    "totalBuildingCount": 28,
    "highFloor": 14,
    "lowFloor": 1,
    "useApproveYmd": "19790901",
    "dealCount": 5,
    "leaseCount": 12,
    "floorAreaRatio": "169%",
    "buildingCoverageRatio": "18%",
    "parkingTotalCount": 2500,
    "parkingHouseholdCount": 0.56,
    "constructionCompanyName": "삼성건설",
    "heatMethodTypeName": "개별난방",
    "heatFuelTypeName": "도시가스"
  },
  "complexPyeongDetailList": [
    {
      "pyeongNo": 1,
      "supplyArea": 115.7,
      "exclusiveArea": 84.43,
      "pyeongName": "34",
      "householdCountByPyeong": 1992,
      "entranceType": "계단식"
    }
  ]
}
```

#### API 4: 단지별 매물 목록 (핵심)

```
GET https://new.land.naver.com/api/articles/complex/{complexNo}?realEstateType=APT&tradeType=A1&tag=:::::::&rentPriceMin=0&rentPriceMax=900000000&priceMin=0&priceMax=900000000&areaMin=0&areaMax=900000000&oldBuildYears&recentlyBuildYears&minHouseHoldCount&maxHouseHoldCount&showArticle=false&sameAddressGroup=true&minMaintenanceCost&maxMaintenanceCost&priceType=TOTAL&directions=&page=1&complexNo={complexNo}&buildingNos=&areaNos=&type=list&order=rank
```

주요 파라미터:

| 파라미터 | 설명 | 값 |
|---------|------|------|
| realEstateType | 부동산 유형 | APT, OPST |
| tradeType | 거래 유형 | A1(매매), B1(전세), B2(월세) |
| priceMin/priceMax | 가격 범위 (만원) | 0 ~ 900000000 |
| areaMin/areaMax | 면적 범위 | 0 ~ 900000000 |
| page | 페이지 번호 | 1, 2, 3... |
| order | 정렬 | rank, prc, spc, date_ |

응답 예시:
```json
{
  "articleList": [
    {
      "articleNo": "2450000001",
      "articleName": "은마",
      "realEstateTypeName": "아파트",
      "tradeTypeName": "매매",
      "floorInfo": "10/14",
      "dealOrWarrantPrc": "25억",
      "areaName": "84",
      "area1": 115.7,
      "area2": 84.43,
      "direction": "남향",
      "articleConfirmYmd": "2026.03.27",
      "articleFeatureDesc": "올수리 남향 로얄층",
      "tagList": ["올수리", "남향", "방3개"],
      "buildingName": "101동",
      "sameAddrCnt": 3,
      "sameAddrDirectCnt": 1,
      "cpLabelName": "네이버부동산",
      "realtorName": "XX공인중개사무소",
      "representativeImgUrl": "https://landthumb-phinf.pstatic.net/..."
    }
  ],
  "mapExposedCount": 45,
  "nonMapExposedCount": 12,
  "totalCount": 57,
  "isMoreData": true
}
```

#### API 5: 단지 시세/가격 이력

```
GET https://new.land.naver.com/api/complexes/{complexNo}/prices?complexNo={complexNo}&tradeType=A1&areaNo={pyeongNo}&type=table
```

응답: 월별 시세 변동 테이블 데이터 (매매가, 전세가, 변동률)

#### API 6: 학군 정보

```
GET https://new.land.naver.com/api/complexes/{complexNo}/schools
```

### 1.2 m.land.naver.com -- 지도 기반 매물 클러스터 조회

모바일 웹에서 지도를 움직일 때 호출되는 API. 좌표 범위 내 매물 클러스터를 반환한다.

#### API 7: 지도 영역 내 매물 목록

```
GET https://m.land.naver.com/cluster/ajax/articleList?rletTpCd={types}&tradTpCd={tradeTypes}&z={zoom}&lat={lat}&lon={lon}&btm={bottom}&lft={left}&top={top}&rgt={right}&spcMin={minArea}&spcMax={maxArea}&dprcMax={maxDeposit}&wprcMax={maxRent}&totCnt={total}
```

| 파라미터 | 설명 | 예시 |
|---------|------|------|
| rletTpCd | 부동산 유형 (복수: 콜론 구분) | APT, OPST, APT:OPST |
| tradTpCd | 거래 유형 (복수: 콜론 구분) | A1, B1, B2, A1:B1 |
| z | 지도 줌 레벨 | 13~19 |
| lat, lon | 중심 좌표 | 37.5018, 127.0565 |
| btm, lft, top, rgt | 지도 영역 경계 좌표 | (위도/경도) |
| spcMin, spcMax | 면적 범위 (m2) | 33, 900000000 |
| dprcMax | 보증금 최대 (만원) | 40000 |
| wprcMax | 월세 최대 (만원) | 10000 |
| totCnt | 전체 매물 수 | 41 |

부동산 유형 코드:

| 코드 | 유형 |
|-----|------|
| APT | 아파트 |
| OPST | 오피스텔 |
| ABYG | 아파트분양권 |
| JGC | 재건축 |
| OBYG | 오피스텔분양권 |
| VL | 빌라/연립 |
| JWJT | 주택/다가구 |
| DDDGG | 단독/다가구 |
| OR | 원룸 |

거래 유형 코드:

| 코드 | 유형 |
|-----|------|
| A1 | 매매 |
| B1 | 전세 |
| B2 | 월세 |
| B3 | 단기임대 |

#### API 8: 지도 영역 내 단지 클러스터

```
GET https://m.land.naver.com/cluster/ajax/complexList?rletTpCd=APT&tradTpCd=A1&z={zoom}&lat={lat}&lon={lon}&btm={btm}&lft={lft}&top={top}&rgt={rgt}
```

### 1.3 fin.land.naver.com -- 매물 상세 정보

네이버 부동산 상세 페이지가 호출하는 API.

#### API 9: 매물 상세 키 정보

```
GET https://fin.land.naver.com/front-api/v1/article/key?articleId={articleNo}
```

#### API 10: 매물 기본 정보

```
GET https://fin.land.naver.com/front-api/v1/article/basicInfo?articleId={articleNo}&realEstateType=A02&tradeType=A1
```

#### API 11: 단지 정보 (fin 도메인)

```
GET https://fin.land.naver.com/front-api/v1/complex?complexNumber={complexNo}
```

#### API 12: 교통 정보

```
GET https://fin.land.naver.com/front-api/v1/article/transport?itemId={articleNo}&itemType=article
```

### 1.4 Rate Limit 및 차단 방지 전략

네이버는 짧은 시간에 대량 요청 시 일시적으로 IP를 차단한다 (CAPTCHA 또는 403 반환).

**권장 전략:**

| 항목 | 설정값 |
|-----|--------|
| 요청 간격 | 2~5초 (랜덤 jitter 포함) |
| 시간당 최대 요청 | 200~300회 |
| User-Agent | 실제 브라우저 UA 문자열 사용, 3~5개 로테이션 |
| Referer | 해당 단지/지역의 실제 URL 설정 |
| 차단 감지 | HTTP 403, 빈 응답, CAPTCHA 리다이렉트 감지 |
| 차단 시 대응 | 30분~1시간 대기 후 재시도 |
| IP 관리 | 개인 프로젝트이므로 단일 IP 사용 (과도한 요청만 피하면 됨) |

```typescript
// 요청 간격 예시 (Supabase Edge Function용)
const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

async function fetchWithThrottle(url: string, headers: Record<string, string>) {
  const jitter = Math.random() * 3000 + 2000; // 2~5초
  await delay(jitter);

  const response = await fetch(url, { headers });

  if (response.status === 403 || response.status === 429) {
    console.log('Rate limited. Backing off for 30 minutes.');
    throw new Error('RATE_LIMITED');
  }

  return response.json();
}
```

---

## 2. 직방 비공식 API 상세 분석

직방도 공식 API를 외부에 제공하지 않지만, 앱/웹이 내부적으로 사용하는 REST API가 존재한다.

### 2.1 API 엔드포인트

#### API 1: 지역 기반 매물 검색 (Geohash 방식)

```
GET https://apis.zigbang.com/v2/items?deposit_gteq=0&domain=zigbang&geohash={geohash}&rent_gteq=0&sales_type_in=전세|월세&service_type_eq=원룸
```

| 파라미터 | 설명 | 예시 |
|---------|------|------|
| geohash | Geohash 문자열 (precision 5) | wydm6 |
| deposit_gteq | 보증금 최소 (만원) | 0 |
| rent_gteq | 월세 최소 (만원) | 0 |
| sales_type_in | 거래 유형 | 전세, 월세, 매매 (파이프로 구분) |
| service_type_eq | 매물 유형 | 원룸, 빌라, 오피스텔, 아파트 |
| domain | 도메인 | zigbang |

Geohash 생성 방법 (precision 5):
```typescript
// npm install ngeohash
import geohash from 'ngeohash';

// 강남역 좌표 -> geohash (precision 5)
const hash = geohash.encode(37.4979, 127.0276, 5); // "wydm6"
```

응답 예시:
```json
{
  "items": [
    { "item_id": 12345678 },
    { "item_id": 12345679 },
    { "item_id": 12345680 }
  ]
}
```

#### API 2: 매물 상세 정보 (일괄 조회)

```
POST https://apis.zigbang.com/v2/items/list
Content-Type: application/json

{
  "item_ids": [12345678, 12345679, 12345680]
}
```

응답 예시:
```json
{
  "items": [
    {
      "item_id": 12345678,
      "section_type": "원룸",
      "images_thumbnail": "https://ic.zigbang.com/...",
      "sales_type": "월세",
      "deposit": 1000,
      "rent": 50,
      "size_m2": 26.4,
      "address1": "서울시 강남구 역삼동",
      "manage_cost": "5",
      "floor": "3",
      "building_floor": "5",
      "title": "역삼역 도보 5분 풀옵션 원룸",
      "is_first_movein": false,
      "random_location": { "lat": 37.4979, "lng": 127.0276 }
    }
  ]
}
```

#### API 3: 지역별 아파트 단지 목록

```
GET https://apis.zigbang.com/property/biglab/apartments/list?type=local&id={localId}
```

| 파라미터 | 설명 | 예시 |
|---------|------|------|
| type | 조회 유형 | local (지역 기반) |
| id | 법정동/시군구 코드 | 11680 (강남구) |

#### API 4: v1 매물 상세 (구버전이지만 아직 동작할 수 있음)

```
GET https://api.zigbang.com/v1/items?detail=true&item_ids={id1},{id2}
```

### 2.2 직방 API 특징 및 제한

| 항목 | 내용 |
|-----|------|
| 인증 | 불필요 (공개 엔드포인트) |
| Rate Limit | 명확하지 않으나, 과도한 요청 시 차단 가능 |
| 데이터 범위 | 원룸/빌라/오피스텔이 주력, 아파트는 상대적으로 적음 |
| 위치 정보 | random_location으로 정확한 위치가 아닌 근사 좌표 제공 |
| 이미지 | 썸네일 URL 제공 |

### 2.3 직방 vs 네이버부동산 비교

| 기준 | 네이버부동산 | 직방 |
|-----|------------|------|
| 아파트 매물 | 압도적 1위 | 상대적으로 적음 |
| 원룸/빌라/오피스텔 | 있음 | 매우 강함 |
| API 안정성 | 중간 (차단 가능) | 상대적으로 관대 |
| 데이터 풍부도 | 매우 높음 | 중간 |
| 인증 필요 | 헤더 필수 | 불필요 |

**결론: 아파트 매물은 네이버부동산, 원룸/빌라는 직방을 primary source로 사용.**

---

## 3. 국토교통부 실거래가 공공 API 상세 스펙

### 3.1 API 종류 (12종)

| # | API명 | 엔드포인트 | 데이터 |
|---|-------|-----------|--------|
| 1 | 아파트 매매 실거래 상세 | `/RTMSDataSvcAptTradeDev/getRTMSDataSvcAptTradeDev` | 매매가, 층, 면적, 건축연도 등 |
| 2 | 아파트 전월세 | `/RTMSDataSvcAptRent/getRTMSDataSvcAptRent` | 보증금, 월세, 계약기간 |
| 3 | 아파트 분양권전매 | `/RTMSDataSvcSilvTrade/getRTMSDataSvcSilvTrade` | 분양권 거래 정보 |
| 4 | 오피스텔 매매 | `/RTMSDataSvcOffiTrade/getRTMSDataSvcOffiTrade` | 오피스텔 매매 |
| 5 | 오피스텔 전월세 | `/RTMSDataSvcOffiRent/getRTMSDataSvcOffiRent` | 오피스텔 임대 |
| 6 | 연립다세대 매매 | `/RTMSDataSvcRHTrade/getRTMSDataSvcRHTrade` | 연립/다세대 매매 |
| 7 | 연립다세대 전월세 | `/RTMSDataSvcRHRent/getRTMSDataSvcRHRent` | 연립/다세대 임대 |
| 8 | 단독/다가구 매매 | `/RTMSDataSvcSHTrade/getRTMSDataSvcSHTrade` | 단독/다가구 매매 |
| 9 | 단독/다가구 전월세 | `/RTMSDataSvcSHRent/getRTMSDataSvcSHRent` | 단독/다가구 임대 |
| 10 | 토지 매매 | `/RTMSDataSvcLandTrade/getRTMSDataSvcLandTrade` | 토지 거래 |
| 11 | 상업업무용 매매 | `/RTMSDataSvcNrgTrade/getRTMSDataSvcNrgTrade` | 상가/사무실 매매 |
| 12 | 공장/창고 매매 | `/RTMSDataSvcInduTrade/getRTMSDataSvcInduTrade` | 공장/창고 거래 |

### 3.2 기본 호출 방법

**Base URL:**
```
https://apis.data.go.kr/1613000/{서비스명}/{오퍼레이션명}
```

**공통 요청 파라미터:**

| 파라미터 | 필수 | 설명 | 예시 |
|---------|------|------|------|
| serviceKey | Y | API 인증키 (공공데이터포털에서 발급) | Decoding된 키 |
| LAWD_CD | Y | 법정동 시군구코드 (5자리) | 11680 (강남구) |
| DEAL_YMD | Y | 계약년월 (6자리) | 202603 |
| pageNo | N | 페이지 번호 (기본 1) | 1 |
| numOfRows | N | 한 페이지 결과 수 (기본 10, 최대 1000) | 100 |

**호출 예시 (아파트 매매):**
```
GET https://apis.data.go.kr/1613000/RTMSDataSvcAptTradeDev/getRTMSDataSvcAptTradeDev?serviceKey={KEY}&LAWD_CD=11680&DEAL_YMD=202603&pageNo=1&numOfRows=100
```

### 3.3 응답 구조 (XML)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<response>
  <header>
    <resultCode>00</resultCode>
    <resultMsg>NORMAL SERVICE.</resultMsg>
  </header>
  <body>
    <items>
      <item>
        <aptNm>은마아파트</aptNm>
        <aptDong>101동</aptDong>
        <aptSeq>11680-19204</aptSeq>
        <buildYear>1979</buildYear>
        <dealAmount>240,000</dealAmount>
        <dealDay>15</dealDay>
        <dealMonth>03</dealMonth>
        <dealYear>2026</dealYear>
        <dealingGbn>중개거래</dealingGbn>
        <excluUseAr>84.43</excluUseAr>
        <floor>10</floor>
        <sggCd>11680</sggCd>
        <umdCd>10600</umdCd>
        <umdNm>대치동</umdNm>
        <estateAgentSggNm>강남구</estateAgentSggNm>
        <buyerGbn>개인</buyerGbn>
        <slerGbn>개인</slerGbn>
        <rgstDate>20260320</rgstDate>
        <landLeaseholdGbn></landLeaseholdGbn>
      </item>
    </items>
    <numOfRows>100</numOfRows>
    <pageNo>1</pageNo>
    <totalCount>48</totalCount>
  </body>
</response>
```

### 3.4 응답 필드 상세 (아파트 매매)

| 필드명 | 설명 | 데이터 타입 |
|--------|------|-----------|
| aptNm | 아파트명 | 문자열 |
| aptDong | 동명 | 문자열 |
| aptSeq | 단지 일련번호 | 문자열 |
| buildYear | 건축연도 | 숫자 |
| dealAmount | 거래금액 (만원, 쉼표 포함) | 문자열 |
| dealDay | 계약일 | 숫자 |
| dealMonth | 계약월 | 숫자 |
| dealYear | 계약연도 | 숫자 |
| dealingGbn | 거래유형 (중개거래/직거래) | 문자열 |
| excluUseAr | 전용면적 (m2) | 실수 |
| floor | 층 | 숫자 |
| sggCd | 시군구코드 | 문자열 |
| umdCd | 읍면동코드 | 문자열 |
| umdNm | 읍면동명 | 문자열 |
| estateAgentSggNm | 중개사 소재지 | 문자열 |
| buyerGbn | 매수자 구분 (개인/법인 등) | 문자열 |
| slerGbn | 매도자 구분 | 문자열 |
| rgstDate | 등기일자 | 문자열 |
| landLeaseholdGbn | 토지임대부 여부 | 문자열 |

### 3.5 주요 법정동 시군구 코드

| 코드 | 지역 |
|-----|------|
| 11110 | 서울 종로구 |
| 11140 | 서울 중구 |
| 11170 | 서울 용산구 |
| 11200 | 서울 성동구 |
| 11305 | 서울 강북구 |
| 11380 | 서울 은평구 |
| 11500 | 서울 강서구 |
| 11560 | 서울 영등포구 |
| 11590 | 서울 동작구 |
| 11620 | 서울 관악구 |
| 11650 | 서울 서초구 |
| 11680 | 서울 강남구 |
| 11710 | 서울 송파구 |
| 11740 | 서울 강동구 |
| 41111 | 경기 수원 장안구 |
| 41131 | 경기 성남 수정구 |
| 41135 | 경기 성남 분당구 |
| 41281 | 경기 하남시 |
| 41465 | 경기 화성시 |

### 3.6 제한 및 유의사항

| 항목 | 내용 |
|-----|------|
| 일일 호출 제한 | 1,000회/일 (기본), 신청 시 확장 가능 |
| 응답 형식 | XML (JSON 미지원) |
| 데이터 지연 | 거래 신고 후 1~2개월 후 반영 |
| 데이터 성격 | 과거 완료 거래만 (현재 매물 아님) |
| 거래 취소 | dealingGbn이 '해제'인 경우 거래 취소건 |

---

## 4. 수정된 데이터 파이프라인 설계

### 4.1 전체 아키텍처

```
[데이터 수집 레이어]
  |
  +-- Supabase Edge Function (Cron) --+-- 네이버부동산 API 크롤러
  |                                     +-- 직방 API 크롤러
  |                                     +-- 국토부 실거래가 API 수집기
  |
  v
[데이터 저장 레이어]
  |
  +-- Supabase PostgreSQL
  |     +-- raw_naver_articles (네이버 매물 원본)
  |     +-- raw_zigbang_items (직방 매물 원본)
  |     +-- raw_molit_trades (실거래가 원본)
  |     +-- listings (정규화된 매물 통합 테이블)
  |     +-- complexes (아파트 단지 마스터)
  |     +-- trade_history (실거래 이력)
  |
  v
[조건 매칭 레이어]
  |
  +-- Supabase Edge Function (Trigger/Cron)
  |     +-- 신규 매물 vs 사용자 조건 매칭
  |     +-- 가격 변동 감지
  |     +-- 급매 신호 탐지
  |
  v
[알림 발송 레이어]
  |
  +-- Supabase Edge Function
        +-- 카카오톡 알림 (카카오 알림톡 API)
        +-- 이메일 (Resend / SendGrid)
        +-- 앱 푸시 (웹 Push API)
```

### 4.2 크롤링 스케줄 설계

| 작업 | 주기 | 대상 | 예상 API 호출 수 |
|------|------|------|-----------------|
| 네이버 관심 단지 매물 수집 | 30분마다 | 사용자가 등록한 관심 단지 (최대 50개) | ~100회/30분 |
| 네이버 관심 지역 매물 수집 | 2시간마다 | 사용자가 등록한 관심 동 (최대 20개) | ~200회/2시간 |
| 직방 매물 수집 | 1시간마다 | 관심 지역 geohash (최대 30개) | ~100회/시간 |
| 국토부 실거래가 수집 | 1일 1회 (새벽 3시) | 관심 지역 시군구 (최대 10개) x 당월 | ~20회/일 |
| 단지 마스터 갱신 | 1주 1회 (일요일 새벽) | 관심 동의 단지 목록 | ~50회 |

### 4.3 핵심 Edge Function 구현

#### 4.3.1 네이버부동산 매물 수집기

```typescript
// supabase/functions/crawl-naver-listings/index.ts

import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const NAVER_HEADERS = {
  'Accept-Encoding': 'gzip',
  'Host': 'new.land.naver.com',
  'Sec-Fetch-Dest': 'empty',
  'Sec-Fetch-Mode': 'cors',
  'Sec-Fetch-Site': 'same-origin',
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
};

const USER_AGENTS = [
  'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
  'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36',
  'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:121.0) Gecko/20100101 Firefox/121.0',
];

function getRandomUA() {
  return USER_AGENTS[Math.floor(Math.random() * USER_AGENTS.length)];
}

function delay(ms: number) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

interface NaverArticle {
  articleNo: string;
  articleName: string;
  realEstateTypeName: string;
  tradeTypeName: string;
  floorInfo: string;
  dealOrWarrantPrc: string;
  areaName: string;
  area1: number;
  area2: number;
  direction: string;
  articleConfirmYmd: string;
  articleFeatureDesc: string;
  tagList: string[];
  buildingName: string;
  realtorName: string;
  representativeImgUrl: string;
}

async function fetchComplexArticles(
  complexNo: string,
  tradeType: string = 'A1',
  page: number = 1
): Promise<{ articleList: NaverArticle[]; totalCount: number; isMoreData: boolean }> {
  const url = new URL(`https://new.land.naver.com/api/articles/complex/${complexNo}`);
  url.searchParams.set('realEstateType', 'APT');
  url.searchParams.set('tradeType', tradeType);
  url.searchParams.set('tag', ':::::::');
  url.searchParams.set('rentPriceMin', '0');
  url.searchParams.set('rentPriceMax', '900000000');
  url.searchParams.set('priceMin', '0');
  url.searchParams.set('priceMax', '900000000');
  url.searchParams.set('areaMin', '0');
  url.searchParams.set('areaMax', '900000000');
  url.searchParams.set('showArticle', 'false');
  url.searchParams.set('sameAddressGroup', 'true');
  url.searchParams.set('page', String(page));
  url.searchParams.set('complexNo', complexNo);
  url.searchParams.set('type', 'list');
  url.searchParams.set('order', 'rank');

  const headers = {
    ...NAVER_HEADERS,
    'User-Agent': getRandomUA(),
    'Referer': `https://new.land.naver.com/complexes/${complexNo}?ms=37.5,127.0,16&a=APT&b=${tradeType}`,
  };

  const response = await fetch(url.toString(), { headers });

  if (!response.ok) {
    throw new Error(`Naver API error: ${response.status}`);
  }

  return response.json();
}

Deno.serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );

  // 관심 단지 목록 조회
  const { data: watchedComplexes } = await supabase
    .from('watched_complexes')
    .select('complex_no, complex_name, trade_types')
    .eq('is_active', true);

  if (!watchedComplexes || watchedComplexes.length === 0) {
    return new Response(JSON.stringify({ message: 'No watched complexes' }), {
      headers: { 'Content-Type': 'application/json' },
    });
  }

  const results = [];

  for (const complex of watchedComplexes) {
    try {
      for (const tradeType of complex.trade_types) {
        // 요청 간격: 2~5초
        await delay(Math.random() * 3000 + 2000);

        const data = await fetchComplexArticles(complex.complex_no, tradeType);

        if (data.articleList && data.articleList.length > 0) {
          // raw 데이터 저장 (upsert)
          const rows = data.articleList.map(article => ({
            article_no: article.articleNo,
            complex_no: complex.complex_no,
            source: 'naver',
            trade_type: tradeType,
            raw_data: article,
            floor_info: article.floorInfo,
            price_text: article.dealOrWarrantPrc,
            area_supply: article.area1,
            area_exclusive: article.area2,
            direction: article.direction,
            confirm_date: article.articleConfirmYmd,
            description: article.articleFeatureDesc,
            tags: article.tagList,
            realtor_name: article.realtorName,
            crawled_at: new Date().toISOString(),
          }));

          const { error } = await supabase
            .from('raw_naver_articles')
            .upsert(rows, { onConflict: 'article_no' });

          if (error) {
            console.error(`Error upserting articles for ${complex.complex_no}:`, error);
          }

          results.push({
            complexNo: complex.complex_no,
            tradeType,
            count: data.articleList.length,
          });
        }
      }
    } catch (err) {
      console.error(`Error crawling ${complex.complex_no}:`, err);

      if (err.message === 'RATE_LIMITED') {
        // Rate limited - 로그 기록 후 중단
        await supabase.from('crawl_logs').insert({
          source: 'naver',
          status: 'rate_limited',
          message: `Rate limited at complex ${complex.complex_no}`,
          created_at: new Date().toISOString(),
        });
        break; // 나머지 단지도 차단될 것이므로 중단
      }
    }
  }

  return new Response(JSON.stringify({ results }), {
    headers: { 'Content-Type': 'application/json' },
  });
});
```

#### 4.3.2 국토부 실거래가 수집기

```typescript
// supabase/functions/crawl-molit-trades/index.ts

import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const MOLIT_BASE_URL = 'https://apis.data.go.kr/1613000';

const API_ENDPOINTS: Record<string, string> = {
  apt_trade: 'RTMSDataSvcAptTradeDev/getRTMSDataSvcAptTradeDev',
  apt_rent: 'RTMSDataSvcAptRent/getRTMSDataSvcAptRent',
  offi_trade: 'RTMSDataSvcOffiTrade/getRTMSDataSvcOffiTrade',
  offi_rent: 'RTMSDataSvcOffiRent/getRTMSDataSvcOffiRent',
  rh_trade: 'RTMSDataSvcRHTrade/getRTMSDataSvcRHTrade',
  rh_rent: 'RTMSDataSvcRHRent/getRTMSDataSvcRHRent',
};

async function fetchTradeData(
  apiType: string,
  lawdCd: string,
  dealYmd: string,
  serviceKey: string,
  pageNo: number = 1,
  numOfRows: number = 1000
): Promise<any[]> {
  const endpoint = API_ENDPOINTS[apiType];
  if (!endpoint) throw new Error(`Unknown API type: ${apiType}`);

  const url = new URL(`${MOLIT_BASE_URL}/${endpoint}`);
  url.searchParams.set('serviceKey', serviceKey);
  url.searchParams.set('LAWD_CD', lawdCd);
  url.searchParams.set('DEAL_YMD', dealYmd);
  url.searchParams.set('pageNo', String(pageNo));
  url.searchParams.set('numOfRows', String(numOfRows));

  const response = await fetch(url.toString());
  const xmlText = await response.text();

  // XML 파싱 (Deno 환경)
  // 간단한 정규식 기반 파싱 (production에서는 XML 파서 라이브러리 권장)
  return parseXmlItems(xmlText);
}

function parseXmlItems(xml: string): any[] {
  const items: any[] = [];
  const itemRegex = /<item>([\s\S]*?)<\/item>/g;
  let match;

  while ((match = itemRegex.exec(xml)) !== null) {
    const itemXml = match[1];
    const item: any = {};
    const fieldRegex = /<(\w+)>(.*?)<\/\1>/g;
    let fieldMatch;

    while ((fieldMatch = fieldRegex.exec(itemXml)) !== null) {
      item[fieldMatch[1]] = fieldMatch[2].trim();
    }

    items.push(item);
  }

  return items;
}

function parseDealAmount(amountStr: string): number {
  // "240,000" -> 240000 (만원)
  return parseInt(amountStr.replace(/,/g, ''), 10);
}

Deno.serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );

  const serviceKey = Deno.env.get('MOLIT_API_KEY')!;

  // 관심 지역 목록
  const { data: watchedRegions } = await supabase
    .from('watched_regions')
    .select('lawd_cd, region_name, api_types')
    .eq('is_active', true);

  if (!watchedRegions || watchedRegions.length === 0) {
    return new Response(JSON.stringify({ message: 'No watched regions' }));
  }

  // 현재 년월 (YYYYMM)
  const now = new Date();
  const dealYmd = `${now.getFullYear()}${String(now.getMonth() + 1).padStart(2, '0')}`;

  const results = [];

  for (const region of watchedRegions) {
    for (const apiType of region.api_types) {
      try {
        const items = await fetchTradeData(apiType, region.lawd_cd, dealYmd, serviceKey);

        if (items.length > 0) {
          const rows = items.map(item => ({
            source_api: apiType,
            lawd_cd: region.lawd_cd,
            deal_ymd: dealYmd,
            apt_name: item.aptNm || item.offiNm || null,
            apt_dong: item.aptDong || null,
            build_year: item.buildYear ? parseInt(item.buildYear) : null,
            deal_amount: item.dealAmount ? parseDealAmount(item.dealAmount) : null,
            deposit: item.deposit ? parseDealAmount(item.deposit) : null,
            monthly_rent: item.monthlyRent ? parseInt(item.monthlyRent) : null,
            deal_year: parseInt(item.dealYear),
            deal_month: parseInt(item.dealMonth),
            deal_day: parseInt(item.dealDay),
            exclusive_area: item.excluUseAr ? parseFloat(item.excluUseAr) : null,
            floor: item.floor ? parseInt(item.floor) : null,
            umd_name: item.umdNm || null,
            dealing_type: item.dealingGbn || null,
            buyer_type: item.buyerGbn || null,
            seller_type: item.slerGbn || null,
            reg_date: item.rgstDate || null,
            raw_data: item,
            crawled_at: new Date().toISOString(),
          }));

          // 중복 방지: source_api + lawd_cd + apt_name + deal_year + deal_month + deal_day + floor + exclusive_area
          const { error } = await supabase
            .from('raw_molit_trades')
            .upsert(rows, {
              onConflict: 'source_api,lawd_cd,apt_name,deal_year,deal_month,deal_day,floor,exclusive_area',
            });

          if (error) console.error(`Molit upsert error:`, error);

          results.push({
            region: region.region_name,
            apiType,
            count: items.length,
          });
        }
      } catch (err) {
        console.error(`Error fetching ${apiType} for ${region.lawd_cd}:`, err);
      }
    }
  }

  return new Response(JSON.stringify({ results }), {
    headers: { 'Content-Type': 'application/json' },
  });
});
```

#### 4.3.3 조건 매칭 및 알림 트리거

```typescript
// supabase/functions/match-and-notify/index.ts

import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

interface UserCondition {
  id: string;
  user_id: string;
  condition_name: string;
  region_codes: string[];      // 관심 지역
  complex_nos: string[];       // 관심 단지
  trade_types: string[];       // A1, B1, B2
  price_min: number | null;    // 최소 가격 (만원)
  price_max: number | null;    // 최대 가격 (만원)
  area_min: number | null;     // 최소 면적 (m2)
  area_max: number | null;     // 최대 면적 (m2)
  floor_min: number | null;    // 최소 층
  keywords: string[];          // 키워드 필터 (올수리, 남향 등)
  notify_channel: string;      // email, kakao, push
}

async function matchNewListings(supabase: any) {
  // 최근 1시간 내 수집된 신규 매물 조회
  const oneHourAgo = new Date(Date.now() - 60 * 60 * 1000).toISOString();

  const { data: newArticles } = await supabase
    .from('raw_naver_articles')
    .select('*')
    .gte('crawled_at', oneHourAgo)
    .eq('notified', false);

  if (!newArticles || newArticles.length === 0) return [];

  // 모든 사용자 조건 조회
  const { data: conditions } = await supabase
    .from('user_conditions')
    .select('*')
    .eq('is_active', true);

  if (!conditions || conditions.length === 0) return [];

  const notifications: any[] = [];

  for (const article of newArticles) {
    for (const condition of conditions as UserCondition[]) {
      if (isMatch(article, condition)) {
        notifications.push({
          user_id: condition.user_id,
          condition_id: condition.id,
          article_no: article.article_no,
          source: article.source,
          complex_no: article.complex_no,
          trade_type: article.trade_type,
          price_text: article.price_text,
          area_exclusive: article.area_exclusive,
          floor_info: article.floor_info,
          description: article.description,
          notify_channel: condition.notify_channel,
          created_at: new Date().toISOString(),
        });
      }
    }
  }

  return notifications;
}

function isMatch(article: any, condition: UserCondition): boolean {
  // 단지 필터
  if (condition.complex_nos.length > 0 &&
      !condition.complex_nos.includes(article.complex_no)) {
    return false;
  }

  // 거래 유형 필터
  if (condition.trade_types.length > 0 &&
      !condition.trade_types.includes(article.trade_type)) {
    return false;
  }

  // 면적 필터
  if (condition.area_min && article.area_exclusive < condition.area_min) {
    return false;
  }
  if (condition.area_max && article.area_exclusive > condition.area_max) {
    return false;
  }

  // 키워드 필터 (태그 또는 설명에 포함)
  if (condition.keywords.length > 0) {
    const articleText = `${article.description || ''} ${(article.tags || []).join(' ')}`;
    const hasKeyword = condition.keywords.some(kw =>
      articleText.includes(kw)
    );
    if (!hasKeyword) return false;
  }

  return true;
}

Deno.serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );

  const notifications = await matchNewListings(supabase);

  if (notifications.length > 0) {
    // 알림 큐에 저장
    await supabase.from('notification_queue').insert(notifications);

    // 알림 발송 처리
    for (const notif of notifications) {
      // 여기서 실제 알림 발송 (이메일, 카카오, 푸시)
      // 별도의 send-notification Edge Function 호출
      console.log(`Notify user ${notif.user_id}: ${notif.price_text} ${notif.description}`);
    }

    // 매물에 notified 플래그 설정
    const articleNos = [...new Set(notifications.map(n => n.article_no))];
    await supabase
      .from('raw_naver_articles')
      .update({ notified: true })
      .in('article_no', articleNos);
  }

  return new Response(JSON.stringify({
    matched: notifications.length,
    timestamp: new Date().toISOString(),
  }), {
    headers: { 'Content-Type': 'application/json' },
  });
});
```

### 4.4 Cron 스케줄 설정 (supabase/config.toml)

```toml
# supabase/config.toml 의 [functions] 섹션 또는
# Supabase Dashboard > Edge Functions > Schedules 에서 설정

# 네이버 매물 수집 - 30분마다
# cron: */30 * * * *
# function: crawl-naver-listings

# 직방 매물 수집 - 1시간마다
# cron: 0 * * * *
# function: crawl-zigbang-listings

# 국토부 실거래가 수집 - 매일 새벽 3시
# cron: 0 3 * * *
# function: crawl-molit-trades

# 조건 매칭 및 알림 - 10분마다
# cron: */10 * * * *
# function: match-and-notify

# 단지 마스터 갱신 - 매주 일요일 새벽 2시
# cron: 0 2 * * 0
# function: refresh-complex-master
```

> 참고: Supabase Edge Functions의 Cron은 pg_cron 또는 외부 Cron 서비스(GitHub Actions, Vercel Cron 등)를 통해 구현한다. Supabase 자체 Cron은 Database Functions (pg_net + pg_cron) 조합으로 Edge Function을 HTTP 호출하는 방식이다.

### 4.5 에러 처리 및 차단 대응 전략

```
[정상 수집]
  |
  v
[HTTP 403 / 429 감지] --> [crawl_logs에 기록] --> [해당 소스 30분 차단 (backoff)]
  |
  v
[연속 3회 차단] --> [해당 소스 6시간 차단] --> [관리자 알림 (이메일)]
  |
  v
[24시간 이상 차단 지속] --> [User-Agent 변경 + 요청 간격 2배 증가]
```

```sql
-- crawl_logs 테이블로 상태 추적
CREATE TABLE crawl_logs (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  source TEXT NOT NULL,           -- 'naver', 'zigbang', 'molit'
  status TEXT NOT NULL,           -- 'success', 'rate_limited', 'error', 'blocked'
  message TEXT,
  items_count INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 최근 차단 여부 확인 함수
CREATE OR REPLACE FUNCTION is_source_blocked(p_source TEXT)
RETURNS BOOLEAN AS $$
  SELECT EXISTS (
    SELECT 1 FROM crawl_logs
    WHERE source = p_source
      AND status IN ('rate_limited', 'blocked')
      AND created_at > NOW() - INTERVAL '30 minutes'
    LIMIT 1
  );
$$ LANGUAGE sql;
```

---

## 5. DB 스키마 설계

### 5.1 전체 ERD 개요

```
users
  |-- user_conditions (1:N)
  |-- watched_complexes (1:N)
  |-- watched_regions (1:N)
  |-- notification_queue (1:N)

complexes (아파트 단지 마스터)
  |-- raw_naver_articles (1:N)
  |-- trade_history (1:N)

raw_naver_articles (네이버 매물 원본)
raw_zigbang_items (직방 매물 원본)
raw_molit_trades (실거래가 원본)

listings (정규화된 통합 매물) -- raw 테이블에서 변환

crawl_logs (크롤링 로그)
```

### 5.2 테이블 정의

```sql
-- ============================================
-- 1. 사용자 관련
-- ============================================

-- 사용자 프로필 (Supabase Auth 확장)
CREATE TABLE user_profiles (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  display_name TEXT,
  email TEXT,
  kakao_id TEXT,                    -- 카카오톡 알림용
  notify_email BOOLEAN DEFAULT true,
  notify_kakao BOOLEAN DEFAULT false,
  notify_push BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 사용자 관심 조건
CREATE TABLE user_conditions (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES user_profiles(id) ON DELETE CASCADE,
  condition_name TEXT NOT NULL,       -- "강남 30평대 매매"
  region_codes TEXT[] DEFAULT '{}',   -- 관심 법정동 코드
  complex_nos TEXT[] DEFAULT '{}',    -- 관심 단지 번호
  trade_types TEXT[] DEFAULT '{A1}',  -- A1(매매), B1(전세), B2(월세)
  real_estate_types TEXT[] DEFAULT '{APT}', -- APT, OPST, VL
  price_min INTEGER,                  -- 최소 가격 (만원)
  price_max INTEGER,                  -- 최대 가격 (만원)
  area_min REAL,                      -- 최소 전용면적 (m2)
  area_max REAL,                      -- 최대 전용면적 (m2)
  floor_min INTEGER,                  -- 최소 층
  floor_max INTEGER,                  -- 최대 층
  keywords TEXT[] DEFAULT '{}',       -- 키워드 필터
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 사용자 관심 단지 (크롤링 대상)
CREATE TABLE watched_complexes (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES user_profiles(id) ON DELETE CASCADE,
  complex_no TEXT NOT NULL,
  complex_name TEXT,
  trade_types TEXT[] DEFAULT '{A1,B1}',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, complex_no)
);

-- 사용자 관심 지역 (실거래가 수집 대상)
CREATE TABLE watched_regions (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES user_profiles(id) ON DELETE CASCADE,
  lawd_cd TEXT NOT NULL,              -- 시군구 5자리 코드
  region_name TEXT,                   -- "서울 강남구"
  api_types TEXT[] DEFAULT '{apt_trade,apt_rent}',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, lawd_cd)
);

-- ============================================
-- 2. 아파트 단지 마스터
-- ============================================

CREATE TABLE complexes (
  complex_no TEXT PRIMARY KEY,        -- 네이버 complexNo
  complex_name TEXT NOT NULL,
  address TEXT,
  road_address TEXT,
  cortarno TEXT,                      -- 법정동 코드
  lawd_cd TEXT,                       -- 시군구 5자리
  total_household INTEGER,
  total_building INTEGER,
  high_floor INTEGER,
  low_floor INTEGER,
  use_approve_ymd TEXT,               -- 사용승인일
  construction_company TEXT,
  heat_method TEXT,
  heat_fuel TEXT,
  parking_total INTEGER,
  floor_area_ratio TEXT,
  building_coverage_ratio TEXT,
  latitude DOUBLE PRECISION,
  longitude DOUBLE PRECISION,
  pyeong_details JSONB,              -- 평형별 상세 정보
  source TEXT DEFAULT 'naver',
  crawled_at TIMESTAMPTZ,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_complexes_lawd ON complexes(lawd_cd);
CREATE INDEX idx_complexes_cortarno ON complexes(cortarno);

-- ============================================
-- 3. Raw 데이터 테이블 (원본 보존)
-- ============================================

-- 네이버부동산 매물 원본
CREATE TABLE raw_naver_articles (
  article_no TEXT PRIMARY KEY,
  complex_no TEXT REFERENCES complexes(complex_no),
  source TEXT DEFAULT 'naver',
  trade_type TEXT NOT NULL,           -- A1, B1, B2
  raw_data JSONB NOT NULL,            -- 전체 원본 JSON
  floor_info TEXT,                    -- "10/14"
  price_text TEXT,                    -- "25억", "3억 5,000"
  area_supply REAL,                   -- 공급면적
  area_exclusive REAL,                -- 전용면적
  direction TEXT,                     -- 남향, 동향 등
  confirm_date TEXT,                  -- 등록확인일
  description TEXT,                   -- 매물 특징
  tags TEXT[],                        -- 태그 목록
  realtor_name TEXT,
  image_url TEXT,
  notified BOOLEAN DEFAULT false,
  is_active BOOLEAN DEFAULT true,     -- 매물 삭제 감지 시 false
  first_seen_at TIMESTAMPTZ DEFAULT NOW(),
  last_seen_at TIMESTAMPTZ DEFAULT NOW(),
  crawled_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_naver_complex ON raw_naver_articles(complex_no);
CREATE INDEX idx_naver_trade ON raw_naver_articles(trade_type);
CREATE INDEX idx_naver_crawled ON raw_naver_articles(crawled_at);
CREATE INDEX idx_naver_notified ON raw_naver_articles(notified) WHERE notified = false;

-- 직방 매물 원본
CREATE TABLE raw_zigbang_items (
  item_id TEXT PRIMARY KEY,
  source TEXT DEFAULT 'zigbang',
  geohash TEXT,
  raw_data JSONB NOT NULL,
  sales_type TEXT,                    -- 매매, 전세, 월세
  service_type TEXT,                  -- 원룸, 빌라, 오피스텔, 아파트
  deposit INTEGER,                    -- 보증금 (만원)
  rent INTEGER,                       -- 월세 (만원)
  size_m2 REAL,                       -- 면적
  address TEXT,
  floor TEXT,
  building_floor TEXT,
  title TEXT,
  manage_cost TEXT,
  latitude DOUBLE PRECISION,
  longitude DOUBLE PRECISION,
  notified BOOLEAN DEFAULT false,
  is_active BOOLEAN DEFAULT true,
  first_seen_at TIMESTAMPTZ DEFAULT NOW(),
  last_seen_at TIMESTAMPTZ DEFAULT NOW(),
  crawled_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_zigbang_geohash ON raw_zigbang_items(geohash);
CREATE INDEX idx_zigbang_sales ON raw_zigbang_items(sales_type);

-- 국토부 실거래가 원본
CREATE TABLE raw_molit_trades (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  source_api TEXT NOT NULL,           -- apt_trade, apt_rent, ...
  lawd_cd TEXT NOT NULL,
  deal_ymd TEXT NOT NULL,             -- YYYYMM
  apt_name TEXT,
  apt_dong TEXT,
  build_year INTEGER,
  deal_amount INTEGER,                -- 거래금액 (만원)
  deposit INTEGER,                    -- 보증금 (만원, 전월세)
  monthly_rent INTEGER,               -- 월세 (만원)
  deal_year INTEGER NOT NULL,
  deal_month INTEGER NOT NULL,
  deal_day INTEGER NOT NULL,
  exclusive_area REAL,
  floor INTEGER,
  umd_name TEXT,
  dealing_type TEXT,                  -- 중개거래/직거래
  buyer_type TEXT,
  seller_type TEXT,
  reg_date TEXT,
  raw_data JSONB,
  crawled_at TIMESTAMPTZ DEFAULT NOW(),
  -- 복합 유니크 키로 중복 방지
  UNIQUE(source_api, lawd_cd, apt_name, deal_year, deal_month, deal_day, floor, exclusive_area)
);

CREATE INDEX idx_molit_lawd ON raw_molit_trades(lawd_cd);
CREATE INDEX idx_molit_apt ON raw_molit_trades(apt_name);
CREATE INDEX idx_molit_deal_date ON raw_molit_trades(deal_year, deal_month);

-- ============================================
-- 4. 정규화된 통합 매물 뷰 (View)
-- ============================================

CREATE VIEW listings_unified AS
SELECT
  'naver_' || article_no AS listing_id,
  'naver' AS source,
  complex_no,
  trade_type,
  price_text AS price_display,
  area_exclusive,
  floor_info,
  direction,
  description,
  tags,
  confirm_date,
  first_seen_at,
  last_seen_at,
  is_active
FROM raw_naver_articles
WHERE is_active = true

UNION ALL

SELECT
  'zigbang_' || item_id AS listing_id,
  'zigbang' AS source,
  NULL AS complex_no,
  CASE sales_type
    WHEN '매매' THEN 'A1'
    WHEN '전세' THEN 'B1'
    WHEN '월세' THEN 'B2'
  END AS trade_type,
  CASE
    WHEN rent > 0 THEN deposit || '/' || rent
    ELSE deposit::TEXT
  END AS price_display,
  size_m2 AS area_exclusive,
  floor AS floor_info,
  NULL AS direction,
  title AS description,
  '{}' AS tags,
  NULL AS confirm_date,
  first_seen_at,
  last_seen_at,
  is_active
FROM raw_zigbang_items
WHERE is_active = true;

-- ============================================
-- 5. 알림 관련
-- ============================================

CREATE TABLE notification_queue (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES user_profiles(id) ON DELETE CASCADE,
  condition_id UUID REFERENCES user_conditions(id),
  listing_id TEXT NOT NULL,           -- listings_unified의 listing_id
  source TEXT NOT NULL,               -- naver, zigbang, molit
  notify_channel TEXT NOT NULL,       -- email, kakao, push
  title TEXT,
  body TEXT,
  metadata JSONB,                     -- 매물 요약 정보
  status TEXT DEFAULT 'pending',      -- pending, sent, failed
  sent_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_notif_status ON notification_queue(status) WHERE status = 'pending';
CREATE INDEX idx_notif_user ON notification_queue(user_id);

-- ============================================
-- 6. 크롤링 로그
-- ============================================

CREATE TABLE crawl_logs (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  source TEXT NOT NULL,
  function_name TEXT,
  status TEXT NOT NULL,
  message TEXT,
  items_count INTEGER DEFAULT 0,
  duration_ms INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_crawl_source_status ON crawl_logs(source, status, created_at);

-- ============================================
-- 7. RLS (Row Level Security) 정책
-- ============================================

ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_conditions ENABLE ROW LEVEL SECURITY;
ALTER TABLE watched_complexes ENABLE ROW LEVEL SECURITY;
ALTER TABLE watched_regions ENABLE ROW LEVEL SECURITY;
ALTER TABLE notification_queue ENABLE ROW LEVEL SECURITY;

-- 사용자는 자신의 데이터만 접근
CREATE POLICY "Users can view own profile"
  ON user_profiles FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
  ON user_profiles FOR UPDATE
  USING (auth.uid() = id);

CREATE POLICY "Users can manage own conditions"
  ON user_conditions FOR ALL
  USING (auth.uid() = user_id);

CREATE POLICY "Users can manage own watched complexes"
  ON watched_complexes FOR ALL
  USING (auth.uid() = user_id);

CREATE POLICY "Users can manage own watched regions"
  ON watched_regions FOR ALL
  USING (auth.uid() = user_id);

CREATE POLICY "Users can view own notifications"
  ON notification_queue FOR SELECT
  USING (auth.uid() = user_id);

-- 매물 데이터는 모든 인증된 사용자가 읽기 가능
ALTER TABLE raw_naver_articles ENABLE ROW LEVEL SECURITY;
ALTER TABLE raw_zigbang_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE raw_molit_trades ENABLE ROW LEVEL SECURITY;
ALTER TABLE complexes ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Authenticated users can read listings"
  ON raw_naver_articles FOR SELECT
  USING (auth.role() = 'authenticated');

CREATE POLICY "Authenticated users can read zigbang"
  ON raw_zigbang_items FOR SELECT
  USING (auth.role() = 'authenticated');

CREATE POLICY "Authenticated users can read trades"
  ON raw_molit_trades FOR SELECT
  USING (auth.role() = 'authenticated');

CREATE POLICY "Authenticated users can read complexes"
  ON complexes FOR SELECT
  USING (auth.role() = 'authenticated');
```

---

## 6. 데이터 수집 전략 요약

### 6.1 3-Layer 데이터 전략

| Layer | 소스 | 데이터 | 합법성 | 안정성 | 역할 |
|-------|------|--------|--------|--------|------|
| Layer 1 (Foundation) | 국토부 실거래가 API | 과거 거래 데이터 | 완전 합법 | 매우 높음 | 시세 기준선, 가격 분석 |
| Layer 2 (Primary) | 네이버부동산 비공식 API | 현재 매물 (아파트) | 개인용 리스크 낮음 | 중간 (차단 가능) | 핵심 매물 알림 |
| Layer 3 (Supplement) | 직방 비공식 API | 현재 매물 (원룸/빌라/오피스텔) | 개인용 리스크 낮음 | 중간 | 보조 매물 소스 |

### 6.2 법적 리스크 평가 (비상업 개인 프로젝트 기준)

| 행위 | 리스크 수준 | 근거 |
|------|-----------|------|
| 국토부 공공 API 사용 | 없음 | 공공데이터 개방 정책 |
| 네이버부동산 비공식 API 소량 호출 (개인용) | 낮음 | 개인 학습/연구 목적, 비상업, 소량 |
| 직방 비공식 API 소량 호출 (개인용) | 낮음 | 동일 |
| 수집 데이터 재배포/공개 | 중간~높음 | 데이터베이스 저작권 침해 가능성 |
| 수집 데이터 상업적 활용 | 높음 | 2024 판례 참조 |

**리스크 완화 조치:**
1. 소량, 저빈도 호출 (aggressive 크롤링 금지)
2. 수집 데이터 외부 공개/재배포 금지
3. robots.txt 존중 (가능한 범위에서)
4. 서비스 약관 변경 시 즉시 대응할 준비
5. 언제든 크롤링을 중단하고 공공 API만으로 전환할 수 있는 구조 유지

### 6.3 Fallback 전략

네이버/직방 API가 차단되거나 사용 불가 시:

```
[Primary] 네이버 + 직방 매물 크롤링
     |
     | (차단 시)
     v
[Fallback 1] 국토부 실거래가 Only 모드
     - 실거래가 기반 가격 인텔리전스 알림
     - 호갱노노형 서비스로 자동 전환
     |
     | (장기 차단 시)
     v
[Fallback 2] 수동 매물 등록 + 공공 데이터
     - 사용자가 직접 관심 매물 URL 등록
     - 시스템이 해당 매물 페이지만 개별 조회
```

---

## 7. 구현 우선순위 (Phased Rollout)

### Phase 1 (Week 1-2): 기반 구축

1. Supabase 프로젝트 생성 및 DB 스키마 마이그레이션
2. 공공데이터포털 API 키 발급 (국토부 실거래가 12종)
3. 국토부 실거래가 수집 Edge Function 구현 및 테스트
4. 관심 단지/지역 등록 UI (최소 기능)

### Phase 2 (Week 3-4): 네이버 크롤링

1. 네이버부동산 매물 수집 Edge Function 구현
2. Rate limit 테스트 및 최적 간격 튜닝
3. 신규 매물 감지 로직 구현 (first_seen_at vs last_seen_at)
4. 조건 매칭 엔진 구현

### Phase 3 (Week 5-6): 알림 및 통합

1. 알림 발송 시스템 구현 (이메일 우선)
2. 직방 매물 수집 추가
3. 통합 매물 뷰 및 프론트엔드 매물 목록 화면
4. 실거래가 vs 호가 비교 분석 기능

### Phase 4 (Week 7-8): 고도화

1. 가격 변동 추적 및 알림
2. 매물 삭제/변경 감지
3. 급매 신호 탐지 로직
4. 대시보드 (관심 단지별 매물 현황, 시세 추이)

---

## Sources

### 네이버부동산 비공식 API
- [네이버 부동산 크롤러 토이프로젝트](https://velog.io/@dev-lop/%ED%86%A0%EC%9D%B4%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EB%84%A4%EC%9D%B4%EB%B2%84-%EB%B6%80%EB%8F%99%EC%82%B0-%ED%81%AC%EB%A1%A4%EB%9F%AC)
- [네이버 부동산 크롤링 - inasie](https://inasie.github.io/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/%EB%84%A4%EC%9D%B4%EB%B2%84-%EB%B6%80%EB%8F%99%EC%82%B0-%ED%81%AC%EB%A1%A4%EB%A7%81/)
- [네이버 부동산 아파트 데이터 가져오기](https://leesunkyu94.github.io/data%20%EB%A7%8C%EB%93%A4%EA%B8%B0/naver-real-estate/)
- [네이버 부동산 크롤링 - FinanceData](https://financedata.github.io/posts/naver-land-crawling.html)
- [네이버 부동산 매물 데이터 수집 - WikiDocs](https://wikidocs.net/267232)

### 직방 비공식 API
- [직방 매물정보 크롤링 - GitHub Gist](https://gist.github.com/9oelM/775e1293ae4714f6b059ffe5560a2ef6)
- [직방에서 서울 아파트 정보 크롤링하기](https://velog.io/@byungjur_96/%EC%A7%81%EB%B0%A9%EC%97%90%EC%84%9C-%EC%84%9C%EC%9A%B8-%EC%95%84%ED%8C%8C%ED%8A%B8-%EC%A0%95%EB%B3%B4-%ED%81%AC%EB%A1%A4%EB%A7%81%ED%95%98%EA%B8%B0-1)
- [직방 크롤링 - gurumii](https://gurumii.com/archive/zigbang-crawling)

### 국토부 공공 API
- [국토교통부 아파트 매매 실거래가 자료 API - 공공데이터포털](https://www.data.go.kr/data/15126469/openapi.do)
- [아파트 매매 실거래가 상세 - Data Doctor Blog](https://datadoctorblog.com/2025/03/17/Py-Crawling-API-gov-APT-trade/)
- [PublicDataReader 실거래가 사용 가이드](https://github.com/WooilJeong/PublicDataReader/blob/main/assets/docs/portal/TransactionPrice.md)
- [공공데이터포털 이용하기 - inasie](https://inasie.github.io/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/%EA%B3%B5%EA%B3%B5%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%9D%B4%EC%9A%A9%ED%95%98%EA%B8%B0-1/)

### 법적 판례
- [네이버 부동산 매물 크롤링 손해배상 판결 - 전자신문](https://www.etnews.com/20241007000224)
