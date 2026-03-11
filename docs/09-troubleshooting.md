# Troubleshooting Notes

## SSH 접속이 안 될 때
1. EC2 Running 상태 확인
2. 퍼블릭 IP 확인
3. Security Group 22 확인
4. pem 권한 확인
5. Ubuntu 인스턴스면 사용자명 `ubuntu` 확인

## 웹 접속이 안 될 때
1. EC2 상태 확인
2. nginx 상태 확인
3. 80 포트 리스닝 확인
4. Security Group 80 확인
5. nginx access/error 로그 확인

## ALB SG로 인한 접속불가.
1. ALB를 관리하는 SG를 생성 (HTTP)
2. 해당 SG의 아웃바운드를 EC2관리 SG에 연결
3. EC2 관리 SG의 인바운드를 ALB관리 SG로 설정.

