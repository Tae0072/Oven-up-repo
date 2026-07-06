# 작업로그 · 푸시 알림(FCM) 실제 연동 완료 (외부 연동 3/3) 🎉

- 날짜: 2026-07-06
- 브랜치/PR: `feature/push-fcm` → dev, PR #43 (머지) + 후속 PR #44 (결제 구매자 이름)
- **E2E 확인: 태블릿에서 실제 OS 푸시 수신 성공** ✅ → **외부 연동 3종 전체 완료**

## 무엇을 했나

관리자가 주문 상태를 바꾸면(또는 결제 완료 시) 손님 기기에 **실제 OS 푸시**가 가도록 FCM(HTTP v1)을 연동했다.

## 변경 내용

### 서버 (PR #43)
- 신규 `device_token` 테이블 + `POST /api/notifications/device-token` (같은 토큰 재등록 시 주인 갱신)
- `RealPushSender` 구현: 서비스 계정 키로 구글 OAuth 토큰 발급 → FCM v1 `messages:send`
  - 손님 기기 토큰마다 발송, UNREGISTERED(앱 삭제 등) 토큰은 자동 정리
  - `FIREBASE_SERVICE_ACCOUNT` 환경변수 = 키 JSON **파일 경로**, project_id는 파일에서 읽음
- 의존성: `google-auth-library-oauth2-http`

### 앱 (PR #43)
- `firebase_core`/`firebase_messaging` + `com.google.gms.google-services` 플러그인(4.5.0)
- `google-services.json`은 **커밋 금지**(.gitignore) — 실행 스크립트가 `D:\ai제작\`에서 복사
- `PushService`: 앱 시작 시 Firebase 초기화 → 로그인 후 홈 진입 시 권한 요청 + 토큰 서버 등록 + 갱신 자동 재등록
- 미설정/웹/테스트에서는 조용히 건너뜀 (CI·mock 흐름 안전)

## E2E 검증 과정 (트러블슈팅 포함)

1. 앱 로그 "[PUSH] 기기 토큰 등록 완료" → DB device_token 저장 확인
2. 관리자 API로 주문 상태 변경 → 서버 로그 "[PUSH] 발송 성공 userId=8"
3. **처음엔 알림바에 안 뜸** → 원인: 앱이 포그라운드(안드로이드는 앱이 화면에 떠 있으면 tray 미표시).
   `flutter run`이 빌드 후 앱을 자동으로 앞에 띄워서 계속 포그라운드였음.
   adb로 홈 이동 후 재발송 → **알림 정상 표시** (`dumpsys notification`으로도 확인)

## 함께 잡은 것들

- **JWT_SECRET 고정** (secrets 파일): 이전엔 서버 재시작마다 로그인 전부 무효화됐음 → 이제 유지됨
- **카드 결제 채널 추가**(KG이니시스 테스트, 콘솔 등록) + `PORTONE_CHANNEL_KEY_CARD` → 카드 버튼이 이니시스 결제창으로 열림
- **결제 구매자 이름 필수** (PR #44): 이니시스 V2는 customer.fullName 필수 → 회원 이름 전달
- **카드 테스트 결제 E2E 성공** (삼성카드 승인 → 서버 검증 → 결제완료). ⚠️ 이니시스 테스트 상점은
  실제 승인 후 당일 자동취소 방식 — 승인 알림톡이 실제로 옴. 즉시 취소는 PortOne API
  `POST /payments/{id}/cancel` 로 가능 (이번에 12,900원 즉시 취소 처리함)
- 빌드 안정화: `org.gradle.daemon=false` + `kotlin.incremental=false` (gradle.properties, 유실됐던 것 복구)
  ※ 유실 원인: 자동머지가 첫 CI 통과 시점에 머지돼서, 그 뒤에 푸시한 커밋이 PR에 포함 안 됨.
  **교훈: PR 생성 후 추가 커밋을 푸시할 때는 머지 전인지 확인할 것.**

## 남은 것 (선택/운영 단계)

- 앱 포그라운드 상태에서도 알림 표시(flutter_local_notifications) — 현재는 인앱 알림함이 역할 수행
- 웹 푸시(VAPID/서비스워커), iOS(APNs) — 해당 플랫폼 배포 시
- 릴리즈 빌드 시 릴리즈 키해시(카카오) 추가 등록
