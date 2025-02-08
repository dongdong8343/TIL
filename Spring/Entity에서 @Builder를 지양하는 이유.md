## Entity에서 @Builder를 지양하는 이유

@Builder를 사용해서 객체를 생성하게되면 다음과 같은 단점이 생깁니다.

**1. 필요한 필드에 값을 넣어야 하는데 누락하고 값을 넣게되면 null값이 들어가게돼서 엔티티의 일관성이 깨집니다.**

아래 코드와 같이 게시글 엔티티가 존재합니다. 이 때 @Builder를 사용해서 엔티티의 값을 생성하도록 하면 꼭 넣어야 하는 값을 빠뜨리고 넣어도 에러가 나지않아서 실수를 해도 모르고 넘어갈 수도 있는 가능성이 생깁니다.

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

    @Builder
    public Post(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }
}

// 엔티티에 값을 추가할 때
Post post = Posts.builder().content("content").author("author").build();
```

**2. 엔티티의 값을 생성하는 목적을 명확하게 알 수 없습니다.**

만약 만드려고하는 웹 특성 상 author에 들어가는 값이 "author1"과 "author2"만 존재한다고 가정을 해보겠습니다. 근데 Builder를 사용해서 엔티티의 값을 생성하면 그냥 코드만 봤을 때 이 엔티티의 값을 생성하는 이유를 빠르게 파악하기가 어렵습니다. 즉 가독성이 떨어집니다.

```java
Post post1 = Posts.builder().title("title").content("content").author("author1").build();

Post post2 = Posts.builder().title("title").content("content").author("author1").build();

Post post3 = Posts.builder().title("title").content("content").author("author2").build();

Post post4 = Posts.builder().title("title").content("content").author("author1").build();

Post post5 = Posts.builder().title("title").content("content").author("author2").build();

Post post6 = Posts.builder().title("title").content("content").author("author1").build();

Post post7 = Posts.builder().title("title").content("content").author("author1").build();
```

그래서 위 단점들을 해결하기 위해서 정적 팩토리 메서드를 사용하면 됩니다. 아래 코드를 보면 정적 팩토리 메서드를 사용함으로써 훨씬 더 가독성이 올라가고 필드의 값을 누락하는 경우도 막을 수 있습니다.

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

    public static fromAuthor1(String title, String content) {
        return new Post(title, content, "author1");
    }

    public static fromAuthor2(String title, String content) {
        return new Post(title, content, "author2");
    }
}

// 엔티티의 값을 생성할 때
Post post1 = Post.fromAuthor1(title, content);
Post post2 = Post.fromAuthor2(title, content);

```
