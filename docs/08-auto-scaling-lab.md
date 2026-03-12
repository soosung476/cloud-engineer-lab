# Auto Scaling Group 구성 실습

## 1. 실습 목적

Application Load Balancer 뒤에서 동작하는 웹 서버 수를  
부하에 따라 자동으로 늘리고 줄일 수 있도록  
Launch Template, Auto Scaling Group, Target Tracking 정책을 구성한다.

이번 실습의 목적은 서버 구성 자체를 템플릿화하고, 부하에 따라 인프라가 자동으로 확장되는 구조를 이해하는 것이다.

특히 아래 항목을 중점적으로 확인했다.

- Launch Template의 역할
- User data를 통한 초기 서버 자동 구성
- Auto Scaling Group의 동작 방식
- ALB / Target Group과의 자동 연동
- CPU 부하 기준 자동 확장(scale out) 확인
- 초기 unhealthy 문제의 원인 분석 및 해결

---

## 2. 실습 환경

- Cloud: AWS
- Network:
  - VPC 1개
  - Public Subnet 2개
  - Internet Gateway
  - Route Table
- Compute:
  - EC2 Auto Scaling Group
  - Launch Template
- Web:
  - nginx
  - Application Load Balancer
  - Target Group
- Monitoring:
  - CloudWatch
- Local:
  - MacBook M1

---

## 3. 실습 목표

- Launch Template 생성
- User data를 이용한 nginx 자동 설치 구성
- Auto Scaling Group 생성
- ALB Target Group과 ASG 연결
- Target tracking 정책 설정
- CPU 부하 테스트를 통한 scale out 확인
- 초기 health check 실패 원인 분석 및 해결

---

## 4. 개념 정리

### Launch Template
Launch Template는 Auto Scaling Group이 새 인스턴스를 생성할 때 사용하는 설계도다.

여기에는 아래 같은 정보가 들어간다.

- AMI
- 인스턴스 타입
- 키 페어
- 보안그룹
- IAM Role
- User data

즉, 사람이 수동으로 EC2를 생성할 때 입력하던 항목을  
자동 생성에 맞게 템플릿화한 것이다.

---

### User data
User data는 인스턴스가 처음 부팅될 때 자동 실행되는 초기 설정 스크립트다.

이번 실습에서는 user data를 이용해 아래 작업을 자동화했다.

- 패키지 업데이트
- nginx 설치
- 기본 HTML 파일 생성
- nginx 서비스 실행

즉, Auto Scaling으로 새 인스턴스가 생성되더라도  
운영자가 직접 SSH로 들어가서 수동 설치하지 않아도  
곧바로 웹 서버 역할을 수행하도록 구성했다.

---

### Auto Scaling Group
Auto Scaling Group은 지정한 최소/기본/최대 인스턴스 수를 유지하면서,  
정책에 따라 인스턴스를 자동으로 늘리거나 줄이는 기능이다.

이번 실습에서는:

- Min: 2
- Desired: 2
- Max: 4

형태로 시작해, CPU 부하 발생 시 최대 4대까지 확장되는 흐름을 확인했다.

---

### Target Tracking Policy
Target Tracking은 특정 지표를 목표값 근처로 유지하도록  
Auto Scaling Group이 자동으로 확장 / 축소하도록 하는 정책이다.

이번 실습에서는 평균 CPU 사용률을 기준으로  
목표값을 초과하면 인스턴스 수를 늘리도록 설정했다.

---

## 5. 작업 과정

### 5-1. Launch Template 생성
먼저 Auto Scaling용 Launch Template를 생성했다.

설정한 주요 항목:
- Ubuntu AMI
- 기존 키 페어
- EC2용 Security Group
- User data
- 이후 디버깅을 위한 IAM Role 고려

이 단계의 핵심은  
**앞으로 생성될 모든 인스턴스가 동일한 초기 구성을 갖도록 만드는 것**이었다.

---

### 5-2. User data 작성
User data에는 nginx 설치와 웹 페이지 생성을 자동화하는 스크립트를 넣었다.

예를 들어 다음과 같은 흐름으로 구성했다.

- `apt update`
- `apt install nginx`
- 인스턴스 ID를 포함한 HTML 생성
- nginx enable / start

이 설정을 통해  
ASG가 새 인스턴스를 만들면 자동으로 웹 서버 준비가 끝나도록 했다.

---

### 5-3. Auto Scaling용 Target Group 생성
기존 ALB 실습에서 사용하던 수동 EC2용 Target Group과 분리하기 위해  
새 Target Group을 생성했다.

설정한 주요 항목:
- Target type: Instances
- Protocol: HTTP
- Port: 80
- Health check path: `/`

이 단계에서 기존 수동 실습용 서버와  
ASG가 자동 생성하는 서버를 분리한 덕분에  
오토스케일링 동작을 더 명확히 확인할 수 있었다.

---

### 5-4. Auto Scaling Group 생성
Launch Template를 기반으로 Auto Scaling Group을 생성했다.

설정한 주요 항목:
- Public Subnet 2개 선택
- 새 Target Group 연결
- Min / Desired / Max 용량 설정
- ELB health check 사용
- target tracking 정책 추가

이 구성으로 인해  
ASG가 생성한 인스턴스는 Target Group에 자동 등록되고,  
ALB Health Check 기준으로 상태를 판단받도록 만들었다.

---

### 5-5. ALB Listener를 새 Target Group으로 전환
기존 수동 EC2용 Target Group이 아니라,  
새 ASG용 Target Group으로 ALB Listener의 기본 전달 대상을 변경했다.

이 단계 이후부터는 브라우저에서 ALB DNS로 접속했을 때  
ASG가 생성한 인스턴스들이 실제 서비스 대상이 되도록 구성했다.

---

## 6. 발생한 문제와 원인 분석

### 6-1. Target Group에서 unhealthy 반복 발생
초기에는 ASG가 인스턴스를 생성하더라도  
Target Group에서 unhealthy 상태가 반복 발생했고,  
결과적으로 인스턴스가 terminate되고 다시 생성되는 루프가 발생했다.

초기에는 ALB health check 문제처럼 보였지만,  
실제 원인은 더 앞 단계에 있었다.

---

### 6-2. 원인: Public IP 부재로 user data의 패키지 설치 실패
ASG 인스턴스는 퍼블릭 서브넷에 생성되었지만  
초기에는 public IP가 자동으로 할당되지 않았다.

그 상태에서는 user data 안의 아래 작업이 정상 수행되지 않았다.

- `apt update`
- `apt install nginx`

즉, 인스턴스가 외부 패키지 저장소에 접근하지 못해  
nginx 설치가 실패했고, 결과적으로 웹 서버가 정상 구동되지 않았다.

그 상태에서 ALB가 `/` 경로로 health check를 보내도  
정상 응답을 받을 수 없었기 때문에  
Target Group에서 unhealthy 상태가 발생한 것이다.

---

### 6-3. 해결: Public IP 할당 후 healthy 전환
퍼블릭 서브넷에 public IP 자동 할당이 가능하도록 수정한 뒤  
새로 생성된 인스턴스는 외부 인터넷에 접근할 수 있게 되었다.

그 결과:
- `apt update` 성공
- `apt install nginx` 성공
- nginx 정상 실행
- ALB health check 성공
- Target Group healthy 전환

이 과정을 통해  
**ALB health check 실패처럼 보이던 문제가 실제로는 user data + 인터넷 egress 문제였음**을 확인할 수 있었다.

---

## 7. 스케일링 테스트

### 7-1. CPU 부하 발생
Auto Scaling Group의 target tracking 정책이  
실제로 동작하는지 확인하기 위해  
ASG 인스턴스들에 CPU 부하를 발생시켰다.

실습에서는 `yes > /dev/null` 명령을 사용해 CPU 사용률을 높였다.

처음에는 1개 프로세스만 실행했을 때 약 50% 수준에서 머무는 현상을 확인했는데,  
이는 인스턴스의 vCPU 개수와 관련된 정상 동작이었다.

이후 각 인스턴스에서 부하를 더 높여  
그룹 평균 CPU 사용률이 target value를 초과하도록 만들었다.

---

### 7-2. Scale out 확인
부하가 일정 시간 유지된 뒤  
Auto Scaling Group이 추가 인스턴스를 생성하는 것을 확인했다.

확인한 내용:
- Desired capacity 증가
- 인스턴스 수 증가
- Target Group에 자동 등록
- Health check 통과 후 healthy 상태 전환

최종적으로 인스턴스 수가 4대까지 증가하는 것을 확인했고,  
이를 통해 target tracking 정책이 실제로 동작함을 검증했다.

---

## 8. 확인 결과

이번 실습을 통해 아래 내용을 확인했다.

- Launch Template를 통해 인스턴스 구성을 템플릿화할 수 있었다
- User data로 nginx 설치 및 초기 구성을 자동화할 수 있었다
- Auto Scaling Group과 Target Group을 연결하면 인스턴스가 자동 등록된다
- ALB Listener를 새 Target Group으로 전환해 자동 생성 인스턴스를 실제 서비스에 연결할 수 있었다
- 초기 unhealthy 문제의 원인이 user data + 인터넷 egress 문제였음을 확인했다
- CPU 부하 테스트를 통해 인스턴스 수가 4대까지 scale out 되는 것을 확인했다

즉,  
**ALB 뒤에서 웹 서버 수가 자동으로 확장되는 기본적인 고가용성 구조를 직접 구성하고 검증했다.**

---

## 9. 운영 관점에서 중요한 이유

이번 실습은 단순히 서버 수를 늘리는 기능을 확인하는 것이 아니라,  
운영 환경에서 왜 자동 확장이 중요한지를 이해하는 과정이었다.

중요한 이유는 다음과 같다.

- 사용량 증가 시 수동 대응 없이 서버 수를 늘릴 수 있다
- Launch Template를 통해 동일한 서버 구성을 반복 재현할 수 있다
- ALB와 결합하면 새 서버가 자동으로 서비스 대상에 편입된다
- 초기 기동 실패 문제를 통해 user data와 네트워크 egress의 중요성을 확인할 수 있다
- 단순 웹 서버 운영에서 자동화된 운영 구조로 한 단계 확장할 수 있다

---

## 10. 트러블슈팅

### Target Group에서 unhealthy가 반복될 때
1. Target Group의 대상 상태 확인
2. ALB health check path와 응답 경로 일치 여부 확인
3. EC2용 보안그룹 인바운드 80 설정 확인
4. User data에서 nginx 설치 / 실행이 정상 수행되는지 확인
5. 인스턴스가 외부 패키지 저장소에 접근 가능한지 확인
6. ASG의 Health check grace period 확인

---

### public IP가 없는데 health check가 실패한 이유
ALB health check 자체는 private 경로로도 가능하다.  
하지만 이번 실습에서는 인스턴스가 먼저 user data 단계에서  
nginx를 설치해야 했기 때문에 외부 인터넷 egress가 필요했다.

즉,  
health check 실패의 직접 원인은 public IP 자체가 아니라  
**public IP 부재로 인해 nginx 설치가 실패한 것**이었다.

---

## 11. 배운 점

- Auto Scaling은 인스턴스를 단순히 더 만드는 기능이 아니라, 서버 구성을 템플릿화하고 자동 운영하는 구조다.
- Launch Template와 User data가 제대로 구성되지 않으면 scale out 이후에도 정상 서비스가 불가능하다.
- Target Group 자동 등록은 ASG와 TG 연결로 이루어지며, 수동 등록이 필요하지 않다.
- Public Subnet이라도 public IP나 NAT 같은 egress 경로가 없으면 user data의 패키지 설치가 실패할 수 있다.
- 헬스체크 실패는 ALB 문제처럼 보여도 실제 원인은 애플리케이션 기동 문제일 수 있다.

---

## 12. 다음 단계

이번 실습을 통해 현재 프로젝트의 1차 목표는 완료되었고,  
이후에는 아래 방향으로 확장할 수 있다.

- HTTPS + ACM
- Route 53
- Private Subnet + NAT Gateway
- RDS 연동
- Systems Manager 기반 운영
- Terraform 기반 IaC 구성