# 작업로그 — 회원정보 수정(프로필·비밀번호 변경)

- 날짜: 2026-07-05
- 브랜치: `feature/profile-edit` → PR #31 (base: dev)
- 관련 화면/명세: S11(내 정보 수정), 05_API §2.4~2.5

## 무엇을 했나 (쉽게)
손님이 마이페이지에서 이름·연락처를 바꾸고, 비밀번호도 바꿀 수 있게 만들었어요.
지금까지 마이페이지는 "보기"만 됐는데, 이제 **정보 수정** 버튼으로 직접 고칠 수 있습니다.

## 서버
- `PATCH /api/users/me` — 이름·연락처 수정(이름이 비면 400 INVALID_INPUT)
- `PATCH /api/users/me/password` — 현재 비밀번호를 확인한 뒤 새 비밀번호로 교체
  - 현재 비밀번호 불일치 → 400 `PASSWORD_MISMATCH`
  - 새 비밀번호 8자 미만 → 400 `INVALID_INPUT`
- `UserService`(BCrypt 재사용) + `UserEntity.updateProfile/changePassword`
- 테스트 6건(`UserProfileControllerTest`): 프로필 수정/빈 이름/미로그인, 비번 변경 성공→새 비번 로그인/현재 불일치/새 비번 짧음

## 앱
- `screens/profile_edit_page.dart`(S11): 이메일(읽기전용) + 이름·연락처 저장 + 비밀번호 변경(현재/새/새 확인)
- `screens/my_page.dart`: 헤더에 **정보 수정** 버튼 진입점
- `data/auth_api.dart`: `fetchProfile/updateProfile/changePassword` + `MyProfile` 모델
- `state/auth_store.dart`: `updateName` — 이름 변경 시 마이페이지 헤더 즉시 갱신 + 기기 저장 반영

## 검증
- 서버: ASCII 복사본(`D:\dev\srvchk13`) H2 `gradlew build` 성공(프로필 테스트 6건 포함, BUILD SUCCESSFUL)
- 앱: ASCII 복사본(`D:\dev\appcheck`) `flutter analyze` 무이슈, `flutter test` 6건 통과

## 메모
- 이메일은 로그인 아이디라 변경 불가로 뒀어요(읽기전용).
- 비밀번호는 서버에서 BCrypt로만 저장하고, 현재 비밀번호 확인을 반드시 거칩니다.
