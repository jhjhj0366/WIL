# HTTP 1.1

## 추가 사항
### 전체적인 변경 사항
- **Host header**
	- HTTP 1.0은 공식적으로 Host header를 요구하지 않는다. HTTP 1.1은 사양에 따라 이를 요구하게 된다. Host header는 프록시 서버를 통해 메시지를 라우팅하는 데 특히 중요하므로 동일한 IP를 가리키는 도메인을 구별할 수 있다.
- **Persistent connections**
	- HTTP 1.0에서 각 요청과 응답 한 쌍마다 새로운 연결을 하게 된다. 그러나 HTTP 1.1 에서는 단일 연결을 사용해 여러 요청을 실행할 수 있다.
- **Continue status**
	- 서버가 처리할 수 없는 요청 거부를 방지하기 위해 클라이언트는 먼저 요청 헤더만 보내고 상태 코드 100을 수신하는지 확인할 수 있다.
- **New methods**
	- HTTP 1.1 버전에는 PUT, PATCH, DELETE, CONNECT, TRACE 및 OPTIONS 6가지 메서드가 추가됐다.
- 기타 사항
	- 압축 및 압축 해제, 다국어 지원 및 바이트 법위 전송 등의 기능도 추가되었다.

### 메서드에서 변경 사항
HTTP 1.1 변경점에서는 HTTP 사용에 있어 실질적인 개선이 있었다. 
- PUT
	- PUT 방식은 이미 존재하는 자원을 대체하는 역할을한다.
- PATCH
	- PUT 방식은 이미 존재하는 자원을 대체하는 역할을 맡았지만, PATCH 방법은 이미 존재하는 리소스의 특정 데이터를 업데이트하게 됐다. 즉 멱등성이 없는 PUT 메서드와는 달리 PATCH 메서드는 멱등성이라는 특징을 가지고 있다. 
- DELETE
	- 리소스를 제거하게 되는 DELETE 메서드가 추가됐다.
- CONNECT
	- CONNECT 메서드는 프록시 서버를 통해 터널을 생성할 수 있다.  
- TRACE
	- TRACE는 클라이언트에서 대상 서버까지 경로에서 루프백 테스트를 실행한다. 
- OPTIONS
	- OPTIONS는 서버와의 사용가능한 통신 옵션에 대한 정보를 반환한다.

## 추가 설명
### Host header는 프록시 서버를 통해 메시지를 라우팅하는 데 특히 중요하다.

> Host header는 프록시 서버를 통해 메시지를 라우팅하는 데 특히 중요하므로 동일한 IP를 가리키는 도메인을 구별할 수 있다.

기존 캐시 서버에서 전송 대상을 판단하기 위해서 리퀘스트 메시지에서 패킷의 헤더를 조사하게 된다. 그리고 패킷의 IP 수신처 주소를 조사해 액세스 대상 웹 서버가 어디인지 알 수 있는데, 이 방법을 트랜스페어런트 프록시라 부른다.

> 캐시 서버는 클라이언트와 서버 간 접속 동작을 중개할 떄 웹 서버에서 받은 데이터를 디스크에 저장하고 웹 서버 대신 데이터를 반송하는 기능을 의미한다.

> 프록시란  웹서 서버와 클라이언트 사이에서 웹 서버에 대한 접근 동작을 중개하는 역할을 한다. 

### 동일한 IP를 가리키는 도메인을 구별할 수 있는 방법

> Host header는 동일한 IP를 가리키는 도메인을 구별할 수 있다.

트랜스페어런트 프록시를 사용하게 된다면 클라이언트는 트랜스페어런트 프록시 서버에 요청을 보내게 되므로 송신처 IP 주소는 프록시 서버의 IP 주소가 된다. 그런다음, 트랜스페어런트 프록시 서버를 경유해 서버에 요청을 보내기 위해서는 `Host header` 헤더 정보를 조사해 해당 서버로 요청을 전송한다.

### HTTP 1.0 과 HTTP 1.1 통신 차이점
1. Non-persistent vs. Persistent
	HTTP 1.0에서는 매번 데이터를 요청할 때마다 TCP 세션을 맺어야 하지만, HTTP 1.1에서는 한 번의 TCP 세션에서 여러 개의 데이터를 요청할 수 있다.
	
    ![이미지](/Network/HTTP/0.png)

	이와 같이 서로 다른 특징으로 인해 HTTP 1.0은 Non-persistent HTTP, HTTP 1.1은 Persistent HTTP라 부른다. HTTP 1.0에서 영구적인 통신을 진행하기 위해서느 Connection 값에 `close retry-after`을 추가하면 영구적인 통신이 가능하다.
2. Pipelining
	HTTP 1.0은 파이프라이닝을 제공하지 않는다. HTTP 1.0 에서 통신은 한 번에 하나의 요청을 받을 수 있다. 즉, 하나의 요청의 응답이 돌아와야지만 다음 요청을 진행할 수 있다.
	반면, HTTP 1.1은 파이프라이닝 기능을 제공한다.  HTTP 1.1은 여러 개의 요청을 동시에 보낼 수 있다. HTTP 1.0과 달리 하나의 요청이 끝나지 않아도 새로운 요청을 보낼 수 있다. 

> 하지만 파이프라이닝은 많은 문제점을 가지고 있기 때문에 최신 브라우저에서는 파이프라이닝을 통한 통신을 제공하지 않는다.
	
> 추가로 [[HTTP 버전 1의 연결 관리]]에서 자세하게 확인 가능하다.

### Request

Host header가 포함된 것을 볼 수 있다. 그리고 HTTP 1.1에서는 Connection: keep-alive  헤더를 사용하지 않더라도 지속적인 연결이 가능하지만, HTTP 1.0

```http
POST /user/create HTTP/1.1  
Host: localhost:8080  
Connection: keep-alive  
Content-Length: 46  
Content-Type: application/x-www-form-urlencoded  
Accept: */*  
  
userId=geonc123

```

### Response
- 응답은 HTTP 1.0과 큰 차이 없다.

```http
HTTP/1.1 200 OK
Server: Apache
Content-Length: 38
Content-Type: text/html; charset=utf-8

<!doctype html>
<html lang="ko">
</html>

```

참고 : [코드 연구소:티스토리](https://code-lab1.tistory.com/196) 
참고 : [HTTP: 1.0 vs. 1.1 vs 2.0 vs. 3.0](https://www.baeldung.com/cs/http-versions) 
