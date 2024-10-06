# 컨테이너와 네트워크

실행 중인 컨테이너는 IP 주소를 할당받아 컨테이너 간 통신이 가능하다.
호스트 내에서 접근 가능한 전용 네트워크를 통해 애플리케이션과 데이터베이스를 연결하는 것이 가능하다.
또한, 컨테이너를 호스트의 외부 네트워크에 공개하는 것도 가능하다.

<img src='./img/1.png' width='50%'>

<img src='./img/2.png' width='60%'>

# 컨테이너 네트워크

docker의 서브 커맨드 network를 사용하면 컨테이너 네트워크를 만들거나 지울 수 있다.

| 컨테이너 네트워크 커맨드  | 설명                                      |
| ------------------------- | ----------------------------------------- |
| docker network ls         | 컨테이너 네트워크를 리스트로 표시         |
| docker network inspect    | 네트워크 명을 지정해서 자세한 내용을 표시 |
| docker network create     | 컨테이너 네트워크를 생성                  |
| docker network rm         | 컨테이너 네트워크를 삭제                  |
| docker network connect    | 컨테이너를 컨테이너 네트워크에 접속       |
| docker network disconnect | 컨테이너를 컨테이너 네트워크에서 분리     |

## docker network ls

<img src='./img/3.png'>

DRIVER열에서 bridge인 경우는 외부 네트워크와 연결되어 있는 네트워크이다.
이 네트워크에 연결된 컨테이너는 외부의 리포지터리에 접근할 수 있으며 `-p` 옵션으로 외부에 포트를 공개할 수도 있다.
컨테이너를 가동할 때 명시적으로 네트워크를 지정하지 않으면 이 네트워크에 연결된다.

## docker network inspect [network id]

<img src='./img/4.png'>

## docker network create [network name]

`docker run --network [network name]` 로 컨테이터를 기동하면 network name에 연결된 컨테이너가 기동된다.
이렇게 기동된 컨테이너는 같은 네트워크에 연결된 컨테이너와만 통신이 가능하다.

<img src='./img/5.png'>

`docker run -d --name webserver1 --network my-network nginx:latest`

# 외부에 포트를 공개하기

`docker run [옵션] 리포지터리[:태그] -p 공개_포트번호:컨테이너_내_포트번호` 를 지정하면 컨테이너 내 포트를 호스트의 IP 주소상의 포트번호로 매핑한다.

`docker run -d --name webserver1 -p 8080:80 nginx:latest` 이렇게 실행 후 curl을 통해 확인할 수 있다.
`curl http://localhost:8080` , `curl http://192.168.0.31:8080` 이 외의 다른 포트번호를 넣으면 에러가 출력된다.

# AP 컨테이너와 DB 컨테이너의 연동 예

## 컨테이너 네트워크 작성

`docker network create apl-net`

컨테이너 간 통신을 위한 전용 네트워크 생성

## MySQL 서버 기동

`docker run -d --name mysql --network apl-net -e MYSQL_ROOT_PASSWORD=qwerty mysql:5.7`

## 애플리케이션 컨테이너 개발

```php
<html>
<head><title>PHP CONNECTION TEST</title></head>
<body>

<?php
$servername = "mysql";
$database = "mysql";

$username = getenv('MYSQL_USER');
$password = getenv('MYSQL_PASSWORD');

try {
    $dsn = "mysql:host=$servername;dbname=$database";
    $conn = new PDO($dsn, $username, $password);
    $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    print("<p>접속에 성공했습니다.</p>");
} catch(PDOException $e) {
    print("<p>접속에 실패했습니다.</p>");
    echo $e->getMessage();
}

$conn = null;
print('<p>종료합니다.</p>');

?>
</body>
</html>
```

mysql에 접속하고 그 결과를 메시지로 출력하는 단순한 프로그램.
기동 시 `-e` 옵션으로 환경변수를 지정해줘야 한다.

```docker
FROM php:7.0-apache
RUN apt-get update && apt-get install -y \
    && apt-get install -y libmcrypt-dev mysql-client \
    && apt-get install -y zip unzip git vim

RUN docker-php-ext-install pdo_mysql session json mbstring
COPY php/ /var/www/html/
```

## 컨테이너 이미지 빌드

`docker build -t php-apl:0.1 .`

## 컨테이너 실행

`docker run -d --name php --network apl-net -p 8080:80 -e MYSQL_USER=root -e MYSQL_PASSWORD=qwerty php-apl:01`

컨테이너 네트워크는 컨테이너 간에 서로 통신할 수 있는 통로와도 같다.
쿠버네티스에서는 비슷한 역할을 수행하는 클러스터 네트워크가 있다.
