# 작업로그 — 회원 탈퇴 + 알림 설정(on/off)

- 날짜: 2026-07-05
- 브랜치: `feature/account-settings` → PR #34 (base: dev)
- 관련 화면/명세: S11(내 정보 수정), 05_API §2.5, §9(알림)

## 무엇을 했나 (쉽게)
손님이 알림을 **켜고 끌 수** 있게 하고, 원하면 **회원 탈퇴**도 할 수 있게 했어요.
알림을 꺼두면 주문·문의 알림이 더 이상 쌓이지 않고, 탈퇴하면 계정이 삭제됩니다(비밀번호 확인 필요).

## 서버
- `PATCH /api/users/me/notify` — 알림 켜기/끄기
- `DELETE /api/users/me` — 현재 비밀번호 확인 후 계정 삭제(불일치 시 400 `PASSWORD_MISMATCH`)
- `UserEntity.notifyDisabled`(기본 false = 알림 켜짐 → 기존 회원도 자동으로 켜짐 상태)
  - 새 컬럼을 "끄기 여부"로 저장해, 이미 있던 행이 0(false)로 들어와도 자연스럽게 '켜짐'
- `MyProfile`에 `notifyEnabled` 추가(내 정보 응답)
- `NotificationService.notifyUser`: 손님이 알림을 꺼뒀으면 저장·푸시를 생략
- 테스트 4건(`UserProfileControllerTest`): 알림 기본 on·off 토글, 탈퇴 비번불일치 400, 탈퇴 후 로그인 실패

## 앱
- `data/auth_api.dart`: `setNotifyEnabled`, `deleteAccount` + `MyProfile.notifyEnabled`
- `screens/profile_edit_page.dart`
  - '알림' 섹션에 `주문·문의 알림 받기` 스위치(즉시 저장, 실패 시 되돌림)
  - '계정' 섹션에 `회원 탈퇴` 버튼 → 현재 비밀번호 확인 다이얼로그 → 탈퇴 → 로그아웃 → 로그인 화면(게이트)

## 검증
- 서버: ASCII 복사본(`D:\dev\srvchk16`) H2 `gradlew build` 성공(계정 테스트 4건 포함, BUILD SUCCESSFUL)
- 앱: ASCII 복사본(`D:\dev\appcheck`) `flutter analyze` 무이슈, `flutter test` 6건 통과

## 메모
- 회원 탈퇴는 되돌릴 수 없어, 반드시 현재 비밀번호를 확인합니다(소셜 전용 계정은 이 경로로 탈퇴 불가 — 추후 소셜 재인증 흐름 필요).
- 알림 설정은 서버에서 발송 시점에 확인하므로, 껐다면 인앱 알림함에도 새 알림이 쌓이지 않아요.
