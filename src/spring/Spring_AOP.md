# Spring AOP (관점지향 프로그래밍

- 여러 메서드에서 동일한 코드가 반복될 경우 AOP를 사용해 한곳에서 처리할 수 있음 (로깅, 트랜잭션, 인증 등)
- spring에서 AOP는 Proxy를 이용하는 방식

### 기본 개념

- Aspect (관점, 관심)
  - 여러 클래스나 기능에 중복되는 관심사를 모듈화
  - 가장 많이 활용되는 부분은 `@Transactional`, `Cacheable` 기능
- Advice (조언)
  - AOP에서 실제로 적용하는 기능
- Join Point (연결 포인트)
  - 모듈화된 특정 기능이 실행될 수 있는 연결 포인트 (모듈화한 기능을 어디에 사용할지)
- Pointcut (포인트 선택 방법)
  - Join point 중 해당 Aspect를(로깅, 트랜잭션 등) 적용할 대상을 뽑을 조건식
- Target Object
  - Advice가 적용될 대상 오브젝트
- AOP Porxy
  - 대상 오브젝트에 Aspect를 적용하는 경우 Advice를 덧붙이기 위해 하는 작업
  - 주로 CGLIB(Code Generation Library, 실행 중 실시간으로 코드를 생성하는 라이브러리) 프록시를 사용해 프록싱 처리를 함
- Weaving
  - Advice를 비즈니스 로직 코드에 삽입하는 것

#### AspectJ 

- AOP를 제대로 사용하기 위해 반드시 필요한 라이브러리

### Code

- Pointcut 생성

```java
@Aspect
@Component
public class AspectTest{
    
    // public 메소드 대상 포인트 컷
    @Pointcut("execution(public * *(..))")  
    private void methodPointCut() {}
  
    // 특정 패키지 대상 포인트 컷
    @Pointcut("within(com.test.aspect..*)")
    private void pacagePointCut() {}
  
    // 위 두 조건을 and 조건으로 묶은 포인트 컷
    @Pointcut("methodPointCut() && pacagePointCut()")
    private void andPointCur() {}
}
```

- Before Advice
  - 내가 정의한 포인트컷이 실행되기 바로 전에 실행되는 로직
```java
@Aspect
public class BeforeTest {
    
    @Before("com.test.my.AspectTest.methodPointCut()")
    public void doAccessCheck() {
        
    }
}
```

- After Returning Advice 
  - 내가 정의한 포인트컷에서 return이 발생된 후 실행되는 로직

```java
@Aspect
public class AfterReturningTest {
    
    @Before("com.test.my.AspectTest.methodPointCut()")
    public void doAccessCheck() {
        
    }
}
```
- Around Advice
- `TestService()`포인트컷의 전, 후에 필요한 동작을 추가하기
  - ProceedingJoinPoint = 실제로 동작하는 코드

```java
@Aspect
public class AroundTest {
    
    @Around("com.test.my.AspectTest.TestService()")
    public Object aroundAdvice(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        // 전
        Object oValue = proceedingJoinPoint.proceed();
        // 후
        return oValue;
    }
}
```