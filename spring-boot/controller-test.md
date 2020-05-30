# Controller Test

> example-code: https://github.com/jongmin92/code-examples/tree/master/spring-boot/controller-test

Spring Boot에서 Controller(Web layer)를 테스트하는 방법은 여러 가지가 있다. 방법에 따라 Unit Test와 Integration Test를 모두 작성할 수 있다.

- Inside-Server Test
    - MockMVC in Standalone Mode
    - MockMVC with WebApplicatonContext
- Outisde-Server Test
    - SpringBootTest with a Mock WebEnvironment
    - SpringBootTest with a Real WebServer

## Server-Side Test

Spring에서 server-side 테스트에는 크게 2가지 전략이 있다. MockMVC 혹은 RestTemplate을 사용해서 Controller 테스트를 작성한다. 보통 Unit Test를 하려면 MockMVC를 사용하는 반면 Integration Test를 작성하는 경우 RestTemplate을 사용한다.

## Inside-Server Test

웹 서버 없이 Controller 로직을 직접 테스트 할 수 있다. 그래서 inside-server 테스트라고 부르며 Unit Test의 정의에 가깝다. 테스트를 위해서는 전체 웹 서버를 Mocking해야 하므로 애플리케이션에서 테스트 해야하는 부분이 누락될 수 있으나, 이는 Integration Test에서 다뤄질 수 있다.

### 1. MockMVC in Standalone Mode

![strategy-1](/spring-boot/image/controller-test/strategy-1.png)

Spring에서 MockMVC를 standlone-mode로 사용하면 inside-server 테스트를 작성할 수 있다. (Spring Context를 로드하지 않는다.)

```java
@ExtendWith(MockitoExtension.class)
public class PersonControllerMockMvcStandloneTest {

    @Mock
    private PersonService personService;

    @InjectMocks
    private PersonController underTest;

    private MockMvc mockMvc;

    @BeforeEach
    public void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(underTest)
                                 .setControllerAdvice(new PersonExceptionHandler())
                                 .addFilters(new PersonFilter())
                                 .addInterceptors(new PersonInterceptor())
                                 .build();
    }

    @Test
    public void getPersonByNameWhenExists() throws Exception {
        lenient().when(personService.getPerson("name"))
                 .thenReturn(new Person("name", 10));

        mockMvc.perform(get("/persons/name")
                                .accept(MediaType.APPLICATION_JSON))
               .andExpect(matchAll(
                       status().isOk(),
                       jsonPath("$.name").value("name"),
                       jsonPath("$.age").value(10)
               ));
    }

    @Test
    public void handleNonExistingPersonException() throws Exception {
        lenient().when(personService.getPerson(anyString()))
                 .thenThrow(NonExistingPersonException.class);

        mockMvc.perform(get("/persons/name")
                                .accept(MediaType.APPLICATION_JSON))
               .andExpect(matchAll(
                       status().isNotFound()
               ));
    }

    @Test
    public void headerIsPresent() throws Exception {
        lenient().when(personService.getPerson("name"))
                 .thenReturn(new Person("name", 10));

        mockMvc.perform(get("/persons/name")
                                .accept(MediaType.APPLICATION_JSON))
               .andExpect(matchAll(
                       status().isOk(),
                       header().string("X-HEADER-1", "filter-value"),
                       header().string("X-HEADER-2", "interceptor-value")
               ));
    }
}
```

### 2. MockMVC with WebApplicatonContext

![strategy-2](/spring-boot/image/controller-test/strategy-2.png)

이번에도 Controller에 대한 Unit Test를 작성하지만, 위와는 다르게 Spring의 `WebApplicationContext`를 사용한다. 아직까지 Inside-Server Test
를 하고 있기 때문에 이 경우에도 웹 서버는 띄우지 않고 테스트한다. (테스트에 하나 이상의 클래스 동작이 포함되므로 실제로는 클래스 간의 Integration Test로 생각할 수도 있다.)

```java
@ExtendWith(SpringExtension.class)
@WebMvcTest(PersonController.class)
public class PersonControllerMockMvcWithContextTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private PersonService personService;

    @Test
    public void getPersonByNameWhenExists() throws Exception {
        when(personService.getPerson("name"))
                .thenReturn(new Person("name", 10));

        mockMvc.perform(get("/persons/name")
                                .accept(MediaType.APPLICATION_JSON))
               .andExpect(ResultMatcher.matchAll(
                       status().isOk(),
                       jsonPath("$.name").value("name"),
                       jsonPath("$.age").value(10)
               ));
    }

    @Test
    public void handleNonExistingPersonException() throws Exception {
        when(personService.getPerson(anyString()))
                .thenThrow(NonExistingPersonException.class);

        mockMvc.perform(get("/persons/name")
                                .accept(MediaType.APPLICATION_JSON))
               .andExpect(matchAll(
                       status().isNotFound()
               ));
    }

    @Test
    public void headerIsPresent() throws Exception {
        when(personService.getPerson("name"))
                .thenReturn(new Person("name", 10));

        mockMvc.perform(get("/persons/name")
                                .accept(MediaType.APPLICATION_JSON))
               .andExpect(matchAll(
                       status().isOk(),
                       header().string("X-HEADER-1", "filter-value"),
                       header().string("X-HEADER-2", "interceptor-value")
               ));
    }
}
```

## Outisde-Server Test

테스트를 위해 애플리케이션에 HTTP 요청을 수행하는 것을 outside-server 테스트라고 부를 수 있다. 그러나 외부에서 테스트 하더라도 테스트 과정에서 mock을 주입하는 것이 가능하다.

Spring에서는 RestTemplate을 사용해서 REST Controller에 대한 outside-server test를 작성하거나 또는 유용한 기능이 포함된 TestRestTemplate를 사용해서 integration test를 작성할 수도 있다.

### 3. SpringBootTest with a Mock WebEnvironment

매개 변수없이 또는 `webEnvironment = WebEnvironment.MOCK`과 함께 `@SpringBootTest`를 사용하는 경우 실제 HTTP server를 로드하지 않는다. **2. MockMVC with WebApplicatonContext**와 비슷한 방식이다.

웹 서버를 띄우지 않기 때문에 RestTemplate을 사용할 수 없다. 그러므로 MockMVC를 계속 사용해야 한다. MockMVC는 `@AutoConfigureMockMVC`로 인해 자동으로 설정된다.

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
@AutoConfigureMockMvc
public class PersonControolerSpringBootMockTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private PersonService personService;

    @Test
    public void getPersonByNameWhenExists() throws Exception {
        when(personService.getPerson("name"))
                .thenReturn(new Person("name", 10));

        mockMvc.perform(get("/persons/name")
                                .accept(MediaType.APPLICATION_JSON))
               .andExpect(ResultMatcher.matchAll(
                       status().isOk(),
                       jsonPath("$.name").value("name"),
                       jsonPath("$.age").value(10)
               ));
    }

    @Test
    public void handleNonExistingPersonException() throws Exception {
        when(personService.getPerson(anyString()))
                .thenThrow(NonExistingPersonException.class);

        mockMvc.perform(get("/persons/name")
                                .accept(MediaType.APPLICATION_JSON))
               .andExpect(matchAll(
                       status().isNotFound()
               ));
    }

    @Test
    public void headerIsPresent() throws Exception {
        when(personService.getPerson("name"))
                .thenReturn(new Person("name", 10));

        mockMvc.perform(get("/persons/name")
                                .accept(MediaType.APPLICATION_JSON))
               .andExpect(matchAll(
                       status().isOk(),
                       header().string("X-HEADER-1", "filter-value"),
                       header().string("X-HEADER-2", "interceptor-value")
               ));
    }
}
```

### 4. SpringBootTest with a Real WebServer

![strategy-4](/spring-boot/image/controller-test/strategy-4.png)

`WebEnvironment.RANDOM_PORT` 또는 `WebEnvironment.DEFINED_PORT`와 함께 `@SpringBootTest`를 사용하는 경우 실제 HTTP server를 로드한다. 이 경우 RestTemplate 또는 TestRestTemplate을 사용해서 테스트해야 한다.

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class PersonControllerSpringBootTest {

    @MockBean
    private PersonService personService;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void getPersonByNameWhenExists() {
        final Person person = new Person("name", 10);
        when(personService.getPerson("name"))
                .thenReturn(person);

        final ResponseEntity<Person> response = restTemplate.getForEntity("/persons/name", Person.class);

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).isEqualTo(person);
    }

    @Test
    public void handleNonExistingPersonException() {
        when(personService.getPerson(anyString()))
                .thenThrow(NonExistingPersonException.class);

        final ResponseEntity<Person> response = restTemplate.getForEntity("/persons/name", Person.class);

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
    }

    @Test
    public void headerIsPresent() {
        final Person person = new Person("name", 10);
        when(personService.getPerson("name"))
                .thenReturn(person);

        final ResponseEntity<Person> response = restTemplate.getForEntity("/persons/name", Person.class);

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getHeaders().get("X-HEADER-1")).containsOnly("filter-value");
        assertThat(response.getHeaders().get("X-HEADER-2")).containsOnly("interceptor-value");
    }
}
```

**1. MockMVC in Standalone Mode** 방식이 다른 방식보다 성능면에서 더 좋다고 생각할 수도 있다. 반대로 Spring Boot Context를 생성하고 HTTP Server까지 띄우는 **4. SpringBootTest with a Real WebServer
** 방식은 매우 비효율적이라고 생각할 수 있다.
완전히 맞는 말은 아니다. 그 이유는 동일한 Test Suite에 대해서는 application context가 기본적으로 재사용되기 때문이다.

그러나, 테스트에서 Spring bean을 수정하는 경우 context를 재사용하면 side effect가 발생할 수 있다. 이 경우에는 `@DirtiesContext`를 사용해서 각 테스트 전에 context를 다시 로드하도록 해야한다.
