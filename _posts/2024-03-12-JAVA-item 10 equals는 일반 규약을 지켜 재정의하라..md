---
layout: post
title: 이펙티브 자바 - item 10. equals는 일반 규약을 지켜 재정의하라 [JAVA]
description: >
  item 6. 불필요한 객체 생성을 피하라
tags: [JAVA]
---

## 이펙티브 자바 - item 10. equals는 일반 규약을 지켜 재정의하라

## 이펙티브 자바 - item 10. equals는 일반 규약을 지켜 재정의하라

### equals 메서드를 재정의하지 않아도 되는 케이스

- **각 인스턴스가 본질적으로 고유하다.**
    - Thread와 같이 값을 표현하는 것이 아니라 동작 개체를 표현하는 클래스

- **인스턴스의 ‘논리적 동치성(logical equality)’를 검사할 일이 없다.**
    - java.util.regex.Pattern의 equals는 정규표현식이 표현은 달라도 같은 표현식 인지 검사해야하는데 이 케이스가 논리적 동치성을 검사하는 케이스
    - 이런 경우가 아니라면 Object의 기본 equals만으로 해결 가능

- **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다**
   - Set 구현체는 AbstractSet이 구현한 equals를 상속받아 사용한다. 
    ~~~java
    public abstract class AbstractSet<E> extends    AbstractCollection<E> implements Set<E> {
         public boolean equals(Object o) {
              if (o == this)
                  return true;

              if (!(o instanceof Set))
                  return false;
              Collection<?> c = (Collection<?>) o;
              if (c.size() != size())
                  return false;
              try {
                  return containsAll(c);
              } catch (ClassCastException |     NullPointerException unused) {
                  return false;
              }
          }
     }
    ~~~
    - List 구현체 : AbstractList이 구현한 equals를 상속
    - MAP 구현체 : AbstractMap이 구현한 equals를 상속

- **클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다**
    - 값 클래스여도 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스의 경우 equals를 만들 필요가 없다. 
    - equals를 호출할 일이 없고 철저하게 위험을 회피하고 싶을 땐 equals 메서드를 호출하면 예외를 던지게 작성하면 된다.
  
    ~~~java
    @Override public boolean equals(Object O) {
    	throw new AssertionError();   //호출 금지
    }
    ~~~

### equals 메서드를 재정의 해야하는 케이스

논리적 동치성을 확인해야 하는 경우이다. 상위 클래스의 equals 가 논리적 동치성을 비교하도록 정의되지 않았을 경우에만 재정의하면 된다. 주로 Integer, String과 같은 값 클래스들이 여기 해당된다.

### equals 메서드를 재정의할 때 지켜야 할 일반 규약 5가지

#### 1. 반사성(Reflexivity)
> null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.

#### 2. 대칭성(symmetry)
> null 이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true이면 y.equals(x)도 true이다.

#### 3. 추이성(transitivity)
> null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true이다.

#### 4. 일관성(consistency)
> null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

두 객체가 같다면 영원히 같아야 한다. 가변 객체와 달리 불변 클래스와 같은 경우는, 값이 수정될 일이 없기 때문에 웬만하면 불변 클래스로 만드는 것이 좋다.

주의 사항은 클래스가 불변이든 가변이든, 일관성을 깨뜨리지 않기 위해서 equals의 판단에 신뢰할 수 없는 자원이 끼어들면 안된다. 

#### 5. 모든 객체가 null과 같지 않아야 함
> null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false 이다.

~~~java
@Override public boolean equals(Object o){
	if (!o instance of MyType))
    	return false
    MyType mt = (MyType) o;
 }
~~~

instanceof 를 사용하면, NullPointerException 과 잘못된 타입이 들어왔을 때 일어날 수 있는 ClassCastException 까지 방지해주므로 명시적으로 null 검사를 하지 않아도 된다.

### equals 메서드 구현 방법

1. 연산자를 사용해 입력이 자기 자신의 참조인지 확인
성능 최적화용으로, 자기 자신이면 true 를 반환한다. (비교 작업이 복잡한 상황에서 효율적임)

~~~java
 if (o == this)
      return true;
~~~

2. instanceof 연산자로 입력이 올바른 타입인지 확인
이때 올바른 타입은, equals 가 정의된 클래스인 것이 보통이지만, 클래스가 해당 구현한 특정 인터페이스가 될 수도 있다. 그런 경우에는 equals에서 클래스가 아닌 해당 인터페이스를 사용해야 한다. Set, List, Map, Map.Entry 등의 컬렉션 인터페이스들이 여기 해당한다.
~~~java
 if (!(o instanceof PhoneNumber))
      return false;
~~~
3. 입력을 올바른 타입으로 형변환
instanceof 검사를 했기에 이 단계는 100% 성공한다.
~~~java
 PhoneNumber pn = (PhoneNumber)o;
 ~~~

4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사
모든 필드가 일치하면 true를, 하나라도 다르면 false 를 반환한다. 2단계에서 인터페이스를 사용했다면, 필드 값을 가져올때 해당 인터페이스의 메서드를 사용해야 한다.
~~~java
 return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
~~~

- 타입에 따른 비교 방법
    - 기본 타입 필드 : == 연산자
    - 참조 타입 필드 : 각각의 equals 메서드
    - float / double : Float.compare(float, float) / Double.compare(double, double)
    - 배열 : 모든 원소가 핵심 필드라면 Arrays.equals()
    - null 정상 값 취급 참조 타입 필드 : Object.equals(Object, Object)

### 주의사항
- equals를 재정의 할 땐 hashCode도 반드시 재정의하기
- 너무 복잡하게 해결하려 들지 말기
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 않기

~~~java
public boolean equals(MyClass o){  // 입력 타입은 반드시 Object
   ...
}
~~~
해당 코드는 Object.equals를 재정의 한것이 아니라 다중정의한 것이 된다. 이 메서드는 하위 클래스에서의 @Override 애너테이션이 긍정 오류(거짓 양성)을 내게하고 보안 측면에서도 잘못된 정보를 준다.


### 핵심 정리
> 꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 많은 경우에 Object.equals가 원하는 비교를 정확히 수행해주기 때문이다. 재정의해야 할 때는, 그 클래스의 핵심 필드를 모두 빠짐없이 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.