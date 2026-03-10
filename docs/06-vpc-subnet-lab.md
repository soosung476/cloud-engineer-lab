# VPC / Public Subnet 구성 실습

## 1. 실습 목적

AWS 기본 VPC가 아닌, 직접 생성한 VPC 환경에서  
Public Subnet, Internet Gateway, Route Table을 구성하고  
그 위에 EC2 인스턴스를 배치하여 외부에서 SSH와 HTTP로 접속 가능한 구조를 만든다.
AWS 네트워크 기본 구성 요소들이 어떻게 연결되어야 실제로 인터넷 통신이 가능한지 이해하는 것이 최족 목표다.

- VPC와 Subnet의 관계
- Internet Gateway의 역할
- Route Table의 기본 동작
- Public Subnet이 성립하는 조건
- 직접 만든 네트워크 위에 EC2를 배치하고 외부 접속까지 검증하는 흐름
을 중점으로 학습했다.

---

## 2. 실습 환경

- Cloud: AWS
- Network:
  - VPC
  - Public Subnet
  - Internet Gateway
  - Route Table
- Compute: EC2 Ubuntu
- Web Server: nginx
- Access Method:
  - SSH
  - Browser HTTP access
- Local Environment: MacBook M1

---

## 3. 실습 목표

이번 실습에서 확인한 목표는 다음과 같다.

- 직접 VPC 생성
- 직접 Public Subnet 생성
- Internet Gateway를 VPC에 연결
- Route Table에 `0.0.0.0/0 -> IGW` 경로 추가
- Public Subnet에 Route Table 연결
- 해당 네트워크에 EC2 배치
- 퍼블릭 IP를 통해 SSH 접속 성공
- nginx 설치 후 HTTP 접속 성공

---

## 4. 네트워크 개념 정리

### VPC
VPC는 AWS 안에서 직접 만드는 **가상 사설 네트워크**다.  
EC2, Load Balancer, 데이터베이스 같은 리소스는 보통 이 VPC 안에 배치된다.
즉, AWS 안에서 내가 직접 설계하는 네트워크 공간이라고 볼 수 있다.

### Subnet
Subnet은 VPC를 더 작은 네트워크 구간으로 나눈 것이다 (Network ID를 쪼개는 것이다). 
리소스는 실제로 VPC가 아니라 Subnet 안에 배치된다.

### Public Subnet
Public Subnet은 인터넷과 통신 가능한 경로가 설정된 Subnet이다.  
단순히 이름이 `public`이라서 public이 되는 것은 아니며,  
다음 조건이 함께 필요하다.

- VPC에 Internet Gateway 연결
- Route Table에 `0.0.0.0/0 -> Internet Gateway` 경로 존재
- 해당 Route Table이 Subnet에 연결
- EC2에 퍼블릭 IP 할당

### Internet Gateway
Internet Gateway는 VPC가 인터넷과 통신할 수 있게 해주는 출입구 역할을 한다.

### Route Table
Route Table은 네트워크 트래픽이 어디로 가야 하는지 결정하는 규칙표다.  
이번 실습에서는 외부 인터넷으로 향하는 트래픽을 Internet Gateway로 보내는 경로를 추가했다.

---

## 5. 작업 과정

### 5-1. VPC 생성
먼저 직접 사용할 VPC를 생성했다.

- 새 VPC 생성
- VPC CIDR 지정

기존 AWS 기본 VPC가 아닌, 내가 직접 제어하는 네트워크 환경을 만들 수 있었다.

---

### 5-2. Public Subnet 생성
생성한 VPC 안에 Public Subnet을 만들었다.

- VPC 내부에 새 Subnet 생성
- Subnet CIDR 지정 (10.0.1.0 / 10.0.2.0 ALB 실습을 위해 두개를 만들어 놨다.)
- 사용할 가용 영역(하나는 2a, 하나는 2c) 선택

Subnet은 VPC 안에서 실제 리소스가 올라갈 네트워크 구간이므로,  
이후 EC2는 이 Subnet 안에 배치된다.

---

### 5-3. Internet Gateway 생성 및 VPC 연결
이후 Internet Gateway를 생성하고,  
방금 만든 VPC에 연결했다.

- VPC가 외부 인터넷과 연결될 수 있는 통로 확보
- Public Subnet 구성을 위한 필수 조건 충족

VPC만 만들어서는 인터넷과 통신할 수 없고,  
Internet Gateway가 연결되어 있어야 외부와 통신할 수 있다.

---

### 5-4. Route Table 생성 및 기본 경로 추가
다음으로 Route Table을 생성하고,  
외부 인터넷으로 향하는 기본 경로를 추가했다.

추가한 핵심 라우팅:
- `0.0.0.0/0 -> Internet Gateway`

이 의미는  
목적지가 내부 네트워크가 아닌 외부 인터넷이라면  
Internet Gateway를 통해 나가도록 하겠다는 뜻이다.

---

### 5-5. Route Table을 Public Subnet에 연결
생성한 Route Table을 Public Subnet에 연결했다.

Subnet이 public으로 동작하려면  
단순히 IGW가 있는 것만으로는 부족하고,  
해당 Subnet이 인터넷 경로가 들어 있는 Route Table을 실제로 사용해야 한다.

Public Subnet이 성립하려면:
- IGW 존재
- Route Table에 외부 경로 존재
- 그 Route Table이 해당 Subnet과 연결

이 세 가지가 함께 필요하다.

---

### 5-6. 직접 만든 Public Subnet에 EC2 배치
이후 EC2 인스턴스를 생성할 때  
기존 기본 VPC 대신 직접 만든 VPC와 Public Subnet을 선택했다.

이 단계에서 함께 확인한 것:
- 퍼블릭 IP 자동 할당
- Security Group에서 SSH(22), HTTP(80) 허용
- 기존 키 페어 사용

이번 실습의 핵심은  
**직접 설계한 네트워크 위에 EC2를 올리는 것**이었다.

---

### 5-7. 퍼블릭 IP를 통한 SSH 접속 확인
EC2 생성 후 퍼블릭 IP를 확인하고,  
기존 키 페어를 사용해 SSH 접속을 시도했다.

SSH 접속이 성공했다는 것은 아래 조건이 모두 맞았다는 뜻이다.

- EC2 인스턴스가 Running 상태
- 퍼블릭 IP가 정상 할당됨
- Security Group 22 허용
- Subnet의 인터넷 경로 정상 구성
- Internet Gateway 연결 정상
- 키 페어 및 사용자명 정상

---

### 5-8. nginx 설치 및 HTTP 접속 확인
SSH 접속 후 다음 작업을 수행했다.

- `sudo apt update`
- `sudo apt install nginx`
- `sudo systemctl status nginx`

이후 nginx가 `active` 상태인 것을 확인하고,  
브라우저에서 `http://퍼블릭IP`로 접속하여 웹 페이지가 정상적으로 열리는지 확인했다.

이 단계는 단순 웹 접속 확인이 아니라,  
아래 항목을 동시에 검증하는 의미가 있다.

- EC2가 Public Subnet에 정상 배치됨
- 80 포트가 외부에 열려 있음
- Internet Gateway와 Route Table 구성이 정상 동작함
- 웹 서버가 실제로 외부 요청을 받을 수 있음

---

## 6. 확인 결과

이번 실습을 통해 아래 내용을 확인했다.

- 직접 만든 VPC에 EC2를 배치할 수 있었다
- 직접 만든 Public Subnet을 사용할 수 있었다
- Internet Gateway를 VPC에 연결했다
- Route Table에 `0.0.0.0/0 -> IGW` 경로를 추가했다
- 퍼블릭 IP를 통해 SSH 접속에 성공했다
- nginx 설치 후 HTTP 접속에 성공했다

AWS 네트워크 기본 구성 요소를 직접 연결하여, 인터넷 통신이 가능한 Public 네트워크 구조를 스스로 구성했다.
---

## 7. 운영 관점에서 중요한 이유
이번 실습이 중요한 이유는

- 서버가 어떤 네트워크 위에 있는지 이해할 수 있다
- 외부 접속 가능 여부가 단순 인스턴스 상태만으로 결정되지 않는다는 점을 알 수 있다
- IGW, Route Table, Public IP, Security Group이 함께 맞아야 외부 접속이 가능하다는 점을 체감할 수 있다
- 이후 ALB, Private Subnet, NAT Gateway, Multi-AZ 구조를 이해하는 기반이 된다

이번 실습은 AWS 네트워크의 가장 기본적인 토대를 직접 만든 경험이라고 볼 수 있다.

---

## 8. 발생할 수 있는 문제

### EC2에 SSH 접속이 되지 않는 경우
가능한 원인:
- 퍼블릭 IP가 할당되지 않음
- Security Group에서 22 포트 미허용
- Internet Gateway 미연결
- Route Table에 외부 경로 미설정
- Route Table이 Subnet에 연결되지 않음
- 키 파일 또는 사용자명 오류

### 브라우저에서 웹 접속이 되지 않는 경우
가능한 원인:
- Security Group에서 80 포트 미허용
- nginx 미설치 또는 미실행
- 퍼블릭 IP 오류
- Public Subnet 구성이 불완전함

### Subnet 이름은 public인데 실제로는 외부 접속이 안 되는 경우
가능한 원인:
- 이름만 public으로 지정했을 뿐
- IGW / Route Table / 퍼블릭 IP 조건이 충족되지 않음

---

## 9. 트러블슈팅 체크리스트

### SSH 접속이 되지 않을 때
1. EC2 인스턴스가 Running 상태인지 확인
2. 퍼블릭 IP가 할당되었는지 확인
3. Security Group에서 22 포트가 허용되었는지 확인
4. Internet Gateway가 VPC에 연결되어 있는지 확인
5. Route Table에 `0.0.0.0/0 -> IGW` 경로가 있는지 확인
6. 해당 Route Table이 Subnet에 연결되어 있는지 확인
7. 키 파일 권한과 사용자명이 올바른지 확인

### HTTP 접속이 되지 않을 때
1. EC2 인스턴스가 Running 상태인지 확인
2. Security Group에서 80 포트가 허용되었는지 확인
3. `systemctl status nginx`로 nginx 상태 확인
4. `ss -tulpn | grep :80` 으로 80 포트 리스닝 확인
5. 퍼블릭 IP가 맞는지 확인
6. Route Table과 IGW 구성이 정상인지 확인

---

## 10. 배운 점

- VPC는 AWS에서 직접 설계하는 네트워크 공간이다.
- Subnet은 실제 리소스가 배치되는 네트워크 구간이다.
- VPC만 만들어서는 인터넷 통신이 되지 않는다.
- Public Subnet은 이름이 아니라 IGW, Route Table, 퍼블릭 IP 조건이 충족되어야 성립한다.
- 외부 접속 가능 여부는 EC2 상태뿐 아니라 네트워크 구조와 보안 설정까지 함께 맞아야 한다.
- AWS 네트워크는 리소스를 **어떻게 연결되는지 이해하는 것**이 중요하다.

---

## 11. 다음 단계

이후에는 아래 실습으로 확장할 예정이다.

- Public Subnet 추가 구성
- EC2 2대 구성
- Application Load Balancer(ALB) 생성
- Target Group 연결
- Health Check 확인
- 다중 서버 로드밸런싱 구조 실습