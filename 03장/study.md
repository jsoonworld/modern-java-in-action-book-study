# ch03 람다 표현식

# 3.1 람다란 무엇인가?

람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다. 람다 표현식에는 이름은 없지만, 파라미터 리스트, 바디, 반환 형식, 발생할 수 있는 예외 리스트는 가질 수 있다.

- 익명
	- 보통의 메서드와 달리 이름이 없으므로 익며이라 표현한다. 구현해야 할 코드에 대한 걱정거리가 줄어든다.
- 함수
	- 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다. 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다.
- 전달
	- 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
- 간결성
	- 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.

람다를 이용해서 간결한 방식으로 코드를 전달할 수 있다.

기존 코드
```java
Compare<Apple> byWeight = new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
}
```

다음은 람다를 이용한 코드
```java
Compartor<Apple> byWeight =
	(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

람다는 세 부분으로 이루어진다.

- 파라미터 리스트
	- `Compartor`의 `compare` 메서드 파라미터(사과 두 개)
- 화살표
	- 화살표 (->)는 람다의 파라미터 리스트와 바디를 구분한다.
- 람다 바디
	- 두 사과의 무게를 비교한다. 람다의 반환값에 해당하는 표현식이다.

# 3.2 어디에, 어떻게 람다를 사용할까?

```java
List<Apple> greenApples = 
	filter(inventory, (Apple a) -> GREEN.equals(a.getColor()));
```

함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.

## 3.2.1 함수형 인터페이스

`Predicate<T>`는 오직 하나의 추상 메서드만 지정하는 함수형 인터페이스다.

```java
public interface Predicate<T> {
	boolean test (T t);
}
 ```

간단히 말해 함수형 인터페이스는 정확히 하나의 추상 메서드를 지정하는 인터페이스다.

함수형 인터페이스로 뭘 할 수 있을까? 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 전체 표현식을 함수형 인터페이스의 인스턴스로 취급(기술 적으로 따지면 함수형 인터페이스를 구현한 클래스의 인스턴스)할 수 있다.

## 3.2.2 함수 디스크립터

함수형 인터페이스의 추상 메서드 시그니처`signature` 는 람다 표현식의 시그니처를 가리킨다. 람다 표현식의 시그니처를 서술하는 메서드를 함수 디스크립터`function descriptor`라고 부른다.
예를 들어 `Runnable` 인터페이스의 유일한 추상 메서드 `run`은 인수와 반환값이 없으므로(`void` 반환)`Runnable` 인터페이스는 인수와 반환값이 없는 시그니처로 생각할 수 있다.

이 장에서는 람다와 함수형 인터페이스를 가리키는 특별한 표기법을 사용한다. () -> void 표기는 파라미터 리스트가 없으며 void를 반환하는 함수를 의미한다. 즉, 앞에서 설명한 `Runnable`이 이에 해당한다. `(Apple, Apple) -> int`는 두 개의 `Apple` 을 인수로 받아 `int`를 반환하는 함수를 가리킨다.

일단은 람다 표현식은 변수에 할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달할 수 있으며, 함수형 인터페이스의 추상 메서드와 같은 시그니처를 갖는다는 사실을 기억하자.

# 3.3 람다 활용 : 실행 어라운드 패턴

람다와 동작 파라미터화로 유연하고 간결한 코드를 구현하는 데 도움을 주는 실용적인 예제를 살펴보자. 자원 처리(예를 들면 데이터베이스의 파일 처리)에 사용하는 순환 패턴`recurrenct pattern`은 자원을 열고, 처리한 다음에, 자원을 닫는 순서로 이루어진다. 설정`setup`과 정리`cleanup`과정은 대부분 비슷한다. 즉, 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다. 이런 형식을 실행 어라운드 패턴 `excute around pattern`이라고 부른다. 

```java
public String processFile() throws IOException{
	try (BufferedReader br = 
		new BufferedReader(new FileReader("data.txt'))){
			return br.readLine(); <- 실제 필요한 작업을 하는 행
		}
}
```

## 3.3.1 1단계: 동작 파라미터화를 기억하라

현재 코드는 파일에서 한 번에 한 줄만 읽을 수 있다. 한 번에 두 줄을 읽거나 가장 자주 사용되는 단어를 반환하려면 어떻게 해야 할까? `processFile`의 동작을 파라미터화하는 것이다. `processFile` 메서드가 `BufferedReader`를 이용해서 다른 동작을 수행할 수 있도록 `processFile` 메서드로 동작을 전달해야 한다.

람다를 이용해서 동작을 전달할 수 있다. `processFile` 메서드가 한 번에 두 행을 읽게 하려면 어떻게 코드를 고쳐야 할까? 우선 `BufferedReader`를 인수로 받아서 `String`을 반환하는 람다가 필요하다.

```java
String result = processFile((BufferedReader br -> 
							 br.readLine() + br.readLine()));
```

## 3.3.2 2단계 : 함수형 인터페이스를 이용해서 동작 전달

함수형 인터페이스 자리에 람다를 사용할 수 있다. 따라서 `BufferedReader -> String`과 `IOException`을 던질`throw` 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야 한다.

## 3.3.3. 3단계 : 동작 실행

이제 `BufferedReaderProcessor`에 정의된 `process` 메서드의 시그니처 `(BufferedReader -> String)` 와 일치하는 람다를 전달할 수 있다. 람다의 코드가 `processFile` 내부에서 어떻게 실행되는지 기억하고 있는가? 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으며 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리한다. 따라서 `processFile` 바디 내에서 `BufferedReaderProcessor` 객체의 `process`를 호출할 수 있다.

```java
public String processFile(BufferedReaderProcessor p) throws IOException{
	try (BufferedReader br = new BufferedReader(new FileReader("data.txt))){
		return p.process(br);
	}
}
```

## 3.3.4 4단계 : 람다 전달

이제 람다를 이용해서 다양한 동적을 `processFile` 메서드로 전달할 수 있다.

다음은 한 행을 처리하는 코드다.
```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
```

다음은 두 행을 처리하는 코드다.
```java
String towLines = 
	processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

# 3.4 함수형 인터페이스 사용

함수형 인터페이스는 오직 하나의 추상 메서드를 지정한다. 함수형 이터페이스의 추상 메서드는 람다 표현식의 시그니처를 묘사한다. 함수형 인터페이스의 추상 메서드 시그니처를 함수 디스크립터`function descriptor`
라고 한다.

다양한 람다 표현식을 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요하다. 3.2 절에서 살펴본 것처럼 이미 자바 API는 `Comparable`, `Runnable`, `Callable` 등의 다양한 함수형 인터페이스를 포함하고 있다.

## 3.4.1 Predicate

`java.util.function.Predicate<T>` 인터페이스는 `test`라는 추상 메서드를 정의하며 `test`는 제네릭 형식 `T`의 객체를 인수로 받아 불리언을 반환한다. 

## 3.4.2 Consumer

`java.util.function.Consumer<T>` 인터페이스는 제네릭 형식 `T`객체를 받아서 `void`를 반환하는 `accept`라는 추상 메서드를 정의한다. `T` 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 `Comsumer` 인터페이스를 사용할 수 있다.

## 3.4.2 Function

`java.util.function.Function<T, R>` 인터페이스는 제네릭 형식 `T`를 인수로 받아서 제네릭 형식 `R` 객체를 반환하는 추상 메서드 `apply`를 정의한다. 입력을 출력으로 매핑하는 람다를 정의할 때 `Function` 인터페이스를 활용할 수 있다(예를 들면 사과의 무게 정보를 추출하거나 문자열을 길이와 매핑). 

### 기본형 특화

자바에서는 기본형을 참조형으로 변환하는 기능을 제공한다. 이 기능을 박싱`boxing`이라고 한다. 참조형을 기본형으로 변환하는 반대 동작을 언방식`unboxing`이라고 한다. 박싱과 언박싱이 자동으로 이루어지는 오토박싱`autoboxing`이라는 기능도 제공한다. 

하지만 이런 변환 과정을 비용이 소모된다. 박싱한 값은 기본형을 감싸는 래퍼며 힙에 저장된다. 따라서 박싱한 값은 메모리를 더 소비하며 기본형을 가져올 때도 메모리를 탐색하는 과정이 필요하다.

자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱 동작을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공한다.

# 3.5 형식 검사, 형식 추론 제약

람다 표현식을 처음 설명할 때 람다로 함수형 인터페이스의 인스턴스를 만들 수 있다고 언급했다. 람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않다. 

## 3.5.1 형식 검사

람다가 사용되는 콘텍스느`context`를 이용해서 람다의 형식`type`을 추론할 수 있다. 어떤 콘텍스트(예를 들면 람다가 전달될 메서드 파라미터나 람다가 할당되는 변수 등)에서 기대되는 람다 표현식의 형식을 대상 형식 `target type` 이라고 부른다.

```java
List<Apple> heavierThan150g = filter(inventory, (Apple apple) -> apple.getWeight() > 150 );
```

1. `filter` 메서드의 선언을 확인한다.
2. `filter` 메서드는 두 번째 파라미터로 `Predicate<Apple>` 형식 (대상 형식)을 기대한다.
3. `Predicate<Apple>`은 `test`라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스다.
4. `test` 메서드는 `Apple`을 받아 `boolean` 을 반환하는 함수 디스크립터를 묘사한다.
5. `filter` 메서드로 전달된 인수는 이와 같은 요구사항을 만족해야 한다.

## 3.5.2 같은 람다, 다른 함수형 인터페이스

대상 형식`target typing` 이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다. 예를 들어 이전에 살펴본 `Callable`과 `PrivilegedAction` 인터페이스는 인수를 받지 않고 제네릭 형식 `T`를 반환하는 함수를 정의한다. 

### 3.5.3 형식 추론

우리 코드를 좀 더 단순화할 수 있는 방법이 있다. 자바 컴파일러는 람다 표현식이 사용된 콘텍스트(대상 형식)를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다. 즉, 대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다.

결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다. 즉, 자바 컴파일러는 람다 파라미터 형식을 추론할 수 있다.

## 3.5.4 지역 변수 사용

지금까지 살펴본 모든 람다 표현식은 인수를 자신의 바디 안에서만 사용했다. 하지만 람다 표현식에서는 익명 함수가 하는 것처럼 자유 변수`free variable` (파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다. 이와 같은 동작을 람다 캡처링`capturing lambda` 이라고 부른다. 

하지만 자유 변수에도 약간의 제약이 있다. 람다는 인스턴스 변수와 정적 변수를 자유롭게 캡처(자신의 바디에서 참조할 수 있도록)할 수 있다. 하지만 그러러면 지역 변수는 명시적으로 final로 선언되어 있어야 하거나 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다. 즉, 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡처할 수 있다.

# 3.6 메서드 참조

메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다. 때로는 람다 표현식보다 메서드 참조를 사용하는 것이 더 가독성이 좋으며 자연스러울 수 있다. 

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compare(a2.getWeight)));
```

```java
inventory.sort(comparing(Apple::getWeight));
```

## 3.6.1 요약

메서드 참조가 왜 중요한가? 메서드 참조는 특정 메서드만들 호출하는 람다의 축약형이라고 생각할 수 있다. 예를 들어 람다가 '이 메서드를 직접 호출해'라고 명령한다면 메서드를 어떻게 호출해야 하는지 설명을 참조하기보다 메서드명을 직접 참조하는 것이 편리하다. 실제로 메서드 참조를 이용하면 기존 메서드 구현으로 람다 표현식을 만들 수 있다. 이때 명시적으로 메서드명을 참조함으로써 가독성을 높일 수 있다. 

### 메서드 참조를 만드는 방법

1. 정적 메서드 참조 
	- 예를 들어 `Integer`의 `parseInt` 메서드는 `Integer::parseInt`로 표현할 수 있다.
2. 다양한 형식의 인스턴스 메서드 참조
	- 예를 들어 `String`의 `length` 메서드는 `String::length` 로 표현할 수 있다.
3. 기존 객체의 인스턴스 메서드 참조
	- 예를 들어 `Transcation` 객체를 할당받은 `expensiveTranscation` 지역 변수가 있고, `Transcation`  객체에는 `getValue` 메서드가 있다면, 이를 `expensiveTranscation::getValue`라고 표현할 수 있다.

## 3.6.2 생성자 참조

`ClassName::new` 처럼 클래스명과 `new` 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다. 이것은 정적 메서드의 참조를 만드는 방법과 비슷하다. 

# 3.7 람다, 메서드 참조 활용하기

## 1단계 : 코드 전달


```java
void sort(Compartor? super E> c)
```

이 코드는 `Compartor` 객체를 인수로 받아 두 사과를 비교한다. 객체 안에 동작을 포함시키는 방식으로 다양한 전략을 전달할 수 있다. 이제 'sort의 동작은 파라미터화되었다'라고 말할 수 있다. 즉, `sort`에 전달된 정렬 전략에 따라 `sort`의 동작이 달라질 것이다.

1단계 코드는 다음과 같이 완성할 수 있다.

```java
public class AppleComparator implements Comparator<Apple> {
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
}
inventory.sort(new AppleComparator());
```

## 2단계 : 익명 클래스 사용

한 번만 사용할 `Comparator`를 위 코드처럼 구현하는 것보다 익명 클래스를 이용하는 것이 좋다.

```java
inventroy.sort(new Compartor<Apple>(){
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
})
```

## 3단계 : 람다 표현식 사용

하지만 아직도 코드가 장황한 편이다. 자바 8에서는 람다 표현식이라는 경량화된 문법을 이용해서 코드를 전달할 수 있다. 함수형 인터페이스를 기대하는 곳 어디에서나 람다 표현식을 사용할 수 있음을 배웠다.

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

자바 컴파일러는 람다 표현식이 사용된 콘텍스트를 활용해서 람다의 파라미터 형식을 추론한다고 설명했다.

```java
inventory.sort( (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

```java
Comparator<Apple> c = Comparator.comparing((Aplle a) -> a.getWeight());
```

```java
import static java.util.Compartor.comparing;

inventory.sort(comparing(apple -> apple.getWeight));
```

## 4 단계 : 메서드 참조 사용

```java
inventory.sort(comparing(Apple::getWeight))
```

# 3.8 람다 표현식을 조합할 수 있는 유용한 메서드

## Compartor 조합

이전에도 보았듯이, 정적 메서드 `Compartor.comparing`을 이용해서 비교에 사용할 키를 추출하는 `Function` 기반의 `Comparator`를 반환할 수 있다.

### 역정렬

```java
inventory.sort(comparing(Apple::getWeight).reversed());
```

### Comparator 연결

```java
inventory.sort(comparing(Apple::getWeight)
			  .reversed()
			  .thenComapring(Apple::getCountry));
```

## Predicate 조합

프레디케이트를 반전시킬 때 `negate()` 메서드
```java
Predicate<Apple> notRedApple = redApple.negate();
```

```java
Predicate<Apple> redAndHeavyApple = 
	redApple.and(apple -> apple.getWeight() > 150);
```

```java
Predicate<Apple> redAndHeavyAppleOrGreen = 
	redApple.and(apple -> apple.getWeight() > 150)
			.or(apple -> GREEN.equals(a.getColor()));
```


# 마치며

- 람다 표현식은 익명 함수의 일종이다. 이름은 없지만, 파라미터 리스트, 바디, 반환 형식을 가지며 예외를 던질 수 있다.
- 람다 표현식으로 간결한 코드를 구현할 수 있다.
- 함수형 인터페이스는 하나의 추상 메서드만을 정의하는 인터페이스다.
- 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 사용할 수 있다.
- 람다 표현식을 이용해서 함수형 인터페이스의 추상 메서드를 즉석으로 제공할 수 있으며 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다.
- `java.util.function` 패키지는 `Predicate<T>`, `Function<T, R>`, `Supplier<T>`, `Consumer<T>`, `BinaryOperator<T>` 등을 포함해서 자주 사용하는 다양한 함수형 인터페이스를 제공한다.
- 자바 8은 `Predicate<T>`와 `Function<T, R>` 같은 제네릭 함수형 인터페이스와 관련한 박싱 동작을 피할 수 있는 `IntPrdicate, IntToLongFunction` 등과 같은 기본형 특화 인터페이스도 제공한다.
- 실행 어라운드 패턴(예를 들면 자원 할당, 자원 정리 등 코드 중간에 실행해야 하는 메서드에 꼭 필요한 코드)을 람다와 활용하면 유연성과 재사용성을 추가로 얻을 수 있다.
- 람다 표현식의 기대 형식`type expected`을 대상 형식`target type`이라고 한다.
- 메서드 참조를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달할 수 있다.
- `Comparator, Predicate, Function` 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메서드를 제공한다.




