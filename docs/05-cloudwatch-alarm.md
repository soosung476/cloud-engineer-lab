# Cloud Watch 생성 및 실습

## 1. 실습 목적

EC2 상태를 Cloud Watch 매트릭으로 확인하여 CPU사용률 임계치 기반 알림을 설정, 알람이 실제로 동작되는지 확인한다. 

운영 관점에서 기본적인 모니터링 흐름을 익힌다.

---

## 2. 실습 환경

- AWS EC2 Ubuntu
- CloudWatch
- SNS Email Notification
- MacBook M1
- nginx 실행 중인 EC2

---

## 3. 실습 목표

- CPUUtilization 메트릭 확인
- Cloud Watch 알람 생성
- SNS 이메일 연결
- CPU 부하 테스트
- OK - In Alarm - OK 상태 확인


---


## 4. 작업 과정

### 4-1. CloudWatch Metrics에서 CPUUtilization 확인
Cloud Watch - Metric - All metrics - EC2 - Per Instance Metrics로 이동하여 CPUUtilization이 있는 것을 확인했다.
인스턴스 네임과 아이디가 알람을 설정하고자 하는 EC2와 맞는지 확인후 알람을 생성한다.

### 4-2. CPU 알람 생성
CPUUtilization 체크박스 항목에 체크 후 Create Alarm 버튼 클릭

주요 설정을 다음과 같다 :
- Statistic : Average - 평균값 통계를 사용.
- Period : 5 min - 통계를 이용하는 시간, 5분동안의 평균값을 토대로 Thresdhold를 넘어가면 알람을 보낸다.
- Threshold Type : Static - 정적으로 표현, 일정 수치를 정해놓고 그 수치를 넘어가면 알람을 보낸다.

### 4-3. SNS 이메일 구독 연결
Notification 설정에 In Alarm시 SNS로 알람을 보내는 세팅, 생성을 완료하면, Confirm subscription 을 선택하는 이메일이 온다.
Confirm subscription을 누르면 구독이 완료됐다는 창이 생성된다.

### 4-4. SSH 접속 후 CPU 부하 테스트
터미널, EC2로 SSH에 접속해서 yes > dev/null 명령어를 입력 CPU에 부하를 준다. 

### 4-5. 알람 상태 변화 확인
알람이 OK에서 In Alarm으로 변화 SNS에 해당 내용이 오면 이메일 구독까지 확인할 수 있다.

### 4-6. 부하 종료 후 정상 상태 복귀 확인
Ctrl + C를 눌러서 부하를 종료하고, In Alarm 에서 OK로 가는지 확인한다.

---

## 5. 확인 결과
이번 실습을 통해서

- Cloud matric에서 EC2 CPU 메트릭을 확인했다.
- CPU가 70%이상일 때 알람이 울리도록 설정하고, 외에 다른 조건들을 설정했다.
- SNS가 구독을 승인하고 알림 수신을 확인했다.
- CPU 부하 발생 후 알람이 In Alarm상태로 가는 것을 확인했다.
- 부하 종료 후 In Alarm상태에서 일정 시간 뒤 OK로 변하는 것을 확인했다.

---

## 6. 운영 관점에서 중요한 이유

- 운영자는 서버에 직접 들어가서 매번 상태를 확인할 수 없다
- 이상 징후를 자동으로 감지하고 통보받는 체계가 필요하다
- CloudWatch Alarm은 기본적인 모니터링의 출발점이다
- 이후 Auto Scaling, 장애 대응 자동화, 로그 분석과 연결되는 기반이 된다

---

## 7. 발생할 수 있는 문제

- 매트릭이 바로 안 보일 수 있음
- 구독 승인하기 전까지는 경고가 오지 않음
- **기본 모니터링이 5분 단위라 상태 변화가 즉각적으로 발생하지 않을 수 있음**

---

## 7. 트러블슈팅
CPU를 70% 이상일 때 알람이 오기로 설정했으나, 임계치가 높았던지 메세지가 오지 않았다.
- 메트릭 확인 결과 CPU가용율이 올라가긴 하나 너무 더디게 올라왔다.
- EC2의 SSH접속을 이용해 2개의 부하를 줘서 해결했다.
- EC2의 SSH접속이 불가능해서, 확인 결과 Security group설정이 My-IP로 설정된 것을 발견, 0.0.0.0/0으로 교체 후 다시 접속했다.

---

## 8. 배운 점
CloudWatch를 통해서 이번에 일정 수준 이상일 때 알람을 받았고, 자동적으로 이상을 감지하고 메세지를 보내주는 점에서 운영할 때 편리성이 증가할 것으로 예상됐다.
Static 방식으로 일정 수준을 설정할 수도 있지만, Dynamic으로 밴드범위를 설정해 시간대별로 학습한 결과값보다 많이 튀는 경우 알람을 받을 수 있는데, 잘 활용하면 트래픽 관리에 아주 훌륭한 모니터링 시스템을 만들 수 있을것 같다.
알람은 그저 생성하는게 중요한 것이 아니라, 알람을 활용해서 대응을 하고 그로 인해서 상태변화를 다시 확인하는 과정이 이번 학습의 핵심이었다.
