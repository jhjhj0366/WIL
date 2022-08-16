# HTTP 1.0

## 추가 사항
- **Header**
	- HTTP 0.9 요청 라인에는 메서드와 리소스 이름만 작성하게 되었다.
	- HTTP 1.0은 HTTP 헤더를 도입해 프로토콜을 유연하고 확장 가능하게 됐고, 생성한 메타데이터를 전송할 수 있도록 도왔다.
- **Versioning**
	- HTTP 버전이 추가되면서 요청 라인에 사용된 버전을 명시적으로 알리고 요청라인에 추가하게 됐다.
- **Status code**
	- HTTP 응답에 상태 코드가 포함되어 수신자가 요청 처리 상태를 확인할 수 있다.
- **Content-type**
	- HTTP 헤더 덕분에 Content-Type 필드와 관련해 HTTP는 일반 HTML 파일 외 다른 문서 유형을 전송할 수 있다.
- **New method**
	- 0.9 버전에서 제공하는 GET외에 POST와 HEAD를 제공한다.

> 프로토콜 개선에 있어 가장 비중이 높은 것은 HTTP 헤더와 메서드와 기능 추가이다. 해당 기능 덕분에 다양한 파일 형식을 보내고 받을 수 있으며, 관련 메터데이터를 교환할 수 있다. 또한 새로운 방법을 통해 클라이언트는 문서에 대한 메타데이터(HEAD)만 복구하고 클라이언트에서 서버로 데이터를 전송(POST)할 수 있다.

## 추가 설명
### Request
- **Header**를 추가한 모습을 볼 수 있다. Header는 `:`로 key와 value로 분리한다.
- 요청 라인에 **Versioning**이 추가된 모습을 볼 수 있다.  요청의 버전은 HTTP/1.1이다.
- 요청에 **Content-type**을 추가해 다양한 파일들을 전송할 수 있다.
- 요청에 **method**를 추가해 다양한 헤더만 보내거나(HEAD) 데이터를 body에 넣어서 보내는 등(POST) 다양한 방식으로 데이터를 전송할 수 있다.

```http
GET http://localhost:8080/user HTTP/1.1  
Content-Type: */*
Content-Length: 1

a

```

### Response
- 응답에서 **Status code**를 확인할 수 있다.
- 응답에 **Content-type**을 사용해 다양한 파일들을 전송할 수 있다.

```http
HTTP/1.1 200 OK
Server: Apache
Content-Length: 38
Content-Type: text/html; charset=utf-8

<!doctype html>
<html lang="ko">
</html>

```