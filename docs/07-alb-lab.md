 # Application Load Balancer(ALB) 구성 실습

## 1. 실습 목적

단일 EC2 인스턴스 기반 웹 서비스 구조의 한계를 보완하기 위해  
Application Load Balancer(ALB)와 Target Group을 구성하고,  
2대의 웹 서버로 트래픽을 분산하는 구조를 만든다.

이번 실습의 목적은  서버 여러 대를 하나의 진입점 뒤에 배치하고, Health Check를 통해 비정상 서버를 자동으로 제외하는 구조를 이해하는 것 이다.

특히 아래 항목을 중점적으로 확인했다.

- ALB의 역할
- Listener와 Target Group의 관계
- Health Check를 통한 서버 상태 판별
- 2대의 웹 서버로 요청 분산
- 비정상 서버 자동 제외 및 복구 확인

---

## 2. 실습 환경

- Cloud: AWS
- Network:
  - VPC 1개
  - Public Subnet 2개
  - Internet Gateway
  - Route Table
- Compute:
  - EC2 Ubuntu 2대
- Web Server:
  - nginx
- Load Balancing:
  - Application Load Balancer
  - Target Group
- Access Method:
  - Browser HTTP access
  - SSH
- Local Environment:
  - MacBook M1

---

## 3. 실습 목표

이번 실습에서 확인한 목표는 다음과 같다.

- 서로 다른 Public Subnet에 EC2 2대 배치
- 각 서버에 nginx 설치
- 각 서버에 서로 다른 HTML 페이지 배포
- Target Group 생성 및 인스턴스 등록
- ALB 생성 및 Listener 연결
- ALB DNS로 웹 접속 확인
- Health Check를 통한 Healthy / Unhealthy 상태 변화 확인
- nginx 중지 후 비정상 서버 자동 제외 확인
- nginx 재시작 후 정상 상태 복귀 확인

---

## 4. 개념 정리

### ALB (Application Load Balancer)
ALB는 사용자의 요청을 받아 여러 서버에 분산해주는 로드밸런서다.  
클라이언트는 개별 EC2에 직접 접속하는 대신,  
ALB의 DNS 이름으로 접속하게 된다.

즉, ALB는 여러 웹 서버 앞단의 **공통 진입점** 역할을 한다.

---

### Listener
Listener는 ALB가 어떤 포트와 프로토콜로 요청을 받을지 정의하는 설정이다.

예를 들어:
- HTTP : 80

이번 실습에서는 ALB가 80 포트의 HTTP 요청을 받고,  
이를 Target Group으로 전달하도록 구성했다.

---

### Target Group
Target Group은 ALB가 요청을 전달할 실제 대상 서버들의 묶음이다.

이번 실습에서는 EC2 2대를 하나의 Target Group에 등록했다.  
ALB는 요청을 받을 때 이 Target Group 안의 인스턴스들 중  
정상 상태인 서버에게만 요청을 전달한다.

---

### Health Check
Health Check는 ALB가 각 서버가 정상적으로 응답 가능한지 주기적으로 검사하는 기능이다.

이번 실습에서는 ALB가 지정된 경로에 HTTP 요청을 보내고,  
응답 결과를 바탕으로 각 인스턴스를 Healthy 또는 Unhealthy로 판단하는 흐름을 확인했다.

ALB는 단순히 요청을 분산하는 것뿐 아니라  
비정상 서버를 자동으로 제외하는 역할도 수행한다.

---

## 5. 작업 과정

### 5-1. Public Subnet 2개 준비
기존에 구성한 VPC 안에 Public Subnet 2개를 준비했다.

이번 실습에서는 ALB를 구성하기 위해  
서로 다른 가용 영역(AZ)의 Public Subnet 2개가 필요했다.

이 단계에서 확인한 것:
- 같은 VPC 내부인지
- 서로 다른 AZ에 위치하는지
- Internet Gateway와 Route Table이 정상 연결되어 있는지

---

### 5-2. EC2 인스턴스 2대 생성
각 Public Subnet에 EC2 인스턴스를 1대씩 생성했다.

이 단계에서 확인한 것:
- 퍼블릭 IP 할당 여부
- Security Group 설정
- 기존 키 페어 사용
- SSH 접속 가능 여부

즉, 동일한 네트워크 구조 안에  
서로 다른 위치의 웹 서버 2대를 준비한 상태를 만들었다.

---

### 5-3. nginx 설치 및 서로 다른 HTML 배포
각 EC2 인스턴스에 nginx를 설치하고 실행 상태를 확인했다.

이후 각 서버에 서로 다른 HTML 내용을 배포했다.

예시:
- 서버 1: `Hello from web-1`
- 서버 2: `Hello from web-2`

이렇게 구성한 이유는  
ALB를 통해 접속했을 때 어느 서버가 응답했는지 쉽게 확인하기 위해서다.

---

### 5-4. Target Group 생성
EC2 2대를 묶는 Target Group을 생성했다.

설정한 주요 항목:
- Target type: Instances
- Protocol: HTTP
- Port: 80
- VPC: 직접 만든 VPC
- Health Check Path: `/`

이후 두 인스턴스를 Target Group에 등록했다.

이 과정에서  
ALB가 단순히 EC2 전체를 대상으로 삼는 것이 아니라,  
**Target Group이라는 단위로 요청을 전달하고 상태를 관리한다는 점**을 이해했다.

---

### 5-5. ALB 생성
Application Load Balancer를 생성했다.

주요 구성:
- Internet-facing ALB
- HTTP 80 Listener
- 같은 VPC 선택
- 서로 다른 AZ의 Public Subnet 2개 선택
- ALB용 Security Group 연결

이 단계의 의미는  
외부 사용자가 직접 EC2에 접속하지 않고,  
ALB를 통해 서버들에 접근하는 구조를 만드는 것이다.

---

### 5-6. Listener와 Target Group 연결
ALB Listener의 기본 동작을  
앞서 만든 Target Group으로 연결했다.

즉, 사용자가 ALB의 80 포트로 접속하면  
ALB가 해당 요청을 Target Group의 인스턴스들로 분산하도록 구성했다.

---

### 5-7. Target Health 상태 확인
ALB 생성 후 Target Group에서 각 인스턴스의 Health 상태를 확인했다.

처음에는 `initial` 상태였지만,  
설정된 Health Check가 누적 성공하면서 `healthy` 상태로 전환되는 것을 확인했다.

이 과정에서  
ALB는 단순히 서버가 켜져 있는지만 보는 것이 아니라,  
지정된 경로에 정상 HTTP 응답이 가능한지를 기준으로 상태를 판단한다는 점을 이해했다.

---

### 5-8. ALB DNS로 접속 및 트래픽 분산 확인
ALB의 DNS 이름으로 브라우저 접속을 시도했다.

이후 새로고침을 반복하면서  
서로 다른 HTML 문구가 번갈아 보이는 것을 확인했다.

이를 통해:
- ALB가 정상 동작하고 있고
- Target Group의 인스턴스들로 요청이 전달되며
- 실제로 트래픽이 분산되고 있다는 점을 확인했다.

---

### 5-9. nginx 중지 후 Unhealthy 상태 확인
이후 한 서버에서 nginx를 중지했다.
그 후 Target Group에서 해당 인스턴스가
healthy에서 unhealthy 상태로 바뀌는 것을 확인했다.

이 과정을 통해:
ALB가 주기적으로 Health Check를 수행하고
비정상 서버를 자동 감지하며
해당 서버를 트래픽 대상에서 제외한다는 점을 확인했다.

---

### 5-10. nginx 재시작 후 Healthy 상태 확인
중지했던 nginx를 다시 시작했다.
이후 일정 시간이 지나자
해당 인스턴스가 다시 healthy 상태로 복귀하는 것을 확인했다.

즉,
ALB는 서버가 비정상일 때만 제외하는 것이 아니라,
정상 복구 후에는 다시 트래픽 대상에 포함시킨다는 점도 확인할 수 있었다.

---

## 6. 확인 결과
이번 실습을 통해 아래 내용을 확인했다.

- 직접 만든 VPC와 Public Subnet 2개를 기반으로 ALB를 구성할 수 있었다
- EC2 2대를 Target Group에 등록할 수 있었다 
- ALB DNS를 통해 단일 진입점으로 웹 접속이 가능했다
- 두 서버에 대한 트래픽 분산을 확인했다
- Health Check를 통해 서버 상태가 `initial -> healthy`로 전환되는 것을 확인했다
- nginx 중지 후 unhealthy 상태 전환을 확인했다
- nginx 재시작 후 다시 healthy 상태로 복귀하는 것을 확인했다
즉,
로드밸런싱과 상태 기반 트래픽 제어가 실제로 동작하는 구조를 직접 구성하고 검증했다.

---


## 7. 운영 관점에서 중요한 이유
클라우드 운영 / 시스템 엔지니어 관점에서
단일 서버 구조는 서버 장애 발생 시 서비스가 즉시 중단될 수 있다는 한계가 있다.

ALB를 사용하는 구조는 아래 측면에서 중요하다.
- 여러 서버에 요청을 분산할 수 있다
- 특정 서버 장애가 전체 서비스 중단으로 이어지는 것을 줄일 수 있다
- 비정상 서버를 자동으로 트래픽 대상에서 제외할 수 있다
- 복구된 서버를 다시 자동으로 서비스 대상에 포함할 수 있다
- 이후 Auto Scaling과 결합해 확장 가능한 구조로 발전시킬 수 있다

즉,
이번 실습은 단순 웹 서버 운영에서 한 단계 나아가
가용성을 고려한 운영 구조를 직접 만들어본 경험이라고 볼 수 있다.

---

## 8. 발생할 수 있는 문제

### Target이 initial 상태에서 오래 머무는 경우
가능한 원인:

- Health Check가 아직 충분히 누적되지 않음
- nginx가 정상 응답하지 않음
- Health Check path가 실제 응답 경로와 다름
- Security Group 설정 문제

### Target이 unhealthy 상태가 되는 경우
가능한 원인:

- nginx 미실행
- 80 포트 미오픈
- EC2 보안그룹 인바운드 설정 문제
- ALB 보안그룹 설정 문제
- Health Check path 응답 실패
- timeout 또는 응답 코드 불일치

### ALB DNS로 접속되지 않는 경우

가능한 원인:
- ALB 보안그룹에서 80 포트 미허용
- Listener 미설정
- Target Group 연결 오류
- 등록된 타겟이 모두 unhealthy 상태

---

## 9. 트러블슈팅

### Target이 healthy가 되지 않을 때

1. systemctl status nginx로 nginx 상태 확인
2. ss -tulpn | grep :80 으로 80 포트 리스닝 확인
3. Target Group Health Check path 확인
4. EC2 보안그룹 인바운드 80 설정 확인
5. ALB 보안그룹 인바운드 / 아웃바운드 설정 확인
6. Target Health의 상태 사유 확인

### ALB DNS로 접속이 안 될 때

1. ALB가 active 상태인지 확인
2. Listener가 HTTP:80 으로 설정되어 있는지 확인
3. ALB 보안그룹에서 80 포트가 허용되는지 확인
4. 타겟 인스턴스가 healthy 상태인지 확인
5. EC2의 nginx가 정상 실행 중인지 확인

---

## 10. 배운 점

- ALB는 단순히 요청을 나누는 장치가 아니라, 서버 상태를 기반으로 트래픽을 제어하는 장치다.
- Target Group은 ALB가 요청을 전달할 실제 서버 묶음이며, Health Check도 이 단위로 관리된다.
- Health Check는 지정된 경로에 대한 HTTP 응답을 기준으로 Healthy / Unhealthy 상태를 판단한다.
- 보안그룹 설정은 로드밸런싱 구조가 정상 동작하는 데 매우 중요한 요소다.
- nginx 중지 및 재시작 테스트를 통해 비정상 서버 자동 제외와 복구 후 재편입 흐름을 직접 확인할 수 있었다.