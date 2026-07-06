# 배포3: 도메인 연결 + HTTPS 적용 (2026-07-06)

## 결과
- **서비스 주소: https://ovenup.duckdns.org** (http 접속 시 https로 자동 이동)
- 인증서: Let's Encrypt (무료), 만료 2026-10-04, **자동 갱신 설정 완료** (certbot 타이머)

## 한 일
1. DuckDNS 무료 도메인 `ovenup.duckdns.org` 생성, 서버 IP `141.147.167.150` 연결
   - DuckDNS 계정: rkdxodh41@gmail.com (Google 로그인), 토큰은 DuckDNS 사이트에서 확인 가능
   - 주의: PC 자동화 브라우저에선 reCaptcha 점수 미달로 실패 → 폰 크롬(인앱 브라우저 X)에서 직접 등록함
   - 카카오톡 인앱 브라우저는 Google 로그인 차단됨(403 disallowed_useragent)
2. VM 방화벽에 443 포트 개방 (iptables + netfilter-persistent 저장)
   - 오라클 클라우드 보안목록(Security List)의 80/443은 이전 세션에서 이미 개방
3. certbot + python3-certbot-nginx 설치
4. nginx server_name을 ovenup.duckdns.org로 지정
5. `certbot --nginx -d ovenup.duckdns.org --redirect` 로 인증서 발급 + https 설정 자동 적용

## 검증
- https://ovenup.duckdns.org/api/menus → 200
- https://ovenup.duckdns.org/api/health → 200
- http → https 301 리다이렉트 확인
- 인증서 발급자 Let's Encrypt, 유효기간 2026-07-06 ~ 2026-10-04

## 다음 단계 (배포4)
- Flutter 웹을 Firebase Hosting에 배포
- 카카오/네이버/PortOne 콘솔에 새 주소(https://ovenup.duckdns.org) 등록
- 태블릿 APK의 API 주소를 https 도메인으로 변경 후 재빌드
