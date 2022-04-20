---
layout: post
title: Filter, Interceptor [Spring]

description: >
  참고 <br>
  - 기록은 재산이다 - (Spring)Filter와 Interceptor의 차이 : <https://supawer0728.github.io/2018/04/04/spring-filter-interceptor/><br>
  - 갓대희의 작은공간 - [Spring] Filter, Interceptor, AOP 차이 및 정리 : <https://goddaehee.tistory.com/154><br>
  - catsbi's DLog - 7. 로그인 처리2 - 필터, 인터셉터 : <https://catsbi.oopy.io/9ed2ec2b-b8f3-43f7-99fa-32f69f059171>
tags: [SPRING]
---

## Filter, Interceptor, AOP 차이 개념 [Spring]

## Filter, Interceptor, AOP 차이 개념 [Spring]<br>

### 공통 프로세스에 대한 고민

자바 웹 개발을 하다보면 로그인 관련(세션체크)처리, 권한체크, XSS(Cross site script)방어, pc와 모바일웹의 분기처리, 로그, 페이지 인코딩 변환 등 **공통적으로 처리**해야 할 업무들이 많다.
 
이렇게 많은 로직에서 공통으로 관심이 있는 부분을 공통 관심사(cross-cutting concerns)라 한다. 이러한 공통 관심사를 **프로그램 흐름의 앞, 중간, 뒤에 추가하여 자동으로 처리**할 수 있는 3가지 방법이 있다.

1. Filter
2. Interceptor
3. AOP

스프링에서 사용되는 Filter, Interceptor, AOP 세 가지 기능은 모두 어떤 행동을 하기전에 먼저 실행하거나, 실행한 후에 추가적인 행동을 할 때 사용되는 기능들이다.

> 웹에 관련된 공통 관심사는 스프링 AOP 보다는 서블릿 필터, 스프링 인터셉터에서 처리하는게 좋다고 한다. 웹과 관련된 공통 관심사를 처리할 때는 HTTP의 헤더나 URL 정보가 필요한데 서블릿 필터나, 스프링 인터셉터는 HttpServletRequest를 제공하기 때문이다.



### Filter, Interceptor, AOP의 흐름

![](https://taeho0304.github.io/assets/img/SPRING/Filter&Interceptor&AOP/flow.jpg)

ㆍInterceptor와 Filter는 Servlet 단위에서 실행된다. 반면 AOP는 메소드 앞에 Proxy패턴의 형태로 실행된다.

ㆍ실행순서를 보면 Filter가 가장 밖에 있고 그안에 Interceptor, 그안에 AOP가 있는 형태이다.


따라서 요청이 들어오면 Filter → Interceptor → AOP → Interceptor → Filter 순으로 거치게 된다.

1. 서버를 실행시켜 서블릿이 올라오는 동안에 init이 실행되고, 그 후 doFilter가 실행된다. 
2. 컨트롤러에 들어가기 전 preHandler가 실행된다
3. 컨트롤러에서 나와 postHandler, afterCompletion, doFilter 순으로 진행이 된다.
4. 서블릿 종료 시 destroy가 실행된다.


### Filter, Interceptor 체인

![](https://taeho0304.github.io/assets/img/SPRING/Filter&Interceptor&AOP/chain.jpg)

* 둘 다 자유롭게 필터 및 인터셉터를 추가할 수 있다. 
* setOrder() 함수를 이용해 체인의 실행순서를 지정할 수 있다.

### Filter

말그대로 요청과 응답을 거른 뒤 정제하는 역할을 한다.

서블릿 필터는 DispatcherServlet 이전에 실행이 되는데 필터가 동작하도록 지정된 자원의 앞단에서 요청내용을 변경하거나,  여러가지 체크를 수행할 수 있다. 

또한 자원의 처리가 끝난 후 응답내용에 대해서도 변경하는 처리를 할 수가 있다. 

일반적으로 인코딩 변환 처리, XSS방어 등의 요청에 대한 처리등에 사용된다.

#### Filter Interface 함수

* init(): 필터 초기화 메서드로 서블릿 컨테이너가 생성될 때 호출된다.
* doFilter(): 고객의 요청이 올 때마다 해당 메서드가 호출된다. (전/후 처리)
* destroy(): 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

>init, destroy 메서드는 default 메서드 이기에 따로 구현하지 않아도 된다.

#### 용도 및 예시

* 공통된 보안 및 인증/인가 관련 작업
* 모든 요청에 대한 로깅 또는 감사
* 이미지/데이터 압축 및 문자열 인코딩
* Spring과 분리되어야 하는 기능
  
필터에서는 기본적으로 **스프링과 무관하게 전역적으로 처리해야 하는 작업**들을 처리할 수 있다.

대표적인 예시로 보안과 관련된 공통 작업이 있다. 필터는 인터셉터보다 앞단에서 동작하기 때문에 전역적으로 해야하는 보안 검사(XSS 방어 등)를 하여 올바른 요청이 아닐 경우 차단을 할 수 있다. 그러면 스프링 컨테이너까지 요청이 전달되지 못하고 차단되므로 안정성을 더욱 높일 수 있다.

또한 필터는 이미지나 데이터의 압축이나 문자열 인코딩과 같이 **웹 애플리케이션에 전반적으로 사용되는 기능**을 구현하기에 적당하다. Filter는 다음 체인으로 넘기는 ServletRequest/ServletResponse 객체를 조작할 수 있다는 점에서 Interceptor보다 훨씬 강력한 기술이다.



### Interceptor

요청에 대한 작업 전/후로 가로챈다고 보면 된다.

필터는 스프링 컨텍스트 외부에 존재하여 스프링과 무관한 자원에 대해 동작한다. 

하지만 인터셉터는 스프링의 DistpatcherServlet이 컨트롤러를 호출하기 전, 후로 끼어들기 때문에 스프링 컨텍스트(Context, 영역) 내부에서 Controller(Handler)에 관한 요청과 응답에 대해 처리한다.

스프링의 모든 빈 객체에 접근할 수 있다.

인터셉터는 여러 개를 사용할 수 있고 로그인 체크, 권한체크, 프로그램 실행시간 계산작업 로그확인 등의 업무처리에 주로 사용된다.

Interceptor를 구현하는 방법은 2가지가 있는데, HandlerInterceptor 인터페이스를 구현하는 방법과 HandlerInterceptorAdapter 클래스를 상속 받는 방법이 있다.

#### Interceptor 메서드

![](https://taeho0304.github.io/assets/img/SPRING/Filter&Interceptor&AOP/interceptor-flow.png)

* (1) preHandler()
  
preHandle 메소드는 컨트롤러가 호출되기 전에 실행된다. 그렇기 때문에 컨트롤러 이전에 처리해야 하는 전처리 작업이나 요청 정보를 가공하거나 추가하는 경우에 사용할 수 있다.

preHandle의 3번째 파라미터인 handler 파라미터는 핸들러 매핑이 찾아준 컨트롤러 빈에 매핑되는 HandlerMethod라는 새로운 타입의 객체로써, @RequestMapping이 붙은 메소드의 정보를 추상화한 객체이다.

또한 preHandle의 반환 타입은 boolean인데 반환값이 true이면 다음 단계로 진행이 되지만, false라면 작업을 중단하여 이후의 작업(다음 인터셉터 또는 컨트롤러)은 진행되지 않는다.


* (4) postHandler()
  
postHandle 메소드는 컨트롤러를 호출된 후에 실행된다. 그렇기 때문에 컨트롤러 이후에 처리해야 하는 후처리 작업이 있을 때 사용할 수 있다. 이 메소드에는 컨트롤러가 반환하는 ModelAndView 타입의 정보가 제공되는데, 최근에는 Json 형태로 데이터를 제공하는 RestAPI 기반의 컨트롤러(@RestController)를 만들면서 자주 사용되지는 않는다.


* (6) afterCompletion(): 뷰가 렌더링 된 후에 호출된다. 

afterCompletion 메소드는 이름 그대로 모든 뷰에서 최종 결과를 생성하는 일을 포함해 모든 작업이 완료된 후에 실행된다. 요청 처리 중에 사용한 리소스를 반환할 때 사용하기에 적합하다.

#### 예외 발생

![](https://taeho0304.github.io/assets/img/SPRING/Filter&Interceptor&AOP/exception.png)

* preHandle은 컨트롤러 호출 전에 호출된다. 
* postHandler은 컨트롤러에서 예외가 발생하면 postHandler은 호출되지 않는다.
* afterCompletion은 항상 호출된다. (try-catch의 finally처럼) 그렇기에 이전에 발생한 예외가 있을 경우 이를 파라미터로 받아서 어떤 예외가 발생했는지 확인할 수 있다

#### 용도 및 예시

* 세부적인 보안 및 인증/인가 공통 작업
* API 호출에 대한 로깅 또는 감사
* Controller로 넘겨주는 정보(데이터)의 가공
  
인터셉터에서는 **클라이언트의 요청과 관련되어 전역적으로 처리해야 하는 작업**들을 처리할 수 있다.

대표적으로 세부적으로 적용해야 하는 인증이나 인가와 같이 클라이언트 요청과 관련된 작업 등이 있다. 예를 들어 특정 그룹의 사용자는 어떤 기능을 사용하지 못하는 경우가 있는데, 이러한 작업들은 컨트롤러로 넘어가기 전에 검사해야 하므로 인터셉터가 처리하기에 적합하다.

또한 인터셉터는 필터와 다르게 HttpServletRequest나 HttpServletResponse 등과 같은 객체를 제공받으므로 객체 자체를 조작할 수는 없다. 대신 해당 객체가 내부적으로 갖는 값은 조작할 수 있으므로 컨트롤러로 넘겨주기 위한 정보를 가공하기에 용이하다. 예를 들어 JWT 토큰 정보를 파싱해서 컨트롤러에게 사용자의 정보를 제공하도록 가공할 수 있는 것이다.

그 외에도 우리는 다양한 목적으로 API 호출에 대한 정보들을 기록해야 할 수 있다. 이러한 경우에 HttpServletRequest나 HttpServletResponse를 제공해주는 인터셉터는 클라이언트의 IP나 요청 정보들을 포함해 기록하기에 용이하다.



#### AOP

객체 지향의 프로그래밍을 했을 때 중복을 줄일 수 없는 부분을 줄이기 위해 종단면(관점)에서 바라보고 처리한다. 흩어진 관심사를 Aspect로 모듈화하고 핵심적인 비즈니스 로직에서 분리하여 재사용하겠다는 것이 AOP의 취지다.

주로 '로깅', '트랜잭션', '에러 처리'등 비즈니스단의 메서드에서 조금 더 세밀하게 조정하고 싶을 때 사용한다.

Interceptor나 Filter와는 달리 메소드 전후의 지점에 자유롭게 설정이 가능하다. 또한, Interceptor와 Filter는 주소로 대상을 구분해서 걸러내야하는 반면, AOP는 주소, 파라미터, 애노테이션 등 다양한 방법으로 대상을 지정할 수 있다.

AOP의 Advice와 HandlerInterceptor의 가장 큰 차이는 파라미터의 차이다.

Advice의 경우 JoinPoint나 ProceedingJoinPoint 등을 활용해서 호출한다.

반면 HandlerInterceptor는 Filter와 유사하게 HttpServletRequest, HttpServletResponse를 파라미터로 사용한다.

#### 주요 개념

* Aspect : 위에서 설명한 흩어진 관심사를 모듈화 한 것. 주로 부가기능을 모듈화함.
* Target : Aspect를 적용하는 곳 (클래스, 메서드 .. )
* Advice : 실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 부가기능을 담은 구현체
* JointPoint : Advice가 적용될 위치, 끼어들 수 있는 지점. 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능
* PointCut : JointPoint의 상세한 스펙을 정의한 것. 'A란 메서드의 진입 시점에 호출할 것'과 같이 더욱 구체적으로 Advice가 실행될 지점을 정할 수 있음
 

#### 스프링 AOP 특징

* 프록시 패턴 기반의 AOP 구현체, 프록시 객체를 쓰는 이유는 접근 제어 및 부가기능을 추가하기 위함이다.
* 스프링 빈에만 AOP를 적용 가능하다.
* 모든 AOP 기능을 제공하는 것이 아닌 스프링 IoC와 연동하여 엔터프라이즈 애플리케이션에서 가장 흔한 문제(중복코드, 프록시 클래스 작성의 번거로움, 객체들 간 관계 복잡도 증가 ...)에 대한 해결책을 지원하는 것이 목적이다.

#### AOP의 포인트컷

* @Before (이전) : 어드바이스 타겟 메소드가 호출되기 전에 어드바이스 기능을 수행한다.
* @After (이후) : 타겟 메소드의 결과에 관계없이(즉 성공, 예외 관계없이) 타겟 메소드가 완료 되면 어드바이스 기능을 수행한다.
* @AfterReturning (정상적 반환 이후)타겟 메소드가 성공적으로 결과값을 반환 후에 어드바이스 기능을 수행한다.
* @AfterThrowing (예외 발생 이후) : 타겟 메소드가 수행 중 예외를 던지게 되면 어드바이스 기능을 수행한다.
* @Around (메소드 실행 전후) : 어드바이스가 타겟 메소드를 감싸서 타겟 메소드 호출전과 후에 어드바이스 기능을 수행한다.
