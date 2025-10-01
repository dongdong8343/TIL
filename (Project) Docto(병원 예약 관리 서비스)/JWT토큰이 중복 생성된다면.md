만약 JWT 토큰을 생성하는데 동일한 claim 값으로 생성하면 중복된 토큰이 생성될 수 있다는 문제가 발생한다.

이런 경우를 방지하기 위해 토큰을 생성할 때 먼저 uuid 같은 랜덤한 문자열을 통해 jti를 생성한다.

그리고 jti를 활용해서 토큰을 생성하면 payload가 달라져 중복된 토큰이 생성되는 것을 막을 수 있다.

```java
String jti = UUID.randomUUID().toString();
String token = Jwts.builder()
        .setSubject("user123")
        .setIssuer("myapp")
        .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60))
        .setId(jti)  // jti 세팅
        .signWith(secretKey)
        .compact();
```
