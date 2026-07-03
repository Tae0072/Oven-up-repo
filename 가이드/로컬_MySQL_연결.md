# 가이드 · 로컬 MySQL 연결

> 이 컴퓨터에는 오븐업용 **로컬 MySQL이 이미 설치·연결**되어 있습니다.
> 서버 기본 DB가 MySQL이라, 보통은 아무것도 안 해도 됩니다. 아래는 조작법과 문제 해결입니다.

---

## 0. 지금 상태 (이 컴퓨터)

| 항목 | 값 |
| --- | --- |
| MySQL | 8.4.9 (무설치본) · `D:\dev\mysql-8.4.9-winx64` |
| 데이터 폴더 | `D:\dev\mysql-data` |
| 설정 파일 | `D:\dev\my.ini` (포트 3306, utf8mb4) |
| 계정 | `root` / 비밀번호 **없음** (로컬 전용) |
| 데이터베이스 | `ovenup` |
| 자동 시작 | 로그인 시 자동 실행 (시작프로그램에 `ovenup-mysql-start.vbs` 등록) |

> ⚠️ 비밀번호 없는 root는 **로컬 개발 전용**입니다. 외부에 공개되는 서버에서는 절대 이렇게 쓰지 마세요.

---

## 1. 서버 실행 (평소)

MySQL은 로그인하면 자동으로 켜져 있습니다. 그냥 서버만 실행하면 됩니다.
```
cd D:\ai제작\Oven-up\server
.\gradlew.bat bootRun
```
- 확인: 브라우저에서 `http://localhost:8080/api/menus` → 메뉴 7종.
- H2와 달리 **데이터가 계속 남습니다**(서버를 껐다 켜도 유지).

---

## 2. MySQL 켜고 끄기

- **켜기**: `D:\dev\ovenup-mysql-start.vbs` 더블클릭 (창 없이 실행됨). 보통은 자동 실행이라 필요 없음.
- **끄기**: `D:\dev\ovenup-mysql-stop.bat` 더블클릭.
- **켜졌는지 확인**(PowerShell):
  ```
  Test-NetConnection localhost -Port 3306 -InformationLevel Quiet
  ```
  `True` 면 실행 중.

---

## 3. 데이터 직접 보기

명령줄 접속:
```
D:\dev\mysql-8.4.9-winx64\bin\mysql.exe -u root --host=127.0.0.1
```
접속 후 SQL:
```sql
USE ovenup;
SHOW TABLES;
SELECT id, name, price FROM menu ORDER BY id;
```
- 화면 도구를 원하면 **MySQL Workbench** 나 **DBeaver** 로 `localhost:3306` / `root` / (비번 없음) 접속.

---

## 4. MySQL 없이 임시로 돌리기 (H2)

MySQL이 문제가 있을 때 설치 없이 임시로 실행:
```
.\gradlew.bat bootRun --args='--spring.profiles.active=h2'
```
- H2는 메모리 DB라 서버를 끄면 초기화됩니다(메뉴는 시더가 다시 채움).

---

## 5. 문제 해결

| 증상 | 해결 |
| --- | --- |
| 서버 실행 시 `Communications link failure` | MySQL이 꺼져 있음 → `ovenup-mysql-start.vbs` 실행, 포트 3306 확인. |
| `Unknown database 'ovenup'` | DB가 없음 → 아래 6번으로 다시 생성. |
| 포트 3306 충돌 | 다른 MySQL이 이미 3306 사용 중. 하나만 켜세요. |
| 한글 깨짐 | DB가 `utf8mb4` 인지 확인(현재 설정은 utf8mb4). |

---

## 6. (참고) 다른 컴퓨터에서 처음부터 만들기

1. MySQL 준비: `winget install Oracle.MySQL` 또는 무설치 zip(https://dev.mysql.com/downloads/mysql/) 을 원하는 폴더에 풀기.
2. 무설치본이면 설정 파일(`my.ini`)에 `basedir`/`datadir`/`port=3306`/`character-set-server=utf8mb4` 지정 후 초기화:
   ```
   bin\mysqld.exe --defaults-file=경로\my.ini --initialize-insecure
   bin\mysqld.exe --defaults-file=경로\my.ini
   ```
3. DB 생성:
   ```sql
   CREATE DATABASE ovenup DEFAULT CHARACTER SET utf8mb4;
   ```
4. 서버 실행: `.\gradlew.bat bootRun` (접속정보가 다르면 환경변수 `DB_URL`/`DB_USERNAME`/`DB_PASSWORD` 로 지정).

> 비밀번호를 쓰는 경우, 코드가 아니라 **환경변수**로 넣으세요. 커밋에는 값 없는 `server/.env.example` 만 올립니다. (06_Git_PR_규칙 §8)
