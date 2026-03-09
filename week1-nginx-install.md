# 09 Mar 2026 - Nginx 설치 및 접속확인

## 목표
EC2 Ubuntu Instace에 nginx를 설치하고 public IP로 웹 접속이 되는지 확인한다.

## 사용한 명령어
- sudo apt update
- sudo apt install -y nginx
- sudo systemctl status nginx
- sudo systemctl enable nginx
- sudo systemctl start nginx



## 결과
- nginx active (running) 확인
- 브라우저에서 Welcome to nginx 페이지 확인

## 장애 실습
- sudo systemctl stop nginx 후 public ip로 접속해서 실패 확인
- sudo systemctl start nginx 후 접속 복구 확인

 
