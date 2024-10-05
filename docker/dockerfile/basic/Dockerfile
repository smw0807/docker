# 베이스 이미지
FROM alpine:latest
# 리눅스 패키지 및 추가 모듈 설치
RUN apk update && apk add figlet
# 컨테이너의 파일 시스템에 파일 추가
ADD ./message /message
# 컨테이너 실행 시 실행할 명령어
CMD cat /message | figlet