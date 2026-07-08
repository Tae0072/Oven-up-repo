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

## 웹·APK 배포 (2026-07-08 16:25 KST)
- 웹: build-ovenup-web.ps1 → Firebase Hosting 배포 완료. 배포본에 건물 주소 포함 확인(main.dart.js 이스케이프 문자열 검사 + 파일 크기 일치)
  ※ 확인 팁: dart2js는 한글을 \uXXXX로 이스케이프하므로 한글 그대로 검색하면 안 나옴
- APK: build-ovenup-apk.ps1 → 성공. 산출물을 `D:\ai제작\오븐업\ovenup-release-building-only.apk` 로 복사해 둠 (63MB)
  ※ 1차 실패: 이전 빌드 잔여물 잠김(mergeReleaseNativeLibs AccessDenied) → gradlew --stop + build 폴더 삭제 후 성공
- 웹은 강력새로고침 2회 필요할 수 있음 (기존 캐시)

## 2차 강화: 위치 필수 + 옛 주소 정리 (2026-07-08 16:45 KST, PR #60)
실사용 확인에서 두 구멍 발견: ①제한 도입 전 등록된 건물 밖 주소가 그대로 사용 가능 ②위치가 안 잡히면(권한 거부 등) 주문 통과.
작업자 결정: **엄격 모드(위치 필수) + 옛 주소 일괄 삭제.**

- 서버: 좌표 없는 주문 거절(`LOCATION_REQUIRED`, `app.building.require-location=true`), 시작 시 건물 밖 주소 자동 정리(`BuildingAddressCleanup`), 건물 밖 주소는 선택도 거절
- 앱: 위치 확인 실패도 주문 차단(안내 문구), 측위 정확도 high·타임아웃 15초
- 테스트: `BuildingLocationOrderTest` 3건 신설(좌표 없음/건물 밖/건물 안), 전체 테스트·analyze 통과
- 배포: 서버(VM)·웹(ven-up.web.app)·APK 모두 재배포 완료. 서버 기동 로그에서 **옛 주소 2건 삭제, 회원 현재주소 1건 초기화** 확인
- APK: `D:\ai제작\오븐업\ovenup-release-building-only.apk` 갱신 (태블릿 연결 시 설치)

⚠️ 이제 위치 권한을 허용해야만 주문 가능. 실내에서 위치가 오래 안 잡히는 기기가 있으면 반경(250m)·타임아웃(15초) 또는 `require-location=false` 완화로 조정 가능.

## 남은 확인
- 태블릿 연결 후 APK 설치, 실기기에서 위치 권한 요청·건물 밖 차단 동작 확인
- 실사용 재확인: 건물 밖에서 주문 시도 → "위치 확인 필요" 또는 "건물 안에서만 주문" 메시지로 차단되는지
