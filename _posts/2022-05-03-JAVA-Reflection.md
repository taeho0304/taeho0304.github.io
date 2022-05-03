---
layout: post
title: Java Reflection

description: >
  참고 <br>
  - 스프링핵심원리 고급편 (김영한) : <https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8>
tags: [ETC]
---
## Reflection이란? [JAVA]

## Reflection이란? [JAVA]

리플렉션은 컴파일한 클래스 정보를 활용해 동적으로 프로그래밍이 가능하도록 지원하는 자바 API이다.
리플렉션 기술을 사용하면 **클래스나 메서드의 메타정보를 동적으로 획득하고, 코드도 동적으로 호출**할 수 있다.
프레임워크나 IDE에서 이런 동적인 바인딩을 이용한 기능을 제공한다. intelliJ의 자동완성 기능, 스프링의 어노테이션이 리플렉션을 이용한 기능이다


### Java Reflection 사용 실습

#### ReflectionTest
```java
@Slf4j
public class ReflectionTest {

    @Test
    void reflection0() {
        Hello target = new Hello();
        //공통 로직1 시작
        log.info("start");
        String result1 = target.callA(); //호출하는 메서드가 다름
        log.info("result={}", result1);
        //공통 로직1 종료

        //공통 로직2 시작
        log.info("start");
        String result2 = target.callB(); //호출하는 메서드가 다름
        log.info("result={}", result2);
        //공통 로직2 종료
    }

    @Slf4j
    static class Hello {
        public String callA() {
            log.info("callA");
            return "A";
        }
        public String callB() {
            log.info("callB");
            return "B";
        }
    }
}
```

공통 로직 1과 공통 로직 2는 호출하는 메서드만 다르고 전체 코드 흐름이 완전히 같다. 이러한 경우 리플렉션을 사용해 호출하는 메서드인 target.callA(), target.callB() 부분만 동적으로 처리하여 문제를 해결 할 수 있다.


#### ReflectionTest
```java
@Test
void reflection1() throws Exception {
    //클래스 정보
    Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

    Hello target = new Hello();

    //callA 메서드 정보
    Method methodCallA = classHello.getMethod("callA");
    dynamicCall(methodCallA, target);

    //callB 메서드 정보
    Method methodCallB = classHello.getMethod("callB");
    dynamicCall(methodCallB, target);
}

private void dynamicCall(Method method, Object target) throws Exception {
    log.info("start");
    Object result = method.invoke(target);
    log.info("result={}", result);
}
```

* Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello") : 클래스 메타정보를 획득한다. 참고로 내부 클래스는 구분을 위해 $ 를 사용한다.
* classHello.getMethod("call") : 해당 클래스의 call 메서드 메타정보를 획득한다.
* method.invoke(target) : 획득한 메서드 메타정보로 실제 인스턴스의 메서드를 호출한다.
  
* dynamicCall(Method method, Object target)
  * 공통 로직1, 공통 로직2를 한번에 처리할 수 있는 통합된 공통 처리 로직이다.
  * Method method : 첫 번째 파라미터는 호출할 메서드 정보가 넘어온다. 이것이 핵심이다. 기존에는 메서드 이름을 직접 호출했지만, 이제는 Method 라는 메타정보를 통해서 호출할 메서드 정보가 동적으로 제공된다.
  * Object target : 실제 실행할 인스턴스 정보가 넘어온다. 타입이 Object 라는 것은 어떠한 인스턴스도 받을 수 있다는 뜻이다. 물론 method.invoke(target) 를 사용할 때 호출할 클래스와 메서드 정보가 서로 다르면 예외가 발생한다.

### 주의

 리플렉션 기술은 **런타임에 동작하기 때문에, 컴파일 시점에 오류를 잡을 수 없다**.
예를 들어서 지금까지 살펴본 코드에서 getMethod("callA") 안에 들어가는 문자를 실수로
getMethod("callZ") 로 작성해도 컴파일 오류가 발생하지 않는다. 그러나 해당 코드를 직접 실행하는 시점에 발생하는 오류인 **런타임 오류가 발생**한다.

가장 좋은 오류는 개발자가 즉시 확인할 수 있는 컴파일 오류이고, 가장 무서운 오류는 사용자가 직접 실행할 때 발생하는 런타임 오류다

따라서 리플렉션은 일반적으로 사용하면 안된다. 리플렉션은 **프레임워크 개발이나 또는 매우 일반적인 공통 처리가 필요할 때 부분적으로 주의해서 사용**해야 한다