---
layout: post
title: [JAVA] equals와 hashCode 함수
description: >
  [JAVA] equals와 hashCode 함수
  참고 
  - https://howtodoinjava.com/java/basics/java-hashcode-equals-methods/ <br>
  - https://www.youtube.com/watch?v=GfYg3imRZsc
tags: [JAVA]
---

## equals와 hashCode 함수

## equals와 hashCode 함수

equals()와 hashCode() 메소드는 모든 Java 객체의 부모 객체인 Object 클래스에 정의되어 있다. 그렇기 때문에 Java의 모든 객체는 Object 클래스에 정의된 equals와 hashCode 함수를 상속받고 있다.

### equals()란?

Object 클래스에 정의된 equals 메소드는 2개의 객체가 동일한지 검사하기 위해 사용된다. eqauls가 구현된 방법은 2개의 객체가 참조하는 것이 동일한지를 확인하는 것이다.

즉, 2개의 객체가 가리키는 곳이 동일한 메모리 주소일 경우에만 동일한 객체가 된다.

```java
//Object의 기본 equals 메서드
public boolean equals(Object obj) {
    return (this == obj);
}
```

equals() 메서드는 최상위 클래스인 Object에 포함되어 있기 때문에 모든 하위 클래스에서 재정의해 사용할 수 있다. 예를들어 동일한 문자열을 2개 생성하면 2개의 문자열은 서로 다른 메모리 상에 할당된다. 하지만 equlas()함수를 사용해 문자열을 비교하면 true 값이 나오는 것을 확인할 수 있다. 이는 하위 클래스인 String 클래스에서 equals 메소드를 오버라이딩하여 값를 비교하는 코드를 추가했기 때문이다.

```java
String s1 = new String("equalsTest");
String s2 = new String("equalsTest");
System.out.println(s1 == s2); // false
System.out.println(s1.equals(s2)); // true;

//
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

### hashCode()란?

런타임에서 개체의 고유 한 정수 값을 반환합니다. 기본적으로 정수 값은 대부분 힙에있는 개체의 메모리 주소에서 파생됩니다 (항상 필수는 아님).

hashCode는 HashTable과 같은 자료구조를 사용할 때 데이터가 저장되는 위치를 결정하기 위해 사용된다.

```java
public native int hashCode();
```

> native method : OS의 method (주로 C언어로 구현)

> 이미 작성되어 있는 method(OS가 가지고 있는 method)를 사용해 내용이 없다.
>
> -> OS가 가지고 있는 C언어로 작성된 메서드를 마치 자바로 작성된 메서드인것처럼 사용가능 (∽JNI)

### equals()와 hashCode()의 관계

일반적으로 equals()를 오버라이딩하면 hashCode()또한 같이 어버라이딩하여 **'동일한 객체에는 동일한 해시 코드가 있어야한다.'**는 관계를 지켜야한다.

이러한 equals와 hashCode의 관계를 정의하면 다음과 같다.

Java 프로그램을 실행하는 동안 equals에 사용된 정보가 수정되지 않았다면, hashCode는 항상 동일한 정수값을 반환해야 한다. (Java의 프로그램을 실행할 때 마다 달라지는 것은 상관이 없다.)

두 객체가 equals()에 의해 동일하다면, 두 객체의 hashCode() 값도 일치해야 한다.

두 객체가 equals()에 의해 동일하지 않다면, 두 객체의 hashCode() 값은 일치하지 않아도 된다.
