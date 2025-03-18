> 상황 
- WSL, PostgreSQL, Docker 간 충돌

### 📌 문제 요약
- 현재 WSL(Windows Subsystem for Linux) 환경에서 로컬 PostgreSQL 서비스와 Docker 컨테이너의 PostgreSQL이 두개가 동시에 띄어져있음
- 인텔리제이에서 DB 연동 시 도커에서 띄운 PostgreSQL이 아닌 WSL 환경에서 띄운 PostgreSQL 이랑 연동이 되버림

### 🔍 해결 
1. `systemctl status postgresql.service` 결과를 보면 WSL 내부에서 PostgreSQL 서비스가 실행되고 있었음
    - 세번째 `Active: active (exited) since Fri 2025-02-14 09:01:18 KST; 6h ago` 실행 중임을 확인 가능
```dotenv
  postgresql.service - PostgreSQL RDBMS
  Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; preset: enabled)
  Active: active (exited) since Fri 2025-02-14 09:01:18 KST; 6h ago
  Process: 613 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
  Main PID: 613 (code=exited, status=0/SUCCESS)
```
2. `systemctl stop postgresql.service` 을 실행한 후에야 Docker 컨테이너의 PostgreSQL에 정상적으로 연결될 것
   - 아래 명령어를 차례대로 칠 것
```dotenv
systemctl status postgresql.service # 실행 중 확인
sudo systemctl stop postgresql.service # 중지
sudo systemctl disable postgresql.service  # 부팅 시 자동 실행 방지 (선택)
```

   - 세번째 `Active: inactive (dead) since Fri 2025-02-14 15:23:51 KST; 4s ago` 중지 됨을 확인 가능
    
```dotenv
   postgresql.service - PostgreSQL RDBMS
   Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; preset: enabled)
   Active: inactive (dead) since Fri 2025-02-14 15:23:51 KST; 4s ago
   Duration: 6h 22min 32.774s
   Process: 613 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 613 (code=exited, status=0/SUCCESS)
```
### ✅ 해결 방법
1. WSL 내부 PostgreSQL 완전히 중지
```dotenv
   sudo systemctl stop postgresql.service
   sudo systemctl disable postgresql.service  # 부팅 시 자동 실행 방지
```
   - disable을 해두면, WSL을 재부팅해도 PostgreSQL이 자동 실행되지 않음

2. Docker PostgreSQL만 실행되도록 설정
   - Docker에서 PostgreSQL을 사용할 것이므로, 로컬 PostgreSQL을 완전히 비활성화
```dotenv
    docker-compose down -v  # 기존 DB 볼륨 제거
    docker-compose up -d --build  # 새로운 DB로 컨테이너 실행
```
→ 이제 Docker PostgreSQL이 5432 포트에서 정상 동작할 것


### 🚨 원인 정리 
- WSL에서는 Windows와 같은 IP를 공유
  - →  localhost(127.0.0.1)로 접근할 때, WSL 내부 PostgreSQL에 연결될 수도 있고, Docker의 PostgreSQL에 연결될 수도 있음

- Docker PostgreSQL과 WSL PostgreSQL이 같은 포트(5432)를 사용
  - → 인텔리제이에서 localhost:5432로 접속할 때, WSL PostgreSQL에 먼저 연결됨!!
  - → 하지만 Docker PostgreSQL을 사용해야 하므로, WSL PostgreSQL을 중지해야 함

- WSL에서 systemd가 기본적으로 활성화되지 않음
  - → systemctl stop postgresql.service를 실행해도 제대로 중지되지 않음
  - → sudo systemctl stop postgresql.service 명령을 실행해야 정상적으로 중지됨
