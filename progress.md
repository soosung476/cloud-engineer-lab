# Progress

## 2026-03-09
### 완료
- AWS 빌링 / 예산 알림 설정
- EC2 Ubuntu 인스턴스 생성
- SSH 접속 성공
- nginx 설치 및 실행
- 기본 페이지를 커스텀 HTML로 교체
- systemctl / journalctl / access.log / error.log 확인
- ps / ss / df / free 명령어 실습
- GitHub push
- EC2 stop

### 배운 점
- 웹 접속 문제는 서비스 상태, 포트, 보안그룹, 로그를 함께 확인해야 한다.
- Linux 명령어는 따로 외우는 것보다 실제 상황과 함께 익혀야 기억에 남는다.

---

## 2026-03-10
### 완료
- Security Group 인바운드 규칙 점검
- 80 포트 제거 / 복구 실습
- SSH 22 포트 접근 범위 점검
- Security Group이 접속 가능 여부에 직접 영향을 준다는 점 확인

### 배운 점
- 서비스가 살아 있어도 보안그룹이 막으면 접속할 수 없다.
- Security Group은 AWS 환경에서 가장 기본적인 네트워크 보안 장치다.

---

## 2026-03-11
### 완료
- CloudWatch에서 EC2 메트릭 확인
- CPUUtilization Alarm 생성
- SNS 이메일 알림 연결
- CPU 부하 테스트 후 Alarm 상태 변화 확인

### 배운 점
- 모니터링은 단순 그래프 확인이 아니라 이상 상황을 자동 감지하는 체계다.
- 알람은 생성보다 상태 변화 검증까지 해야 실습이 완성된다.

---

## 2026-03-11
### 완료
- 직접 VPC 생성
- Public Subnet 구성
- Internet Gateway 연결
- Route Table에 `0.0.0.0/0 -> IGW` 설정
- 직접 만든 네트워크에 EC2 배치
- SSH / HTTP 접속 성공

### 배운 점
- VPC만 만들어서는 인터넷 통신이 되지 않는다.
- Public Subnet은 이름이 아니라 IGW, Route Table, Public IP 조건으로 성립한다.

---

## 2026-03-12
### 완료
- ALB 구성
- Target Group 생성 및 Health Check 확인
- Public Subnet 2개 기반 다중 서버 구조 구성
- 서로 다른 웹 서버 2대로 트래픽 분산 확인
- nginx 중지 후 unhealthy 전환 및 자동 제외 확인
- nginx 재시작 후 healthy 복귀 확인

### 배운 점
- ALB는 단순 분산 장치가 아니라 상태 기반 트래픽 제어 장치다.
- Target Group과 Health Check가 가용성 유지의 핵심이다.

---

## 2026-03-12
### 완료
- Launch Template 생성
- User data 기반 nginx 자동 설치 구성
- Auto Scaling Group 생성
- Target tracking 정책 설정
- CPU 부하 테스트로 scale out 확인
- 최대 4대까지 인스턴스 증가 확인

### 트러블슈팅
- 초기에는 Target Group unhealthy가 반복 발생
- 원인은 public IP 부재로 user data의 `apt update`, `apt install nginx`가 실패한 것이었다
- public IP 할당 후 nginx 자동 설치와 health check가 정상 동작했다

### 배운 점
- Auto Scaling은 인스턴스 수 조절보다 Launch Template와 User data 설계가 더 중요하다.
- 헬스체크 실패는 ALB 문제처럼 보여도 실제로는 애플리케이션 초기 구성 실패일 수 있다.

---

## 현재 상태
- cloud-lab 관련 인스턴스 및 실습 리소스 삭제 완료
- 현재 프로젝트 1차 실습 마무리 상태

## 다음 단계
- 실습 문서 보완
- 아키텍처 다이어그램 정리
- 다음 확장 주제 검토
  - HTTPS + ACM
  - Route 53
  - Private Subnet + NAT Gateway
  - Terraform