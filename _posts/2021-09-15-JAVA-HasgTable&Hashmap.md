---
layout: post
title: HashTable과 HashMap [JAVA]
description: >
  [JAVA] HashTable과 HashMap <br>
  참고 <br>
  - https://howtodoinjava.com/java/basics/java-hashcode-equals-methods/ <br>
  - https://www.youtube.com/watch?v=GfYg3imRZsc
tags: []
---

## HashTable과 HashMap

## HashTable과 HashMap

Hashtable 이란?
Hashtable 클래스는 컬렉션 프레임웍이 만들어지기 이전부터 존재하던 것이기 때문에 컬렉션 프레임워의 명명법을 따르지 않습니다.

Vector나 Hashtable과 같은 기존의 컬렉션 클래스들은 호환을 위해, 설계를 변경해서 남겨두었지만 가능하면 사용하지 않는 것이 좋습니다. (대신 ArrayList와 HashMap을 사용하는 것이 좋습니다.)

 

Hashtable는 자바에서 해시 테이블을 구현한 클래스 중 가장 오래되었습니다. 그리고 두 번째로 구현한 클래스는 HashMap 클래스입니다. 즉, 일반적으로 hashMap과 사용법이 거의 동일합니다. (예를들면 key - value 형태이고 key는 중복될 수 없고, value는 중복될 수 있다는 특징들 입니다.)

public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {

    private float loadFactor;

    public Hashtable(int initialCapacity, float loadFactor) {

    }

    public Hashtable(int initialCapacity) {
        this(initialCapacity, 0.75f);
    }

    public Hashtable() {
        this(11, 0.75f);
    }
}
위와 같이 기본 생성자로 객체를 생성하게 되면 초기용량(버킷의 수) = 11, 로드팩터 = 0.75로 설정됩니다.
버킷의 수와 로드팩터, HashMap에 대해서 좀 더 자세히 알고 싶다면 여기 에서 확인하면 됩니다. 위의 글을 읽고 왔다면 해시테이블 자료구조에 대해서는 어느정도 안다고 가정하겠습니다.

 

이번 글에서는 HashMap과 Hashtable의 차이점에 대해서 정리를 해보겠습니다.

 

HashMap과 Hashtable 클래스의 차이점
Thread-safe 여부
Hashtable은 Thread-safe하고, HashMap은 Thread-safe하지 않다는 특징을 가지고 있습니다. 그렇기에 멀티스레드 환경이 아니라면 Hashtable은 HashMap 보다 성능이 떨어진다는 단점을 가지고 있습니다.
Null 값 허용 여부
Hashtable은 key에 null을 허용하지 않지만, HashMap은 key에 null을 허용합니다.
Enumeration 여부
Hashtable은 not fail-fast Enumeration을 제공하지만, HashMap은 Enumeration을 제공하지 않습니다.
HashMap은 보조해시를 사용하기 때문에 보조 해시 함수를 사용하지 않는 Hashtable에 비하여 해시 충돌(hash collision)이 덜 발생할 수 있어 상대적으로 성능상 이점이 있습니다.
최근까지 Hashtable은 구현에 거의 변화가 없지만, HashMap은 현재까지도 지속적으로 개선되고 있습니다.
