# 작업로그 · 2026-07-04 — 푸시 알림 (§9)

> 주문 상태 변경·결제 시 손님에게 알림. 인앱 알림 기본 동작 + 실제 FCM 연동 준비/가이드. (PR #25)

## 1. 워크플로우
1. [x] 서버 알림 도메인/API + PushSender(mock) + 결제/상태변경 훅 + 테스트
2. [x] 서버 빌드검증(ASCII,H2) + MySQL 실측(상태변경→알림)
3. [x] 앱 알림 목록 화면 + 홈 종 아이콘(배지) + analyze/test
4. [x] 커밋/PR(#25) → dev 머지 + FCM 가이드/작업로그

## 2. 리포트 (PR #25)

### 서버 (05_API §9)
- `notification` 도메인 + API: `GET /api/notifications`, `/unread-count`, `POST /{id}/read`, `/read-all` (로그인, 본인만).
- `PushSender` — Mock(기본 `app.push.mock=true`, 로그만) / Real(FCM 자리, false). **인앱 알림은 항상 저장**.
- 훅: 결제 완료(ORDER_PAID) + 관리자 상태 변경(ORDER_STATUS) 시 손님 알림 생성 + 푸시. 상태별 문구.
- 테스트: 상태변경→알림 생성, 안읽음수, 읽음 처리, 미로그인 401.

### 앱
- `NotificationApi` + 알림 목록 화면(읽음·모두 읽음).
- 홈 상단 종 아이콘 + 안읽음 배지 → 알림 목록.

## 3. 검증
- 서버 gradlew build(H2) 통과. MySQL 실측: 결제 → unread 1, 상태변경(준비완료) → unread 2, 알림 한글 정상
  ("결제가 완료되어 주문이 접수됐어요." / "준비가 완료됐어요! 픽업/수령해 주세요."), 읽음 처리 확인.
- 앱 flutter analyze 무결점 + test 6건.

## 4. 실제 FCM 연동
- 가이드: `가이드/푸시_알림_FCM.md`. 요약: Firebase 프로젝트+서비스계정 →
  기기 토큰 등록 API + RealPushSender(FCM HTTP v1) 구현 → `app.push.mock=false`.

## 5. 다음 단계
- 실제 FCM 푸시, 예약 주문 리마인드, 단체주문/고객의소리 답변 알림.
