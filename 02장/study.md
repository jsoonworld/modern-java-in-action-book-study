# ch02: 동작 파라미터화 코드 전달하기

 우리가 어떤 상황에서 일을 하든 소비자의 요구사항은 항상 바뀐다.
 동작 파라미터화`behavior parameterization`를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다. 동작 파라미터화는 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다. 이 코드 블록은 나중에 프로그램에서 호출한다. 즉, 코드 블록의 실행은 나중으로 미뤄진다.

## 2.1 변화하는 요구사항에 대응하기

변화에 대응하는 코드를 구현하는 것은 어려운 일이다. 일단 하나의 예제를 선정한 다음 예제 코드를 점차 개선하면서 유연한 코드를 만드는 모범사례를 보여줄 것이다.

기존의 농장 재고 목록 애플리케이션에 리스트에서 녹색`green` 사과만 필터링 하는 기능을 추가한다고 가정하자.

### 2.1.1 첫 번째 시도: 녹색 사과 필터링

```java
public enum Color {  
    RED, GREEN  
}
```

```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {  
    List<Apple> result = new ArrayList<>();  
    for (Apple apple : inventory) {  
        if (Color.GREEN.equals(apple.getColor())) {  
            result.add(apple);  
        }  
    }  
    return result;  
}
```

그런데 갑자기 농부가 변심하여 녹색 사과 말고 빨간`red` 사과도 필터링하고 싶어졌다. 어떻게 고쳐야 할까?

거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.

### 2.1.2 두 번째 시도 : 색을 파라미터화

어떻게 해야 `filterGreenApples`의 코드를 반복 사용하지 않고 `filterRedApples`를 구현할 수 있을까? 
색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하면 변화하는 요구사항에 좀 더 유연하게 대응하는 코드를 만들 수 있다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {  
    List<Apple> result = new ArrayList<>();  
    for (Apple apple : inventory) {  
        if (apple.getColor().equals(color)) {  
            result.add(apple);  
        }  
    }  
    return result;  
}
```

이제 농부도 만족할 것이다. 그런데 갑자기 농부가 다시 나타나서는 '색 이외에도 가벼운 사과와 무거운 사과로 구분할 수 있으면 정말 좋겠네요. 보통 무게가 150그램 이상인 사과가 무거운 사과입니다.'

농부의 다양한 요구사항을 듣다보면 색과 마찬가지로 앞으로 무게의 기준도 얼마든지 바뀔 수 있다는 사실을 눈치 챘을 것이다. 그래서 앞으로 바뀔 수 있는 다양한 무게에 대응할 수 있도록 무게 정보 파라미터도 추가했다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {  
    List<Apple> result = new ArrayList<>();  
    for (Apple apple : inventory) {  
        if (apple.getWeight() > weight) {  
            result.add(apple);  
        }  
    }  
    return result;
```

하지만 구현 코드를 자세히 보면 목록을 검색하고, 각 사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 중복된다. 이는 소프트웨어 공학의 DRY`don't repeat yourself` (같은 것을 반복하지 말 것) 원칙을 어기는 것이다. 탐색 과정을 고쳐서 성능을 개선하려면 무슨 일이 일어날까? 한 줄이 아니라 메서드 전체 구현을 고쳐야 한다. 즉, 엔지니어링적으로 비싼 대가를 치러야 한다.

### 세 번째 시도: 가능한 모든 속성으로 필터링

색이나 무게 중 어떤 것을 기준으로 필터링할지 가리키는 플래그를 추가할 수 있다. 하지만 실전에서는 절대 이 방법을 사용하지 말아야 한다.

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {  
    List<Apple> result = new ArrayList<>();  
    for (Apple apple : inventory) {  
        if (flag && apple.getColor().equals(color) || !flag && apple.getWeight() > weight) {  
            result.add(apple);  
        }  
    }  
    return result;  
}
```

```java
List<Apple> greenApples = filterApples(inventory, Green, 0, true);
List<Apple> greenApples = filterApples(inventory, null, 150, false);
```

위 같이 코드를 사용하게 되면, `true`, `false` 가 무엇을 의미하는지 모르고, 앞으로 요구사항이 바뀌었을 대 유연하게 대응할 수도 없다.

`filterApples`에 어떤 기준으로 사과를 필터링할 것인지 효과적으로 전달할 수 있다면 더 좋을 것이다.

## 2. 2  동작 파라미터화

한 걸음 물러서서 전체를 보자. 우리의 선택 조건을 다음처럼 결정할 수 있다. 사과의 어떤 속성에 기초해서 불리언값을 반환(예를 들어 사과가 녹색인가? 150그램 이상인가?)하는 방법이 있다. 참 또는 거짓을 반환하는 함수를 프레디케이트라고 한다. 선택 조건을 결정하는 인터페이스를 정의하자.

```java
public interface ApplePredicate {  
    boolean test(Apple apple);  
}
```

사과를 선택하는 다양한 전략을 사용한다.`ApplePredicate`는 사과 선택 전략을 캡슐화함. 위 조건에 따라 `filter`가 다르게 동작할 것이라고 예상할 수 있다. 

이를 전략 디자인 패턴`strategy design pattern` 이라고 부른다. 전략 디자인 패턴은 각 알고리즘(전략이라 불리는)을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다.

여기서는 `ApplePredicate`가 알고리즘 패밀리고 `AppleHeavyWeightPredicate`와 `AppleGreenColorPredicate`가 전략이다.

그런데 `ApplePredicate`는 어떻게 다양한 동작을 수행할 수 있을까?
동작 파라미터화, 즉 메서드가 다양한 동작(또는 전략)을 받아서 내부적으로 다양한 동작을 수행할 수 있다.

### 2.2.1 네 번째 시도 : 추상적 조건으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate predicate) {  
    List<Apple> result = new ArrayList<>();  
    for (Apple apple : inventory) {  
        if (predicate.test(apple)) {  
            result.add(apple);  
        }  
    }  
    return result;  
}
```

#### 코드/동작 전달하기

이제 요구사항이 변경되면 `ApplePredicate`를 적절하게 구현하는 클래스만 만들면 된다. 우리가 전달한 `ApplePredicate` 객체에 의해 `filterApples` 메서드의 동작이 결정된다니 정말 멋지다.
즉, 우리는 `filterApples` 메서드의 동작을 파라미터화한 것이다.

#### 한 개의 파라미터, 다양한 동작

지금까지 살펴본 것처럼 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다는 것이 동작 파라미터화의 강점이다.

## 2.3 복잡한 과정 간소화

사용하기 복잡한 기능이나 개념을 사용하고 싶은 사람은 아무도 없다. 다음 예제에서 요약하는 것처럼 현재 `filterApples` 메서드로 새로운 동작을 전달하려면 `ApplePredicate` 인터페이스를 구현하는 여러 클래스를 정의한 다음에 인스턴스화해야 한다. 

자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 익명 클래스`anonymous class`라는 기법을 제공한다. 익명 클래스를 이용하면 코드의 양을 줄일 수 있다. 하지만 익명 클래스가 모든 것을 해결하는 것은 아니다. 

### 2.3.1 익명 클래스

익명 클래스는 자바의 지역 클래스 `local class` (블록 내부에 선언된 클래스)와 비슷한 개념이다. 익명 클래스는 말 그대로 이름이 없는 클래스다. 익명 클래스를 이용하면 클래스 선언과 인스턴스화를 동시에 할 수 있다. 즉, 즉석에서 필요한 구현을 만들어서 사용할 수 있다.

### 2.3.2 다섯 번째 시도 : 익명 클래스 사용

익명클래스의 부족한 점.

1. 익명 클래스는 여전히 많은 공간을 차지한다.
2. 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다.

코드의 장황함`verbosity` 은 나쁜 특성이다. 장황한 코드는 구현하고 구현하고 유지보수하는 데 시간이 오래 걸릴 뿐 아니라 읽는 즐거움을 뺴앗는 요소로, 개발자로부터 외면받는다. 한눈에 이해할 수 있어야 좋은 코드다.

### 2.3.3 여섯 번쨰 시도 : 람다 표현식 사용

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor)));
```

이전 코드보다 훨씬 간단해졌다.

### 2.3.4 일곱 번째 시도 : 리스트 형식으로 추상화

```java
public static List<Apple> filter(List<T> list, Predicate<T> p) {  
  List<T> result = new ArrayList<>();  
  for (T e : list) {  
    if (p.test(e)) {  
      result.add(e);  
    }  
  }  
  return result;  
}
```

## 2. 4 실전 예제

### 2.4.1 Comparator로 정렬하기

컬렉션 정렬은 반복되는 프로그래밍 작업이다. 

```java
public interface Comparator<T> {
	int compare(T o1, T o2)
}
```

`Comparator`를 구현해서 `sort` 메서드의 동작을 다양화할 수 있다. 

```java
inventory sort( (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

### 2.4.2 Runnalbe로 코드 블록 실행하기

자바 스레드를 이용하면 병렬로 코드 블록을 실행할 수 있다. 어떤 코드를 실행할 것인지 스레드에게 알려줄 수 있을까? 여러 스레드가 각자 다른 코드를 실행할 수 있다. 나중에 실행할 수 있는 코드를 구현할 방법이 필요하다.

자바에서는 `Runnable` 인터페이스를 이용해서 실행할 코드 블록을 지정할 수 있다.

```java
public interfcate Runnable{
	void run();
}
```

`Runnalbe`을 이용해서 다양한 동작을 스레드로 실행할 수 있다.

```java
Thread t = new Thread( () -> System.out.println("Hello word"));
```

### 2.4.3 Callable을 결과로 반환하기

```java
Future<String> threadName = executorService.submit( () -> Thread.currentThread().getName());
```






