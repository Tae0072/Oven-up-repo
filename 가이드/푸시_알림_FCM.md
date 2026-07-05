# 푸시 알림(FCM) 연동 가이드

> 오븐업 알림은 **인앱 알림**(서버 저장 + 앱에서 조회/배지)이 기본으로 동작해요.
> 손님 휴대폰에 **실제 OS 푸시**를 보내려면 Firebase Cloud Messaging(FCM) 연동이 필요합니다.
> 버전: v0.1 · 작성일: 2026-07-04

---

## 0. 지금 상태

- 주문 **결제 완료**·**관리자 상태 변경** 시 서버가 손님 알림을 **저장**하고, 앱은 홈 종 아이콘 배지 + 알림 목록으로 보여줘요. (이미 동작)
- `app.push.mock=true`(기본) → 실제 기기 푸시는 로그만 남김(`MockPushSender`).
- 실제 FCM 푸시를 켜려면 아래를 설정하고 `app.push.mock=false`.

---

## 1. 흐름

```
[이벤트] 결제완료 / 상태변경
   → 서버: 알림 저장(NotificationEntity)  ← 항상
   → 서버: PushSender.send(userId, title, body)
              ├ (mock)  로그만
              └ (real)  FCM HTTP v1 로 그 손님 기기 토큰에 푸시 전송
[손님 앱] 인앱 알림 목록 + (실연동 시) OS 푸시 수신
```

---

## 2. 준비물 (Firebase)

1. [Firebase 콘솔](https://console.firebase.google.com) 프로젝트 생성.
2. **Cloud Messaging** 활성화.
3. **서비스 계정 키(JSON)** 발급(프로젝트 설정 → 서비스 계정). 서버가 FCM 호출 시 인증에 사용.
4. 앱(Flutter)에 Firebase 등록:
   - Android: `google-services.json`
   - iOS: `GoogleService-Info.plist` + APNs 키
   - Web: Firebase 설정 + VAPID 키 + 서비스워커

---

## 3. 서버에서 해야 할 일

### 3-1. 기기 토큰 저장 (추가 구현 필요)
손님 앱이 받은 FCM 토큰을 서버에 저장할 곳이 필요해요.
- `device_token` 테이블(user_id, token) + 등록 API(`POST /api/notifications/device-token`) 추가.
- 앱 로그인 후 `firebase_messaging`으로 토큰을 받아 이 API로 등록.

### 3-2. RealPushSender 구현
`server/.../notification/RealPushSender.java` 의 `send()`를 채운다.
1. 서비스 계정 키로 OAuth2 액세스 토큰 발급(google-auth 라이브러리).
2. `POST https://fcm.googleapis.com/v1/projects/{PROJECT_ID}/messages:send`
   - 헤더: `Authorization: Bearer <access_token>`
   - 본문: 해당 userId의 기기 토큰 + `notification{title, body}`.
3. 여러 기기면 토큰마다 전송, 만료 토큰은 정리.

### 3-3. 설정
- `app.push.mock=false`
- 서비스 계정 키 경로/프로젝트 ID는 **환경변수**로(키는 커밋 금지).

---

## 4. 앱에서 해야 할 일

1. `firebase_core`, `firebase_messaging` 패키지 추가 + 플랫폼 설정.
2. 로그인 후 FCM 토큰 받아 `POST /api/notifications/device-token` 등록.
3. 포그라운드/백그라운드 푸시 수신 핸들러 → 필요 시 인앱 알림 목록 새로고침.
4. 알림 권한 요청(iOS/웹 필수).

---

## 5. 체크리스트
- [ ] Firebase 프로젝트 + 서비스 계정 키
- [ ] 앱에 Firebase 등록(플랫폼별)
- [ ] 기기 토큰 등록 API + 앱 연동
- [ ] RealPushSender FCM 전송 구현 + `app.push.mock=false`
- [ ] 실제 기기로 푸시 수신 테스트
- [ ] 키는 환경변수로만(커밋 금지)

## 6. 다음 단계
- 예약 주문 리마인드, 단체주문/고객의소리 답변 알림도 같은 방식으로 확장.
