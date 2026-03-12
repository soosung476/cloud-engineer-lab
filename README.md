# cloud-engineer-lab

클라우드 운영 / 시스템 엔지니어 취업을 목표로  
AWS, Linux, Network, Monitoring, Load Balancing, Auto Scaling 실습을 기록하는 저장소입니다.

이 저장소는 단순 이론 정리가 아니라,  
직접 구축하고 운영하고 문제를 확인하고 복구하는 과정을 기록하는 실습형 포트폴리오입니다.

---

## 1. 목표

- Linux 서버 운영 기본기 익히기
- AWS 인프라 구성 경험 축적
- 로그 확인 및 장애 점검 흐름 익히기
- 모니터링과 네트워크 구조 이해하기
- 가용성과 자동 확장 구조 경험하기
- 장기적으로 DevOps / SRE 방향으로 확장하기

---

## 2. 현재까지 진행한 실습

### Linux / EC2 기초
- EC2 Ubuntu 인스턴스 생성
- SSH 접속 성공
- nginx 설치 및 실행
- systemctl / journalctl / access.log / error.log 실습
- ps / ss / df / free 명령어 실습
- 기본 nginx 페이지를 직접 만든 HTML로 교체

### 보안 / 운영
- Security Group 인바운드 규칙 점검
- SSH / HTTP 접근 제어 실습
- CloudWatch Alarm 생성
- CPU 기준 알람 및 상태 변화 확인

### 네트워크
- 직접 VPC 생성
- Public Subnet 구성
- Internet Gateway 연결
- Route Table에 `0.0.0.0/0 -> IGW` 설정
- 직접 만든 네트워크 위에 EC2 배치 및 SSH / HTTP 접속 확인

### 가용성 / 확장성
- Application Load Balancer(ALB) 구성
- Target Group 생성 및 Health Check 확인
- 서로 다른 웹 서버 2대로 트래픽 분산 확인
- nginx 중지 후 unhealthy 전환 및 자동 제외 확인
- Auto Scaling Group 구성
- Launch Template + User data 기반 자동 구성
- Target tracking 정책으로 CPU 부하 시 인스턴스 수 증가 확인

---

## 3. 실습 문서

### 기초
- [EC2 생성 및 초기 접속](docs/01-ec2-setup.md)
- [nginx 설치 및 서비스 제어](docs/02-nginx-setup.md)
- [Linux 운영 명령어 정리](docs/03-linux-commands.md)

### 운영 / 네트워크 / 가용성
- [Security Group 점검 및 실습](docs/04-security-group-lab.md)
- [CloudWatch Alarm 생성](docs/05-cloudwatch-alarm.md)
- [VPC / Public Subnet 구성](docs/06-vpc-subnet-lab.md)
- [Application Load Balancer 구성](docs/07-alb-lab.md)
- [Auto Scaling Group 구성](docs/09-auto-scaling-lab.md)

### 트러블슈팅
- [Troubleshooting Notes](troubleshooting.md)

---

## 4. 실습 환경

- Local: MacBook M1
- Cloud: AWS
- OS: Ubuntu
- Web Server: nginx
- Monitoring: CloudWatch
- Load Balancing: ALB
- Scaling: Auto Scaling Group
- Version Control: Git / GitHub

---

## 5. 학습 방식

이 저장소는 이론을 먼저 길게 정리하기보다,  
직접 구성하고 문제를 겪은 뒤 필요한 개념을 보완하는 방식으로 진행합니다.

정리 기준은 아래와 같습니다.

- 무엇을 만들었는가
- 어떻게 구성했는가
- 어떤 문제가 발생했는가
- 어떻게 점검하고 해결했는가
- 운영 관점에서 왜 중요한가

---

## 6. 이번 프로젝트에서 구현한 아키텍처

- VPC 1개
- Public Subnet 2개
- Internet Gateway
- Route Table
- EC2 기반 nginx 웹 서버
- Application Load Balancer
- Target Group + Health Check
- Auto Scaling Group
- Launch Template + User data
- CloudWatch Alarm

---

## 7. 이번 프로젝트에서 배운 핵심

- 웹 접속 문제는 서비스 상태, 포트, 보안그룹, 라우팅, 로그를 함께 확인해야 한다.
- Public Subnet은 이름이 아니라 IGW + Route Table + Public IP 조건으로 동작한다.
- ALB는 단순 분산기가 아니라 상태 기반으로 트래픽을 제어한다.
- Auto Scaling은 단순히 인스턴스를 늘리는 기능이 아니라, Launch Template과 User data까지 포함한 자동 구성 체계다.
- Health check 실패는 ALB 문제만이 아니라 애플리케이션 기동 실패나 인터넷 egress 문제일 수도 있다.

---

## 8. 다음 확장 방향

이번 저장소의 1차 목표는 완료되었고, 이후 아래 방향으로 확장할 수 있습니다.

- HTTPS + ACM
- Route 53
- Private Subnet + NAT Gateway
- RDS
- SSM 기반 운영
- CI/CD
- Terraform

---

## 9. Progress

- [progress.md](progress.md)
- 실습 리소스 정리 완료
- 현재는 문서화 회고 단계

---

## 10. Repository 목적

이 저장소의 목적은 공부한 흔적을 남기는 것뿐 아니라,  
클라우드 운영 / 시스템 엔지니어로 성장하는 과정을  
실습 중심으로 증명할 수 있는 포트폴리오를 만드는 것입니다.