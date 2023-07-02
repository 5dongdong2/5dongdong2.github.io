---
layout: post
title: Effective Java
date: 2023-05-31
categories: [java, effective]
tags: [java, effective]
---
# Effective Java

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











