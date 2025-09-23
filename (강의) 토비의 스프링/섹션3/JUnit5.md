- 테스트는 작고 빠르게 동작하게 만들어야 한다. 그리고 그런 연습을 해야한다.

- @Test 테스트 메서드

  하나의 테스트 기능을 하나의 메서드로 만들어서 제대로 동작하는지 쉽게 검증할 수 있다.

- @BeforeEach

  각 테스트 전에 실행돼서 초기화와 같이 테스트 진행 전에 수행할 작업들을 작성할 수 있다.

- 테스트마다 새로운 인스턴스가 만들어진다.

  각 테스트는 서로 영향을 끼치면 안된다. 독립적으로 실행돼야해서 매번 새로운 인스턴스가 만들어지는 것이다.

- 보통 클래스 명 + Test로 이름을 작성한다.

- 준비, 실행, 검증 순으로 작성한다. (정해진 것은 아님)

  ```java
  public class SortTest {
      Sort sort;

      @BeforeEach
      void beforeEach() {
          sort = new Sort();
      }

      @Test
      void sort() {
          // 실행
          List<String> list = sort.sortByLength(Arrays.asList("aa", "b"));

          // 검증
          Assertions.assertThat(list).isEqualTo(List.of("b", "aa"));
      }

      @Test
      void sort3Items() {
          // 실행
          List<String> list = sort.sortByLength(Arrays.asList("aa", "ccc", "b"));

          // 검증
          Assertions.assertThat(list).isEqualTo(List.of("b", "aa", "ccc"));
      }

      @Test
      void sortAlreadySorted() {
          // 준비
          Sort sort = new Sort();

          // 실행
          List<String> list = sort.sortByLength(Arrays.asList("b", "aa", "ccc"));

          // 검증
          Assertions.assertThat(list).isEqualTo(List.of("b", "aa", "ccc"));
      }
  }

  ```
