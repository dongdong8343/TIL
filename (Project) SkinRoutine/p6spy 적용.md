개발을 하면서 DB에 날리는 쿼리를 보고 싶은데 별다른 설정을 하지않으면 어떤 값들이 날라가는지 보지 못한다는 단점이 있다.

실제로 개발할 때 어떤 값들이 날라가는지 보고싶은데 그러지 못한다는 것이 불편했다.

그래서 p6spy라는 것을 적용해볼 것이다.

### p6spy?

p6spy는 기존 애플리케이션 코드를 변경하지 않고 데이터베이스의 데이터를 가로채서 로그를 남길 수 있는 프레임워크다.

애플리케이션이 DB에 던지는 모든 JDBC 호출을 감싸서 가로채고 그것들을 분석해서 로그로 남겨주는 역할을 한다.

### 구성 방법

build.gradle에 p6spy starter 의존성을 추가해준다.

```java
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.9.0'
```

이를 수동 구성 방식으로 커스터마이징도 할 수 있지만 나는 날라가는 쿼리를 자세하게 보고싶을뿐 별다른 커스터마이징은 필요없다고 생각해서 자동 구성 방식으로 진행했다.

<center>
  <img
    src="https://github.com/user-attachments/assets/a6faa0c7-49c8-487e-b660-a20d13982355"
    width="100%"
  />
</center>

위와 같은 폴더 구조를 구성하고 `org.springframework.boot.autoconfigure.AutoConfiguration.imports` 파일을 만들어서 아래 내용을 추가해준다.

```java
com.github.gavlyukovskiy.boot.jdbc.decorator.DataSourceDecoratorAutoConfiguration
```

이 파일은 스프링부트가 DataSourceDecoratorAutoConfiguration 클래스를 찾아 자동으로 로드하도록 지시한다.

위의 내용까지 적용하면 아래와 같이 쿼리가 한 줄로 나가는 것을 볼 수 있다. 하지만 우리는 쿼리를 가독성 좋게 보고싶다.

<center>
  <img
    src="https://github.com/user-attachments/assets/8ded4f01-7f42-4d20-a751-38a5abe1a3da"
    width="100%"
  />
</center>

그렇게 하기 위해서는 P6SpyConfig 클래스를 만들어서 쿼리를 가독성있게 볼 수 있도록 만들 수 있다.

```java
@Configuration
public class P6SpyConfig implements MessageFormattingStrategy {

	@PostConstruct
	public void setLogMessageFormat() {
		P6SpyOptions.getActiveInstance().setLogMessageFormat(this.getClass().getName());
	}

	@Override
	public String formatMessage(int connectionId, String now, long elapsed, String category,
		String prepared, String sql, String url) {
		return String.format("[%s] | %d ms | %s", category, elapsed, formatSql(category, sql));
	}

	private String formatSql (String category, String sql){
		if (sql != null && !sql.trim().isEmpty() && Category.STATEMENT.getName()
			.equals(category)) {
			String trimmedSQL = sql.trim().toLowerCase(Locale.ROOT);
			if (trimmedSQL.startsWith("create") || trimmedSQL.startsWith("alter")
				|| trimmedSQL.startsWith("comment")) {
				sql = FormatStyle.DDL.getFormatter().format(sql);
			} else {
				sql = FormatStyle.BASIC.getFormatter().format(sql);
			}
			return sql;
		}
		return sql;
	}
}
```

<center>
  <img
    src="https://github.com/user-attachments/assets/4b909a2d-5dea-4b42-afab-71f2e5385b06"
    width="100%"
  />
</center>

참고로 p6spy를 적용하기 전에 나오던 쿼리가 나오면서 중복으로 출력되는 것을 막으려면 아래와 같이 application.yml 파일을 수정해야한다.

```java
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        dialect:
          storage_engine: innodb
        format_sql: false // 중복 출력 방지를 위해 false로
        show_sql: false // 중복 출력 방지를 위해 false로
        use_sql_comments: false
    open-in-view: false
```
