+++
title = "Part1 기초"
+++

## 1장 - 자바8을 눈여겨봐야 하는 이유 
**Stream processing** : stream이란 한번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임을 말한다.

**동작 파라미터화** : 메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공한다.

**병렬성과 공유 가변 데이터** : 다른 코드와 동시에 실행하더라도 안전하게 실행할 수 있는 코드를 만드려면 공유된 가변 데이터에 접근하지 말아야 한다. 이런 함수를 pure 함수, stateless 함수라 부른다. 

### 자바 함수
프로그래밍 언어에서 함수라는 용어는 메서드 특히 정적 메서드와 같은 의미로 사용된다. 자바의 함수는 이에 더해 수학적인 함수처럼 사용되며 부작용을 일으키지 않는 함수를 의미한다.

자바8에서는 함수를 새로운 값의 형식으로 추가했다.(즉, 함수 자체가 값)

**메서드 레퍼런스**

ex) 디렉토리에서 모든 숨겨진 파일을 필터링하는 문제에서 우선 주어진 파일이 숨겨져 있는지 체크하는 기능
```java
File[] hiddenFiles = new File(“.”).listFiles(new FileFilter(){
	public boolean accept(File file){
		return file.isHidden(); //숨겨진 파일 필터링. 
	}
}
```
위의 코드를 보면 자바8 전까지는 File 클래스에 이미 isHidden이라는 메서드가 있는데 FileFilter로 감싼 다음에 FileFilter를 인스턴스화해야 했다. 
```java
File[] hiddenFiles = new File(“.”).listFiles(File::isHidden);
```
**자바8의 메서드 레퍼런스 ::** (이 메서드를 값으로 사용하라는 의미)를 이용해서 listFiles에 직접 전달할 수 있다. 기존에 객체 레퍼런스(new로 객체 레퍼런스를 생성함)를 이용해서 객체를 이리저리 주고받았던 것처럼 자바 8에서는 File::isHidden을 이용해서 메서드 레퍼런스를 만들어 전달할 수 있게 되었다. 

**람다: 익명 함수**

함수도 값으로 취급할 수 있다. ex) (int x) -> x+1 : x라는 인수를 호출하면 x+1을 반환하라.

**코드 넘겨주기: 예제**

Apple이라는 클래스와 getColor라는 메서드가 있고, Apples 리스트를 포함하는 inventory라는 변수가 있다고 가정하자. 이때 모든 녹색 사과를 선택해서 리스트를 반환하는 프로그램을 구현하려 한다. 이처럼 특정 항목을 선택해서 반환하는 동작을 ‘필터'라고 한다. 자바8 이전에는 다음처럼 filterGreenApples라는 메서드를 구현했을 것이다.
```java
public static List<Apple> filterGreenApples(List<Apple> inventory){
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory){
		if(“green”.equals(apple.getColor()){
			result.add(apple);
		}
	}
	return result; 
}
```
하지만 누군가는 애플을 무게로 필터링 하고 싶을 수 있다. 그러면 전체 코드를 복붙해서 다음처럼 구현할 수 있을 것이다. 
```java
public static List<Apple> filterGreenApples(List<Apple> inventory){
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory){
		if(apple.getWeight() > 150){
			result.add(apple);
		}
	}
	return result; 
}
```
중복의 단점이 드러나는 안좋은 방법이다. 자바8 에서는 코드를 인수로 넘겨줄 수 있어서 filter 메서드를 중복으로 구현할 필요가 없다. 
```java
public static boolean isGreenApple(Apple apple){
    return "green".equals(apple.getColor());
}

public static boolean isHeavyApple(Apple apple){
    return apple.getWeight() > 150;
}

static List<Apple> filterApples(List<Apple> inventory,
                                Predicate<Apple> p){
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory){
        if(p.test(apple)){
            result.add(apple);
        }
    }
    return result;
}
```
다음 처럼 메서드를 호출 할 수 있다.

`filterApples(inventory, Apple::isGreenApple);`

`filterApples(inventory, Apple::isHeavyApple);`

cf) Predicate : 수학에서는 인수로 값을 받아 true / false를 반환하는 함수를 Predicate라고 한다.

**메서드 전달에서 람다로**

isHeavyApple, isGreenApple처럼 한두 번만 사용할 메서드를 매번 정의하는 것은 귀찮은 일이다. 자바8 에서는 다음처럼 새로운 개념을 이용해서 코드를 구현할 수 있다. 
`filterApples(inventory, (Apple a) -> “green”.equals(a.getColor());`

`filterApples(inventory, (Apple a) -> a.getWeight() > 150);`

`filterApples(inventory, (Apple a) -> a.getWeight() < 80 || “brown”.equals(a.getColor());`

즉, 한 번만 사용할 메서드는 따로 정의를 구현할 필요가 없다. 하지만 람다가 몇 줄 이상으로 길어진다면(복잡한 동작을 수행하는 상황) 익명 람다보다는 코드가 수행하는 일을 잘 설명하는 이름을 가진 메서드를 정의하고 메서드 레퍼런스를 활용하는 것이 바람직하다. **코드의 명확성이 우선시 되어야 한다.**

### 스트림
다음은 리스트에서 고가의 거래(Transcation)만 필터링한 다음에 통화로 결과를 그룹화하는 코드다. 
``` java
Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>(); // 그룹화된 트랜잭션을 더할 Map 생성

for (Transaction transaction : transactions){ // 트랜잭션의 리스트를 반복
	if (transaction.getPrice() > 1000){ // 고가의 트랜잭션을 필터링
		Currency currency = transaction.getCurrency(); // 트랜잭션의 통화 추출
		List<Transcation> transactionsForCurrency = transactionsByCurrencies.get(currency); 
		if (transactionsForCurrency == null){ // 현재 통화의 그룹화된 맵에 항목이 없으면 새로 만든다.
			transactionsForCurrency = new ArrayList<>();
			transactionsByCurrencies.put(currency, transactionsForCurrency);
		}
		transactionsForCurrency.add(transaction); // 현재 탐색된 트랜잭션을 같은 통화의 트랜잭션 리스트에 추가한다. 
	}
}
```
위 예제 코드에는 중첩된 제어 흐름 문장이 많아서 코드를 한 번에 이해하기 어렵다. 스트림 API를 이요하면 다음처럼 문제를 해결할 수 있다. 
```java
import static java.util.stream.Collections.toList;

Map<Currency, List<Transaction>> transactionByCurrencies = 
	transactions.stream()
           		   .filter((Transaction t) -> t.getPrice() > 1000) //고가의 트랜잭션 필터링
			   .collect(groupingBy(Transaction::getCurrency);	
```
컬렉션에서는 반복 과정을 직접 처리해야 했다. 이런 방식의 반복을 외부 반복이라고 한다. 반면 스트림 API를 이용하면 루프를 신경쓸 필요가 없다. 라이브러리 내부에서 모든 데이터가 처리된다. 이와 같은 반복을 내부 반복이라고 한다.

컬렉션은 어떻게 데이터를 저장하고 접근할지에 중점을 두는 반면 스트림은 데이터에 어떤 계산을 할 것인지 묘사하는 것에 중점을 둔다는 점을 기억하자. 스트림은 스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공한다는 것이 핵심이다. 

**컬렉션을 필터링할 수 있는 가장 빠른 방법은 컬렉션을 스트림으로 바꾸고, 병렬로 처리한 다음에, 리스트로 다시 복원하는 것이다.**

### 디폴트 메서드 
디폴트 메서드는 특정 프로그램을 구현하는 데 도움을 주는 기능이 아니라 미래에 프로그램이 쉽게 변화할 수 있는 환경을 제공하는 기능이다.

## 2장 - 동작 파라미터화 코드 전달하기
동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다. **동작 파라미터화란 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미한다.** 이 코드 블록은 나중에 프로그램에서 호출한다. 즉, 코드 블록의 실행은 나중으로 미뤄진다. 결과적으로 코드 블록에 따라 메서드의 동작이 파라미터화된다. 

**동작 파라미터화**

```java
public interface ApplePredicate{
	boolean test (Apple apple);
}
```
선택 조건을 결정하는 인터페이스이다. 이와 같은 동작을 프레디케이트(불린을 반환하는 함수)라고 한다.

다음 예제처럼 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있다. 
```java
//무거운 사과만 선택
public class AppleHeavyWeightPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return apple.getWeight() > 150;
	}
}
//녹색 사과만 선택
public class AppleGreenColorPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return “green”.equals(apple.getColor());
	}
}

// 템플릿 부분
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory){
		if (p.test(apple)){
			result.add(apple);
		}
	}
	return result;
}

```
하지만 익명 클래스를 사용해도 반복되어 지저분한 코드는 여전히 많고 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다. 

**람다 표현식 사용**

자바8의 람다 표현식을 이용해서 간단히 재구현할 수 있다.

`List<Apple> result = filterApples(inventory, (Apple apple) -> “red”.equals(apple.getColor()));`

![](/assets/parameterization.jpg)

**리스트 형식으로 추상화**

Apple 이외의 다양한 물건에서 필터링이 작동하도록 리스트 형식을 추상화할 수 있다. 
```java
public interface Predicate<T>{
	boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p){
	List<T> result = new ArrayList<>();
	for (T e : list){
		if (p.test(e)){
			result.add(e); 
		}
	}
	return result; 
}
```
이제 바나나, 오렌지, 정수, 문자열 등의 리스트에 필터 메서드를 사용할 수 있다. 
`List<Apple> redApples = filter(inventory, (Apple apple) -> “red”.equals(apple.getColor());`

`List<String> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);`

**실전 예제 - Comparator로 정렬하기**

자바8의 List에는 sort 메서드가 포함되어 있다.(물론 Collections.sort도 존재) 다음과 같은 인터페이스를 갖는 java.util.Comparator 객체를 이용해서 sort의 동작을 파라미터화 할 수 있다. 
``` java
//java.uitl.Comparator
public interface Comparator<T>{
	public int compare(T o1, T o2);
}
```
Comparator를 구현해서 sort 메서드의 동작을 다양화할 수 있다. 예를 들어 익명 클래스를 이용해서 무게가 적은 순으로 목록에서 사과를 정렬할 수 있다.
```java
inventory.sort(new Comparator<Apple>{
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
});
```
람다 표현식을 이용하면 다음처럼 간단하게 코드를 구현할 수 있다.
```java
inventory.sort(
	(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

## 3장 - 람다 표현식
### 람다란 무엇인가
람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한것이라고 할 수 있다. 
```java
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
};
```
```java
Comparator<Apple> byWeight =
        (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
람다는 세 부분으로 이루어진다.

**파라미터 리스트** :  Comparator의 compare 메서드의 파라미터(두개의 사과).

**화살표** : 화살표(->)는 람다의 파라미터 리스트와 바디를 구분한다.

**람다의 바디** : 두 사과의 무게를 비교한다. 람다의 반환값에 해당하는 표현식이다.

**자바8의 유효한 5가지 람다 표현식**

`(String s) -> s.length()` 

첫 번째 람다 표현식은 String 형식의 파라미터 하나를 가지며 int를 반환한다. 람다 표현식에는 return이 함축되어 있으므로 return 문을 명시적으로 사용하지 않아도 된다.

`(Apple a) -> a.getWeight() > 150`

두번째 람다 표현식은 Apple 형식의 파라미터를 가지며 boolean을 반환한다. 

``` java
(int x, int y) -> {
	System.out.println(“Result:”);
	System.out.println(x+y);
```
세 번째 람다 표현식은 int 형식의 파라미터 두 개를 가지며 리턴값이 없다(void 리턴). 이 예제에서 볼 수 있듯이 람다 표현식은 여러 행의 문장을 포함할 수 있다. 

`() -> 42` 

네 번째 람다 표현식은 파라미터가 없으며 int를 반환한다. 

`(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());`

다섯 번째 람다 표현식은 Apple 형식의 파라미터 두 개를 가지며 int를 반환한다. 
```java
//cf) 유효하지 않은 람다 표현식
(Integer i ) -> return “Alan” + i; 
(String s) -> {“Iron Man”;}

//return은 흐름 제어문이다. { } 안에 있어야 한다. 
(Integer i ) -> { return “Alan” + i }; 

//“Iron Man”은 구문(statement)이 아니라 표현식(expression)이다. 
(String s) -> “Iron Man” 또는 (String s) -> { return “Iron Man” }
```
### 어디에, 어떻게 람다를 사용할까?
**함수형 인터페이스**라는 문맥에서 람다 표현식을 사용할 수 있다. 함수형 인터페이스는 정확히 하나의 추상 메서드를 지정하는 인터페이스다. 지금까지 살펴본 자바 API의 함수형 인터페이스로 Comparator, Runnable 등이 있다. 
```java
//java.util.Comparator 
public interface Comparator<T>{
	int compare(T o1, T o2);
}

//java.lang.Runnable 
public interface Runnable{
	void run();
}
```
cf) 인터페이스는 디폴트 메서드(인터페이스의 메서드를 구현하지 않은 클래스를 고려해서 기본 구현을 제공하는 바디를 포함하는 메서드)를 포함할 수 있다. 많은 디폴트 메서드가 있더라도 추상 메서드가 오직 하나면 함수형 인터페이스다.  

람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**(기술적으로 따지면 함수형 인터페이스를 concrete 구현한 클래스의 인스턴스)할 수 있다. 

**함수 디스크립터**

함수형 인터페이스의 추상메서드 시그너처를 함수 디스크립터라고 부른다. 예를 들어 () -> void 라는 표기는 파라미터 리스트가 없으며 void를 반환하는 함수를 의미한다. 람다 표현식은 함수형 인터페이스의 추상 메서드와 같은 시그니처를 갖는다는 사실을 기억하자.

함수형 인터페이스를 인수를 받는 메서드에만 람다 표현식을 사용할 수 있다.
```java
execute(() -> {});
public void execute(Runnable r){
	r.run();
}
```

cf) `@FunctionalInterface`은 함수형 인터페이스임을 가리키는 애노테이션이다. 만약 실제로 함수형 인터페이스가 아니면 컴파일러가 에러를 발생시킨다. 

### 람다 활용: 실행 어라운드 패턴 
자원 처리(예를 들면 DB의 파일 처리)에 사용하는 순환 패턴은 자원을 열고, 처리한 다음에, 자원을 닫는 순서로 이루어진다. 설정과 정리 과정은 대부분 비슷하다. **즉, 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는데 이 같은 형식의 코드를 실행 어라운드 패턴이라고 부른다.**

```java
public String processFile() throws IOException{
    try(BufferedReader br =
            new BufferedReader(new FileReader("data.txt"))){
        return br.readLine();
    }
}
```
cf) 자바7에 새로 추가된 try-with-resources 구문을 사용했다. 이를 사용하면 자원을 명시적으로 닫을 필요가 없다.

현재 코드는 파일에서 한 번에 한 줄만 읽을 수 있다. 기존의 설정, 정리 과정은 재사용하고 processFile 메서드만 다른 동작을 다른 동작을 수행하도록 해보자. processFile의 동작을 파라미터화하는 것이다. 즉, processFile 메서드가 BufferedReader를 이용해서 다른 동작을 수행할 수 있도록 processFile 메서드로 동작을 전달해야 한다.

`String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());`

함수형 인터페이스 자리에 람다를 사용할 수 있다. 따라서 BufferedReader -> String과 IOException을 던질 수 있는 시그니처와 일치하는 함수형 인터페이스를 만들어야 한다.
```java
@FuntionalInterface
public interface BufferedReaderProcessor{
	String process(BufferedReader b) throws IOException;
}

public String processFile(BufferedReaderProcessor p) throws IOException{
    try(BufferedReader br =
            new BufferedReader(new FileReader("data.txt"))){
        return p.process(br);
    }
}
```
이제 BufferedReaderProcessor에 정의된 process 메서드의 시그니처와 일치하는 다양한 람다를 전달할 수 있다. 
```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

**람다와 함수형 인터페이스 예제**

| 사용 사례 | 람다 예제 | 대응하는 함수형 인터페이스 |
| -- | -- | -- |
| 불린 표현 | `(List<String> list)` -> list.isEmpty() | `Predicate<List<String>>` |
| 객체 생성 | () -> new Apple(10) | `Supplier<Apple>` |
| 객체에서 소비 | (Apple a) -> System.out.println(a.getWeight()) | `Consumer<Apple>` |
| 객체에서 선택/추출 | (String s) -> s.length() | `Function<String, Integer>`또는 `ToInteFunction<String>` |
| 두 값 조합 | (int a, int b) -> a * b | IntBinaryOperator |
| 두 객체 비교 | (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()) | `BiFunction<Apple,Apple,Integer>` 또는 `ToIntBiFunction<Apple,Apple>` |

**예외, 람다, 함수형 인터페이스의 관계**

함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다. 그래서 예외를 던지를 람다 표현식을 만들려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 람다를 try/catch 블록으로 감싸야 한다. 
```java
@FunctionalInterface
public interface BufferedReaderProcessor{
	String process(BufferedReader b) throws IOException;
}
BufferedReaderProcessor p = (BufferedReader br) -> br.readLine();
```
위의 예제는 IOException을 명시적으로 선언하는 함수형 인터페이스 BufferedReaderProcessor 이다. 그러나 우리는 `Function<T, R>` 형식의 함수형 인터페이스를 기대하는 API를 사용하고 있으며 직접 함수형 인터페이스를 만들기 어려운 상황이다. 이런 상황에서는 아래 예제처럼 명시적으로 확인된 예외를 잡을 수 있다.
```java
Function<BufferedReader, String> f = 
	(BufferedReader b) -> {
		try{
			return b.readLine();
		}
		catch(IOException e){
			throw new RuntimeException(e);
		}
	};
```
### 형식 검사, 형식 추론, 제약
람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않다. 람다가 사용되는 context를 이용해서 람다의 형식(type)을 추론할 수 있다. 
![](/assets/type_inspection_process.jpg)

**특별한 void 호환 규칙**

람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다(물론 파라미터 리스트도 호환되어야 함). 예를 들어 List의 add 메서드는 boolean을 반환하지만  Consumer 컨텍스트(T -> void)에도 유효한 코드다. 
```java
// Predicate는 불린 반환값을 갖는다.
Predicate<String> p = s -> list.add(s);
// Consumer는 void 반환값을 갖는다. 
Consumer<String> b = s -> list.add(s);
```

**형식 추론**

``` java
//형식을 추론하지 않음
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
//형식을 추론함
Comparator<Apple> c = (a1,  a2) -> a1.getWeight().compareTo(a2.getWeight());

//형식 추론한 다른 예제
List<Apple> greenApples = filter(inventory, a -> “green”.equals(a.getColor()));
```
**지역 변수 사용**

람다 표현식에서는 자유 변수(파라미터로 넘겨진 변수가 아니라 외부에서 정의된 변수)를 활용할 수 있다. 이와 같은 동작을 람다 캡쳐링이라고 부른다. 
```java
int portNumber = 123;
Runnable r = () -> System.out.println(portNumber);
```
하지만 자유 변수에도 약간의 제약이 있다. 람다는 인스턴스 변수와 정적 변수를 자유롭게 캡쳐(자신의 바디에서 참조할 수 있도록) 할 수 있다. 하지만 그러려면 **지역 변수는 final로 선언되거나 실질적으로 final 처럼 취급되어야 한다.**
``` java
//컴파일 에러
int portNumber = 123;
Runnable r = () -> System.out.println(portNumber);
portNumber = 321;
```

**지역변수에 왜 이러한 제약이 필요한지 알아보자.**

우선 내부적으로 지역 변수는 스택에 위치한다. 람다에서 지역 변수에 바로 접근할 수 있다는 가정 하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다. 따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다. 따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴것이다.

람다와 익명 클래스 모두 메서드의 인수로 전달될 수 있으며 자신의 외부 영역의 변수에 접근할 수 있다. 다만 람다와 익명 클래스는 람다가 정의된 메서드의 지역 변수의 값은 바꿀 수 없다. 람다가 정의된 메서드의 지역 변숫값은 final 변수여야 한다. **람다는 변수가 아닌 값에 국한되어 어떤 동작을 수행한다는 사실이 명확해진다.** 지역 변숫값은 스택에 존재하므로 자신을 정의한 스레드와 생존을 같이 해야 하며 따라서 지역 변수는 final 이어야 한다. 가변 지역 변수를 새로운 스레드에서 캡쳐할 수 있다면 안전하지 않은 동작을 수행할 가능성이 생긴다(인스턴스 변수는 스레드가 공유하는 힙에 존재하므로 특별한 제약이 없다).

### 메서드 레퍼런스
메서드 레퍼런스를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다. 때로는 메서드 레퍼런스를 사용하는 것이 더 가독성이 좋으며 자연스러울 수 있다.
```java
// 기존 코드
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 메서드 레퍼런스와 java.util.Comparator.comparing을 활용한 코드 
inventory.sort((comparing(Apple::getWeight()); 
```
메서드 레퍼런스는 특정 메서드만을 호출하는 람다의 축약형이라고 생각할 수 있다. 예를 들어 `Apple::getWeight`는 Apple 클래스에 정의된 getWeight의 메서드 레퍼런스다. 실제로 메서드를 호출하는 것은 아니므로 괄호는 필요없다. 결과적으로 메서드 레퍼런스는 람다 표현식 `(Apple a) -> a.getWeight()`를 축약한 것이다.


람다와 메서드 레퍼런스 단축 표현 예제

| 람다 | 메서드 레퍼런스 단축 표현 |
| -- | -- |
| (Apple a) -> a.getWeight() | Apple :: getWeight |
| () -> Thread.currentThread().dumpStack() | Thread.currentThread() :: dumpStack |
| (str, i) -> str.substring(i) | String :: substring |
| (String s) -> System.out.println(s) | System.out :: println |

**메서드 레퍼런스를 만드는 방법**

![](/assets/change_methodreference.jpg)

메서드 레퍼런스는 세 가지 유형으로 구분할 수 있다. ‘정적 메서드 레페런스’, ‘다양한 형식의 인스턴스 메서드 레퍼런스’, ‘기존 객체의 인스턴스 메서드 레퍼런스'

**생성자 레퍼런스**

ClassName :: new 처럼 클래스명과 new 키워드를 이용해서 기존 생성자의 레퍼런스를 만들 수 있다. 
```java
Supplier<Apple> c1 = Apple :: new; // 디폴트 생성자 Apple()의 생성자 레퍼런스
Apple a1 = c1.get(); // Supplier의 get 메서드를 호출해서 새로운 Apple 객체를 만들 수 있다.

// 위 예제는 다음 코드와 같다.
Supplier<Apple> c1 = () -> new Apple(); // 람다 표현식은 디폴트 생성자를 가진 Apple을 만든다.
Apple a1 = c1.get(); // Supplier의 get 메서드를 호출해서 새로운 Apple 객체를 만들 수 있다.
```

Apple(Integer weight) 라는 시그니처를 갖는 생성자는 Function 인터페이스의 시그니처와 같다. 
```java
Function<Integer, Apple> c2 = Apple :: new; // Apple (Integer weight)의 생성자 레퍼런스
Apple a2 = c2.apply(110); // Function의 apply 메서드를 무게를 인수로 호출해서 새로운 Apple 객체를 만들 수 있다.

//위 예제는 다음 코드와 같다.
Function<Integer, Apple> c2 = (weight) -> new Apple(weight); // 특정 무게의 사과를 만드는 람다 표현식
Apple a2 = c2.apply(110); // Function의 apply 메서드를 무게를 인수로 호출해서 새로운 Apple 객체를 만들 수 있다.
```

### 람다, 메서드 레퍼런스 활용하기
```java
// 1단계 
public class AppleComparator implements Comparator<Apple>{
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
}
inventory.sort(new AppleComparator());

// 2단계: 익명 클래스 사용
inventory.sort(new Comparator<Apple>(){
	public int compare(Apple a1, Apple a2){
		return a1.getWeight().compareTo(a2.getWeight());
	}
});

// 3단계 : 람다 표현식 사용
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

//형식 추론을 통해 더 간소화
inventory.sort((a1,  a2) -> a1.getWeight().compareTo(a2.getWeight()));

 inventory.sort(comparing((a) -> a.getWeight()));

// 4단계 : 메서드 레퍼런스 사용
inventory.sort(comparing(Apple::getWeight));
```

### 람다 표현식을 조합할 수 있는 유용한 메서드
여러 개의 람다 표현식을 조합해서 복잡한 람다 표현식을 만들 수 있다(디폴트 메서드 덕분).

**Comparator 조합**

``` java
// 역정렬
inventory.sort(comparing(Apple::getWeight)).reversed();

// Comparator 연결
inventory.sort(comparing(Apple::getWeight).reversed().thenComparing(Apple::getCountry));
```

**Predicate 조합**

``` java
// 반전시킴
Predicate<Apple> notRedApple = redApple.negate();

// and 조합
Predicate<Apple> redAndHeavyApple = redApple.and(a -> a.getWeight() > 150);
```
cf)  a.or(b).and(c)는 (a || b) && c 와 같다.

**Function 조합**

```java
// andThen - 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수를 반환
Function<Integer, Integer> f = x -> x+1;
Function<Integer, Integer> g = x -> x*2;
Function<Integer, Integer> h = f.andThen(g);;
int result = h.apply(1); // 4를 반환 

//compose - 인자로 주어진 함수를 먼저 실행한 다음에 그 결과를 외부 함수의 인수로 제공
Function<Integer, Integer> f = x -> x+1;
Function<Integer, Integer> g = x -> x*2;
Function<Integer, Integer> h = f.compose(g);;
int result = h.apply(1); // 3를 반환 
```