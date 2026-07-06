# 작업로그 · 알림 다듬기 + 운영 알림 확장 (①·③)

- 날짜: 2026-07-06
- 브랜치/PR: `feature/polish-notifications` → dev PR #45 (머지), `fix/desugaring` PR #46
- 외부 연동 3종 완료 후의 다듬기(①)와 기능 확장(③) 단계

## 앱 (①)

1. **포그라운드 알림 표시**: 안드로이드는 앱이 화면에 떠 있으면 푸시를 알림바에 안 그려줌
   → `flutter_local_notifications`로 FCM `onMessage`(포그라운드 수신)를 받아 직접 표시.
   전용 채널 `ovenup_default`(오븐업 알림, importance high) 생성.
   - 요구사항: core library desugaring 활성화 필요 (PR #46 — compileOptions + desugar_jdk_libs)
2. **로그인 만료 자동 처리**: 스플래시에서 저장된 토큰을 `GET /api/users/me`로 검증.
   서버가 거부(만료)하면 조용히 로그아웃 → 로그인 화면. 서버 연결 불가면 세션 유지(홈에서 재시도).

## 서버 (③)

1. **사장님 새 주문 알림**: 결제완료 시 `notifyAdmins()` — ADMIN 역할 전원에게 인앱+푸시.
   "새 주문 20260706-000X — 포장 주문이 들어왔어요. (12,900원)"
   (사장님이 앱에 관리자 계정으로 로그인해 두면 기기 푸시도 받음)
2. **예약 주문 리마인드**: `@Scheduled` 1분 주기 — 예약 시각 30분 전에 손님에게 알림.
   `orders.reminder_sent` 컬럼으로 중복 발송 방지. 대상 상태: 결제완료/준비중/준비완료.
   `@EnableScheduling` 활성화.
3. 문의 답변 알림은 기존에 이미 구현돼 있었음(INQUIRY_REPLY) — 추가 작업 없음.

## 검증

- 서버 gradlew build 통과 / 앱 analyze 0건, test 6건 통과 / 태블릿 빌드·설치 확인
- 백그라운드 푸시 재확인(dumpsys notification에 기록). 포그라운드 표시는 기기에서 육안 확인 예정

## 빌드 환경 메모

- `org.gradle.daemon=false` + `kotlin.incremental=false` → 레포 gradle.properties에 확정 커밋
- 반복되던 build 폴더 잠금: 원인은 좀비 dart/adb/java 프로세스 → run-ovenup-tablet.ps1에
  "서버 보호 + 나머지 정리 + build 폴더 삭제" 단계 내장 (이후 재발 없음)

## 다음 단계

- ② 배포 준비: 서버 클라우드 호스팅 + https + 도메인, 릴리즈 빌드·스토어 등록, PG 실계약 전환, 카카오 비즈니스 인증
