## Entity에서 @Setter를 지양하는 이유

아래와 같은 엔티티가 있고 @Setter를 통해서 값을 생성하거나 수정할 수 있게 만들었다.

```java
@Setter
@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    private Post(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
```

그럼 아무 곳에서 set메서드를 통해서 엔티티에 값을 넣을수 있게된다. 그냥 단순히 set 메서드를 사용하면 엔티티인 post의 값을 생성하는 것인지 아니면 수정하는 것인지 명확하게 알기 어렵습니다. 아래 코드를 보면 알수있듯이 명확한 이유를 알기 어렵습니다.

```java
posts.setAuthor("test1");
posts.setTitle("title2");
```

그리고 자유롭게 값을 변경할 수 있기 때문에 엔티티의 일관성 유지가 어렵습니다. 그래서 이 문제를 해결하기 위해서 메서드를 만들어서 메서드에 수정하는 이유가 들어간 이름으로 지어서 명확하게 엔티티인 게시글의 값을 생성하는 것인지 수정하는 것인지를 알릴 수 있도록 해야합니다. 아래 코드처럼 작성하면 사용의도를 정확히 파악할 수 있게 됩니다.

```java
@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    private Post(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public static Post newPost(String title, String content, String author) {
        return new Post(title, content, author);
    }

    public void updatePost(String title, String content) {
        this.title = title;
        this.content = content;
    }
}

// 사용할 때
Post post1 = new newPost(title, content, author);

Post post2 = post1.updatePost(title, content);
```

### 결론

객체를 생성하거나 수정을 할 때 Setter 메서드를 사용하면 편할 수는 있으나 데이터 일관성을 유지하기 어렵습니다. 그리고 수정하는 것인지 생성하는 것인지 명확한 목적을 파악하기 어렵습니다.

그러니 생성을 할 때는 정적 팩토리 메서드를 활용하고 수정할 때는 메서드를 사용해서 생성하는 것인지 수정하는 것인지 명확한 목적을 명시할 수 있도록 합니다.
