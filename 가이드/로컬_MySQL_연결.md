# 가이드 · 로컬 MySQL 연결 (입문자용)

> 오븐업 서버는 지금 **H2(설치 불필요)** 로 바로 돌아갑니다.
> 실제 MySQL로 바꾸고 싶을 때 이 문서를 따라 하세요. (급하지 않으면 나중에 해도 됩니다.)

---

## 0. 왜 MySQL?

- H2는 "연습용 임시 창고"라 서버를 끄면 데이터가 사라집니다(메뉴는 시더가 다시 채움).
- MySQL은 "진짜 창고"라 데이터가 계속 남습니다. 주문·회원이 생기면 필요합니다.

---

## 1. MySQL 설치 (Windows)

### 방법 A) winget (간단)
PowerShell에서:
```
winget install Oracle.MySQL
```
설치 후 MySQL이 서비스로 실행됩니다.

### 방법 B) 공식 설치 파일
- https://dev.mysql.com/downloads/installer/ 에서 "MySQL Installer" 받아 설치.
- 설치 중 **root 비밀번호**를 정합니다(꼭 기억!).

> 설치가 부담되면 **MySQL 대신 Docker**로도 가능합니다(선택):
> ```
> docker run --name ovenup-mysql -e MYSQL_ROOT_PASSWORD=1234 -e MYSQL_DATABASE=ovenup -p 3306:3306 -d mysql:8
> ```

---

## 2. 데이터베이스 만들기

MySQL에 접속해서(예: MySQL Workbench 또는 명령줄 `mysql -u root -p`) 아래 한 줄 실행:
```sql
CREATE DATABASE ovenup DEFAULT CHARACTER SET utf8mb4;
```
- `utf8mb4` 는 한글·이모지까지 안전하게 저장하기 위한 설정입니다.

---

## 3. 접속 정보 넣기 (비밀번호는 코드에 쓰지 않기)

서버는 아래 **환경변수**로 접속 정보를 받습니다. 없으면 기본값(root / 빈 비밀번호)을 씁니다.

| 환경변수 | 예시 | 설명 |
| --- | --- | --- |
| `DB_URL` | `jdbc:mysql://localhost:3306/ovenup?serverTimezone=Asia/Seoul&characterEncoding=UTF-8` | 접속 주소 |
| `DB_USERNAME` | `root` | 계정 |
| `DB_PASSWORD` | `1234` | 비밀번호 |

PowerShell에서 이번 실행에만 적용하려면:
```
$env:DB_PASSWORD = "여기에_본인_비밀번호"
```

> ⚠️ 비밀번호를 코드나 `application.properties` 에 **직접 쓰지 마세요.** (06_Git_PR_규칙 §8)
> 커밋에는 값 없는 예시(`server/.env.example`)만 올립니다.

---

## 4. MySQL로 서버 실행

`server` 폴더에서:
```
./gradlew bootRun --args='--spring.profiles.active=mysql'
```
Windows PowerShell:
```
.\gradlew.bat bootRun --args='--spring.profiles.active=mysql'
```

- 처음 실행 시 JPA가 `menu`, `menu_option` 테이블을 자동 생성하고(시더가) 메뉴 7종을 넣습니다.
- 확인: 브라우저에서 `http://localhost:8080/api/menus` → 메뉴 7종이 보이면 성공.

---

## 5. 잘 안 될 때 (자주 겪는 문제)

| 증상 | 원인/해결 |
| --- | --- |
| `Access denied for user 'root'` | `DB_PASSWORD` 가 실제 비밀번호와 다름. 다시 설정. |
| `Unknown database 'ovenup'` | 2번의 `CREATE DATABASE` 를 안 함. |
| `Communications link failure` | MySQL이 안 켜져 있음. 서비스/Docker 실행 확인, 포트 3306. |
| 한글이 깨져 저장됨 | DB를 `utf8mb4` 로 만들었는지 확인. |

---

## 6. 다시 H2로 돌아가려면

그냥 프로필 없이 실행하면 됩니다:
```
.\gradlew.bat bootRun
```
