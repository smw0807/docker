# 컨테이너 환경 표시

| 커맨드 실행    | 설명                             |
| -------------- | -------------------------------- |
| docker version | 도커 클라이언트와 서버 버전 표시 |
| docker info    | 구체적인 환경 정보 표시          |

# 컨테이너의 3대 기능

## 컨테이너 이미지 빌드

```
docker build -t 리포지터리:태그 .
docker image build -t 리포:태그 .	현 디렉터리에 있는 Dockerfile을 바탕으로 이미지를 빌드
docker images
docker image ls	로컬 이미지 목록
docker rmi 이미지
docker image rm 이미지	로컬 이미지 삭제
docker rmi -f ‘docker images -aq’
docker image prune -a	로컬 이미지 일괄 삭제
```

## 이미지의 이동과 공유

```md
docker pull 원격*리포지토리[:태그]
docker image pull 원격*리포지토리[:태그] 원격 리포지토리에서 이미지 다운로드
docker tag 이미지[:태그] 원격*리포지터리[:태그]
docker image tag 이미지[:태그] 원격*리포지터리[:태그] 로컬 이미지에 태그 부여
docker login 레지스트리*서버\_URL 레지스트리 서비스에 로그인
docker push 원격*리포지토리[:태그]
docker image push 원격\_리포지토리[:태그] 로컬 이미지를 레지스트리 서비스에 등록
docker save -o 파일명 이미지
docker image save -o 파일명 이미지 이미지를 아카이브 형식 파일로 기록
docker load -i 파일명
docker image load -i 파일명 아카이브 형식 파일을 리포지터리에 등록
docker export <컨테이너명|컨테이너ID> -o 파일명
docker container export <컨테이너명|컨테이너ID> -o 파일명 컨테이너명 또는 컨테이너ID로 컨테이너를 지정해서 tar 형식 파일로 기록
docker import 파일명 리포지터리[:태그]
docker image import 파일명 리포지터리[:태그] 파일로 저장된 이미지를 리포지터리에 입력
```

## 컨테이너 실행

```
docker run —rm -it 이미지 커맨드
docker container run —rm -it 이미지 커맨드	대화형으로 컨테이너를 기동해서 커맨드를 실행.
종료 시에는 컨테이너를 삭제.
커맨드에 sh와 bash를 지정하면 대화형 셸로 리눅스 명령어 실행 가능
docker run -d -p 5000:80 이미지
docker container run -d -p 5000:80 이미지	백그라운드로 컨테이너를 실행.
컨테이너 내 프로세스의 표준 출력과 에러 출력은 로그에 보존.
-p는 포트포워딩으로 ‘호스트_포트:컨테이너_포트’로 지정
docker run -d —name 컨테이너명 -p 5000:80 이미지
docker container run -d —name 컨테이너명 -p 5000:80 이미지	컨테이너에 이름을 지정하여 실행
docker run -v ‘pwd’/html:/usr/share/nginx/html -d -p 5000:80 nginx
docker container run -v ‘pwd’/html:/usr/share/nginx/html -d -p 5000:80 nginx	컨테이너의 파일 시스템에 디렉터리를 마운트하면서 실행.
-v 는 로컬_절대_경로:컨테이너_내_경로
docker exec -it <컨테이너명 | 컨테이너ID> sh
docker container exec -it <컨테이너명 | 컨테이너ID> sh	실행 중인 컨테이너에 대해서 대화형 셸ㅇ르 실행
docker ps
docker container ps	실행 중인 컨테이너 목록 출력
docker ps -a
docker container ls -a	정지된 컨테이너도 포함하여 출력
docker stop <컨테이너명 | 컨테이너ID>
docker container stop <컨테이너명 | 컨테이너ID>	컨테이너의 주 프로세스에 시그널 SIGTERM을 전송하여 종료 요청.
타임 아웃 시 강제 종료 진행
docker kill <컨테이너명 | 컨테이너ID>
docker container kill <컨테이너명 | 컨테이너ID>	컨테이너를 강제 종료
docker rm <컨테이너명 | 컨테이너ID>
docker container rm <컨테이너명 | 컨테이너ID>	종료한 컨테이너를 삭제
docker rm ‘docker ps -a -q’
docker container prume -a	종료한 컨테이너를 일괄 삭제
docker commit <컨테이너명 | 컨테이너ID> 리포지터리[:태그]
docker container commit <컨테이너명 | 컨테이너ID> 리포지터리[:태그]	컨테이너를 이미지로서 리포지터리에 저장
```

# 디버그 관련 기능

## 컨테이너 디버깅 관련

```
docker logs <컨테이너명 | 컨테이너ID>
docker container logs <컨테이너명 | 컨테이너ID>	컨테이너 로그 출력
docker logs -f <컨테이너명 | 컨테이너ID>
docker container logs -f <컨테이너명 | 컨테이너ID>	컨테이너 로그를 실시간으로 표시
docker ps -a
docker container ls -a	컨테이너 목록 표시
docker exec -it <컨테이너명 | 컨테이너ID>커맨드
docker container exec -it <컨테이너명 | 컨테이너ID> 커맨드	실행 중인 컨테이너에 대해서 대화형으로 커맨드 실행
docker inspect <컨테이너명 | 컨테이너ID>
docker container inspect <컨테이너명 | 컨테이너ID>	상세한 컨테이너의 정보를 표시
docker stats
docker container stats	컨테이너 실행 상태를 실시간으로 표시
docker pause <컨테이너명 | 컨테이너ID>
docker container pause <컨테이너명 | 컨테이너ID>	컨테이너를 일시정지
docker unpause <컨테이너명 | 컨테이너ID>
docker container unpause <컨테이너명 | 컨테이너ID>	컨테이너 일시정지를 해제
docker start -a <컨테이너명 | 컨테이너ID>
docker container start -a <컨테이너명 | 컨테이너ID>	정지한 컨테이너를 실행.
이때 표준 출력과 표준 에러 출력을 터미널에 출력
```
