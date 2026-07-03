# 작업로그 · 2026-07-03 — 서버 장바구니(§3.3~3.5) + 주문 리팩터

> 회원별 서버 장바구니를 추가하고, 주문 생성이 서버 장바구니에서 항목을 읽도록 문서 원래 설계대로 리팩터. (PR #17)

## 1. 워크플로우
1. [x] 서버 장바구니 API(§3.3~3.5) + 주문 리팩터(장바구니에서 읽기·비우기)
2. [x] 서버 테스트 갱신(cart/order) + 빌드검증(ASCII,H2) + MySQL 실측
3. [x] 앱 주문 흐름 동기화 + analyze/test
4. [x] 커밋/PR(#17) → dev 머지 + 문서

## 2. 리포트 (PR #17)

### 서버 — 장바구니 API (05_API §3.3~3.5)
- `cart_item` 엔티티/리포지토리/서비스/컨트롤러(GET/POST/PATCH/DELETE, 전체 비우기). 모두 로그인 필요.
- optionIds는 콤마 문자열로 저장, 금액은 조회 때마다 메뉴 DB로 재계산.

### 서버 — 주문 리팩터 (§4.1~4.2)
- 주문 생성이 요청 items 대신 **서버 장바구니**에서 읽음. 빈 장바구니 400(EMPTY_CART).
- 주문 성공 시 장바구니 비움. 배달조건·delivery-check도 장바구니 기준 서버 재검증.

### 앱
- `OrderApi.createOrder`: 주문 전에 화면 장바구니를 서버로 동기화(비우기→담기) 후 주문.

## 3. 검증
- 서버 gradlew build(H2) 성공 — 테스트 18건(cart 4 + order 5 + auth 5 + menu 3 + contextLoads).
- 앱 flutter analyze 통과 + test 4건.
- MySQL 실측: 담기 → GET /api/cart(25,800) → POST /api/orders(본문 항목 없이) → 주문 20260703-0005(25,800) → 장바구니 자동 비움. cart_item 테이블 자동 생성.

## 4. 다음 단계
결제 연동(§5, PortOne), 소셜 로그인(§2.3), 쿠폰·적립(§10).

> 환경 메모: Windows-MCP가 계속 불안정해 Desktop Commander로 진행. gradlew를 대화형 셸에서 돌리면 셸이 종료되어, 출력을 로그 파일로 받아 read_file로 확인. clean은 실행 중 서버 jar가 build/를 잠가 실패 → 새 폴더로 복사해 빌드.
