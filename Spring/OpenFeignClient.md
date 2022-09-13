# Spring OpenFeign

## 참고 자료
[Spring Docs](https://spring.io/projects/spring-cloud-openfeign)

## Spring Cloud Open Feign
- Feign은 플러그형 인코더와 디코더도 지원합니다.
- Spring Cloud는 Spring MVC 주석에 대한 지원을 추가하고 Spring Web에서 기본적으로 사용되는 동일한 HttpMessageConverters를 사용합니다.

> Feign은 선언적 웹 서비스 클라이언트입니다. 웹 서비스 클라이언트를 더 쉽게 작성할 수 있습니다. Feign을 사용하려면 인터페이스를 만들고 주석을 답니다.


## 사용방법

### 의존성 추가
프로젝트에 Feign을 포함하려면 그룹 `org.springframework.cloud` 및 artifact ID가 `spring-cloud-starter-openfeign`인 스타터를 사용하세요.

### @EnableFeignClients 추가
```java
@SpringBootApplication
@EnableFeignClients
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

### 연결 서비스 구현

#### 인터페이스를 이용한 구현
##### @FeignClient 
- name
	name(여기에서는 `stores`)은 Spring Cloud LoadBalancer 클라이언트를 생성하는 데 사용되는 임의의 클라이언트 이름입니다. 
- url
	- url 속성(절대값 또는 호스트 이름만)을 사용하여 URL을 지정할 수도 있습니다.
- qualifiers
	- 애플리케이션 컨텍스트에서 빈의 이름은 인터페이스 이름(여기에서는 `StoreClient`)이지만, 빈을 고유한 별칭으로 지정하려면 qualifiers 사용할 수 있습니다.

> 이전에는 url 속성을 사용할 때 name 속성이 필요하지 않았지만, 현재는 이름을 필수적으로 등록해야 합니다.

추가로 Name, URL 속성에  PlaceHolders가 제공됨에 따라 프로퍼티에서 설정이 가능합니다.

```java
@FeignClient(name = "${feign.name}", url = "${feign.url}")
public interface StoreClient {
    //..
}
```

#### 예시

```java
@FeignClient("stores")
public interface StoreClient {
    @RequestMapping(
	    method = RequestMethod.GET, 
	    value = "/stores"
    )
    List<Store> getStores();

    @RequestMapping(
	    method = RequestMethod.GET, 
	    value = "/stores"
    )
    Page<Store> getStores(Pageable pageable);

    @RequestMapping(
	    method = RequestMethod.POST, 
	    value = "/stores/{storeId}", 
	    consumes = "application/json"
    )
    Store update(@PathVariable("storeId") Long storeId, Store store);

    @RequestMapping(
	    method = RequestMethod.DELETE, 
	    value = "/stores/{storeId:\\d+}"
    )
    void delete(@PathVariable Long storeId);
}
```

#### 수동으로 Feign 클라이언트 구현

어떤 경우에는 위의 방법으로는 불가능한 방식으로 Feign 클라이언트를 사용자 정의해야 할 수도 있습니다. 이 경우 [Feign Builder API](https://github.com/OpenFeign/feign/#basics) 를 사용하여 클라이언트를 생성할 수 있습니다.

```java
@Import(FeignClientsConfiguration.class)
class FooController {

    private FooClient fooClient;

    private FooClient adminClient;

    @Autowired
    public FooController(Client client, Encoder encoder, Decoder decoder, Contract contract, MicrometerCapability micrometerCapability) {
        this.fooClient = Feign.builder().client(client)
                .encoder(encoder)
                .decoder(decoder)
                .contract(contract)
                .addCapability(micrometerCapability)
                .requestInterceptor(new BasicAuthRequestInterceptor("user", "user"))
                .target(FooClient.class, "https://PROD-SVC");

        this.adminClient = Feign.builder().client(client)
                .encoder(encoder)
                .decoder(decoder)
                .contract(contract)
                .addCapability(micrometerCapability)
                .requestInterceptor(new BasicAuthRequestInterceptor("admin", "admin"))
                .target(FooClient.class, "https://PROD-SVC");
    }
}
```

### 로드 밸런서

위의 로드 밸런서 클라이언트는 `store` 서비스에 대한 물리적 주소를 검색하기를 원할 것입니다. 애플리케이션이 Eureka 클라이언트인 경우 Eureka 서비스 레지스트리에서 서비스를 확인합니다. Eureka를 사용하지 않으려면 `SimpleDiscoveryClient`를 사용하여 외부 구성에서 서버 목록을 구성할 수 있습니다. Spring Cloud OpenFeign은 Spring Cloud LoadBalancer의 차단 모드에서 사용할 수 있는 모든 기능을 지원합니다.

## Feign 기본값 오버라이딩
각 Feign 클라이언트는 요청 시 원격 서버에 연결하기 위해 함께 작동하는 구성 요소 앙상블이며 @FeignClient 주석에 설정한 이름을 사용해 앙상블을 사용하고 있습니다.

### FeignClientsConfiguration

Spring Cloud는 `FeignClientsConfiguration`을 사용하여 각 명명된 클라이언트에 대한 요청 시 `ApplicationContext`로 새 앙상블을 생성합니다. `FeignClientsConfiguration`는 Decoder, Encoder, Contract가 포함되어 있고, @FeignClient 주석의 contextId 속성을 사용하여 해당 앙상블의 이름을 재정의할 수 있습니다.

> 앙상블은 @FeignClient 주석이 달린 클라이언트들을 의미합니다.

### configuration 설정

Spring Cloud를 사용하면 @FeignClient를 사용하여 (FeignClientsConfiguration 위에) 추가 구성을 선언하여 Feign 클라이언트를 완전히 제어할 수 있습니다.

```java
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    //..
}
```

> 이 상황에서 `FooConfiguration` 구성 파일이 우선 순위로 설정되고, 다음으로`FeignClientsConfiguration` 구성 파일이 설정됩니다.


>  `FooConfiguration`은 `@Configuration`으로 주석을 달 필요가 없지만, 인코더와 디코더 설정이 된 경우 무조건적으로 `@ComponentScan`에 의해 스캔되어서는 안됩니다. `@ComponentScan` 또는 `@SpringBootApplication`에서 별도의 겹치지 않는 패키지에 넣어 방지하거나 `@ComponentScan`에서 명시적으로 제외해야 합니다.

### OpenFeign Default Bean 
Spring Cloud OpenFeign은 기본적으로 feign(BeanType beanName: ClassName)에 대해 다음 빈을 제공합니다.

- Decoder : feignDecoder
	- `ResponseEntityDecoder` (`SpringDecoder`를 래핑하는)
- Encoder : feignEncoder
	- `SpringEncoder`
- Logger : feignLogger
	- `Slf4jLogger`
- MicrometerCapability : micrometerCapability
	- `feign-micrometer`가 클래스 경로에 있고 `MeterRegistry`를 사용할 수 있는 경우
- CachingCapability : cachingCapability
	- `@EnableCaching` 주석이 사용되는 경우 `feign.cache.enabled`를 통해 비활성화할 수 있습니다.
- Contract : feignContract
	- `SpringMvcContract`
- Feign.Builder : feignBuilder
	- `FeignCircuitBreaker.Builder`
- Client : feignClient
	- Spring Cloud LoadBalancer가 클래스 경로에 있으면 `FeignBlockingLoadBalancerClient`가 사용됩니다. 클래스 경로에 아무 것도 없으면 기본 Feign 클라이언트가 사용됩니다.

> spring-cloud-starter-openfeign은 spring-cloud-starter-loadbalancer를 지원하지만 선택적으로 종속되므로 사용하려면 프로젝트에 추가되었는지 확인해야 합니다.

Spring Cloud OpenFeign은 기본적으로 feign에 대해 다음 빈을 제공하지 않지만 여전히 애플리케이션 컨텍스트에서 이러한 유형의 빈을 검색하여 feign 클라이언트를 생성합니다.


## Feign 요청과 응답 압축
Feign 요청에 대해 요청 또는 응답 GZIP 압축을 활성화하는 것을 고려할 수 있습니다.

### 응답 압축

```properties
feign.compression.response.enabled=true
```

### 요청 압축

```properties
feign.compression.request.enabled=true
```

Feign 요청 압축은 웹 서버에 대해 설정할 수 있는 것과 유사한 설정을 제공합니다.

```properties
feign.compression.request.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048
```

> 이러한 속성을 사용하면 압축된 미디어 유형과 최소 요청 임계값 길이를 선택할 수 있습니다.

## Feign 로깅
생성된 각 Feign 클라이언트에 대해 Logger가 생성됩니다. 기본적으로 로거의 이름은 Feign 클라이언트를 생성하는 데 사용되는 인터페이스의 전체 클래스 이름입니다. ## Feign 로깅은 DEBUG 수준에만 응답합니다.

### 로깅 레벨 설정
클라이언트별로 구성할 수 있는 Logger.Level 개체는 Feign에 로깅 설정을 도와줍니다.

- NONE : 로깅 없음(DEFAULT).
- BASIC : 요청 방법 및 URL, 응답 상태 코드 및 실행 시간만 기록합니다.
- HEADERS : 요청 및 응답 헤더와 함께 기본 정보를 기록합니다.
- FULL : 요청과 응답 모두에 대한 헤더, 본문 및 메타데이터를 기록합니다.

### 빈 등록 방법

#### Yaml을 이용한 기본 값 등록

```yaml
feign:
    client:
        config:
            default:
                loggerLevel: basic
```

#### 수동 빈 등록

```java
@Configuration
public class FooConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

## Feign Common Application Properties
[참고](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/appendix.html)

### Spring Data Support
`org.springframework.data.domain.Page `및 `org.springframework.data.domain.Sort `디코딩을 위해 Jackson 모듈을 활성화하는 것을 고려할 수 있습니다.
```properties
feign.autoconfiguration.jackson.enabled=true
```

### defaultQueryParameters and defaultRequestHeaders
`feign.client.config.feignName.defaultQueryParameters` 및 `feign.client.config.feignName.defaultRequestHeaders`를 사용하여 `feignName`이라는 클라이언트의 모든 요청과 함께 전송될 쿼리 매개변수 및 헤더를 지정할 수 있습니다.

```yaml
feign:
    client:
        config:
            feignName: # 이름을 지정해 요청과 함께 전송될 쿼리와 헤더를 지정할 수 있다.
				defaultQueryParameters:
                    query: queryValue
                defaultRequestHeaders:
                    header: headerValue
```

### 구성 설정

하나의 빈을 만들고 @FeignClient 구성에 배치하면 설명된 각 빈을 재정의할 수 있습니다
```java
@Configuration
public class FooConfiguration {
    @Bean
    public Contract feignContract() {
        return new feign.Contract.Default();
    }

    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("user", "password");
    }
}
```

@FeignClient는 configuration properties를 사용하여 구성할 수도 있습니다.

```yaml
feign:
    client:
        config:
            feignName:
                connectTimeout: 5000
                readTimeout: 5000
                loggerLevel: full
                errorDecoder: com.example.SimpleErrorDecoder
                retryer: com.example.SimpleRetryer
                defaultQueryParameters:
                    query: queryValue
                defaultRequestHeaders:
                    header: headerValue
                requestInterceptors:
                    - com.example.FooRequestInterceptor
                    - com.example.BarRequestInterceptor
                decode404: false
                encoder: com.example.SimpleEncoder
                decoder: com.example.SimpleDecoder
                contract: com.example.SimpleContract
                capabilities:
                    - com.example.FooCapability
                    - com.example.BarCapability
                queryMapEncoder: com.example.SimpleQueryMapEncoder
                metrics.enabled: false
```

> 기본 구성은 위에서 설명한 것과 유사한 방식으로 `@EnableFeignClients` 속성 `defaultConfiguration`에 지정할 수 있습니다. 차이점은 이 구성이 모든 Feign 클라이언트에 적용된다는 것입니다.

> 기본적으로 Feign 클라이언트는 슬래시/문자를 인코딩하지 않습니다. `feign.client.decodeSlash` 값을 false로 설정하여 이 동작을 변경할 수 있습니다.
