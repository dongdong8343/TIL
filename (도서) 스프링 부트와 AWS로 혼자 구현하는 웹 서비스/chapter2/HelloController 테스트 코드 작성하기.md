1. 컨트롤러를 작성하기 전에 프로젝트의 메인 클래스를 만들어줍니다.

```java
package org.example.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // 스프링 부트 자동설정, 빈 읽기 생성 자동으로 해줌. (이 위치부터 설정들 읽어나감.)
public class Application {
    public static void main(String[] args) {
        // 내장 WAS(웹 애플리케이션 서버) 실행
        SpringApplication.run(Application.class, args);
    }
}

```

내장 WAS를 사용함으로써 톰캣을 직접 설치할 필요가 없게됩니다. 만약 외장 WAS 쓴다고 가정을 하면 WAS의 버전이 달라지면 내부의 코드도 다시 다 설정을 해줘야하기 때문에 매우 번거로워질겁니다. 하지만 내장 WAS를 씀으로써 그런 일은 생기지 않습니다.

2. HelloController를 생성합니다.

```java
package org.example.springboot.web;

import org.example.springboot.web.dto.HelloResponseDto;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

// JSON을 반환하는 컨트롤러로 만들어줌
@RestController
public class HelloController {
    // /hello로 get 요청을 하면 아래 메서드를 실행해서 hello를 반환
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount);
    }
}

```

3. HelloController의 hello api가 잘 동작하는지 확인하기 위해서 테스트 코드를 작성합니다.

- 테스트 코드를 작성하는 클래스의 이름은 보통 대상 클래스 이름에 Test를 붙여서 사용합니다.
- src > test > 이후 메인 패키지와 동일
<center>
  <img
    src="https://github.com/user-attachments/assets/45c7e521-f838-4d46-8b09-3a4713bbd13c"
    width="50%"
  />
</center>

```java
package org.example.springboot.web;

import org.example.springboot.web.dto.HelloResponseDto;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.web.servlet.MockMvc;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@ExtendWith(SpringExtension.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {
    @Autowired // 스프링이 관리하는 빈 주입 받음
    private MockMvc mvc; // 웹 api 테스트 할 때 사용

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello")) // /hello 주소로 get요청 보내면
                .andExpect(status().isOk()) // Header의 Status 검증
                .andExpect(content().string(hello)); // 응답 본문 내용 검증
    }

    @Test
    public void helloDto가_리턴된다() throws Exception{
        String name = "hello";
        int amount = 199;
        mvc.perform(
                        get("/hello/dto")
                                .param("name", name)
                                .param("amount", String.valueOf(amount)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value(name))
                .andExpect(jsonPath("$.amount").value(amount)
                );
    }
}

```
