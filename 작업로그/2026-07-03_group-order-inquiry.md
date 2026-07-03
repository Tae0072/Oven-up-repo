# 작업로그 · 2026-07-03 — 단체 주문 문의 · 고객의 소리

> 단체 주문(협의형)과 고객의 소리(문의·답변) 기능. 서버 도메인/API + 앱 화면 + 목업 데이터. (PR #20)

## 1. 워크플로우
1. [x] 서버 단체주문(§6) + 고객의소리(§7) 도메인/API + 테스트
2. [x] 서버 빌드검증(ASCII,H2) + MySQL 실측
3. [x] 앱 단체주문(목록·작성) + 고객의소리(목록·작성·상세) + 진입점 + analyze/test
4. [x] 커밋/PR(#20) → dev 머지 + 문서(작업로그)

## 2. 리포트 (PR #20)

### 서버 (05_API §6, §7)
- **단체 주문(grouporder)** — 협의형(즉시 결제 없음). 상태: 접수/협의중/확정/취소.
  - `POST /api/group-orders` {desiredAt, headcount, detail, contact} → 201 {groupOrderId, status:"접수"}
  - `GET /api/group-orders` — 내 문의 목록(사장님 답변 `adminMemo` 포함)
- **고객의 소리(inquiry)** — 본인 글만 조회.
  - `POST /api/inquiries` {title, content, imageUrl} → 201 {inquiryId, status:"접수"}
  - `GET /api/inquiries` — 내 목록 / `GET /api/inquiries/{id}` — 상세(사장님 답변 `reply` 포함)
  - 타인 문의 접근 시 **404**(존재 여부 노출 방지). 답변은 `inquiry_reply` 테이블.
- 모든 엔드포인트 로그인 필요(AuthTokenFilter). 입력 검증(인원수>0, 제목·내용 필수)은 서버에서.
- **목업 데이터**(DemoDataInitializer): 단체주문 1건, 고객의소리 2건(1건 답변완료+답변).

### 앱
- 모델(GroupOrder, InquirySummary/Detail/Reply) + API(GroupOrderApi, InquiryApi).
- 화면: 단체주문 목록/작성(날짜·시간 선택), 고객의 소리 목록/작성/상세(답변 카드).
- 메뉴 화면 계정 메뉴에 진입점 추가: **단체 주문 문의**, **고객의 소리**.

## 3. 검증
- 서버 gradlew build(H2) 성공 — 테스트 8개 클래스(grouporder 3 + inquiry 4 + cart 4 + order 5 + auth 5+4 + menu 3 + contextLoads).
- 앱 flutter analyze 통과(No issues) + test 4건.
- MySQL 실측: 단체주문 접수/목록, 고객의소리 작성/목록/상세(답변 포함) — **한글 정상 저장·조회**(HEX 확인: status=`ECA091EC8898`="접수", 클라이언트 전송 한글 왕복 일치).

## 4. 참고 — 한글 검증 시 주의
로컬 검증에서 PowerShell(5.1)이 UTF-8 `.ps1`을 CP949로 읽어 스크립트 안의 한글 문자열을 깨뜨린다. 요청 본문 한글은 스크립트 리터럴 대신 파일(write)로 만들어 `curl --data-binary @파일`로 보내야 정확히 검증된다. 서버·DB(utf8mb4)·JDBC는 정상.

## 5. 다음 단계
관리자(사장님) 화면 — 단체주문 상태 변경·답변, 고객의 소리 답변 등록(§11). 이후 결제(§5), 쿠폰·적립(§10), 푸시(§9).
