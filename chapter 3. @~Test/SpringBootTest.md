
> # Chapter 3. 스프링 부트 테스트

- 스프링 부트는 기본적인 테스트 스타터 spring-boot-starter-test 를 제공. 크게 2가지 모듈.
    1. spring-boot-test : Test 실행 관련 기능
    2. spring-boot-test-autoconfiguration : 관련 Test 자동 설정 기능

- 이 장에서 알아볼 @Test
    1. @SpringBootTest
    2. @WebMvcTest
    3. @DataJpaTest
    4. @RestClientTest
    5. @JsonTest

> ### 1) @SpringBootTest
- 특징과 쓰이는 때,
    1. 통합 테스트를 제공
    2. 여러가지 단위 테스트를 하나의 통합된 테스트로 수행할 때
    3. 만능이긴 한데, 애플리케이션에 설정된 Bean을 모두 로드하여 규모가 크면 느려짐.
        - 단위 테스트하기엔 부적합해 질 수 있다.

``` 기본적으로 ```
