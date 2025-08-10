### 1. 라이브러리 추가

일단은 필요한 라이브러리들을 추가해주기 위해서 build.gradle 파일에 아래와 같이 작성했다.

```java
dependencies {
    ...
    ...
    // 1
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    // 2
    implementation 'com.h2database:h2'
}
```

1. Spring Data Jpa 추상화 라이브러리를 추가 - 스프링부트 버전에 맞춰서 자동으로 JPA 관련 라이브러리들의 버전 관리
2. 인메모리 데이터베이스 추가 - 메모리에서 실행돼서 프로젝트를 재시작하면 데이터들이 날라간다. 그렇기 때문에 테스트 용도로 많이 사용된다.

### 2. 엔티티 생성

따로 domain 패키지를 만들어서 Post라는 클래스를 생성했다.

<center>
  <img
    src="https://github.com/user-attachments/assets/0b096f8f-550e-4a47-b7ec-5c7b7f5660f4"
    width="50%"
  />
</center>

코드는 아래와 같다.

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity // 테이블과 매핑될 클래스라는 것을 나타낸다.
public class Post {
    @Id // 해당 테이블의 pk 필드임을 나타낸다.
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

    // 정적 팩토리 메서드
    public static Post ofPosts(String title, String content, String author) {
        return new Post(title, content, author);
    }
}
```

주요 어노테이션을 클래스와 가까이 둬서 나중에 새 언어로 전환할 때 필요없는 롬복 어노테이션은 바로 삭제할 수 있도록했다.

위 코드에서 정적 팩토리 메서드를 만들어서 객체를 생성하도록 한 것을 확인할 수 있다. 이는 나중에 필드가 추가되거나 특정한 값만 넣는 경우 그냥 생성자를 통해서 넣게되면 어떤 객체를 생성하는 것인지 명확하게 알 수 없습니다.

그래서 정적 팩토리 메서드를 만들어서 어떤 객체를 만드는 것인지 명시를 해주게되면 나중에 코드를 읽을 때 가독성도 올라가고 유지보수 하기도 편리해집니다.

자세한 내용은 아래 링크에서 확인 가능합니다.

[정적 팩토리 메서드란?](https://github.com/dongdong8343/TIL/blob/main/JAVA/%EC%A0%95%EC%A0%81%20%ED%8C%A9%ED%86%A0%EB%A6%AC%20%EB%A9%94%EC%84%9C%EB%93%9C.md)

그리고 아래 코드를 한 번 보자.

@NoArgsConstructor(access = AccessLevel.PROTECTED)

기본 생성자를 왜 protected로 지정을 했을까? 그 이유는 다음과 같다.

private로 설정하게되면 나중에 JPA가 프록시 객체를 생성할 때 리플렉션을 통해 생성하는데 이 때 기본 생성자가 필요하다. 그렇기때문에 private로 지정하면 기본 생성자를 사용못하기 때문에 안된다.

public으로 설정하게되면 여기저기서 막 생성자를 호출할 수 있기 때문에 protected를 사용해서 생성자를 호출할 때 한 번 더 체크할 수 있도록 하기위해서다.

자세히보면 @Setter도 없는 것을 확인할 수 있습니다. 그 이유는 Setter를 사용해서 여기저기서 값 변경이 가능해지고 나중에 객체를 생성하거나 수정할 때 어디서 왜 변경이 되는것인지 명확하게 알기 어렵다. 그렇기 때문에 Setter는 지양하고 메서드를 만들어서 왜 변경을 하는것이지 아니면 생성을 하는 것인지 명확하게 알려주는 것이 좋다.

### 데이터베이스에 접근을 위해 JpaRepository를 생성

엔티티 클래스와 기본 Entity Repository는 같은 패키지에 위치해야한다. 그 이유는 둘은 아주 밀접한 관계를 가지고 있기 때문이다. 그래서 아래와 같이 posts 패키지 안에 Post 클래스와 동일한 곳에 위치하도록 했다.

<center>
  <img
    src="https://github.com/user-attachments/assets/4c8dfaa0-e0d1-4d82-b881-86b21994d9e4"
    width="50%"
  />
</center>

그래서 아래 코드와 같이 Repository에 JpaRepository<Entity 클래스, PK 타입>을 상속해주게 되면 기본으로 CRUD 메서드가 생성이 된다.

```java
public interface PostsRepository extends JpaRepository<Post, Long> { }
```

### PostsRepository가 제대로 동작하는지 확인하기 위한 테스트 코드 작성

잘 동작하는지 확인하기 위해서 테스트 코드를 작성해보겠다.

```java
package org.example.springboot.domain.posts;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import java.time.LocalDateTime;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@ExtendWith(SpringExtension.class)
@SpringBootTest // 자동으로 h2 데이터베이스를 실행
public class PostRepositoryTest {
    @Autowired
    PostsRepository postsRepository;

    @AfterEach // 단위 테스트 끝나면 실행됨. - 다음 테스트 진행할 때 영향 안끼치게 하기 위함.
    public void cleanup() {
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기() {
        // given
        String title = "테스트 게시글";
        String content = "테스트 본문";

        postsRepository.save(Post.ofPosts(title, content, "author"));

        // when
        List<Post> postList = postsRepository.findAll();

        // then
        Post post = postList.get(0);
        assertThat(post.getTitle()).isEqualTo(title);
        assertThat(post.getContent()).isEqualTo(content);
    }
}
```

### 실행된 쿼리 보는 방법

<center>
  <img
    src="https://github.com/user-attachments/assets/cbc811f1-7f92-45a6-8a57-58906785567e"
    width="50%"
  />
</center>

application.properties 파일에서 아래 코드를 추가하고 실행하면 콘솔에서 쿼리를 확인할 수 있다.

```java
spring.jpa.show-sql=true
```

근데 저 코드만 넣으면 h2 데이터베이스 쿼리로 보여져서 MySQL 쿼리로 변경해서 보고싶다면 아래 코드를 작성해줘야한다. 근데 이거는 버전 별로 달라질 수 있어서 공식문서를 통해 확인하고 작성하는 것이 바람짐하다.

```java
spring:
  jpa:
    show-sql: true
  h2:
    console:
      enabled: true
  datasource:
    // DB url 설정
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:
```

그럼 이제 DB에 작업을 진행하면 콘솔에서 MySQL 쿼리로 확인할 수 있게된다.
