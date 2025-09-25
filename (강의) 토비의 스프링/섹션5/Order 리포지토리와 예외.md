- Order를 저장하는 save 메서드 만들기

1. repository 만들기

   - 만약 데이터를 처리하다가 정상 수행이 어렵다면 롤백 시켜줘야한다. 그래서 예외가 발생하면 catch로 잡아서 롤백을 시켜주는 부분을 작성해줬다.

     그리고 마지막에는 엔티티 매니저를 close 해줘야해서 finally 부분에 작성했다.

   - 하지만 이런 작업들은 매번 저장, 수정 등의 작업을 할 때 공통적으로 일어나는 작업들이다. 그래서 스프링에서 템플릿으로 이미 제공하고 있다.

   ```java
   public class OrderRepository {
       private final EntityManagerFactory emf;

       public OrderRepository(EntityManagerFactory emf) {
           this.emf = emf;
       }

       public void save(Order order) {
           EntityManager em = emf.createEntityManager();
           EntityTransaction transaction = em.getTransaction();

           // 트랜잭션
           transaction.begin();
           try {
               em.persist(order);
               em.flush();
               transaction.commit();
           } catch (RuntimeException e) {
               // 롤백 처리하는 부분이 없다. - 예외 발생시 close가 실행안됨
               // 그래서 try-catch로 롤백하는 부분을 만들었음
               if(transaction.isActive()) transaction.rollback();
               throw e;
           } finally {
               if(em.isOpen()) em.close();
           }
       }
   }
   ```

2. EntityManagerFactory는 Repository에서 하나만 생성해서 사용하면 되기 때문에 DataConfig에 작성해줬다.

   ```java
   @Configuration
   public class DataConfig {
       // data source
       @Bean
       public DataSource dataSource() {
           return new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseType.H2).build();
       }

       // entity manager factory
       @Bean
       public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
           LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
           emf.setDataSource(dataSource());
           emf.setPackagesToScan("tobyspring.tobyspring");
           emf.setJpaVendorAdapter(new HibernateJpaVendorAdapter() {{
                   setDatabase(Database.H2);
                   setGenerateDdl(true);
                   setShowSql(true);
           }});

           return emf;
       }

       @Bean
       public OrderRepository orderRepository(EntityManagerFactory emf) {
           return new OrderRepository(emf);
       }
   }
   ```

3. DataClient에서 동작 확인을 한다.
   - 지금 no 부분에 unique한 값이 들어가야하는데 서로 동일한 값을 집어넣어서 예외가 발생할 것이다.
     지금 상황은 주문 번호가 동일해서 일어나는 예외이기 때문에 주문 번호를 다르게 해서 다시 복구시킬 수 있다.
     그럼 예외를 catch해야하는데 어떤 예외를 catch 해야할까?
     다른 종류의 JPA를 지원하는 라이브러리를 사용하면 동일한 상황에서 서로 다른 예외를 던질 수 있다. 앞에서 기술마다 서로 다른 예외를 던질 수 있다고 했는데 이런 상황을 말한 것이다.
     그래서 특정 예외를 잡아서 해결하면 나중에 다른 라이브러리를 사용하게되면 확장성이 떨어지게 된다.
     이 문제를 다음 파일에서 해결할 것이다.
     ```java
     public class DataClient {
         public static void main(String[] args) {
             BeanFactory beanFactory = new AnnotationConfigApplicationContext(DataConfig.class);
             OrderRepository repository = beanFactory.getBean(OrderRepository.class);

             Order order = new Order("100", BigDecimal.TEN);
             repository.save(order);
             System.out.println(order);

             Order order2 = new Order("100", BigDecimal.ONE);
             repository.save(order2);
         }
     }
     ```
