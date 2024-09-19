# 마스터와 노드

클러스터 관리를 담당하는 마스터, 컨테이너화된 애플리케이션을 실제로 실행하는 노드
마스터는 마스터 노드, 노드는 워커 노드라고도 한다.

마스터는 kubectl과 같은 API 클라이언트로부터 요청을 받아서 애플리케이션의 배포, 스케일 업/다운 컨테이너의 버전 업 등의 요구를 처리한다.
마스터는 쿠버네티스 클러스터의 단일 장애점이 되지 않도록 다중화할 수 있다.

# 도커의 아키텍처

## 도커 데몬

클라이언트인 도커 커맨드 명령을 받아들여서 도커 오브젝트인 이미지, 컨테이너, 볼륨, 네트워크 등을 관리한다.
도커 데몬은 네트워크 너머에 있는 원격 클라이언트로부터 요청을 받는 것도 가능하다.

## 도커 클라이언트

도커 커맨드는 컨테이너를 조작하는 커맨드 라인 유저 인터페이스로 도커 데몬의 클라이언트다.
도커 커맨드는 도커 API를 사용하여 도커 데몬에 요청을 보낸다.

### 자주 사용하는 커맨드

`docker build` : 새로운 이미지를 만들 때 사용한다.  
`docker pull` : 레지스트리에서 이미지를 로컬에 다운로드할 때 사용한다.  
`docker run` : 이미지를 바탕으로 컨테이너를 실행한다.

## 이미지

읽기 전용인 컨테이너의 템플릿을 말한다.

## 컨테이너

하나의 프로세스로 볼 수 있고, 명확하게 표현하자면 ‘실행 가능한 이미지의 인스턴스’라고 할 수 있다.

## 도커 레지스트리

컨테이너의 이미지가 보관되는 곳이다.  
도커는 기본으로 도커 허브에 있는 이미지를 찾도록 설정되어 있다.  
개발한 컨테이너 이미지를 쿠버네티스에서 실행하기 위한 중간 창고와도 같은 존재이다.

레지스트리와 리포지터리는 이름이 비슷한데,  
레지스트리는 리포지터리를 여러개 가지는 보관 서비스이다.  
리포지터리는 하나의 이미지에 대해 태그를 사용하여 다양한 출시 버전을 함께 보관하는 곳이다.

## 퍼블릭 레지스트리

공개된 레지스트리이다.

## 클라우드 레지스트리

퍼블릭 레지스트리가 제공하는 레지스트리 서비스이다.  
접근 가능한 계정을 제한하여 비공개로 운영할 수 있다.

## 비공개 레지스트리

회사나 팀 전용으로 레지스트리를 구축하여 운영하는 경우에 해당된다.

# 컨테이너를 위한 기술과 표준

## 리눅스 표준 규격과 리눅스 ABI(Application Binary Interface)

도커를 사용하면 다양한 리눅스 배포판을 기반으로 한 컨테이너 실행이 가능하다.

## 리눅스 커널 기술

### 네임스페이스(Namespace)

리눅스 커널에 사용된 기술로 컨테이너가 하나의 독립된 서버와 같이 동작할 수 있게 한다.  
네임스페이스를 사용하면 특정 프로세스를 다른 프로세스로부터 분리시켜 같은 네임스페이스 내에서만 접근할 수 있도록 제한할 수 있다.

### 컨트롤 그룹(cgroup)

도커는 리눅스 커널의 cgroup을 사용한다.
cgroup은 프로세스별로 CPU 시간이나 메모리 사용량과 같은 자원을 감시하고 제한한다.

## 유니온 파일 시스템(UnionFS)
