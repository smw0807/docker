# hello-world 실행

`docker run hello-world`

실행 시 출력되는 문구

```bash
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

- 도커의 '이미지'는 운영체제와 소프트웨어를 담고 있는 컨테이너 실행 이전의 상태다. 각 이미지는 '리포지터리:태그'로 식별된다.
- 도커의 리포지터리는 이미지 보관소를 말한다. 리포지터리의 이름에 버정 등을 의미하는 태그를 붙여서 각각의 이미지를 구별하여 보관할 수 있다. 태그를 생략하면 최신을 의미하는 latest가 사용된다. 클라우드 서비스의 문서 등에서는 리포지터리 대신에 레지스트리란 표현이 쓰이기도 한다.
- 레지스트리는 windows의 설정 정보 데이터베이스를 말하기도 하지만, 도커에서는 리포지터리의 집합체로서 리포지터리를 제공하는 서버를 말한다.
