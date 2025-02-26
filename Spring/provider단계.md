아래 코드를 보면 게시글을 찾아오는 코드가 반복되는 것을 확인할 수 있다. 그래서 데이터를 조회하는 부분을 따로 분리하면 훨씬 코드가 더 깔끔해지겠다고 느꼈다.

```java
@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public SavePost.Response save(SavePost.Request request) {
        Long postId = postsRepository.save(request.toEntity()).getId();

        return SavePost.Response.fromRequest(postId, request);
    }

    @Transactional
    public Long update(Long id, UpdatePost.Request request) {
        Post post = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));
        post.update(request);
        return id;
    }

    @Transactional(readOnly = true)
    public SearchPost.Response findById(Long id) {
        Post entity = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 사용자가 없습니다. id=" + id));

        return SearchPost.Response.fromEntity(entity);
    }

    @Transactional(readOnly = true)
    public List<ReadPosts.Response> findAllDesc() {
        return postsRepository.findAllDesc().stream()
                .map(ReadPosts.Response::new)
                .collect(Collectors.toList());
    }


    @Transactional
    public void delete(Long id){
        Post post = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id="+id));

        postsRepository.delete(post);
    }
}

```

위 코드를 보면 동일한 코드가 반복된다는 것을 알 수 있다.

```java
postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));
```

그래서 위 코드를 provider 계층으로 빼서 코드의 중복을 줄여보려고 한다. 사실 private 메서드를 사용해서 service 클래스에서 공통된 로직을 처리할 수 있는데 이렇게 하면 테스트 코드를 작성할 때 문제가 발생한다.

JUnit 같은 테스트 프레임워크는 public, protected 메서드만 직접 호출할 수 있기 때문이다. 물론 private 메서드를 테스트할 수 있는 방법은 있지만 여기서는 넘어가도록 하겠다.

나는 provider 클래스를 만들어서 데이터를 조회하는 로직은 따로 메서드를 만들어서 넣어줬다.

```java
@Component
@RequiredArgsConstructor
public class PostProvider {
    private final PostsRepository postsRepository;

    public Post searchPost(Long id) {
        return postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 게시물이 없습니다. id=" + id));
    }

    public List<ReadPosts.Response> searchPosts() {
        return postsRepository.findAllDesc().stream()
                .map(ReadPosts.Response::new)
                .collect(Collectors.toList());
    }
}

```

그리고 service 클래스에서 provider의 메서드를 호출하는 방식으로 로직의 흐름 제어에 더 집중할 수 있도록 만들었다.

```java
@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;
    private final PostProvider postProvider;

    @Transactional
    public SavePost.Response save(SavePost.Request request) {
        Long postId = postsRepository.save(request.toEntity()).getId();

        return SavePost.Response.fromRequest(postId, request);
    }

    @Transactional
    public Long update(Long id, UpdatePost.Request request) {
        Post post = postProvider.searchPost(id);
        post.update(request);
        return id;
    }

    @Transactional(readOnly = true)
    public SearchPost.Response findById(Long id) {
        Post entity = postProvider.searchPost(id);

        return SearchPost.Response.fromEntity(entity);
    }

    @Transactional(readOnly = true)
    public List<ReadPosts.Response> findAllDesc() {
        return postProvider.searchPosts();
    }


    @Transactional
    public void delete(Long id){
        Post post = postProvider.searchPost(id);

        postsRepository.delete(post);
    }
}

```

훨씬 코드가 깔끔해진 기분이다.
