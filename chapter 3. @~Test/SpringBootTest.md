
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
<summary> Book (도메인) </summary>
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
<summary>Controller (/books 로 GET 요청시) </summary>
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
<summary>BookControllerTest (컨트롤러 테스트 코드)</summary>
<div markdown="1">

```java
@RunWith(SpringRunner.class)
@WebMvcTest(BookController.class) // 테스트에 사용할 클래스 명시
public class BookControllerTest {

    // 모든 의존성이 아닌 BookController 관련 Bean 만 로드. 여기서는 HTTP 서버를 대신 하려고.
    @Autowired
    private MockMvc mvc; 

    // 구현체는 없지만 Mock(목) 가짜 객체로 쓸 것임.
    @MockBean
    private BookService bookService;

    @Test
    public void Book_MVC_테스트() throws Exception {
        Book book = new Book("Spring Boot Book", LocalDateTime.now());
        given(bookService.getBookList()).willReturn(Collections.singletonList(book));
        //given( 어떤 메서드를 주면).willReturn( XXX가 return 될거다 )

        mvc.perform(get("/books"))
                .andExpect(status().isOk())         // HTTP status 200
                .andExpect(view().name("book"))     // 반환 view 이름이 'book'
                .andExpect(model().attributeExists("bookList")) // model 프로퍼티 중 'bookList' 존재하는지
                .andExpect(model().attribute("bookList", contains(book))); // 해당 프로퍼티에 book 객체가 담겨 있는지
    }
}
```
</div>
</details>
---

> ### 3) @DataJpaTest
- 특징과 쓰이는 때,
    1. JPA 관련 테스트 설정만 로드
    2. 데이터 소스의 설정, JPA로 데이터를 생성, 수정, 삭제 정상적으로 하는지
    3. 내장형 데이터베이스 (H2) 사용 등
 
 - 기본으로 인메모리 임베디드 DB 사용
 - ```@Entity``` 클래스를 스캔하여 JPA Repositories 구성
<details>
<summary>별도의 DataSource 를 사용하도록 Profile 사용</summary>
<div markdown="1">

```java
@RunWith(SpringRunner.class)
@DataJpaTest
//dev 프로파일 설정값으로 사용
@ActiveProfiles("dev")
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE) 
//.Any는 기본 내장 DataSource 사용
//@AutoConfigureTestDatabase(connection = H2)
public class BookJpaTest {
    .....
}
```
</div>
</details>

- ```@DataJpaTest```는 자동으로 테스트 후 Rollback
- EntityManager 대체하여, 테스트용인 TestEntityManager 를 사용해서 persist, flush, find 등 기본 JPA 테스트가 가능하다.
  - persist : 저장하고나서 return 없이 끝 / save 는 저장하고나서 저장된 객체를 돌려준다.
  - flush :  DB의 상태를 맞추는 작업. 영속성 컨텍스트의 변경 내용을 DB에 동기화(반영) 한다.
  - 궁금하면 https://velog.io/@jayjay28/%EC%97%94%ED%8B%B0%ED%8B%B0Entity


<details>
<summary>Book 에 JPA 설정 추가</summary>
<div markdown="1">

```java
@NoArgsConstructor
@Getter
@Entity // 테이블과 매핑되어 JPA가 관리할 클래스
@Table  // 매핑할 테이블 지정, 생략시 엔티티 네임으로 테이블 매핑
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
```
</div>
</details>

<details>
<summary>BookRepository 인터페이스 ( JPA )</summary>
<div markdown="1">

```java
public interface BookRepository extends JpaRepository<Book, Integer> { //Integer는 PK?
}
```
</div>
</details>

<details>
<summary></summary>
<div markdown="1">

```java
@RunWith(SpringRunner.class)
@DataJpaTest //자동으로 Rollback
public class BookJpaTest {
    private final static String BOOT_TEST_TITLE = "Spring Boot Test Book";

    @Autowired
    private TestEntityManager testEntityManager; // 테스트용 EntityManager

    @Autowired
    private BookRepository bookRepository;

    @Test
    public void Book저장하기_테스트() {
        Book book = Book.builder().title(BOOT_TEST_TITLE).publishedAt(LocalDateTime.now()).build();
        testEntityManager.persist(book); // persist 테스트
        assertThat(bookRepository.getOne(book.getIdx()), is(book));
    }

    // book 3개 저장하고 3개 맞는지, 각각 맞는 객체인지
    @Test
    public void BookList저장하고_찾기_테스트() {
        Book book1 = Book.builder().title(BOOT_TEST_TITLE+"1").publishedAt(LocalDateTime.now()).build();
        testEntityManager.persist(book1);
        Book book2 = Book.builder().title(BOOT_TEST_TITLE+"2").publishedAt(LocalDateTime.now()).build();
        testEntityManager.persist(book2);
        Book book3 = Book.builder().title(BOOT_TEST_TITLE+"3").publishedAt(LocalDateTime.now()).build();
        testEntityManager.persist(book3);

        List<Book> bookList = bookRepository.findAll();
        assertThat(bookList, hasSize(3));
        assertThat(bookList, contains(book1, book2, book3));
    }
    
    // 2개 잘 삭제 됐는지.
    @Test
    public void BookList저장하고_삭제_테스트() {
        Book book1 = Book.builder().title(BOOT_TEST_TITLE+"1").publishedAt(LocalDateTime.now()).build();
        testEntityManager.persist(book1);
        Book book2 = Book.builder().title(BOOT_TEST_TITLE+"2").publishedAt(LocalDateTime.now()).build();
        testEntityManager.persist(book2);

        bookRepository.deleteAll();
        assertThat(bookRepository.findAll(), IsEmptyCollection.empty());
    }
    
    // 클래스 전체를 한 번에 테스트하면 (RUN - public class BookJpaTest), 서로 영향이 없나?
    // 클래스 전체 테스트 시 메서드의 순서는 보장되지 않음.
}
```
</div>
</details>



---
<details>
<summary></summary>
<div markdown="1">

```java

```
</div>
</details>

