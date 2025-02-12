보통 엔티티에는 생성시간과 수정시간이 들어가게 된다. 언제 만들어졌는지 언제 수정했는지에 대한 정보는 굉장히 중요한 정보다. 그렇기 때문에 이 정보들을 넣을 때 여기저기서 동일한 코드를 반복해서 작성하게 된다.

이번에는 JPA Auditing을 사용해서 생성시간과 수정시간을 자동화해보려고 한다.

### domain 패키지에 BaseTimeEntity 클래스 생성

```java
@Getter
// 이 클래스를 상속하면 상속받은 클래스에서 이 클래스의 필드를 컬럼으로 인식
@MappedSuperclass
// 엔티티를 생성하거나 수정할 때마다 값을 넣어줘야하는 번거러움 없앤다.
// 왜냐하면 자동으로 값을 넣어주기 때문이다.
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {
    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime modifiedDate;
}

```

Post 클래스가 위 클래스를 상속받도록 만들어준다.

```java
public class Post extends BaseTimeEntity
```

이제 Post 클래스는 createdDate, modifiedDate 필드를 컬럼으로 인식하고 새로 생성하거나 수정할 때 알아서 값을 넣게된다.

근데 하나 더 설정할 것이 남았다. Application 클래스에 JPA Auditing 어노테이션들을 활성화할 수 있도록 어노테이션을 아래와 같이 추가한다.

```java
@EnableJpaAuditing // 추가
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

그리고 잘 수행되는지 테스트 코드를 작성해서 확인했다.

```java
@Test
public void BaseTimeEntity_등록() {
    // given
    LocalDateTime now = LocalDateTime.of(2025, 2, 4, 0, 0, 0);
    postsRepository.save(Post.ofPosts("title", "content", "author"));

    // when
    List<Post> postList = postsRepository.findAll();

    // then
    Post post = postList.get(0);
    System.out.println(">>>>>>>>>> " + post.getTitle());
    System.out.println(">>>>>>>>>> createDate=" + post.getCreatedDate() + "modifiedDate=" + post.getModifiedDate());
    assertThat(post.getCreatedDate()).isAfter(now);
    assertThat(post.getModifiedDate()).isAfter(now);
}
```
