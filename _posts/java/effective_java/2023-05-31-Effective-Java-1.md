---
layout: post
title: Effective Java (1)
date: 2023-05-31
categories: [java, effective]
tags: [java, effective]
---
# Effective Java (1)

***
# Ch1 들어가기
- 책은 총 90개의 아이템
- 보는 순서는 무관
- 책의 규칙은 명료성(clarity)과 단순성(simplicity)
- 컴포넌트는 사용자를 놀라게 하는 일 없이, 정해진 동작이나 예측할 수 있는 동작만 수행
  - 여기서 컴포넌트란, 메서드부터 여러 패키지로 된 복잡한 프레임워크까지 재사용 가능한 모든 소프트웨어 요소
- 코드는 복사가 아닌 재사용
- 컴포넌트 간 의존성 최소로 유지
- 위에 나열된 것들을 어겨도 되나, 합당한 이유가 필요

### 용어 정리
- 메서드 시그니처: 메서드 이름과 입력 매개변수의 타입들(반환 타입은 제외)
- 상속(inheritance)와 서브클래싱(subclassing)은 같은 용어
- 클래스가 인터페이스를 상속이 아닌 구현(implement)
- 인터페이스가 인터페이스를 확장(extend)
- 나머지 자바 용어는 자바 언어 명세와 같다.
- 공개 API(exported API) = API(application programming interface)
  - 프로그래머가 접근할 수 있는 모든 클래스, 인터페이스, 생성자, 멤버, 직렬화된 형태
- API를 사용하는 사람: user(사용자)
- API를 사용하는 코드: client

### java9의 module system
모듈 개념을 적용하면, 공개하겠다고 한 패키지들의 공개 API만 공개된다. 즉 공개할 패키지를 선택할 수 있다.

***
# Ch2 객체 생성과 파괴
- 객체를 만들어야 할 때와 만들지 말아야 할 떄를 구분하는 법
- 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법
- 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령

***
## item1. 생성자 대신 정적 팩토리 메서드를 고려하라
- 디자인 패턴에서 말하는 팩토리 메서드(Factory Method)와 정적 팩토리 메서드는 다르다.
- 클래스는 client에 public constructor 대신(혹은 생성자와 함께) 정적 팩토리 메서드를 제공 가능

### 정적 팩토리 메서드의 장점(사용할 이유)
1. 이름을 가질 수 있다.
   - 생성자는 모두 이름이 같다! 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명할 수 없다.
   - 매개변수의 순서와 타입이 같아도 이름을 다르게 할 수 있다. -> 이런 상황이 필요하면 사용!
2. 호출될 떄마다 인스턴스를 새로 생성하지 않아도 된다.(선택)
   - 불변 클래스(immutable class)는 인스턴스를 미리 생성할 수 있다.
   - 새로 생성한 인스턴스를 캐싱하여 재활용할 수 있다.
   - 불필요한 객체 생성을 줄일 수 있다.
   - `Flyweight pattern`도 비슷한 기법
   - 언제 어느 인스턴스를 살아 있게 할지를 통제할 수 있다.
     - 싱글톤 가능
     - 인스턴스화 불가(noninstantiable)
3. 반환 타입의 하위 타입 객체를 반환할 수 있다.
   - 반환할 객체의 클래스를 선택할 수 있게 하는 유연성 제공
   - 구현 클래스를 공개하지 않고 반환 가능하기 때문에 API를 작게 유지
4. 입력 매개변수에 따라 매번 다른 클래스의 객체 반환 가능
   - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 무관
   - 다음 릴리즈 때 객체가 변경돼도 무관
5. 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 됨
   - 이런 유연함은 서비스 제공자 프레임워크(service provider framework)를 만드는 근간

### service provider framework
- JDBC(Java Database Connectivity)가 대표적
- provider는 서비스의 구현체
- 구현체들을 클라이언트에게 제공하는 역할을 프레임워크가 통제 -> 클라이언트를 구현체로부터 분리
- 3개의 핵심 컴포넌트로 구성(4번은 종종 쓰이는 컴포넌트)
  1. 서비스 인터페이스(service interface): 동작을 정의(~~client를 위한 설명서~~)
     - JDBC의 Connection
  2. 제공자 등록 API(provider registration API): 제공자가 구현체를 등록할 떄 사용(~~구현체 등록~~)
     - JDBC의 DriverManager, registerDriver
  3. 서비스 접근 API(service access API): 클라이언트가 서비스의 인스턴스를 얻을 때 사용(~~구현체 생성(제공)~~)
     - JDBC의 DriverManager.getConnection
     - client는 원하는 구현체의 조건을 명시
     - 조건을 명시하지 않으면 기본 구현체 반환
     - 이 부분이 `유연한 정적 팩토리`의 핵심
  4. 서비스 제공자 인터페이스(service provider interface): 서비스 인터페이스의 인스턴스를 생성하는 팩토리 객체를 설명
     - JDBC의 Driver
     - 서비스 제공자 인터페이스가 없으면 각 구현체를 인스턴스로 만들 때 리플렉션을 사용
     - java6부터 `ServiceLoader`제공(https://alkhwa-113.tistory.com/entry/ServiceLoader)

### 정적 팩토리 메서드의 단점
1. 상속을 하려면 public이나 protected 생성자가 필요하다.
   - 정적 팩토리 메소드만 제공하면 하위 클래스를 만들 수 없다.
   - 상속보다 구성(composition)을 유도, 불변 타입을 위해서는 이 제약을 지켜야하기 때문에 장점으로 볼 수 있다.
2. 프로그래머가 정적 팩토리 메서드를 찾기 어렵다.
   - API 설명에 명확히 드러나지 않으므로, 메서드를 확인하며 인스턴스화할 방법을 알아내야 한다.
   - 이를 위해 널리 알려진 규약을 따라 메서드 이름을 지어 문제를 완화

### 정적 팩토리 메서드 네이밍 규약
- from
  - 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환
  - `Date d = Date.from(instant);`
- of
  - 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환
  - `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`
- valueOf
  - from과 of의 더 자세한 버전
  - `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`
- instance / getInstance
  - 매개변수로 명시한 인스턴스를 반환, 같은 인스턴스를 보장하지는 않음
  - `StackWalker = luke = StackWalker.getInstance(options);`
- create / newInstance
  - 매번 새로운 인스턴스를 생성해 반환함을 보장
  - `Object newArray = Array.newInstance(classObject, arrayLen);`
- getType
  - 생성할 클래스가 아닌 다른 클래스에 팩토리 메서드를 정의, Type은 반환할 객체의 타입
  - `FileStore fs = Files.getFileStore(path);`
- newType
  - 매번 새로운 인스턴스를 생성해 반환함을 보장
  - `BufferedReader br = Files.newBufferedReader(path);`
- type
  - getType, newType의 간결한 버전
  - `List<Complaint> litany = Collections.list(legacyLitany);`

#### 결론
- 정적 팩토리 메서드와 public 생성자는 각자의 쓰임새가 있다.
- 상황에 맞게 장단점을 이해하고 사용하는 것이 좋다.
- 정적 팩토리 메서드를 사용하는 게 유리한 경우가 더 많으니 무작정 public 생성자를 작성하던 습관을 고치자































