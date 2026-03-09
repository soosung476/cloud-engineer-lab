# cloud-engineer-lab

클라우드 운영 / 시스템 엔지니어 취업을 목표로  
AWS, Linux, Network, Monitoring, Load Balancing 실습을 기록하는 저장소입니다.

이 저장소는 단순 이론 정리가 아니라,  
직접 구축하고 운영하고 장애를 확인하고 복구하는 과정을 기록하는 실습형 포트폴리오입니다.

---

## 1. 목표

- Linux 서버 운영 기본기 익히기
- AWS 인프라 구성 경험 축적
- 로그 확인 및 장애 점검 흐름 익히기
- 모니터링과 네트워크 구조 이해하기
- 장기적으로 DevOps / SRE 방향으로 확장하기

---

## 2. 현재까지 진행한 내용

- Mac(M1) 로컬 터미널 환경 세팅
- Git / GitHub 연동
- AWS 빌링 / 예산 알림 설정
- EC2 Ubuntu 인스턴스 생성
- SSH 접속 성공
- nginx 설치 및 실행
- systemctl stop/start/status 실습
- journalctl, access.log, error.log 확인
- ps, ss, df, free 명령어 실습
- 기본 nginx 페이지를 커스텀 HTML로 교체
- GitHub push 완료
- EC2 인스턴스 stop 완료

---

## 3. 실습 문서

### Linux / EC2
- [EC2 생성 및 초기 접속](docs/01-ec2-setup.md)
- [nginx 설치 및 서비스 제어](docs/02-nginx-setup.md)
- [Linux 운영 명령어 정리](docs/03-linux-commands.md)

### 앞으로 추가할 실습
- Security Group 점검 및 실습
- CloudWatch Alarm 생성
- VPC / Public Subnet 구성
- ALB + EC2 2대 구성
- Auto Scaling 기초
- IAM / 운영 보안 기초

---

## 4. 실습 환경

- Local: MacBook M1
- Cloud: AWS EC2 (Ubuntu)
- Web Server: nginx
- Version Control: Git / GitHub

---

## 5. 학습 방식

이 저장소는 이론 선행형 보다는  
직접 만들어보고, 문제를 만나고, 필요한 이론을 보완하는 방식 으로 진행합니다.

정리 방식은 아래 기준을 따릅니다.

- 무엇을 만들었는가
- 어떻게 설정했는가
- 어떤 문제가 발생할 수 있는가
- 어떻게 점검하고 복구하는가
- 운영 관점에서 왜 중요한가

---

## 6. 현재 학습 우선순위

1. Security Group
2. CloudWatch
3. VPC / Subnet
4. ALB
5. Auto Scaling

---

## 7. 트러블슈팅 관점에서 배우는 것

이 저장소에서는 단순 구축보다  
"접속이 안 될 때 어디를 확인해야 하는지" 를 중요하게 생각합니다.

예를 들어 웹 접속 장애가 발생하면 다음 순서로 점검합니다.

1. EC2 인스턴스 상태 확인
2. Security Group 인바운드 규칙 확인
3. nginx 서비스 상태 확인
4. 80 포트 리스닝 여부 확인
5. nginx 로그 확인
6. 파일 경로 및 권한 확인

---

## 8. 앞으로의 방향

단일 서버 운영 실습에서 시작해서,  
점차 아래 구조로 확장할 예정입니다.

- 서버 1대 운영
- 모니터링
- 네트워크 분리
- 로드밸런싱
- 오토스케일링
- 운영 자동화 / DevOps 확장

---

## 9. Progress

진행 로그는 아래 문서에 계속 기록합니다.

- [progress.md](progress.md)

---

## 10. Repository 목적

이 저장소의 목적은 “공부한 흔적”을 남기는 것뿐 아니라,  
클라우드 운영 / 시스템 엔지니어로서의 성장 과정을  
실습 중심으로 증명할 수 있는 포트폴리오를 만드는 것입니다.