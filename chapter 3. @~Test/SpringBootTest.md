
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

```java
@RunWith(SpringRunner.class)
@SpringBootTest(value = "value=test", 
//properties = {"property.value=propertyTest"}, value랑 같이 사용 불가
classes = {SpringBootTestApplication.class},
webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SpringBootTestApplicationTests {

	@Value("${value}")
	private String value;

	@Value("${property.value}")
	private String propertyValue;

	@Test
	public void contextLoads() {
		assertThat(value, is("test"));
		assertThat(propertyValue, is("propertyTest"));
	}

}
```
- ```@RunWith(SpringRunner.class)``` 를 사용하면 JUnit에 내장된 러너 대신 정의된(SpringRunner.class) 러너 클래스를 사용한다.
- ```@SpringBootTest```는 JUnit 실행에 필요한 ```SpringJUnit4ClassRunner``` 클래스를 상속 받은 ```@RunWith```가 필요함.
---
- ### ```@SpringBootTest```의 프로퍼티와 사용법
    1. ```value = "value=test" --> @Value("${value}")``` : 테스트 실행 전 적용할 프로퍼티를 주입, 기존 프로퍼티 오버라이드
    2. ```properties = {"property.value=propertyTest"} --> @Value("${property.value}")``` : 테스트 실행 전 {key = value} 형식으로 프로퍼티 추가
    3. ```classes = {SpringBootTestApplication.class}``` ApplicationContext에 로드할 클래스 지정. 지정 안할 시 @SpringBootConfiguration을 찾아서 로드
    4. ```webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT``` 실행될 때의 웹 환경을 설정, 기본값은 Mock서블릿 로드해서 구동.

- ### ```@SpringBootTest``` 사용시 Tip
    1. ```@ActiveProfiles("local")[dev, real 등..]``` 처럼 프로파일 설정 가능 (DataSource 등등에 사용)
    2. ```테스트```에서 ```@Transactional``` 사용시 테스트 마치고 Rollback함.
    3. ```@SpringBootApplication```이나 ```@SpringBootConfiguration``` 어노테이션 중 하나는 필수
---

<br>

> ### 2) @WebMvcTest
- 주로 Controller를 테스트 할 때. ( 요청 / 응답 ) 
    - 시큐리티, 필터까지 자동으로 테스트

- MVC 관련 설정인 ```@Controller, @ControllerAdvice, @JsonComponent, Filter, WebMvcConfigurer, HandlerMethodArgumentResolver``` 만 로드되어 가볍다.

<details>
<summary> Book 도메인 </summary>
<div markdown="1">

```java
@NoArgsConstructor // 기본 생성자
@Getter
public class Book {

    @Id
    @GeneratedValue
    private Integer idx;

    @Column
    private String title;

    @Column
    private LocalDateTime publishedAt;

    @Builder
    public Book(String title, LocalDateTime publishedAt) {
        this.title = title;
        this.publishedAt = publishedAt;
    }
}
```
</div>
</details>

<details>
<summary>/books 로 GET 요청시 Controller </summary>
<div markdown="1">

```java
@Controller
public class BookController {

    @Autowired
    private BookService bookService;

    @GetMapping("/books")
    public String getBookList(Model model) {
        model.addAttribute("bookList", bookService.getBookList());
        return "book";
    }
}
```
</div>
</details>

<details>
<summary>BookService 인터페이스</summary>
<div markdown="1">

```java
public interface BookService {
    List<Book> getBookList();
}
// 구현체는 만들지 않고 Mock 데이터로 테스트 함.
```
</div>
</details>


<details>
<summary></summary>
<div markdown="1">

</div>
</details>





---
<details>
<summary></summary>
<div markdown="1">

</div>
</details>

