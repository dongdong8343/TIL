1. build.gradle에 추가

```java
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
    annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"
```

2. gradle - build

빌드를 하게되면 Q 파일이 생성된다.

<center>
  <img
    src="https://github.com/user-attachments/assets/f8192604-b98c-44b3-aee2-89bdf43882c2"
    width="50%"
  />
</center>


3. config 파일 만들어서 자바 설정하기
<center>
  <img
    src="https://github.com/user-attachments/assets/0f9e29fd-8284-400a-b01f-f7bb49228b88"
    width="50%"
  />
</center>


```java
    package org.dongdong.sb2.config;

    import com.querydsl.jpa.impl.JPAQueryFactory;
    import jakarta.persistence.EntityManager;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class QuerydslConfig {

        @Bean
        public JPAQueryFactory jpaQueryFactory(EntityManager em) {
            return new JPAQueryFactory(em);
        }

    }
```

4. 잘 적용이 됐는지 테스트 코드 만들어서 확인하기

   ```java
   @Autowired
   private JPAQueryFactory queryFactory;

   @Test
   public void testQuery() {
       log.info("====> " + queryFactory);

       QTodo todo = QTodo.todo;

       JPQLQuery<Todo> query = queryFactory.selectFrom(todo);

       query.where(todo.tno.gt(0L));
       query.where(todo.title.like("AAA"));

       query.orderBy(new OrderSpecifier<>(Order.DESC, todo.tno));

       query.limit(10);
       query.offset(5);

       log.info(query);
       List<Todo> entityList = query.fetch();

       long count = query.fetchCount();

       log.info(entityList);
       log.info(count);
   }
   ```

5. 새로운 Repository 생성

<center>
  <img
    src="https://github.com/user-attachments/assets/be43ae9d-f766-4c2c-9013-be0706c0909f"
    width="50%"
  />
</center>
    
```java
    public interface TodoSearch {
        List<Todo> list1(Pageable pageable);
    }
```


6. querydsl을 작성할 클래스 파일 생성

<center>
  <img
    src="https://github.com/user-attachments/assets/0fd47fac-3cb2-405c-bfbf-a5549b54f8e1"
    width="50%"
  />
</center>
    
    - 규칙 - 인터페이스 이름 + Impl
    
```java
    package org.dongdong.sb2.todo.repository;
    
    import java.util.List;
    
    import com.querydsl.core.types.Order;
    import com.querydsl.core.types.OrderSpecifier;
    import com.querydsl.jpa.JPQLQuery;
    import lombok.extern.log4j.Log4j2;
    import org.dongdong.sb2.todo.entities.QTodo;
    import org.springframework.data.domain.Pageable;
    import org.dongdong.sb2.todo.entities.Todo;
    import com.querydsl.jpa.impl.JPAQueryFactory;
    import lombok.RequiredArgsConstructor;
    
    @Log4j2
    @RequiredArgsConstructor
    public class TodoSearchImpl implements TodoSearch{
    
        private final JPAQueryFactory queryFactory;
    
        @Override
        public List<Todo> list1(Pageable pageable) {
            log.info("list.........................");
            log.info(queryFactory);
    
            QTodo todo = QTodo.todo;
    
            JPQLQuery<Todo> query = queryFactory.selectFrom(todo);
    
            int size = pageable.getPageSize();
            int offset = pageable.getPageNumber() * size;
            
            query.limit(size);
            query.offset(offset);
            query.orderBy(new OrderSpecifier<>(Order.DESC, todo.tno));
    
            List<Todo> list = query.fetch();
            
            return null;
        }
    
    }
```


7. Repository에 가서 TodoSearch(인터페이스) 추가

```java
    public interface TodoRepository extends JpaRepository<Todo, Long>, TodoSearch
```
