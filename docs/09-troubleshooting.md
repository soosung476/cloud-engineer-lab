# Troubleshooting Notes

이 문서는 실습 중 실제로 겪은 문제와  
원인, 해결 방법, 배운 점을 정리한 문서다.

---

## 1. SSH 접속이 되지 않을 때

### 확인 항목
1. EC2 인스턴스가 Running 상태인지 확인
2. 퍼블릭 IP가 맞는지 확인
3. Security Group에서 22 포트 허용 여부 확인
4. `.pem` 키 파일 권한 확인
5. Ubuntu 인스턴스인 경우 사용자명이 `ubuntu`인지 확인

### 배운 점
SSH 접속 문제는 단순히 키 파일 문제만이 아니라  
인스턴스 상태, 네트워크, 보안그룹, 사용자명까지 함께 맞아야 해결된다.

---

## 2. 웹 페이지가 열리지 않을 때

### 확인 항목
1. EC2 인스턴스가 Running 상태인지 확인
2. `systemctl status nginx` 확인
3. `ss -tulpn | grep :80` 으로 80 포트 리스닝 확인
4. Security Group에서 80 포트 허용 여부 확인
5. 퍼블릭 IP가 현재 인스턴스와 일치하는지 확인
6. `journalctl -u nginx --no-pager | tail -n 30` 확인
7. `/var/log/nginx/access.log`, `/var/log/nginx/error.log` 확인

### 배운 점
웹 접속 실패는 nginx 서비스 문제일 수도 있고,  
포트, 보안그룹, 퍼블릭 IP 문제일 수도 있다.

---

## 3. Security Group 변경 후 접속이 안 될 때

### 증상
- nginx는 정상 실행 중인데 브라우저 접속 불가
- SSH 또는 HTTP가 갑자기 실패

### 원인
- 인바운드 규칙에서 필요한 포트가 제거됨
- 허용 범위가 현재 접속 IP와 맞지 않음

### 해결
- 80 포트 복구
- 22 포트는 필요 시 My IP 기준으로 허용
- 인바운드 규칙 재확인

### 배운 점
서비스 상태가 정상이어도 Security Group이 막으면 외부 접속은 실패한다.

---

## 4. CloudWatch 알람이 바로 오지 않을 때

### 증상
- 알람을 만들었는데 이메일이 오지 않음
- CPU를 올렸는데 상태 변화가 늦음

### 원인
- SNS 구독 승인을 하지 않음
- 기본 모니터링은 5분 단위라 반응이 느릴 수 있음
- CPU 부하가 충분하지 않음

### 해결
- SNS 메일에서 구독 승인
- CPU 부하 테스트를 더 명확히 수행
- 상태 변화는 몇 분 정도 기다림

### 배운 점
CloudWatch 알람은 생성 자체보다  
실제 상태 변화와 알림 수신까지 검증해야 의미가 있다.

---

## 5. ALB Target Group이 `initial` 상태에서 오래 머무를 때

### 증상
- Target Group 상태가 바로 healthy가 되지 않음

### 원인
- 헬스체크 성공 누적이 충분하지 않음
- nginx 응답 준비 전 상태일 수 있음

### 해결
- 몇 분 기다리며 health check 누적 확인
- `systemctl status nginx`, `curl localhost`로 서버 상태 확인
- Health check path와 응답 경로 일치 여부 확인

### 배운 점
ALB는 즉시 healthy로 바꾸는 것이 아니라  
연속 성공 조건을 충족해야 healthy로 전환한다.

---

## 6. ALB Health Check 실패처럼 보였지만 실제 원인은 다른 경우

### 증상
- Target Group에서 unhealthy 반복
- ASG 인스턴스가 생성 / 종료 반복

### 초기 추정
- ALB 설정 오류
- Target Group 문제
- Security Group 문제

### 실제 원인
- ASG 인스턴스에 public IP가 없어 외부 패키지 저장소에 접근할 수 없었음
- 그 결과 user data의 `apt update`, `apt install nginx`가 실패
- nginx가 구동되지 않아서 health check도 실패

### 해결
- 퍼블릭 서브넷에서 public IP 할당 가능하도록 수정
- 새 인스턴스가 정상적으로 nginx 설치 및 실행
- Target Group health check 통과

### 배운 점
헬스체크 실패는 ALB 문제처럼 보일 수 있지만,  
실제로는 앱 초기 구성 실패나 인터넷 egress 문제일 수도 있다.

---

## 7. Auto Scaling에서 Target Group 자동 등록이 헷갈릴 때

### 헷갈린 점
- 새 인스턴스를 Target Group에 직접 수동 등록해야 하는지 혼동

### 정리
- Auto Scaling Group에 Target Group을 연결하면
- ASG가 생성한 인스턴스는 자동으로 Target Group에 등록된다
- 수동 등록은 필요하지 않다

### 배운 점
ALB는 Target Group만 보고,  
ASG는 생성한 인스턴스를 해당 Target Group에 자동 연결한다.

---

## 8. CPU 부하 테스트에서 49~50% 근처에서 멈출 때

### 증상
- `yes > /dev/null` 실행 후 CPU가 약 49.9% 수준에서 멈춤

### 원인
- `yes` 프로세스 하나가 CPU 코어 1개만 계속 점유
- 2 vCPU 인스턴스에서는 전체 평균이 약 50% 수준으로 보일 수 있음

### 해결
- 같은 인스턴스에서 `yes`를 여러 개 실행
- 살아 있는 ASG 인스턴스 여러 대에 동시에 부하 발생

### 배운 점
Target tracking은 그룹 평균 CPU를 보기 때문에  
한 인스턴스에만 약한 부하를 주면 scale out 테스트가 잘 보이지 않을 수 있다.

---

## 9. Auto Scaling에서 scale out이 바로 안 보일 때

### 확인 항목
1. ASG의 Max capacity가 충분한지
2. target tracking 정책의 target value가 몇인지
3. 그룹 평균 CPU가 실제로 기준을 넘었는지
4. 새 인스턴스 warmup이나 health check 누적 시간이 필요한지

### 배운 점
오토스케일링은 한순간 지표만 보고 즉시 늘어나는 것이 아니라,  
정책 평가, 평균 지표, 준비 시간 등을 함께 고려해 동작한다.

---

## 10. 이번 프로젝트 전체에서 가장 크게 배운 점

- 장애처럼 보이는 현상과 실제 원인은 다를 수 있다.
- 접속 문제는 서비스, 포트, 보안그룹, 라우팅, 앱 기동, user data까지 함께 봐야 한다.
- AWS 인프라 실습은 “리소스를 만드는 것”보다 “어떻게 연결되고 왜 실패하는지 이해하는 것”이 더 중요하다.