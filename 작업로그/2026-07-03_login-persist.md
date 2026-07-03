# 작업로그 · 2026-07-03 — 로그인 유지(기기 저장)

> 로그인 상태를 기기에 저장해 앱을 껐다 켜도 유지되게 함. (PR #16)

## 1. 워크플로우
1. [x] shared_preferences 추가 + AuthStore 저장/복원 + main 시작 시 복원
2. [x] ASCII 복사로 analyze/test 검증
3. [x] 커밋/PR(#16) → dev 머지 + 문서 저장

## 2. 리포트 (PR #16)
- `shared_preferences` 의존성 추가.
- `AuthStore`: 로그인 시 토큰·회원(id/name/role) 기기 저장, 로그아웃 시 삭제, 앱 시작 시 `load()`로 복원.
- `main()`에서 `WidgetsFlutterBinding.ensureInitialized()` 후 `AuthStore.instance.load()` → `runApp`.

## 3. 검증
- flutter analyze 통과(No issues found), flutter test 통과(4건). (한글 경로 이슈로 analyze는 ASCII 복사본에서 실행)

## 4. 확인 방법
1) 로그인(demo@oven.com / demo1234) → 2) 앱 완전 종료 후 재실행 → 여전히 로그인 상태

## 5. 참고 / 다음 단계
- 토큰 만료(7일) 검증은 아직 없음(복원만). 이후 시작 시 GET /api/users/me로 검증해 만료 시 자동 로그아웃 개선 가능.
- 다음: 서버 장바구니(§3.3), 결제(§5), 소셜 로그인(§2.3).

## 부록 — 이번 세션 환경 이슈 해결
- Windows-MCP 연결이 끊겨 Desktop Commander(PowerShell)로 전환해 커밋/PR/실측 진행.
- 한글 경로/커밋메시지가 stdin에서 깨져, 텍스트는 파일로 써서 우회(git commit -F, gh --body-file).
- 깨진 Flutter 설치(D:\TOOL\flutter, git 손상)가 PATH 우선이라 `flutter run` 실패 → 사용자 PATH에서 제거해 정상 설치(D:\dev\flutter)로 정리. 앱 web 빌드 성공 확인.
