## 09 Mar 2026 -Linux commands used in EC2 Nginx Lab

## 목표
EC2 Ubuntu Instance에서 nginx를 설치하고 서비스 상태/로그/포트/자원 상태를 확인하는 과정에서 하용한 리눅스 명령어를 정리한다.

## 1. 서비스 관리

### sudo systemctl status nginx
- sudo, 관리자권한으로; systemctl, 서비스 확인; status, 상태; nginx : 관리자 권한으로 nginx 서비스 상태를 확인한다.
- 언제 사용했는지: nginx가 Active중인지 Inactive중인지 확인할 때 사용햇었음, 사용결과 Activd: active(running) 인지, Active: Inactive (dead)인지 확인 가능.

### sudo systemctl stop nginx
- sudo,; systemctl,; stop, 중지한다; nginx : 관리자 권한으로 nginx서비스 중지한다.
- 언제 사용했는지: nginx를 중지하고 public IP로 접근했을 때의 반응을 살피기 위해서 사용.

### sudo systemctl start nginx
- sudo,; systemctl,; start, 시작한다,; nginx : 관리자 권한으로 nginx 실행
- 언제 사용했는지: nginx설치 후 시작을 위해서 사용.

## 2. 로그 확인
### sudo journalctl -u nginx --no-pager | tail -n 30
- sudo, journalctl, systemd 서비스 로그를 본다;  -u, 유닛; nginx ; --no-pager, 한페이지에 ; |, 파이프라인, 뒤로 결과값 넘김  ; tail,맨  마지막; -n 30 , 30줄 보여줘;
- 언제 사용했는지: nginx의 서비스 로그를 보는데 사용. 언제 해당 서비스가 동작했는지,  스탑했는지 알 수 있었다.

### sudo tail -n 30 /var/log/nginx/access.log
-  언제 사용했는지: nginx에 접근했던 로그를 볼 때 사용, 어떤 ip에서 어떤 브라우저를 타고 언제 접속했는지 알 수 있었다.

### sudo tail -n 30 /var/log/nginx/error.log
- 언제 사용했는지: nginx 에러를 볼 때 사용, notice로 소켓을 inherited받아서 사용했다는 로그가 있었다. 

## 3. 프로세스/포트/자원 확인
### ps aux | grep nginx
- ps, 현재 실행중인 프로세스를 본다 ; aux, a: 모든 사용자 프로세스 u: 사용자 정보 포함 x: 터미널 안에 묶임 프로세스까지 포함; | ; grep, 특정 문자열이 들어간 것만 찾음; nginx : 현재 프로세스 중에 nginx관련된 것들만 보여줘.
- 언제 사용했는지: 마스터프로세스, 워커프로세스 등을 볼 때 사용했다. 

### ss -tulpn | grep :80
- ss, 소켓 상태를 보는 명령어 (netstat와 비슷); -tulpn, t:tcp, u:udp, l:listening 상태만, p:프로세스정보 표시 n:포트/주소를 number로 표시 ; 즉 80번 포트의 정보들(tulpn)을 보는데 사용
- 언제 사용했는지: 80번 포트에 어떤 프로세스가 listening 중인지 보고시을 때 사용, 이 때 nginx가 있는지 확인했다.


### df -h
- 디스크 사용량 확인 명령어, -h 사람이 읽기 쉬운 단위로 보기(KB, MB, GB)
- 언제 사용했는지: 서버 디스크가 얼마나 있는지 확인할 때 사용.

### free -m
- 메모리 사용량 확인, -m 메가바이트 표시로
- 언제 사용했는지: 토탈 메로리, 사용중인게 어느정도인지 확인할 때 사용했다.

### 후기
- 평소 알던 명령어 ps, netstat, ss, pwd, df, exit 등을 직접 CLI로 써보고, 결과를 확인할 수 있어서 좋았다.
- systemctl이나 journalctl 같은 명령어는 익숙하지 않은 명령어들이었지만, 한 번 써보고 나니 서비스 상태를 확인할 때 필수적인 명령어인 것을 체감하게 되었다.
- ubuntu라서 apt명령어를 썼지만 인스턴스에서 다른 배포판을 체험할 때 yum이나 dnf사용법을 익혀야 나중에 어떤 배포판이 와도 당황하지 않고 설치/배포 할 수 있을것 같은 느낌이 든다.