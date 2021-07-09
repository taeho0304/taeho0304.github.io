---
layout: post
title: equals와 hashCode 함수 [JAVA]
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

일반적으로 equals()를 오버라이딩하면 hashCode()또한 같이 어버라이딩하여 **'동일한 객체에는 동일한 해시 코드가 있어야한다'**는 관계를 지켜야한다.

이러한 equals와 hashCode의 관계를 정의하면 다음과 같다.

1. Java 응용 프로그램을 실행하는 동안 equals에 사용된 정보가 수정되지 않았다면, hashCode는 항상 동일한 정수값을 반환해야 한다.

2. 두 객체가 equals()에 의해 동일하다면, 두 객체의 hashCode() 값도 일치해야 한다.

3. 두 객체가 equals()에 의해 동일하지 않다면, 두 객체의 hashCode() 값은 일치하지 않아도 된다.

### hashCode() 및 equals()의 Override

#### 기본동작

만약 애플리케이션에 아래와 같은 Employee 클래스가 있다고 하자.

###### Employee.java

```java
public class Employee{
    private Integer id;
    private String firstname;
    private String lastName;
    private String department;

    //Setters and Getters
}
```

만약 아래와 같이 동일한 id 값을 갖는 2개의 Employ를 서로 다른 처리 과정에 의해 얻었다고 하자. 2개의 Employee는 동일한 id를 갖기 때문에 equals 연산을 한다면 true를 반환해야 한다. 하지만 아래의 예제는 깊게 볼 필요도 없이 false를 반환할 것이다.

```java
public class EqualsTest {
    public static void main(String[] args) {
        Employee e1 = new Employee();
        Employee e2 = new Employee();

        e1.setId(100);
        e2.setId(100);

        System.out.println(e1.equals(e2));  //false
    }
}
```

올바른 동작을 얻기 위해서는 Employee 클래스내에 equals() 메소드를 오버라이드 해야 한다.

```java
public boolean equals(Object o) {
    if(o == null) {
        return false;
    }
    if (o == this) {
        return true;
    }
    if (getClass() != o.getClass()) {
        return false;
    }

    Employee e = (Employee) o;
    return (this.getId() == e.getId());
}
```

위와 같이 equals() 메소드를 오버라이드하면 eqauls() 메서드가 원하는대로 수행된다.

하지만 Employee를 HashSet과 같은 자료구조에 저장하면 또 다른 문제가 발생한다.

```java
import java.util.HashSet;
import java.util.Set;

public class EqualsTest
{
    public static void main(String[] args)
    {
        Employee e1 = new Employee();
        Employee e2 = new Employee();

        e1.setId(100);
        e2.setId(100);

        //Prints 'true'
        System.out.println(e1.equals(e2));

        Set<Employee> employees = new HashSet<Employee>();
        employees.add(e1);
        employees.add(e2);

        System.out.println(employees);  // [study.Employee@2a139a55, study.Employee@15db9742]
    }
}
```

hashCode() 메소드는 해당 메모리 주소값을 이용해 정수값을 반환한다. 그렇기 때문에 위의 e1과 e2는 다른 해시값을 가져 HashSet에 2개의 객체가 서로 다른 위치에 저장된다. 따라서 equals()를 수정하면 hashCode()또한 수정이 필요하다.

```java
@Override
public int hashCode()
{
    final int PRIME = 31;
    int result = 1;
    result = PRIME * result + getId();
    return result;
}
```

위와 같이 Employee 클래스내에 hashCode() 메서드를 오버라이드 작업을 수행하면 동일한 객체 하나만 출력하는 것을 볼 수 있다.

```java
import java.util.HashSet;
import java.util.Set;

public class EqualsTest
{
    public static void main(String[] args)
    {
        Employee e1 = new Employee();
        Employee e2 = new Employee();

        e1.setId(100);
        e2.setId(100);

        //Prints 'true'
        System.out.println(e1.equals(e2));

        Set<Employee> employees = new HashSet<Employee>();
        employees.add(e1);
        employees.add(e2);

        System.out.println(employees); // [study.Employee@83]
    }
}
```
