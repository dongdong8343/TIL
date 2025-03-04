실제로 서비스를 운영한다고 가정을 했을 때 사용자가 어떤 글이나 정보를 삭제하는 경우가 있다. 이 때 삭제하고 아무 일도 안일어나면 괜찮지만 어떤 경우는 사용자가 데이터 날라간거 복구할 수 있냐라는 문의가 오기도 한다.

그래서 이런 경우를 대비하기 위해서 DB에서 실제로 삭제하는 것이 아니라 삭제했다고 표시만 하고 실제 서비스에서는 보여주지 않는 방식으로 운영을 한다고 한다.

이것을 soft delete라고 한다. 그래서 아래 코드는 어노테이션을 사용해서 soft delete를 적용해봤다. 간단하게 설명을 하자면 **@SQLRestriction("is_deleted = false")** 이 코드를 통해서 sql 뒤에 is_deleted가 false인 친구만 가지고 오도록 할 수 있다.

그리고 삭제 명령이 내려진 경우 실제로 삭제하지 않고 update 쿼리가 나가도록 작성을 했다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@SQLDelete(sql = "UPDATE post set is_deleted = true WHERE id = ?")
@SQLRestriction("is_deleted = false")
@Entity
public class Post extends BaseTimeEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(length = 1000, nullable = false)
    private String content;

    private String author;

    private Boolean isDeleted;

    private Post(String title, String content, String author, Boolean isDeleted) {
        this.title = title;
        this.content = content;
        this.author = author;
        this.is_deleted = isDeleted;
    }

    public static Post ofPosts(String title, String content, String author) {
        return new Post(title, content, author, false);
    }

    public void update(String title, String content) {
        this.title = title;
        this.content = content;
    }
}

```

하지만 위 코드를 봤을 때 어떤 게시글을 가지고 온다고 했을 때 삭제된 게시글도 포함해서 가지고 오고 싶은 경우도 있을 것이다. 실제로 운영할 때 필요한 데이터가 될 수도 있기 때문이다.

하지만 이 때 어노테이션으로 삭제 표시가 된 친구만 항상 가지고 오도록했기 때문에 삭제 표시된 게시글을 가지고 오기 힘들겠다 라는 생각이 들었다.

그리고 실제 삭제 명령을 내렸을 때 update문을 통해 is_deleted가 true가 되도록 했는데 이는 직관적이지 않다고 판단이 돼서 아래와 같이 코드를 수정했다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class Post extends BaseTimeEntity {
    ...

    public void delete(){
        this.isDeleted = LocalDateTime.now();
    }
}
```

```java
@Component
@RequiredArgsConstructor
public class PostProvider {
    ...

    @Transactional
    public void delete(Long id) {
        Post post = searchPost(id);
        post.delete();
    }
}
```

위 코드와 같이 Post 엔티티안에 delete 메서드를 구현해서 isDeleted 필드가 삭제됐다는 표시를 하도록 만들었다.

그리고 Provider 계층에서 실제 삭제를 하는 것이 아니라 post 엔티티의 delete 메서드를 통해 삭제 표시를 하도록 만들었다.

```java
public interface PostsRepository extends JpaRepository<Post, Long> {
    @Query("SELECT p FROM Post p ORDER BY p.id DESC")
    List<Post> findAllDesc();

    List<Post> findByIsDeletedIsNull();

    Optional<Post> findByIdAndIsDeletedIsNull(Long id);
}
```

그리고 실제 운영자가 삭제된 데이터도 가지고 와서 보고싶을 때를 대비해서 위와 같이 경우에 따라 쿼리가 호출될 수 있도록 만들었다.
