# 파드 API(Pod v1 core)

| 주요 항목  | 설명                                                                                   |
| ---------- | -------------------------------------------------------------------------------------- |
| apiVersion | v1 설정                                                                                |
| kind       | Pod 설정                                                                               |
| metadata   | 파드의 이름을 지정하는 name은 필 수 항목이며, 네임스페이스 내에서 유일한 이름이어야 함 |
| spec       | 파드의 사양을 기술                                                                     |

# 파드의 사양(PodSpec v1 core)

| 주요 항목      | 설명                                             |
| -------------- | ------------------------------------------------ |
| containers     | 컨테이너의 사양을 배열로 기술                    |
| initContainers | 초기화 전용 컨테이너의 사양을 배열로 기술        |
| nodeSelector   | 파드가 배포될 노드의 레이블을 지정               |
| volumes        | 파드 내 컨테이너 간에 공유할 수 있는 볼륨을 설정 |

containers, initContainers, volumess는 여러 개를 정의할 수 있다.

# 컨테이터 설정(Container v1 core)

| 주요 항목      | 설명                                                                          |
| -------------- | ----------------------------------------------------------------------------- |
| image          | 이미지의 리포지터리 명과 태그                                                 |
| name           | 컨테이너를 여러개 기술할 경우 필수 항목                                       |
| livenessProbe  | 컨테이너 애플리케이션이 정상적으로 동작 중인지 검사하는 프로브                |
| readinessProbe | 컨테이너 애플리케이션이 사용자의 요청을 받을 준비가 되었는지 검사하는 프로브  |
| ports          | 외부로부터 요청을 전달받기 위한 포트 목록                                     |
| resources      | CPU와 메모리 요구량과 상한치                                                  |
| volumeMounts   | 파드에 정의한 볼륨을 컨테이너 파일 시스템에 마운트하는 설정. 복수개 설정 가능 |
| command        | 컨테이너 기동 시 실행할 커맨드, args가 인자로 적용                            |
| agrs           | command의 실행 인자                                                           |
| env            | 컨테이너 내에 환경 변수 설정                                                  |

# 매니페스트 적용 방법

`kubectl apply -f 매니페스트_파일명`

매니페스트를 통해 오브젝트를 만드는 kubectl의 서브 커맨드로는 `create`와 `apply`가 있다.
apply는 동일한 이름의 오브젝트가 있을 때 매니페스트의 내용에 따라 오브젝트의 스펙을 변경하는 한편, create는 에러를 반환한다.
쿠버네티스 오브젝트를 지우기 위해서는 delete를 사용하면 된다.

`kubectl delete -f YAML 매니페스트_파일명`

# 파드의 동작 확인

`kubectl apply -f nginx-pod.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

nginx-pod.yml

현재 이 파드는 백그라운드로 돌면서 클러스터 네트워크의 TCP 80번 포트에서 요청을 대기한다.
클러스터 네트워크는 쿠버네티스 클러스터를 구성하는 노드 간의 통신을 위한 폐쇄형 네트워크다.
다른 말로 파드 네트워크라고도 불린다.
클러스터 네트워크에서 오픈한 쿠버네티스 클러스터를 호스팅하는 컴퓨터에서도 접근할 수 없다.

파드의 클러스터 네트워크의 IP 주소를 확인하고 싶은 경우에는 `-o wide` 옵션을 추가하면 된다.

쿠버네티스 클러스터 외부에서 클러스터 네트워크에 있는 파드의 포트에 접근하기 위해서는 서비스를 이용해야 한다.

파드는 클러스터 네트워크 상의 IP 주소를 가지며 이 주소를 바탕으로 파드와 파드가 서로 통신할 수 있다.

# 파드의 헬스 체크 기능

파드의 컨테이너에는 애플리케이션이 정상적으로 기동 중인지 확인하는 헬스 체크 기능을 설정할 수 있다.
이상이 감지되면 컨테이너를 강제 종료를 하고 재시작시킬 수 있다.

쿠버네티스에서는 노드에 상주하는 kubelet이 컨테이너의 헬스 체크를 담당한다.
kubelet의 헬스 체크는 다음 두 종류의 프로브를 사용하여 실행 중인 파드의 컨테이너를 검사한다.

- 활성 프로브(Liveness Probe) : 컨테이너의 애플리케이션이 정상적으로 실행 중인 것을 검사한다. 검사에 실패하면 파드상의 컨테이너를 강제 종료하고 재시작한다. 이 기능을 사용하기 위해서는 매니페스트에 명시적으로 설정해야 한다.
- 준비 상태 프로브(Readiness Probe) : 컨테이너의 애플리케이션이 요청을 받을 준비가 되었는지 아닌지를 검사한다. 검사에 실패하면 서비스에 의한 요청 트래픽 전송을 중지한다. 파드가 기동하고 나서 준비가 될 때까지 요청이 전송되지 않기 위해 사용한다. 이 기능을 사용하기 위해서는 매니페스트에 명시적으로 설정해야 한다.

로드밸런서와 마찬가지로 HTTP로 헬스 체크를 하는 경우에는 예를 들어 `http://파드IP주소:포트번호/healthz` 를 정기적으로 확인하도록 하고 서버에는 이에 대한 적절한 응답을 반환하도록 구현해야 한다. 그리고 파드의 컨테이너에는 프로브에 대응하는 핸들러를 구현해야 한다. 이 핸들러는 컨테이너의 특성에 따라 다음 세 가지 중 하나를 선택할 수 있다.

## 프로브 대응 핸들러의 종류와 설명

| 핸들러 명칭 | 설명                                                                                                                                                                                |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| exec        | 컨테이너 내 커맨드를 실행. Exit 코드 0으로 종료하면 진단 결과는 성공으로 간주되며, 그 외의 값은 실패로 간주                                                                         |
| tcpSocket   | 지정한 TCP 포트번호로 연결할 수 있다면, 진단 결과는 성공으로 간주                                                                                                                   |
| httpGet     | 지정한 포트워 경로로 HTTP GET 요청이 정기적으로 실행. HTTP 상태 코드가 200이상 400 미만이면 성공으로 간주하고, 그 외에는 실패로 간주. 지정 포트가 열려 있지 않은 경우도 실패로 간주 |

## 실습

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapl
spec:
  containers:
    - name: webapl
      image: webapl:0.1 # (1)핸드러를 구현한 애플리케이션
      livenessProbe: # (2)애플리케이션이 살아있는지 확인
        httpGet:
          path: /healthz # 확인 경로
          port: 3000
        initialDelaySeconds: 3 # 검사 개시 대기 시간
        periodSeconds: 5 # 검사 간격
      readinessProbe: # (3) 애플리케이션이 준비되었는지 확인
        httpGet:
          path: /ready # 확인 경로
          port: 3000
        initialDelaySeconds: 15
        periodSeconds: 6
```

활성프로브가 연속해서 3번 실패하면 kubelet이 컨테이너를 강제 종료하고 재기동한다.
컨테이너가 재시작되면 컨테이너에 있었던 정보들은 별도로 저장하지 않은 이상 지워진다.

```json
{
  "name": "webapl",
  "version": "1.0.0",
  "description": "",
  "main": "webapl.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.16.3"
  }
}
```

```jsx
// 모의 애플리케이션
//
const express = require('express');
const app = express();
var start = Date.now();

// Liveness 프로브 핸들러
// 기동 후 40초가 되면, 500 에러를 반환한다.
// 그 전까지는 HTTP 200 OK를 반환한다.
// 즉, 40초가 되면, Liveness프로브가 실패하여 컨테이너가 재기동한다.
//
app.get('/healthz', function (request, response) {
  var msec = Date.now() - start;
  var code = 200;
  if (msec > 40000) {
    code = 500;
  }
  console.log('GET /healthz ' + code);
  response.status(code).send('OK');
});

// Rediness 프로브 핸들러
// 애플리케이션의 초기화 시간으로
// 기동 후 20초 지나고 나서부터 HTTP 200을 반환한다.
// 그 전까지는 HTTPS 200 OK를 반환한다.
app.get('/ready', function (request, response) {
  var msec = Date.now() - start;
  var code = 500;
  if (msec > 20000) {
    code = 200;
  }
  console.log('GET /ready ' + code);
  response.status(code).send('OK');
});

// 첫 화면
//
app.get('/', function (request, response) {
  console.log('GET /');
  response.send('Hello from Node.js');
});

// 서버 포트 번호
//
app.listen(3000);
```

```docker
FROM alpine:latest

RUN apk update && apk add --no-cache nodejs npm

## 의존 모듈
WORKDIR /
ADD ./package.json /
RUN npm install
ADD ./webapl.js /

## 애플리케이션 기동
CMD node /webapl.js
```

`docker build --tag webapl:0.1 .`

`kubectl apply -f webapl-pod.yaml`

`kubectl get pod`

`kubectl get pod -o wide`

`kubectl logs webapl`

`kubectl describe pod webapl`
