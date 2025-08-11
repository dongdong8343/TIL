JPA, Hibernate, Spring Data JPA 이 3가지 개념이 비슷한 것 같은데 각각이 뭘 의미하는지 헷갈려서 전체적인 그림을 파악하고자 정리했다.

## JPA

jpa는 자바 어플리케이션에서 관계형 DB를 어떻게 사용할 것인지 정의한 명세서다.

즉 api 명세서처럼 내부 로직은 모르더라도 어떻게 사용하는지 사용하는 방법에 대한 명세서라고 이해하면 된다.

그래서 JPA의 핵심인 EntityManager는 아래와 같이 인터페이스로 정의된 것을 확인할 수 있다.

```java
public interface EntityManager {

    public void persist(Object entity);

    public <T> T merge(T entity);

    public void remove(Object entity);

    ...
}
```

## Hibernate

하이버네이트는 JPA를 구현한 구현체를 말한다. 즉 JPA를 통해 사용 방법을 알고 사용을 했다면 실제 동작은 하이버네이트에 구현된 방식으로 동작을 하게된다.

구현체는 하이버네이트 말고도 다른 구현체들도 존재한다.

## Spring Data JPA

JPA 서적을 읽은 적이 있는데 앞 부분에 전부다 EntityManager를 통해 구현하는 것을 본적이 있다.

하지만 실제 JPA를 통해 개발할 때는 Repository를 선언해서 사용을 했다.

이렇게 사용할 수 있었던 이유는 JPA를 추상화시킨 Spring Data JPA가 Repository를 제공하기 때문이다.

Spring Data JPA를 사용하는 목적은 JPA만을 통해 작성하면 반복적인 CRUD 코드를 크게 줄여줄 수 있기 때문에 한 단계 더 추상화 한 것이다.

### JPA만 사용한 경우

엔티티마다 비슷한 코드를 계속해서 작성해줘야하는 불편함이 존재한다.

```java
@Repository
public class UserRepository {
    @PersistenceContext
    private EntityManager em;

    public void save(User user) {
        em.persist(user);
    }

    public User findById(Long id) {
        return em.find(User.class, id);
    }

    ...
}

```

하지만 Spring Data JPA를 사용하게되면 기본적인 CRUD는 상속을 받기 때문에 아래와 같이 코드가 간단해진다는 장점이 있다.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // CRUD 메서드 전부 상속받음
}

```

위 설명들을 그림으로 표현하면 아래와 같이 표현할 수 있다.

<center>
  <img
    src="https://github.com/user-attachments/assets/0a411dce-cdb5-4427-84f1-0e7750fe8a1e"
    width="50%"
  />
</center>
