# 2026-07-08 건물 전용 앱 — 명지에코펠리스 제한

## 요구사항
"한 건물(상가 빌딩) 안에서만 앱을 사용하도록" — 빌딩 내 주소로만 사용 가능하게.
대상 건물: **명지에코펠리스** (부산 강서구 명지국제2로28번길 34, 지하2층~지상10층 상가)

## 선택한 방식 (입문자 설명)
GPS만으로 막으면 실내에서 위치가 수십 m 틀어져 정상 손님도 차단될 수 있어요. 그래서:

1. **주소 고정(1차 방어선)**: 주소 검색(다음 우편번호)을 아예 없애고, 건물 주소는 고정 + **층/호수만 입력**받아요. 다른 주소는 입력 자체가 불가능.
2. **GPS 보조 확인(2차)**: 주문(결제) 직전에 현재 위치를 확인해서 건물 반경 250m **밖이면 차단**. 권한 거부·실내 미수신 등 "확인 불가"면 통과(주소가 이미 고정돼 있으므로).
3. **서버 재검증**: 앱을 우회해도 서버가 주소·좌표를 다시 검사해요.

## 변경 내용
- 서버: `building/BuildingPolicy.java` 신설(건물명·주소·좌표 35.0928292/128.9088756·반경 250m, `app.building.*` 설정). 주소 등록/수정/가입 시 건물 주소만 허용(`ADDRESS_NOT_ALLOWED`), 주문에 lat/lng 오면 반경 검증(`LOCATION_NOT_ALLOWED`).
- 앱: `building_config.dart`(상수) + `building_gate.dart`(geolocator 위치 확인). 주소 입력 3개 화면(주소록·회원가입·소셜 온보딩) 층/호수 입력으로 교체. 주문서에서 GPS 확인 후 좌표 전송. Android 위치 권한 추가.

## 검증
- 서버 `gradlew test` 전체 통과, 앱 `flutter analyze` 0건 / `flutter test` 6건 통과 (한글경로 제약으로 D:\dev 복사 후 검증)
- 좌표 출처: 구글지도 (명지국제2로28번길 34)

## PR
- https://github.com/Tae0072/Oven-up/pull/59 (dev 대상)

## 서버 재배포 (2026-07-08 16:15 KST 완료)
- PR #59 자동머지 확인(dev c640e99) → D:\dev\ovenup-server-build에서 bootJar → scp → /opt/ovenup/server.jar 교체 → systemctl restart
- 확인: ovenup.service active, 내부 /api/menus 200, 외부 https://ovenup.duckdns.org/api/menus 200, 부팅 로그 오류 없음
- 이제 서버가 건물 밖 주소 등록·건물 밖 좌표 주문을 실제로 거절함

## 남은 확인
- 실기기(태블릿)에서 위치 권한 요청·차단 동작 확인 — 앱은 APK 재빌드·재설치 필요 (D:\dev\build-ovenup-apk.ps1)
- 웹(https://ven-up.web.app )도 재빌드·재배포해야 새 주소 UI가 반영됨 (D:\dev\build-ovenup-web.ps1)
- 기존 회원 중 건물 밖 주소로 저장된 계정은 다음 주소 변경 때부터 건물 주소만 선택 가능
