---
layout: post
title: CGlib vs Dynamic Proxy

description: >

tags: [ETC]
---

## CGlib vs Dynamic Proxy [Spring]

## CGlib vs Dynamic Proxy [Spring]

## Dynamic Proxy

JDK 에서 제공하는 Dynamic Proxy는 1.3 버젼부터 생긴 기능이며, Interface를 기반으로 Proxy를 생성해주는 방식이다. 따라서 JDK 동적 프록시는 **인터페이스가 필수적이라는 단점**이 있다.

또한 Invocation Handler를 상속받아서 실체를 구현하게 되는데, 이 과정에서 특정 Object에 대해 **Reflection을 사용하기 때문에 성능이 조금 떨어지는 단점**이 있다.

#### AImpl

```java
@Slf4j
public class AImpl implements AInterface {
    @Override
    public String call() {
        log.info("A 호출");
        return "a";
    }
}
```

### JDK 동적 프록시가 제공하는 InvocationHandler

```java
package java.lang.reflect;

public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

- Object proxy : 프록시 자신
- Method method : 호출한 메서드
- Object[] args : 메서드를 호출할 때 전달한 인수

### TimeInvocationHandler

InvocationHandler를 상속받아 구현한 예시는 다음과 같다.

```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    private final Object target; // 동적 프록시가 호출할 대상

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws
            Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = method.invoke(target, args); // invoke 함수를 통해 동적으로 proxy 적용 가능

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```

### JdkDynamicProxyTest

```java
@Test
  void dynamicA() {
      AInterface target = new AImpl(); // target
      TimeInvocationHandler handler = new TimeInvocationHandler(target); // InvocationHandler
      AInterface proxy = (AInterface)
              Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]
                      {AInterface.class}, handler);
      proxy.call();
      log.info("targetClass={}", target.getClass());
      log.info("proxyClass={}", proxy.getClass());
  }
```

> TimeInvocationHandler - TimeProxy 실행
> AImpl - A 호출
> TimeInvocationHandler - TimeProxy 종료 resultTime=0
> JdkDynamicProxyTest - targetClass=class hello.proxy.jdkdynamic.code.AImpl
> JdkDynamicProxyTest - proxyClass=class com.sun.proxy.$Prox

#### 실행순서

![](https://taeho0304.github.io/assets/img/Etc/cglib/d-p.png)

## CGLIB

CGLIB(Code Generator Libray)는 클래스의 바이트 코드를 조작하여 프록시 객체를 생성해 주는 라이브러리다. CGLIB를 사용하면 인터페이스가 아닌 타겟 클래스에 대해서도 프록시 객체를 만들어 줄 수 있다.

스프링의 ProxyFactory라는 것이 이 기술을 편리하게 사용하게 도와주기 때문에, 너무 깊이있게 파기 보다는 CGLIB가 무엇인지 대략 개념만 잡도록 하자!

### MethodInterceptor - CGLIB 제공

DK 동적 프록시에서 실행 로직을 위해 InvocationHandler 를 제공했듯이, CGLIB는
MethodInterceptor를 제공한다.

```java
package org.springframework.cglib.proxy;

public interface MethodInterceptor extends Callback {
    Object intercept(Object obj, Method method, Object[] args, MethodProxy
            proxy) throws Throwable;
}
```

### TimeMethodInterceptor

```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {

  private final Object target;

  public TimeMethodInterceptor(Object target) {
      this.target = target;
  }

  @Override
  public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
      log.info("TimeProxy 실행");
      long startTime = System.currentTimeMillis();

      Object result = proxy.invoke(target, args); // 실제 대상을 동적으로 호출

      long endTime = System.currentTimeMillis();
      long resultTime = endTime - startTime;
      log.info("TimeProxy 종료 resultTime={}", resultTime);
      return result;
  }
}
```

- obj : CGLIB가 적용된 객체
- method : 호출된 메서드
- args : 메서드를 호출하면서 전달된 인수
- proxy : 메서드 호출에 사용

### TimeMethodInterceptor

```java
@Test
  void cglib() {
    ConcreteService target = new ConcreteService();

    Enhancer enhancer = new Enhancer(); // CGLIB는 Enhancer 를 사용해서 프록시를 생성
    enhancer.setSuperclass(ConcreteService.class); //  CGLIB는 구체 클래스를 상속 받아서 프록시를 생성할 수 있다. 어떤 구체 클래스를 상속 받을지 지정
    enhancer.setCallback(new TimeMethodInterceptor(target)); // 프록시에 적용할 실행 로직을 할당
    ConcreteService proxy = (ConcreteService)enhancer.create(); // 프록시를 생성한다. 앞서 설정한 enhancer.setSuperclass(ConcreteService.class) 에서 지정한 클래스를 상속 받아서 프록시가 만들어진다
    log.info("targetClass={}", target.getClass());
    log.info("proxyClass={}", proxy.getClass());

    proxy.call();
  }
```

> CglibTest - proxyClass=class hello.proxy.common.service.ConcreteService$
> $EnhancerByCGLIB$$25d6b0e3
> TimeMethodInterceptor - TimeProxy 실행
> ConcreteService - ConcreteService 호출
> TimeMethodInterceptor - TimeProxy 종료 resultTime=9

#### 실행순서

![](https://taeho0304.github.io/assets/img/Etc/cglib/cglib.png)

### 성능차이

- CGIB는 타겟에 대한 정보를 직접적으로 제공 받고, 타겟 클래스에 대한 바이트 코드를 조작하여 프록시를 생성하므로 리플렉션을 사용하는 JDK Dynamic Proxy에 비해 성능이 좋다.

- CGLIB는 메소드가 처음 호출되었을 때 동적으로 타겟 클래스의 바이트 코드를 조작하고, 이후 호출 시엔 조작된 바이트 코드를 재사용한다
