- 기존의 WebApiExRateProvider를 보면 예외를 외부로 던지는 코드가 존재한다. 그리고 이 예외를 외부로 던지는 코드는 WebApiExRateProvider에서만 필요한 코드다.

  내부적으로 환율을 임의로 반환하는 구현 클래스에서는 예외를 던지는 코드가 필요없다. 하지만 ExRateProvider에 있기 때문에 이 인터페이스를 구현한 클래스에 예외를 던지는 코드가 다 들어가있다.

  그래서 코드에 존재하는 throws 부분을 다 없앤다.

- try/catch를 통해 처리한다.

- try-with-resources

  괄호 안에 AutoCloseable을 구현한 객체를 넣으면 자원을 다 사용하고 자동으로 반환해서 리소스 누수를 방지해준다.

  자동으로 close()를 호출한다.

```java
@Component
public class WebApiExRateProvider implements ExRateProvider {
	@Override
	public BigDecimal getExRate(String currency) {
		String url = "https://open.er-api.com/v6/latest/" + currency;

		URI uri;
		try {
			uri = new URI(url);
		} catch (URISyntaxException e) {
			throw new RuntimeException(e);
		}

		String response;
		try {
			HttpURLConnection connection = (HttpURLConnection)uri.toURL().openConnection();

			try (BufferedReader br = new BufferedReader(new InputStreamReader(connection.getInputStream()))) {
				response = br.lines().collect(Collectors.joining());
			}
		} catch (IOException e) {
			throw new RuntimeException(e);
		}

		ObjectMapper mapper = new ObjectMapper();
		ExRateData data;
		try {
			data = mapper.readValue(response, ExRateData.class);
			return data.rates().get("KRW");
		} catch (JsonProcessingException e) {
			throw new RuntimeException(e);
		}
	}
}
```
