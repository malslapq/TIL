# 스프링 의존성 주입 DI (Dependency Injection)

 자바 프로젝트를 만들면서 느꼈던 점은 MVC 패턴을 만들어도 서비스 레이어에서 만들어 놓은 인터페이스의 구현체를 어떤걸로 사용할지 정해줘야 했다.
그러다 보니 **서로간의 강한 결합을 이루고, 어딘가에서 수정이 필요한 경우 하나하나 코드를 따라가면서 전체적인 수정이 불가피했다.**

    스프링을 사용할 때 내가 어떤 것을 사용할지 스프링 컨테이너에서 내가 설정해놓은 파일을 읽어 주입해준다면 어떻게 될까?
    
🍊 모양과 🍎 모양의 젤리가 있다고 생각하자. 두 젤리를 만드는 재료와 양은 같고 모양만 다를 뿐이다.<br>
이럴 경우 만드는 과정 전체를 바꾸는게 아니고 만드는 모양의 틀만 바꿔주면 되지 않을까? 

이를 Spring DI라고 생각한다면 우리가 어떤 객체(틀)를 사용할지는 Spring이 우리가 만들어놓은 설정파일을 읽은 후 이 객체를 사용하려고 하는구나 하고 자동적으로 주입한다. 

그러면 우리는 전체적으로 코드를 수정 할 필요가 없어지는 엄청난 효과를 얻을 수 있다!!!

### Spring DI를 사용하는 이유

- 결합도를 낮춰준다.
- 재사용성, 확장성, 유연성을 높여준다.
- 테스트를 하기 용이하다.

### DI 시작하기

- DI를 위해서는 ApplicationContext가 관리하도록 객체를 Bean으로 등록해야 한다. (인터페이스를 implements 한 class들에 @Component 어노테이션을 달아줘야 함 )
  - Bean으로만 등록할 경우 해당 메소드가 Bean name이 됨 
- DI에는 필드 주입, setter주입, 생성자 주입이 있지만 일반적으로 3개 중 생성자 주입을 사용하는 것이 좋다고 한다.

### Code

- config class를 생성해 어떤 객체를 사용할지 설정
  - config class에 `@ComponentScan(basePakages = "com.test.jelly")` 어노테이션을 이용해 패키지를 스캔하는 방법
  - config class에 `@ComponentScan(basePakageClasses = "JellyApllication.class")` 어노테이션을 이용해 클래스를 스캔하는 방법
  - xml에 작성하는 방법도 있지만 class자체에 설정해주는 것이 가독성이 더 좋다고 생각하기에 설명하지 않겠음
- 구현체가 여러개일 경우
  - 사용할 객체 class에 `@Primary` 어노테이션을 달아주는 방법
  - 생성자로 받을 때 `@Qualifier()` 어노테이션을 달아주는 방법  

```java
@Configuration
public class ApplicationConfig {

    @Bean
    public JellyService JellyService() {
        return new JellyService(new OrangeFrame());
    }

    @Bean
    public OrangeFrame orangeFrame() {
        return new OrangeFrame();
    }

    @Bean
    public AppleFrame appleFrame() {
        return new AppleFrame();
    }

}
```

- 설정해놓은 객체를 자동으로 주입

```java
@Service
public class JellyService {
    
//    @Autowired    필드 주입
    private final JellyFrame jellyFrame;

//    @Autowired    setter 주입
    public void setJellyFrame(JellyFrame jellyFrame) {
        this.jellyFrame = jellyFrame;
    }
    
//    @Autowired    생성자 주입
    public JellyService(
//            @Qualifier("orangeFrame") 구현체가 여러개일 경우
            JellyFrame jellyFrame) {
        this.jellyFrame = jellyFrame;
    }

}
```

### 특정 환경에서만 동작하게 하기 @Profile

- 실행/디버그 구성의 환경변수에 -Dspring.profiles.active="@Profile명"을 추가해야 한다.
  - 표현식으로 !, &, || 표현 가능하다. 
- 클래스 단위
  - `
    @Configuration
    @Profile("test")
    `
  - `
    @Component
    @Profile("test")
    `
- 메소드 단위
  - `
    @Bean
    @Profile("test")
    `

#### 참고
Lombok을 사용할 때 class에 `@RequiredArgsConstructor` 어노테이션을 달 경우 해당 class 내 `final` 또는 `@NonNull` 필드에 대해서 생성자를 생성해준다.
이로써 우리가 얻을 수 있는 것은 해당 class에 생성자가 하나만 존재하고 생성자의 파라미터 타입이 빈으로 등록 가능한 경우 `@Autowired`없이도 의존성 주입이 가능하기 떄문에 상황에 맞는다면
`@RequiredArgsConstructor`어노테이션 하나로 간단하게 DI를 사용할 수 있다.