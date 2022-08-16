# HTTP Header

HTTP 헤더를 사용하면 클라이언트와 서버가 HTTP 요청 또는 응답과 함께 추가 정보를 전달할 수 있다. HTTP 헤더는 대소문자를 구분하지 않는 이름과 콜론(`:`) 그리고 값으로 구성된다. 추가로 값 앞의 공백은 무시된다.


### 요청 헤더

### 응답 헤더

### 프록시가 헤더를 처리하는 방법에 따라 그룹화

### End-to-end header
End-to-end header는 메시지의 최종 수신자에게 전송되어야 한다. 중간 프록시는 End-to-end 와 관련된 헤더를 수정하지 않고 재전송해야 하며 캐시는 이를 저장해야 한다.

### hop-by-hop header
hop-by-hop header는 단일 전송 수준 연결에 대해서만 의미가 있으며 프록시에 의해 캐싱되거나 재전송되어서는 안된다. `Connection` 헤더를 사용해 hop-by-hop 헤더에만 설정할 수 있다.

[MDN : HTTP 헤더](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#end-to-end_headers)