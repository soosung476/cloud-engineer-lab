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