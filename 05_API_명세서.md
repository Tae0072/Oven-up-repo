# 05. API 명세서

> 화면(프론트, Flutter)과 서버(백엔드, Spring Boot)가 주고받는 규칙 정의. 이 문서대로 서버 API를 만들고, 화면에서 호출한다.
> 버전: v0.1 · 작성일: 2026-07-02

---

## 0. API가 뭐예요? (아주 쉽게)

- **API** = 화면이 서버에게 "이거 해줘"라고 부탁하는 **약속된 창구**.
- 예: 화면이 `GET /api/menus` 를 부르면 → 서버가 메뉴 목록을 돌려준다.
- **요청(request)**: 화면 → 서버 (무엇을 해달라). **응답(response)**: 서버 → 화면 (결과).
- 아래 표기법: `방식 경로` (예: `POST /api/auth/login`).
  - GET=조회, POST=생성/실행, PUT/PATCH=수정, DELETE=삭제.

---

## 1. 공통 규칙

### 기본 주소 (Base URL)
- 개발: `http://localhost:8080`
- 실제: (배포 후 정해질 도메인)

### 인증 방식
- 로그인하면 서버가 **토큰(JWT)** 을 준다. 이후 요청엔 헤더에 토큰을 넣는다.
- 헤더: `Authorization: Bearer {토큰}`
- 로그인 필요 API는 이 헤더가 없으면 `401 Unauthorized`.

### 요청/응답 형식
- 데이터 형식은 **JSON**.
- 요청 헤더: `Content-Type: application/json`

### 공통 응답 형태
성공:
```json
{ "success": true, "data": { ... } }
```
실패:
```json
{ "success": false, "error": { "code": "INVALID_INPUT", "message": "이메일 형식이 올바르지 않습니다." } }
```

### 자주 쓰는 상태코드
| 코드 | 뜻 |
| --- | --- |
| 200 | 성공 |
| 201 | 생성 성공 |
| 400 | 잘못된 요청(입력값 오류) |
| 401 | 로그인 필요/토큰 없음 |
| 403 | 권한 없음(예: 손님이 관리자 API 호출) |
| 404 | 없는 데이터 |
| 409 | 충돌(예: 이메일 중복) |
| 500 | 서버 오류 |

---

## 2. 인증 (회원가입 / 로그인 / 소셜)

### 2.1 회원가입
`POST /api/auth/signup`
```json
// 요청
{ "email": "test@oven.com", "password": "12345678", "name": "홍길동", "phone": "010-1234-5678" }
```
```json
// 응답 201
{ "success": true, "data": { "userId": 1 } }
```
- 실패: 이메일 중복 → 409, 비밀번호 8자 미만 → 400.

### 2.2 로그인
`POST /api/auth/login`
```json
// 요청
{ "email": "test@oven.com", "password": "12345678" }
```
```json
// 응답 200
{ "success": true, "data": { "accessToken": "eyJhbGci...", "user": { "id": 1, "name": "홍길동", "role": "USER" } } }
```

### 2.3 소셜 로그인 (카카오 / 네이버)
`POST /api/auth/social/{provider}`  · `{provider}` = `kakao` 또는 `naver`
```json
// 요청 (앱이 카카오/네이버에서 받은 액세스 토큰을 전달)
{ "accessToken": "카카오/네이버가 준 토큰" }
```
```json
// 응답 200 (우리 서비스 토큰 발급, 첫 로그인이면 자동 회원가입)
{ "success": true, "data": { "accessToken": "eyJhbGci...", "user": { "id": 5, "name": "카카오사용자", "role": "USER" }, "isNew": true } }
```
> 서버는 받은 토큰으로 카카오/네이버에 "이 사람 누구야?"를 물어 확인한 뒤, 우리 회원과 연결한다.

### 2.4 내 정보
`GET /api/users/me`  (로그인 필요)
```json
// 응답 200
{ "success": true, "data": { "id": 1, "email": "test@oven.com", "name": "홍길동", "phone": "010-1234-5678", "role": "USER", "pointBalance": 1500 } }
```

### 2.5 로그아웃
- 서버 저장 없이 **앱에서 토큰을 지우면** 로그아웃. (별도 API 불필요)

---

## 3. 메뉴

### 3.1 메뉴 목록
`GET /api/menus?category={카테고리}`  · `category` 생략 시 전체
```json
// 응답 200
{ "success": true, "data": [
  { "id": 1, "name": "아메리카노", "price": 4000, "category": "커피", "imageUrl": "...", "status": "ON_SALE" },
  { "id": 10, "name": "햄치즈 샌드위치", "price": 6500, "category": "샌드위치", "imageUrl": "...", "status": "ON_SALE" }
] }
```

### 3.2 메뉴 상세
`GET /api/menus/{id}`
```json
// 응답 200
{ "success": true, "data": {
  "id": 1, "name": "아메리카노", "description": "...", "price": 4000, "category": "커피", "imageUrl": "...", "status": "ON_SALE",
  "options": [ { "id": 100, "name": "샷 추가", "extraPrice": 500 }, { "id": 101, "name": "사이즈업", "extraPrice": 500 } ]
} }
```

### 3.3 장바구니 조회 (서버 저장)
`GET /api/cart`  (로그인 필요)
```json
// 응답 200
{ "success": true, "data": {
  "items": [
    { "cartItemId": 1, "menuId": 10, "menuName": "햄치즈 샌드위치", "quantity": 2, "optionIds": [], "optionsDesc": "", "lineprice": 13000 }
  ],
  "totalPrice": 13000
} }
```

### 3.4 장바구니 담기
`POST /api/cart/items`  (로그인 필요)
```json
// 요청
{ "menuId": 10, "quantity": 2, "optionIds": [] }
```
```json
// 응답 201
{ "success": true, "data": { "cartItemId": 1 } }
```

### 3.5 장바구니 수량 변경 / 삭제 / 비우기
- `PATCH /api/cart/items/{cartItemId}` — 수량 변경 `{ "quantity": 3 }`
- `DELETE /api/cart/items/{cartItemId}` — 항목 삭제
- `DELETE /api/cart` — 장바구니 전체 비우기

> **장바구니는 서버에 저장**된다(회원별). 웹에서 담고 갤럭시탭에서 열어도 그대로 유지된다. 주문(§4)이 완료되면 서버가 장바구니를 비운다.

---

## 4. 주문 (배달/픽업/예약 포함)

### 4.1 배달 가능 여부 확인 🆕
`POST /api/orders/delivery-check`  (로그인 필요)
```json
// 요청 (배달지만 전달 — 항목은 서버가 현재 장바구니에서 읽음)
{ "address": "명지에코펠리스 305호" }
```
```json
// 응답 200 (가능)
{ "success": true, "data": { "deliverable": true } }
```
```json
// 응답 200 (불가)
{ "success": true, "data": { "deliverable": false, "reason": "직배송은 샌드위치 2개 이상부터 가능해요. 픽업으로 주문해 주세요." } }
```
**서버 판정 규칙**
1. 주소가 **명지에코펠리스 건물**인가? (아니면 불가 — 픽업만 가능)
2. 항목 중 `category="샌드위치"` 수량 합계 **≥ 2** 인가?
- 둘 다 만족해야 배달 가능.

### 4.2 주문 생성
`POST /api/orders`  (로그인 필요)
```json
// 요청 (주문 항목은 보내지 않는다 — 서버가 현재 장바구니 내용으로 주문 생성)
{
  "fulfillmentType": "DELIVERY",        // DINE_IN | TAKEOUT | DELIVERY
  "scheduledAt": "2026-07-05T14:30:00",  // 예약주문이면 시간, 지금 주문이면 null
  "deliveryAddress": "명지에코펠리스 305호", // 배달일 때만
  "requestMsg": "덜 맵게 해주세요",
  "couponId": 3,                          // 없으면 null
  "usePoint": 500                         // 사용할 적립금, 없으면 0
}
```
```json
// 응답 201 (아직 결제 전 상태)
{ "success": true, "data": { "orderId": 55, "orderNo": "20260705-0055", "totalPrice": 13500, "status": "결제대기" } }
```
**서버 처리 규칙 (중요)**
- 주문 항목은 **서버의 장바구니(cart_item)에서 가져온다**. 장바구니가 비어 있으면 400.
- 금액은 **서버가 다시 계산**(메뉴가·옵션가·쿠폰·적립). 화면이 보낸 금액 신뢰 안 함.
- 배달이면 **§4.1 조건을 서버가 재검증**. 실패 시 400.
- 주문 시점의 이름·가격을 항목에 함께 저장하고, **주문 성공 시 장바구니를 비운다**.

### 4.3 내 주문 목록
`GET /api/orders`  (로그인 필요)
```json
// 응답 200
{ "success": true, "data": [
  { "orderId": 55, "orderNo": "20260705-0055", "totalPrice": 13500, "fulfillmentType": "DELIVERY", "scheduledAt": "2026-07-05T14:30:00", "status": "준비중", "createdAt": "2026-07-02T10:00:00" }
] }
```

### 4.4 주문 상세
`GET /api/orders/{id}`  (로그인 필요, 본인 주문만)
```json
// 응답 200
{ "success": true, "data": {
  "orderId": 55, "orderNo": "20260705-0055", "status": "준비중",
  "fulfillmentType": "DELIVERY", "scheduledAt": "2026-07-05T14:30:00", "deliveryAddress": "명지에코펠리스 305호",
  "totalPrice": 13500, "discountPrice": 500,
  "items": [ { "menuName": "햄치즈 샌드위치", "unitPrice": 6500, "quantity": 2, "optionsDesc": "" } ]
} }
```

> **예약 주문**은 별도 API가 아니라, 위 주문 생성에서 `scheduledAt` 을 채우면 된다.

---

## 5. 결제 (PortOne 연동)

결제 흐름: **① 서버에 결제 준비 → ② 앱에서 PortOne 결제창 → ③ 서버가 결제 검증**

### 5.1 결제 준비
`POST /api/payments/prepare`  (로그인 필요)
```json
// 요청
{ "orderId": 55 }
```
```json
// 응답 200 (PortOne 결제창에 넘길 정보)
{ "success": true, "data": { "merchantUid": "order_55_abc", "amount": 13500, "orderName": "햄치즈 샌드위치 외 1건" } }
```

### 5.2 결제 검증
`POST /api/payments/verify`  (로그인 필요)
```json
// 요청 (앱이 PortOne 결제 성공 후 받은 값 전달)
{ "impUid": "imp_123456", "merchantUid": "order_55_abc" }
```
```json
// 응답 200
{ "success": true, "data": { "orderId": 55, "status": "결제완료", "paidAmount": 13500 } }
```
**서버 처리 (중요)**
- 서버가 PortOne에 `impUid`로 조회해 **실제 결제금액 == 주문금액**인지 확인.
- 일치하면 주문 `결제완료`, **적립 지급 + 쿠폰 사용 처리**. 불일치면 결제 취소/실패 처리.

### 5.3 결제 웹훅 (선택)
`POST /api/payments/webhook`  — PortOne 서버가 결제 결과를 우리 서버로 통지. (검증 이중 안전장치)

**지원 결제수단**: 카드 / 카카오페이 / 네이버페이 / 토스페이 / 삼성페이 (PortOne 결제창에서 선택)

---

## 6. 단체 주문

### 6.1 단체주문 문의 등록
`POST /api/group-orders`  (로그인 필요)
```json
{ "desiredAt": "2026-07-10T12:00:00", "headcount": 20, "detail": "아메리카노 20잔 정도", "contact": "010-1234-5678" }
```
```json
// 응답 201
{ "success": true, "data": { "groupOrderId": 7, "status": "접수" } }
```

### 6.2 내 단체주문 목록
`GET /api/group-orders`  → 상태와 사장님 답변(adminMemo) 포함.

---

## 7. 고객의 소리

### 7.1 문의 등록
`POST /api/inquiries`  (로그인 필요)
```json
{ "title": "매장 좌석 문의", "content": "콘센트 자리 있나요?", "imageUrl": null }
```
```json
// 응답 201
{ "success": true, "data": { "inquiryId": 12, "status": "접수" } }
```

### 7.2 내 문의 목록 / 상세
- `GET /api/inquiries` — 내 문의 목록
- `GET /api/inquiries/{id}` — 상세 + 사장님 답변
```json
// 상세 응답 200
{ "success": true, "data": {
  "inquiryId": 12, "title": "매장 좌석 문의", "content": "...", "status": "답변완료",
  "reply": { "content": "네, 창가에 있어요!", "createdAt": "2026-07-02T12:00:00" }
} }
```

---

## 8. 쿠폰 & 적립

- `GET /api/me/coupons` — 내 보유 쿠폰(사용 가능/사용됨)
```json
{ "success": true, "data": [ { "userCouponId": 3, "name": "신규가입 2,000원 할인", "discountType": "AMOUNT", "discountValue": 2000, "minOrderPrice": 5000, "validUntil": "2026-08-01", "isUsed": false } ] }
```
- `GET /api/me/points` — 적립금 잔액 + 내역
```json
{ "success": true, "data": { "balance": 1500, "history": [ { "changeAmount": 135, "reason": "주문적립", "createdAt": "2026-07-02T10:05:00" } ] } }
```

---

## 9. 관리자(사장님) API

> 모두 `Authorization` 헤더의 회원 `role` 이 **ADMIN** 인지 서버가 확인. 아니면 `403`.
> 경로 앞에 `/api/admin` 을 붙인다.

### 9.1 메뉴 관리
- `POST /api/admin/menus` — 메뉴 등록
- `PUT /api/admin/menus/{id}` — 메뉴 수정
- `PATCH /api/admin/menus/{id}/status` — 판매/품절 전환 `{ "status": "SOLD_OUT" }`
- `DELETE /api/admin/menus/{id}` — 삭제
```json
// 등록 요청 예시
{ "name": "BLT 샌드위치", "price": 7000, "category": "샌드위치", "description": "...", "imageUrl": "...",
  "options": [ { "name": "베이컨 추가", "extraPrice": 1000 } ] }
```

### 9.2 주문 관리
- `GET /api/admin/orders?status={상태}` — 주문 목록(상태/날짜 필터)
- `PATCH /api/admin/orders/{id}/status` — 상태 변경
```json
// 요청
{ "status": "준비완료" }   // 접수됨→준비중→준비완료→픽업완료 / 배달중→배달완료
```
- 상태 변경 시 손님에게 **푸시 알림** 발송.

### 9.3 예약주문 관리
- `GET /api/admin/orders?scheduled=true&date={날짜}` — 예약 주문을 시간대별로 조회. (주문 API 재사용)

### 9.4 단체주문 관리
- `GET /api/admin/group-orders` — 목록
- `PATCH /api/admin/group-orders/{id}` — 답변/상태 `{ "status": "확정", "adminMemo": "견적 안내드렸습니다" }`

### 9.5 고객의 소리 관리
- `GET /api/admin/inquiries` — 목록
- `POST /api/admin/inquiries/{id}/reply` — 답변 등록 `{ "content": "답변 내용" }`

### 9.6 쿠폰 관리
- `POST /api/admin/coupons` — 쿠폰 생성
```json
{ "name": "여름 이벤트 10% 할인", "discountType": "RATE", "discountValue": 10, "minOrderPrice": 10000, "validUntil": "2026-08-31" }
```

---

## 10. 개발 순서와의 연결

이 API는 로드맵(01번) 단계에 맞춰 조금씩 만든다:
1. **메뉴 API**(§3) → 화면에 진짜 메뉴 표시
2. **회원 API**(§2.1~2.2, 2.4) → 로그인
3. **주문 API**(§4) → 주문 저장·조회
4. **결제 API**(§5) + 소셜 로그인(§2.3)
5. **단체/문의/쿠폰·적립**(§6~8)
6. **관리자 API**(§9)

> 처음부터 전부 만들 필요 없다. 화면이 필요로 하는 API부터 하나씩 붙인다.
