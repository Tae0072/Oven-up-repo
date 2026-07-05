# 작업로그 — 관리자 고객의 소리 답변 + 단체주문 관리

- 날짜: 2026-07-05
- 브랜치: `feature/admin-support` → PR #33 (base: dev)
- 관련 화면/명세: A6(고객의 소리 답변), A7(단체주문 관리), 05_API §6·§7, §9(알림)

## 무엇을 했나 (쉽게)
사장님이 손님 문의에 **답변**을 달고, **단체주문 문의**를 처리(상태 변경·메모)할 수 있게 만들었어요.
답변을 등록하거나 단체주문 상태를 바꾸면 손님에게 **알림**이 갑니다.

## 서버 (`AdminSupportController`, ADMIN 전용, 손님은 403)
- `GET /api/admin/inquiries` — 전체 문의(내용·답변 포함, 최신순)
- `POST /api/admin/inquiries/{id}/reply` — 답변 등록/수정
  - 답변 저장 → 문의 상태 `답변완료` → 손님에게 알림(`INQUIRY_REPLY`)
  - 빈 답변은 400 `INVALID_INPUT`
- `GET /api/admin/group-orders` — 전체 단체주문 문의
- `PATCH /api/admin/group-orders/{id}` — 상태(접수/협의중/확정/취소)·메모 갱신 → 손님 알림(`GROUP_ORDER`)
  - 허용되지 않은 상태는 400 `INVALID_INPUT`
- 재사용: `InquiryReplyEntity`(기존), `NotificationService.notifyUser`
- 리포지토리에 `findAllByOrderByIdDesc` 추가(문의·단체주문)
- 테스트 6건(`AdminSupportControllerTest`)

## 앱
- `data/admin_api.dart`: 문의 목록/답변, 단체주문 목록/갱신 + `AdminInquiry` 모델, 상태 상수
- `screens/admin_inquiries_page.dart`: 문의 목록 + 답변 바텀시트(상태 칩 표시)
- `screens/admin_group_orders_page.dart`: 단체주문 목록 + 상태 칩·메모 편집 바텀시트
- `screens/my_page.dart`: 관리자 메뉴에 '고객의 소리 답변', '단체주문 관리' 진입점 추가

## 검증
- 서버: ASCII 복사본(`D:\dev\srvchk15`) H2 `gradlew build` 성공(고객지원 테스트 6건 포함, BUILD SUCCESSFUL)
- 앱: ASCII 복사본(`D:\dev\appcheck`) `flutter analyze` 무이슈, `flutter test` 6건 통과
  - (BuildContext async-gap 린트 2건은 `mounted` 가드로 해결)

## 메모
- 답변/상태 변경 시 손님에게 알림이 가므로, 손님은 홈 벨 아이콘 배지로 바로 확인할 수 있어요(기존 알림 기능 재사용).
- 단체주문은 즉시 결제가 아니라 협의형이라, 상태 흐름(접수→협의중→확정/취소)과 메모로 관리합니다.
