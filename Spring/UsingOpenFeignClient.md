# Spring OpenFeignClient 사용기
### 사용하기 전

1. URL을 등록하고, 연결을 요청한다.
2. 연결한 후, BufferedReader를 이용해 데이터를 읽어온다.

```java
public TokenDto joinUser(String code) {
    KakaoUserInfo user = null;
		String result;
    String reqURL = "https://kapi.kakao.com/v2/user/me";
    try {
        URL url = new URL(reqURL);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("GET");
				// write
        conn.setRequestProperty("Authorization", code);
        result = getResult(conn);
    } catch (IOException e) {
        e.printStackTrace();
    }
		//...
}

private String getResult(HttpURLConnection conn) throws IOException {
    int responseCode = conn.getResponseCode();
    log.info("Get responseCode using access token: {}", responseCode);
    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(conn.getInputStream()));

    String line = "";
    String result = "";

    while ((line = bufferedReader.readLine()) != null) {
        result += line;
    }

    log.info("Get response body using access token: {}", result);
    bufferedReader.close();

    return result;
}
```

### 단점

1. 예외 상황을 핸들링해야 한다.
2. Body에 데이터까지 들어가게 된다면 코드는 더욱 더 길어진다.

<aside>
💡 이러한 단점은 OpenFeignClient를 사용하게 된다면 쉽게 데이터를 가져올 수 있다.

</aside>

### OpenFeignClient 사용

OpenFeignClient를 사용하게 된다면 애너테이션을 이용해 원하는 값을 매핑할 수 있고, 리턴값으로 원하는 값을 파싱해 반환받을 수 있다.

1. 넣어야 할 값을 인자로 넘기고, 애너테이션을 이용해 원하는 위치에 값을 추가한다.
2. GetMapping, PostMapping 등 메서드 매핑을 이용해 URI를 매핑한다.

```java
@Component
@FeignClient(url = "https://kapi.kakao.com", name = "kakaoClient")
public interface KakaoAuthorizationClient {

    @GetMapping(value = "/v2/user/me", consumes = MediaType.APPLICATION_JSON_VALUE)
    ResponseEntity<KakaoUserInfo> getUserInfo(
        @RequestHeader("Content-type") String contentType,
        @RequestHeader("Authorization") String code
    );

}
```

### 주의할 점

주의할점은 예외 상황과 파싱 실패 경우를 대비하는 것이다. 나머지 하나는 필터링이라는 기능이 있으니 문제 상황을 미리 캐치해서 예외 상황을 피할 수 있다.

1. 예외 상황일 경우 처리
2. 파싱 실패할 경우를 대비
3. 중간에 값을 필터링해야 할 경우를 설정

아래와 같이 생성한 객체와 완벽하게 매칭되지 않으면 파싱에서 문제가 발생할 수 있다.  `@JsonIgnoreProperties`을 이용해 원하지 않은 값을 커스텀할 필요가 있다.

```java
@JsonIgnoreProperties(ignoreUnknown = true)
@JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class)
public final class KakaoUserInfo implements UserInfo {
    private Long id;
    private Map<String, String> properties;
    private Map<String, Object> kakaoAccount;

    private KakaoUserInfo() {
    }
}
```

<aside>
💡 PropertyNamingStrategy.SnakeCaseStrategy.class는 Deprecated 메서드이니 사용하지 말자!

</aside>