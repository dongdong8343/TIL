게시판을 만들어보는데 DTO가 계층 간에 데이터를 넘겨줄 때마다 새롭게 생성하니까 너무 많은 DTO가 생겨버렸다.

심지어 동일한 필드를 가지고 있는데 굳이 DTO를 계속 만들어야 하나? 라는 의문이 들었고 이에 대한 해답을 찾던 중 inner class를 활용해서 API당 Request, Response로 나눠서 DTO를 관리하면 DTO 파일의 수를 줄일 수 있고 효율적으로 관리할 수 있다고 해서 이번에 한 번 적용해보려고 한다.

추가로 아래 사진처럼 Dto 파일 이름들이 있으니까 언제 사용하는 DTO인지 한 눈에 파악하기 어려웠다. 그래서 API당 이너 클래스를 사용해서 관리하면 가독성이 더 좋아질 것 같았다.

![alt text](image.png)

적용해 볼 코드는 아래와 같다.

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

    public Post toEntity(){
        return Post.ofPosts(this.title, this.content, this.author);
    }
}

public class PostsSaveResponseDto {
    private Long id;
    private String title;
    private String content;
    private String author;

    private Response(Long id, String title, String content, String author) {
        this.id = id;
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public static Response fromRequest(Long postId, Request saveRequest) {
        return new Response(postId, saveRequest.title, saveRequest.content, saveRequest.author);
    }
}
```

위 코드를 보면 게시글을 작성하면 그 게시글을 저장하기 위해서 필요한 요청과 응답에 필요한 DTO다. 하지만 위 클래스는 동일한 API에 사용이 되는데 굳이 파일을 구분해서 만들게되면 나중에 DTO가 엄청 많을 때 가독성도 떨어지고 유지보수성 측면에서도 안좋은 영향을 끼친다.

그래서 아래와 같이 API별로 DTO를 구분해서 정리를 해봤다.

![alt text](image-1.png)

SavePost 안의 코드는 아래와 같다.

```java
public class SavePost {
    @Getter
    @NoArgsConstructor
    public static class Request {
        private String title;
        private String content;
        private String author;

        public Post toEntity() {
            return Post.ofPosts(this.title, this.content, this.author);
        }
    }

    @Getter
    public static class Response {
        private Long id;
        private String title;
        private String content;
        private String author;

        private Response(Long id, String title, String content, String author) {
            this.id = id;
            this.title = title;
            this.content = content;
            this.author = author;
        }

        public static Response fromRequest(Long postId, Request saveRequest) {
            return new Response(postId, saveRequest.title, saveRequest.content, saveRequest.author);
        }
    }
}
```

Request, Response에 대한 DTO를 SavePost라는 클래스에서 inner class를 활용해서 관리를 하니까 훨씬 가독성도 올라가고 파일 구조가 훨씬 명확해졌다는 기분이 들었다.

### inner 클래스에 static 키워드를 붙인 이유

추가로 inner class를 static으로 작성한 이유는 다음과 같다. inner class를 static 없이 작성을 하면 outer 클래스가 반드시 있어야 inner 클래스를 생성할 수 있다. 그렇기 때문에 둘의 관계는 연결이 된 상태다.

이 상태에서 outer 클래스는 사용하지 않는데 inner 클래스 때문에 GC 대상이 되지못해서 메모리를 잡아먹고 있는 경우가 발생할 수 있다.

하지만 static을 붙여서 정적 inner 클래스로 만들게되면 메모리 누수를 방지할 수 있다는 장점이 있다.

그래서 static을 붙여서 inner 클래스를 만들었다.

### 그런데 왜 동일한 필드들을 가지고 있는데 API별로 굳이 Request, Response 분리해야하는 것일까?

**그 이유는 가짜 중복과 관련이 있다.**

서로 다른 곳에서 사용되는 DTO지만 같은 필드를 가지고 있다는 이유로 하나의 DTO만 만들어서 관리한다고 가정하자.

그리고 이 DTO를 Save하는 곳와 Update 하는 곳에서 사용한다고 가정하자.

그런데 Update 하는 곳에서 DTO를 넘길 때 하나의 필드가 더 필요없어졌다고 가정하자. 그럼 필드를 하나 제거해줘야 한다.

제거하는 경우 Save에서 사용할 때 문제가 발생한다. 즉 나아가는 방향이 서로 다르기 때문에 DTO를 하나만 사용해서 여러 곳에서 사용하는 경우 문제가 발생한다.

필드는 동일하지만 시간이 지나면서 코드의 변경이 서로 다르게 일어나는 경우 이를 가짜 중복이라고 한다.

그렇기 때문에 이 경우에는 DTO를 API 별로 따로 만들어서 관리하는게 유지 보수 측면에서 효율적이다.

그럼 DTO가 많아져서 파일이 늘어나는거 아니냐는 질문을 할 수 있는데 이는 유지보수 측면에서 보면 API 별로 나눠서 관리하는게 효율적이기 때문에 파일이 많아지는 것보다 유지보수를 더 중요시 여긴다면 API별로 분리하는 것이 좋은 선택이 될 것이다.
