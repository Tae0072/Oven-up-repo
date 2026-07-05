# 작업로그 · 결제(PortOne) 실제 연동 (외부 연동 2/3)

- 날짜: 2026-07-05
- 브랜치/PR: `feature/payment-portone` → dev, PR #39 (머지)
- 관련 가이드: `가이드/결제_연동.md`

## 무엇을 했나

모의(mock) 결제만 있던 결제 흐름에 **실제 PortOne V2 결제창**을 연결했다.
서버 검증(RealPaymentVerifier: PortOne 조회 → status=PAID + 금액 일치 확인)은 기존 코드 그대로 사용.

## 결제 흐름 (real 모드)

1. 앱: 주문 생성(결제대기) → 결제 화면에서 수단 선택 → **PortOne 결제창** 열림
2. 결제 완료 → `paymentId` 획득
3. 앱 → 서버 `POST /api/orders/{id}/pay { method, paymentRef=paymentId }`
4. 서버: PortOne API로 해당 결제 조회 → **결제완료 + 금액 일치** 확인 → 주문 '결제완료' + 적립 지급

## 변경 내용 (앱)

- 웹: `web/index.html`에 PortOne browser-sdk 로드, `portone_web_real.dart`(js_interop)로 `PortOne.requestPayment` 호출
- 모바일: 공식 `portone_flutter ^1.0.1`(웹뷰) — `portone_mobile_real.dart`, appScheme `ovenup` 인텐트 필터 추가
- `payment_config.dart`: `PORTONE_STORE_ID`/`PORTONE_CHANNEL_KEY`는 `--dart-define` 주입, 없으면 기존 mock 결제(테스트/CI 안전)
- 결제수단 매핑: CARD → CARD, 카카오페이/네이버페이/토스/삼성페이 → EASY_PAY (실사용 가능 여부는 콘솔 채널 설정에 따름)

## 서버 (변경 없음, 실행 설정만)

- `--app.payment.mock=false` + 환경변수 `PORTONE_API_SECRET` (V2 API Secret)
- ⚠️ API Secret은 "channel-key-..."가 아니라 콘솔 → 결제연동 → **API Keys**에서 발급하는 별도 값

## 실행 스크립트 (레포 밖, 키는 secrets 파일에서 자동 로드)

- 서버: `D:\dev\run-ovenup-server-real.ps1` (social+payment 둘 다 real)
- 웹: `D:\dev\run-ovenup-web.ps1` (http://localhost:8091)
- 태블릿: `D:\dev\run-ovenup-tablet.ps1`

## 검증

- appcheck: flutter analyze 0건, flutter test 6/6 통과
- CI: secret-scan 통과 (키 커밋 없음)
- E2E:
  - [x] 태블릿에서 결제창(카카오페이 테스트) 열림 확인
  - [x] 서버 검증 파이프라인 확인 (API 테스트): 주문 생성 → 가짜 paymentRef로 /pay 호출
        → 서버가 PortOne 조회(인증 성공) → PAYMENT_NOT_FOUND → **PAYMENT_FAILED로 거부**, 주문은 '결제대기' 유지 ✅
  - [ ] 실제 카카오페이 테스트 결제 완료까지: 카카오페이가 "주 사용 기기"에서만 승인돼 태블릿에선 불가.
        PC 웹(localhost:8091)에서 결제하면 QR을 본인 폰으로 찍는 방식이라 가능할 것으로 보임.

## 트러블슈팅 기록 (키)

- `PORTONE_API_SECRET`에 채널 키가 잘못 들어가 있었음. 진짜 Secret은 접두어 없는 긴 랜덤 문자열.
  → PortOne `POST /login/api-secret`으로 유효성 직접 확인 후 교정.
- 결제수단을 뭘 눌러도 카카오페이로 열림 = 정상. 채널 키가 결제사를 결정하는데 등록된 테스트 채널이
  카카오페이 하나뿐. 카드 등은 콘솔에서 채널 추가 후 수단별 채널 키 매핑 필요(추후).

## 남은 것 / 주의

- PortOne **테스트 모드**에서만 검증할 것 (실 결제 전환은 사용자가 직접 결정)
- 다음 단계: 외부 연동 3/3 푸시(FCM) — `가이드/푸시_알림_FCM.md`
