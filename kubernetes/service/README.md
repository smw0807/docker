# 서비스

파드는 일시적인 존재라 언제든지 할당된 IP가 바뀔 수 있다.
클라이언트 입장에서는 늘 변하는 파드의 IP 주소를 알기 어렵기 때문에 쿠버네티스에는 서비스라는 오브젝트가 존재한다.
서비스에는 다음과 같이 여러 타입이 있어 이를 매니페스트에 지정함으로써 접근이 가능한 클라이언트의 범위를 한정할 수 있다.

| 서비스 타입  | 접근 가능 범위                                                                                               |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| ClusterIP    | 타입을 지정하지 않으면 기본으로 설정되며, 클러스터 내부의 파드에서 서비스의 이름으로 접근할 수 있다.         |
| NodePort     | ClusterIP의 접근 범위 뿐만 아니라 쿠버네티스 클러스터 외부에서도 노드의 IP 주소와 포트번호로 접근할 수 있다. |
| LoadBalancer | NodePort의 접근 범위 뿐만 아니라 쿠버네티스 클러스터 외부에서 대표 IP 주소로 접근할 수 있다.                 |
| ExternalName | 쿠버네티스 클러스터 내의 파드에서 외부 IP 주소에 서비스의 이름으로 접근할 수 있다.                           |

# ClusterIP

클러스터 내부에서 내부 DNS에 등록한 이름으로 특정 파드 집합에 요청을 전송할 수 있게 해준다.
매니페스트에 ClusterIP: none이라고 지정하면, 헤드리스 설정으로 서비스가 동작한다.
이 설정에서는 대표 IP 주소를 획득하지 않고, 부하분산도 이뤄지지 않는다.
그 대신에 파드들의 IP 주소를 내부 DNS에 등록하여, 파드의 IP 주소 변경에 대응하여 최신 상태를 유지한다.

# 서비스 타입 NodePort

서비스 타입에 NodePort를 지정하면, ClusterIP의 기능에 더해 노드의 IP 주소에 공개 포트가 열린다.
이를 통해 쿠버네티스 클러스터 외부에서 내부의 파드에 요청을 보낼 수 있게 된다.

공개 포트번호의 범위는 기본적으로 30000~32767이다.
클라이언트가 노드의 IP와 포트로 전송한 요청은 최종적으로 파드에 전달된다.

NodePort 타입의 서비스를 만들면 클러스터의 모든 노드에 이정한 포트가 열리게 된다.
각 노드가 수령한 요청은 대상이 되는 파드들에게 부하분산되어 전송된다.
이때 요청을 받은 노드 내에 있는 파드로만 전송하도록 설정할 수도 있다.
노드들 앞에 로드밸런서가 있다면 매우 유용한 설정이다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service-np
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
  type: NodePort
```

# 서비스 타입 LoadBalancer

서비스 타입 LoadBalancer는 로드밸런서와 연동하여 파드의 애플리케이션을 외부에 공개한다.
또한, LoadBalancer는 NodePort를 사용하기 때문에 ClusterIP도 자동적으로 만들어 진다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service-lb
spec:
  selector:
    app: web
  ports:
    - name: webserver
      protocol: TCP
      port: 80
  type: LoadBalancer
```

# 서비스 타입 ExternalName

서비스 타입 ExternalName은 지금까지 살펴본 것들과 반대로, 파드에서 쿠버네티스 클러스터 외부의 엔드포인트에 접속하기 위한 이름을 해결해준다.
예를 들어, 퍼블릭 클라우드의 데이터베이스나 인공지능 API 서비스 등을 접근할 때 사용될 수 있다.

ExternalName은 서비스의 이름과 외부 DNS 이름의 매핑을 내부 DNS에 설정한다.
이를 통해 파드는 서비스의 이름으로 외부 네트워크와 엔드포인트에 접근할 수 있다.
포트 번호는 지정할 수 없다.

```yaml
kind: Service
apiVersion: v1
metadata:
  name: yahoo
spec:
  type: ExternalName
  externalName: www.yahoo.co.jp
```

이 서비스 타입은 파드에서 쿠버네티스 클러스터 외부의 엔드포인트에 접속할 때 편리하다.
네임스페이스에서의 서비스 이름으로 IP 주소를 얻을 수 있기 때문이다.
또한, 쿠버네티스 클러스터 내의 서비스로 교체하기도 쉽다.

다만, 외부 DNS 명을 등록하는 항목 `spec.externalName` 에는 IP 주소를 설정할 수 없다.
서비스의 매니페스트에 IP 주소를 설정하고 싶은 경우에는 헤드레스 서비스를 이용해야 한다.

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: server1
subsets:
  - addresses:
      - ip: 192.168.1.16
---
apiVersion: v1
kind: Service
metadata:
  name: server1
spec:
  clusterIP: None
```

# 서비스와 파드의 연결

서비스가 요청을 전송할 파드를 결정할 때는 셀렉터의 라벨과 일치하는 파드를 etcd로 부터 선택한다.

```yaml
## 서비스
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec: # type을 생략하여 ClusterIP가 적용된다.
  selector: # service - 백엔드 pod와 연결
    app: web
  ports:
    - protocol: TCP
      port: 80
```

```yaml
## 디플로이먼트
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector: # deployment - pod 대응용
    matchLabels:
      app: web
  template: # 여기서부터 파드 템플릿
    metadata:
      labels:
        app: web # 파드의 라벨
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

서비스의 셀렉터(selector)와 디플로이먼트 파드 템플릿의 metadata.labels에 같은 라벨 `app.web` 을 기술한다.

하나의 파드 템플릿으로 만들어지는 파드들은 같은 속성이 부여되므로 라벨도 같다.
따라서 디플로이먼트에 의해 만들어지는 파드들은 같은 라벨을 가지게 되어 서비스의 요청을 전송받는다.

이처럼 라벨에 의해 전송되는 파드를 결정하는 방식은 큰 유연성을 가져다 준다.
셀렉터의 값을 바꾸는 것만으로 서비스가 전송되는 파드의 그룹을 바꿀 수 있어 운영상의 유연성을 가질 수 있다.
그러나 라벨이 중복되면 의도치 않은 파드로 요청이 전송될 수 있어 중복되지 않도록 프로젝트 운영 규칙을 정하는 것이 중요하다.
