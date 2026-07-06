# 배포 1·2단계: 오라클 무료 VM 생성 + 오븐업 서버 배포 (2026-07-06)

## 요약
오라클 클라우드 무료 계정(도쿄 리전)에 VM을 만들고, 오븐업 서버(Spring Boot)를 실제 인터넷에서 접속 가능하게 배포했다.

- **서버 주소: http://141.147.167.150** (예: http://141.147.167.150/api/menus → 200 OK)
- 지금은 http(80)까지 완료. https(443)는 도메인 결정 후 다음 단계에서 연결.

## 만든 것들

### VM (오라클 클라우드, ap-tokyo-1)
| 항목 | 값 |
|---|---|
| 인스턴스 이름 | ovenup-server |
| Shape | VM.Standard.E2.1.Micro (Always Free, AMD 1 OCPU / RAM 1GB) |
| OS | Ubuntu 24.04 |
| 공인 IP | 141.147.167.150 (임시/ephemeral) |
| 사설 IP | 10.0.0.215 |
| SSH | `ssh -i D:\dev\ovenup-server-key ubuntu@141.147.167.150` |

⚠️ 원래 계획은 A1(ARM 4코어/24GB)이었지만 도쿄 리전 무료 용량 고갈로 생성 불가 → 사용자 선택으로 E2.1.Micro 진행. 나중에 A1 용량이 풀리면 이전 가능.

### 네트워크
- VCN `vcn-20260706-1704`, 공개 서브넷 `subnet-20260706-1704`
- 퀵액션 "Connect public subnet to internet"으로 인터넷 게이트웨이/라우팅 연결
- NSG `ig-quick-action-NSG`: TCP 80, 443 인그레스 허용 (0.0.0.0/0)
- VM 내부 iptables도 80/443 허용 + netfilter-persistent로 영구 저장
- 공인 IP는 생성 마법사에서 지정 불가(버그성 UI) → 생성 후 VNIC → IP administration → Edit에서 Ephemeral 할당

### VM 안 구성
- 스왑 3GB (`/swapfile`) — RAM 1GB 보강. fstab 등록됨
- OpenJDK 21 (jre-headless), MySQL 8.0.46
- MySQL: DB `ovenup`(utf8mb4), 계정 `ovenup@localhost` (비밀번호는 secrets 파일에 `OVENUP_DB_PASSWORD`)
- 앱 배치: `/opt/ovenup/server.jar`
- 환경변수: `/etc/ovenup/ovenup.env` (chmod 600, 소유자 ovenup) — DB 접속·카카오/네이버/포트원/JWT 키, mock 3종 전부 false
- Firebase 키: `/etc/ovenup/firebase-service-account.json`
- systemd 서비스 `ovenup.service`: 자동시작, 죽으면 10초 후 재시작, 힙 -Xmx400m
- nginx 리버스 프록시: 80 → 127.0.0.1:8080

## 서버 운영 명령어 (SSH 접속 후)
```
sudo systemctl status ovenup      # 상태 확인
sudo systemctl restart ovenup     # 재시작
sudo journalctl -u ovenup -f      # 실시간 로그
```

## 새 jar 배포 방법 (로컬 PC에서)
```powershell
robocopy "D:\ai제작\Oven-up\server" "D:\dev\srvchkN" /MIR /XD build .gradle .git
cd D:\dev\srvchkN; $env:GRADLE_USER_HOME='D:\dev\gradlehome'; .\gradlew.bat bootJar
& "D:\Git\usr\bin\scp.exe" -i D:/dev/ovenup-server-key D:/dev/srvchkN/build/libs/server-0.0.1-SNAPSHOT.jar ubuntu@141.147.167.150:/home/ubuntu/
& "D:\Git\usr\bin\ssh.exe" -i D:/dev/ovenup-server-key ubuntu@141.147.167.150 "sudo mv /home/ubuntu/server-0.0.1-SNAPSHOT.jar /opt/ovenup/server.jar && sudo chown ovenup:ovenup /opt/ovenup/server.jar && sudo systemctl restart ovenup"
```

## 트러블슈팅 기록
1. **A1 Out of capacity**: 4→2→1 OCPU 전부 실패. 도쿄 리전 Free Tier에서 상시 발생하는 문제. E2.1.Micro로 우회.
2. **Windows OpenSSH 고장**: `C:\Windows\System32\OpenSSH\ssh.exe`가 무출력 exit 255 (ssh-keygen도 동일했음). **Git 내장 ssh/scp(`D:\Git\usr\bin\ssh.exe`, `scp.exe`) 사용**으로 해결.
3. **PowerShell에서 `'ovenup'@'localhost'` 인용 문제**: `@'`를 here-string으로 해석. SQL을 파일로 저장해 `type file | ssh "sudo mysql"`로 우회.
4. **생성 마법사 공인 IP 토글 비활성**: "public subnet을 선택하라"는 경고가 떠도 무시하고 생성 → 생성 후 VNIC에서 할당하면 됨.

## 남은 단계
- 배포3: 도메인 결정 → https(Let's Encrypt) + Flutter 웹 Firebase 호스팅
- 배포4: 카카오/네이버/PortOne 콘솔 주소 갱신, 앱 API_BASE_URL 교체, APK 추출, E2E 검증
