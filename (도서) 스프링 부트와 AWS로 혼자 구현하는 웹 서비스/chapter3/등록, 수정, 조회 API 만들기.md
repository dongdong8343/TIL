API를 만들기 위해서는 총 3개의 클래스가 필요하다.

1. DTO 클래스 (데이터를 계층 간 전달할 때 사용)
2. 컨트롤러 클래스 (API 요청 처리)
3. 서비스 클래스 (트랜잭션, 도메인 간 순서를 보장)

### 스프링 웹 계층

<center>
  <img
    src="https://github.com/user-attachments/assets/d9c1cc48-75eb-4641-8c35-841849b735a6"
    width="50%"
  />
</center>

위 사진을 보면 알 수 있듯이 스프링 웹 계층은 위와 같이 이루어져있다. 하나씩 살펴보자.

1. Web Layer - 요청, 응답에 대한 영역 (컨트롤러, JSP 등)
2. Service Layer - 트랜잭션, 도메인 간 순서 보장하는 영역
3. Repository Layer - DB에 접근 가능한 영역 (DAO)
4. DTOs - 계층 간 데이터를 전달할 때 사용하는 객체들의 영역
5. Domain Model - 도메인(개발 대상)을 모든 사람들이 이해할 수 있도록 단순화 한 것

비즈니스를 처리해야 하는 영역은 Domain이다. 책임에 맞는 역할을 수행해야한다. 이를 보통 Service 영역에서 처리하는 경우가 많은데 이렇게 하게되면 Service 영역이 무의미해지고 Domain은 그저 데이터만 모아놓는 곳이 된다. 즉 Domain이 자기 역할을 수행을 못하게 되는 것이다.

따라서 비즈니스 처리는 Domain 영역에서하고 도메인 간 순서를 보장하기 위해서 Service 영역이 존재한다고 이해하면 좋을 것 같다.

### 등록 API

일단 파일 구조는 아래와 같다.

<center>
  <img
    src="https://github.com/user-attachments/assets/6cfeff3f-edc3-400f-bc98-21c29eb58446"
    width="50%"
  />
</center>

### Controller

```java
@RequiredArgsConstructor
@RestController
public class PostsApiController {
    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto) {
        return postsService.save(requestDto);
    }
}

```

### Service

```java
@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto) {
        return postsRepository.save(requestDto.toEntity()).getId();
    }
}
```

컨트롤러와 서비스에 보면 생성자가 없고 초기화를 하지 않았는데 postsService, postsRepository를 사용하는 것을 확인할 수 있다. 사실 스프링에서 Bean을 주입해서 사용을 하는 것이다. 해당 계층과 의존 관계를 맺어야 원활한 동작이 가능하기 때문에 의존성 주입이라고 한다.

Bean을 주입하는 방법에는 3가지가 있다.

1. @Autowired (권장하지 않는다.)

- 순환 참조 - 서로 다른 빈들이 서로를 필요로 하면서 스프링이 어떤 스프링 빈을 먼저 생성할지 모르는 문제가 발생할 수 있다. 해당 어노테이션을 사용하면 객체 생성과 빈을 주입하는 시점이 달라서 컴파일 시에는 문제가 발생안하고 나중에 객체의 메서드를 사용할 때 문제가 터지게 된다.
- 강한 결합 - A라는 클래스 내부에서 B 객체를 직접 생성하고 있으면 강한 결합이 발생한다. 왜냐하면 B 객체말고 다른 C 객체로 변경하고자 할 때 A 클래스 내부의 코드를 수정해줘야 하기 때문이다.
- final 사용 불가 - Autowired는 객체가 생성된 후 주입되기 때문에 final을 사용할 수 없다. final은 선언과 동시에 초기화를 하거나 생성자에서 초기화를 하는데 둘의 시점이 다르기 때문에 final 키워드는 사용할 수 없다.

2. Setter

Setter를 사용해서 주입을 하게되면 나중에 다른 객체를 주입할 수 있어서 안정성에 문제가 생길 수 있다.

3. 생성자

@RequiredArgsConstructor 어노테이션이 붙어있어서 final이 붙은 필드를 인자값으로 하는 생성자를 만들어준다. 그래서 따로 생성자를 만들지 않아도 생성자를 사용해서 필드에 객체를 주입할 수 있게된다.

그리고 final 키워드가 붙었기 때문에 나중에 다른 객체를 의존하게 될 가능성을 차단해서 안정성이 올라가게 된다.

객체를 생성할 때 생성자가 호출되고 빈을 주입하기 때문에 컴파일 시점에 순환 참조가 있다면 바로 에러를 내준다.

추가로 컨트롤러에 새로운 서비스가 추가되거나 컴포넌트가 제거돼도 기존 코드를 수정할 필요가 없어서 편리하다.

### DTO

```java
@Getter
@NoArgsConstructor
public class PostsSaveRequestDto {
    private String title;
    private String content;
    private String author;

    private PostsSaveRequestDto(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public static PostsSaveRequestDto saveRequest(String title, String content, String author) {
        return new PostsSaveRequestDto(title, content, author);
    }

    public Post toEntity() {
        return Post.ofPosts(this.title, this.content, this.author);
    }
}
```

DTO를 보면 엔티티와 구조가 비슷하다. 그럼 엔티티를 사용하면되지 왜 굳이 DTO를 새로 만드는지 의문이 들 수 있다. 엔티티는 DB와 아주 밀접하게 닿아있는 친구다. 그래서 엔티티를 변경하고 수정한다는 것은 큰 위험이 따를 수 있다. 그리고 DTO는 계층 간에 데이터를 전송할 때 사용을 많이 하는데 엔티티를 계층 간 데이터 전송을 위해 막 생성하고 수정하는 것은 위험이 크다. 엔티티를 기준으로 테이블이 생성되고 스키마 변경된다. 그래서 DTO를 따로 만드는 것이다.

### 테스트 코드

잘 동작하는지 확인하기 위해서 아래와 같이 테스트 코드를 작성했다.

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {
    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @AfterEach
    public void tearDown() throws Exception {
        postsRepository.deleteAll();
    }

    @Test
    public void Post_등록된다() throws Exception {
        // given
        String title = "title";
        String content = "content";
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.saveRequest(title, content, "author");

        String url = "http://localhost:" + port + "/api/v1/posts";

        // when
        ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

        // then
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);

        List<Post> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }
}
```

### 수정 API

### 컨트롤러

```java
@RequiredArgsConstructor
@RestController
public class PostsApiController {
    private final PostsService postsService;

    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto) {
        return postsService.update(id, requestDto);
    }
}
```

### 서비스

```java
@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long update(Long id, PostsUpdateRequestDto requestDto) {
        Post post = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));
        post.update(requestDto);
        return id;
    }
}
```

자세히 보면 update 기능에 쿼리를 날리는 부분이 없다. 그런데 어떻게 update를 할 수 있는 것일까? 그 이유는 JPA 영속성 컨텍스트 때문이다.

영속성 컨텍스트는 엔티티를 영구적으로 저장하는 환경을 말한다. JPA의 엔티티 매니저가 활성화 된 상태로 DB에 접근해 데이터를 가져오면 이 데이터는 영속성 컨텍스트가 유지된 상태다.

이 상태에서 데이터를 변경하면 트랜잭션이 끝나는 시점에 알아서 해당 테이블의 변경분을 반영한다. 그래서 별도로 update 쿼리를 날릴 필요가 없는 것이다. 이 개념을 더티 체킹이라고 한다.

### DTO

```java
@Getter
public class PostsUpdateRequestDto {
    private String title;
    private String content;

    private PostsUpdateRequestDto(String title, String content) {
        this.title = title;
        this.content = content;
    }

    public static PostsUpdateRequestDto ofPostsUpdateRequestDTO(String title, String content) {
        return new PostsUpdateRequestDto(title, content);
    }
}
```

### 조회 API

### 컨트롤러

```java
@RequiredArgsConstructor
@RestController
public class PostsApiController {
    private final PostsService postsService;

    @GetMapping("/api/v1/posts/{id}")
    public PostResponseDto findById(@PathVariable Long id) {
        return postsService.findById(id);
    }
}
```

### 서비스

```java
@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    public PostResponseDto findById(Long id) {
        Post entity = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));

        return PostResponseDto.fromEntity(entity);
    }
}
```

### DTO

```java
@Getter
public class PostResponseDto {
    private Long id;
    private String title;
    private String content;
    private String author;

    private PostResponseDto(Long id, String title, String content, String author) {
        this.id = id;
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public static PostResponseDto fromEntity(Post entity){
        return new PostResponseDto(entity.getId(), entity.getTitle(), entity.getContent(), entity.getAuthor());
    }
}
```
