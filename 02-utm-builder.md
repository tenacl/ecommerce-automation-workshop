# 2. UTM 빌더 및 관리
> UTM 파라미터의 개념부터 실전 적용, 데이터 분석까지 한 번에 다룹니다.

---

## 2-1. UTM이란?

### 한 줄 정의
> **UTM = 링크에 붙이는 추적 꼬리표.** "이 고객이 어디서 왔는지" 알려주는 태그.

### 왜 필요한가?

스마트스토어에 하루 100명이 들어왔다고 합시다.
- 인스타 광고에서 왔나?
- 블로그 포스팅에서 왔나?
- 카카오톡 메시지에서 왔나?

UTM 없이는 **알 수 없습니다.** UTM을 붙이면 정확히 알 수 있습니다.

### UTM 파라미터 5가지

| 파라미터 | 뜻 | 예시 | 필수 |
|----------|-----|------|------|
| `utm_source` | 어디서 왔나 (출처) | naver, instagram, kakao | ✅ |
| `utm_medium` | 어떤 방식으로 (매체) | cpc, social, email | ✅ |
| `utm_campaign` | 어떤 캠페인 | summer_sale, new_product | ✅ |
| `utm_term` | 어떤 검색어 (유료 검색) | 여성운동화, 에어팟케이스 | 선택 |
| `utm_content` | 어떤 소재 (A/B 테스트용) | banner_a, text_link | 선택 |

### UTM 적용 전 vs 후

**Before (UTM 없이):**
```
https://smartstore.naver.com/myshop/products/12345
```
→ 분석: "누군가 들어왔다" (끝)

**After (UTM 포함):**
```
https://smartstore.naver.com/myshop/products/12345?utm_source=instagram&utm_medium=social&utm_campaign=summer_sale&utm_content=story_ad
```
→ 분석: "인스타그램 스토리 광고를 통해 여름 세일 캠페인으로 들어왔다" ✅

---

## 2-2. UTM을 적용하는 방법: 타입봇

### 타입봇(Typebot)이란?
> 노코드 챗봇 빌더. UTM 파라미터를 자동으로 수집해서 스프레드시트에 저장해줍니다.

### 왜 타입봇?
- 무료 플랜으로 충분
- 스프레드시트 연동 쉬움
- UTM 파라미터를 자동으로 읽어옴
- 고객에게 추가 정보(이름, 연락처 등)도 받을 수 있음

### 타입봇 UTM 수집 설정 방법

```
1. typebot.io 가입 → 새 봇 만들기

2. 시작 블록 설정:
   - "URL에서 변수 가져오기" 기능 사용
   - utm_source → 변수 {{source}}
   - utm_medium → 변수 {{medium}}
   - utm_campaign → 변수 {{campaign}}

3. 챗봇 대화 흐름 설계:
   - "안녕하세요! 어떤 상품에 관심 있으신가요?"
   - 고객 응답 → 변수 {{interest}} 에 저장
   - "연락처를 남겨주시면 할인 쿠폰을 보내드려요!"
   - 고객 응답 → 변수 {{phone}} 에 저장

4. Google Sheets 연동:
   - 결과 블록 → Google Sheets 선택
   - 시트에 기록할 컬럼: source, medium, campaign, interest, phone, 시간
```

### UTM이 달린 링크로 타입봇 호출

```
https://typebot.io/my-bot?utm_source=instagram&utm_medium=story&utm_campaign=summer_sale
```

→ 고객이 챗봇에 응답하면, 스프레드시트에 이렇게 기록됩니다:

| 시간 | source | medium | campaign | interest | phone |
|------|--------|--------|----------|----------|-------|
| 2026-03-31 14:00 | instagram | story | summer_sale | 원피스 | 010-xxxx |

---

## 2-3. UTM을 적용해서 실제 데이터 받아보기 + CRM 자동화

### 실습: UTM 링크 만들기

직접 UTM 링크를 만들어봅시다.

**내 스마트스토어 URL:** `https://smartstore.naver.com/myshop`

| 채널 | 완성 UTM 링크 |
|------|---------------|
| 인스타 피드 | `?utm_source=instagram&utm_medium=feed&utm_campaign=spring_new` |
| 카카오톡 단체 | `?utm_source=kakao&utm_medium=message&utm_campaign=spring_new` |
| 블로그 포스팅 | `?utm_source=blog&utm_medium=post&utm_campaign=spring_new` |
| 네이버 검색광고 | `?utm_source=naver&utm_medium=cpc&utm_campaign=spring_new&utm_term=여성원피스` |

### CRM 자동화 아이디어

타입봇 + 구글 시트 데이터가 쌓이면 자동화할 수 있습니다:

```
[구글 시트에 데이터 쌓임]
        ↓
[자동화 트리거 — Make.com 또는 Zapier]
        ↓
┌─────────────────────────────────┐
│ source가 "instagram"이면        │ → 인스타 전용 쿠폰 발송
│ campaign이 "summer_sale"이면    │ → 여름 상품 추천 메시지
│ interest가 "원피스"이면         │ → 원피스 카테고리 링크 발송
│ 3일 후 미구매 시                │ → 리마인드 메시지 발송
└─────────────────────────────────┘
```

### Make.com(구 Integromat) 자동화 시나리오

```
트리거: Google Sheets — 새 행 추가됨
    ↓
필터: utm_source = "instagram"
    ↓
액션: 카카오 알림톡 발송
    → "인스타에서 보신 상품, 오늘만 10% 추가 할인!"
```

---

## 2-4. 스프레드시트에서 제미나이로 분석/대시보드 만들기

### Google Sheets + Gemini (제미나이) 분석

구글 시트에 UTM 데이터가 쌓였다면, 제미나이를 활용해 분석합니다.

### 실습 1: 제미나이에게 분석 요청

구글 시트 사이드패널에서 Gemini를 열고:

```
이 시트의 데이터를 분석해줘.

1. utm_source별 유입 수 비교 (인스타 vs 카카오 vs 블로그)
2. utm_campaign별 전환율 (관심상품 입력까지 도달한 비율)
3. 시간대별 유입 패턴 (몇 시에 가장 많이 들어오는지)
4. 핵심 인사이트 3가지를 한국어로 요약
```

### 실습 2: 대시보드 차트 만들기

제미나이에게 차트 요청:

```
이 데이터로 대시보드를 만들어줘.

차트 1: utm_source별 유입 수 — 파이 차트
차트 2: 날짜별 유입 추이 — 라인 차트
차트 3: utm_campaign별 평균 전환율 — 바 차트
차트 4: 시간대별 유입 수 — 히트맵
```

### 실습 3: 자동 리포트 생성

```
이 데이터를 바탕으로 주간 마케팅 리포트를 만들어줘.

포함할 내용:
- 이번 주 총 유입 수
- 가장 효과 좋은 채널 (utm_source)
- 가장 효과 좋은 캠페인 (utm_campaign)
- 다음 주 추천 액션
- 한국어로, 대표님께 보고할 수 있는 톤으로
```

### 분석 결과 예시

| 지표 | 결과 |
|------|------|
| 가장 효과 좋은 채널 | 인스타그램 (유입의 45%) |
| 가장 전환 높은 캠페인 | spring_new (전환율 23%) |
| 피크 시간대 | 오후 8~10시 |
| 추천 액션 | 인스타 스토리 광고 예산 증액 |

---

## 수강생이 막히는 포인트

### ❶ "UTM 파라미터가 안 넘어가요"
**원인:** 링크 중간에 `?`가 이미 있는 경우 `&`로 연결해야 함.
**해결:**
- 첫 번째 파라미터: `?utm_source=...`
- 두 번째부터: `&utm_medium=...`
- URL에 이미 `?`가 있으면: `&utm_source=...`로 시작

### ❷ "스프레드시트에 데이터가 안 쌓여요"
**원인:** 타입봇 → 구글 시트 연동 설정 누락.
**해결:** 타입봇 결과 블록에서 Google Sheets 인증이 완료되었는지 확인.

### ❸ "제미나이가 차트를 못 만들어요"
**원인:** 데이터 양이 너무 적거나, 컬럼 헤더가 없음.
**해결:** 최소 10행 이상 + 첫 행에 명확한 헤더 입력.

### ❹ "한국어가 깨져요"
**원인:** UTM 값에 한국어 사용 시 URL 인코딩 문제.
**해결:** UTM 값은 **영어/숫자/언더스코어**만 사용. `여성원피스` → `womens_dress`

---

## 강의 시간 배분 (1시간 기준)

| 시간 | 내용 |
|------|------|
| 0:00~0:10 | UTM 개념 설명 + 실제 사례 |
| 0:10~0:25 | 타입봇으로 UTM 수집 설정 |
| 0:25~0:40 | UTM 링크 직접 만들기 + 테스트 |
| 0:40~0:55 | 구글 시트 + 제미나이 분석/대시보드 |
| 0:55~1:00 | CRM 자동화 아이디어 + Q&A |

---

## 핵심 메시지

> **UTM = 마케팅의 GPS**
>
> "어디서 고객이 오는지" 모르면 마케팅비를 태우는 겁니다.
> UTM 태그 하나로 모든 채널의 효과를 숫자로 증명할 수 있습니다.
>
> 오늘 배운 흐름:
> 1. UTM 링크 만들기 → 채널별 추적
> 2. 타입봇으로 데이터 수집 → 자동 기록
> 3. 제미나이로 분석 → 인사이트 도출
> 4. 자동화 연결 → CRM까지

---

*UTM 마케팅 추적 + 자동화 — 바이브코딩 워크숍*
