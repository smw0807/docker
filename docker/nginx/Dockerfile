# Nginx 베이스 이미지를 사용합니다
FROM nginx:latest

# nginx.conf 파일을 이미지 내의 Nginx 설정 디렉토리로 복사합니다
COPY nginx.conf /etc/nginx/nginx.conf

# 기본 Nginx 포트 노출
EXPOSE 8080

# 컨테이너가 시작되면 Nginx를 실행합니다
CMD ["nginx", "-g", "daemon off;"]

# docker build -t custom-nginx .
# docker run -d -p 8080:8080 --name my-nginx custom-nginx

# nginx.conf 수정 시
# 컨테이너 정지 및 삭제
# docker stop my-nginx
# docker rm my-nginx

# 이미지 재빌드
# docker build -t custom-nginx .

# 컨테이너 재실행
# docker run -d -p 8080:8080 --name my-nginx custom-nginx

###
# 파일 수정
# docker cp nginx.conf my-nginx:/etc/nginx/nginx.conf

# 재실행
# docker exec my-nginx nginx -s reload