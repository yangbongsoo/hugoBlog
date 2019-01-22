+++
title = "Part2-1 함수형 데이터 처리"
+++

## 4장 - 스트림 소개
DB에서는 `select name from dishes where calorie < 400`문장 처럼 선언형으로 연산을 표현할 수 있다(직접 구현할 필요가 없다). SQL 질의 언어에서는 우리가 기대하는 것이 무엇인지 직접 표현할 수 있다.

### 스트림이란 무엇인가?
**스트림**이란 자바 API에 새로 추가된 기능으로, 스트림을 이용하면 선언형(즉, 데이터를 처리하는 임의 구현 코드 대신 질의로 표현할 수 있다)으로 컬렉션 데이터를 처리할 수 있다. 또한 스트림을 이용하면 멀티 스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다. 다음 예제는 저칼로리의 요리명을 반환하고, 칼로리를 기준으로 요리를 정렬하는 자바7 코드다. 
```java
List<Dish> lowCaloricDishes = new ArrayList<>();
for(Dish d : menu){
	if(d.getCalories() < 400){
		lowCaloricDishes.add(d);
	}
}

Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
	public int compare(Dish d1, Dish d2){
		return Integer.compare(d1.getCalories(), d2.getCalories());
	}
});

List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish d : lowCaloricDishes){
	lowCaloricDishesName.add(d.getName());
}
```
위 코드에서는 lowCaloricDishes라는 ‘가비지 변수’가 사용되었다. 즉 lowCaloricDishes는 컨테이너 역할만 하는 중간 변수다. 자바8에서 이러한 세부 구현은 라이브러리 내에서 모두 처리한다. 

```java
//자바8 코드 
import static java.util.Comparator.comparing;
import static java.uitl.stream.Collectors.toList;

List<String> lowCaloricDishesName =

			menu.stream()

				.filter(d -> d.getCalories() < 400) // 400칼로리 이하의 요리 선택

				.sorted(comparing(Dish::getCalories)) // 칼로리로 요리 정렬

				.map(Dish::getName) // 요리면 추출

				.collect(toList()); // 모든 요리명을 리스트에 저장 

```

stream()을 parallelStream()으로 바꾸면 이 코드를 멀티코어 아키텍처에서 병렬로 실행할 수 있다.

```java

List<String> lowCaloricDishesName =

			menu.parallelStream()

				.filter(d -> d.getCalories() < 400) // 400칼로리 이하의 요리 선택

				.sorted(comparing(Dish::getCalories)) // 칼로리로 요리 정렬

				.map(Dish::getName) // 요리면 추출

                .collect(toList()); // 모든 요리명을 리스트에 저장 

```
자세한 내용은 7장에서 설명하겠다. 

### 스트림 시작하기 

스트림이란 정확히 뭘까? 스트림이란 **데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소**로 정의할 수 있다. 이 정의를 하나씩 살펴보자.

**연속된 요소** : 컬렉션과 마찬가지로 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공한다. 컬렉션은 자료구조이므로 컬렉션에서는 (예를 들어 ArrayList를 사용할 것인지 아니면 LinkedList를 사용할 것이지에 대한) 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룬다. 반면 스트림은 filter, sorted, map처럼 표현 계산식이 주를 이룬다. **즉, 컬렉션의 주제는 데이터고 스트림의 주제는 계산이다.**

**소스** : 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다. 정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지된다. 즉, 리스트로 스트림을 만들면 스트림의 요소는 리스트의 요소와 같은 순서를 유지한다. 

**데이터 처리 연산** : 스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 DB와 비슷한 연산을 지원한다. 예를 들어 filter, map, reduce, find, match, sort 등으로 데이터를 조작할 수 있다.

또한 스트림은 다음과 같은 두 가지 중요한 특징을 갖는다.

**파이프라이닝** : 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림을 자신을 반환한다.

**내부 반복** : 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원한다. 

```java
import static java.util.stream.Collectors.toList;

List<String> threeHighCaloricDishNames = 

	menu.stream() // 메뉴(요리 리스트)에서 스트림을 얻는다.

		.filter(d -> d.getCalories() > 300) // 파이프라인 연산 만들기. 첫 번째로 고칼로리 요리를 필터링한다.

		.map(Dish::getName) // 요리명 추출

		.limit(3) //선착순 세 개만 선택

		.collect(toList()); // 결과를 다른 리스트로 저장 

System.out.println(threeHighCaloricDishNames); 
// 결과는 [pork, beef, chicken] 이다. 
```
우선 menu에 stream 메서드를 호출해서 요리 리스트(menu)로부터 스트림을 얻었다. 여기서 **데이터 소스**는 요리 리스트(menu)다. 데이터 소스는 **연속된 요소**를 스트림에 제공한다. 다음으로 스트림에 filter, map, limit, collect로 이어지는 일련의 **데이터 처리 연산**을 적용한다. collect를 제외한 모든 연산은 서로 **파이프라인**을 형성할 수 있도록 스트림을 반환한다. 마지막으로 collect 연산으로 파이프라인을 처리해서 결과를 반환한다(collect는 스트림이 아니라 List를 반환한다). 마지막에 collect를 호출하기 전까지는 menu에서 아무것도 선택되지 않으며 출력 결과도 없다. 즉, collect가 호출되기 전까지 메서드 호출이 저장되는 효과가 있다. 

![](/streamprocess.jpg)

### 스트림과 컬렉션

자바의 기존 컬렉션과 새로운 스트림 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다. 여기서 ‘연속된’이라는 표현은 순서와 상관없이 아무 값에나 접속 하는 것이 아니라 순차적으로 값에 접근한다는 것을 의미한다. 이제 컬렉션과 스트림의 차이를 살펴보자.

**데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이라고 할 수 있다.** 컬렉션은 현재 자료구조가 포함하는 **모든** 값을 메모리에 저장하는 자료구조다. 즉, 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다(컬렉션에 요소를 추가하거나 컬렉션의 요소를 삭제할 수 있다. 이런 연산을 수행할 때마다 컬렉션의 모든 요소를 메모리에 저장해야 하며 컬렉션에 추가하려는 요소는 미리 계산되어야 한다).

반면 스트림은 이론적으로 **요청할 때만 요소를 계산**하는 고정된 자료구조다(스트림에 요소를 추가하거나 스트림에서 요소를 제거할 수 없다). 사용자가 요청하는 값만 스트림에서 추출한다는 것이 핵심이다. 결과적으로 스트림은 생산자와 소비자 관계를 형성한다. 또한 스트림은 게으르게 만들어지는 컬렉션과 같다. 즉, 사용자가 데이터를 요청할 때만 값을 계산한다.

반면 컬렉션은 적극적으로 생성된다(생산자 중심: 팔기도 전에 창고를 가득 채움). 소수 예제를 적용해보면 컬렉션은 끝이 없는 모든 소수를 포함하려 할 것이므로 무한 루프를 돌면서 새로운 소수를 계산하고 추가하기를 반복할 것이다. 결국 소비자는 영원히 결과를 볼 수 없게 된다.

스트림은 단 한번만 소비 할 수 있다.

```java

List<String> title = Arrays.asList(“java8”, “in”, “action”);

Stream<String> s = title.stream();

s.forEach(System.out::println); // title의 각 단어를 출력

s.forEach(System.out::println); // java.lang.IllegalStateException : 스트립이 이미 소비되었거나 닫힘

```
cf) 스트림과 컬렉션의 철학적 접근

스트림을 시간적으로 흩어진 값의 집합으로 간주할 수 있다. 반면 컬렉션은 특정 시간에 모든 것이 존재하는 공간(컴퓨터 메모리)에 흩어진 값으로 비유할 수 있다. for-each 루프 내에서 반복자를 이용해서 공간에 흩어진 요소에 접근할 수 있다.

컬렉션과 스트림의 또 다른 차이점은 데이터 반복 처리 방법이다. 컬렉션 인터페이스를 사용하려면 사용자가 직접 요소를 반복해야 한다. 이를 외부 반복이라고 한다. 반면 스트림 라이브러리는 반복을 알아서 처리하고 결과 스트림값을 어딘가에 저장해주는 내부 반복을 사용한다. 스트림 라이브러리의 내부 반복은 데이터 표현과 하드웨어를 활용한 병렬성 구현을 자동으로 선택한다. 반면 for-each를 이용하는 외부 반복에서는 병렬성을 스스로 관리해야 한다.

### 스트림 연산
스트림 인터페이스의 연산을 크게 두 가지로 구분할 수 있다. 

```java

List<String> threeHighCaloricDishNames = 

	menu.stream() // 메뉴(요리 리스트)에서 스트림을 얻는다.

		.filter(d -> d.getCalories() > 300) // 중간 연산

		.map(Dish::getName) // 중간 연산

		.limit(3) // 중간 연산

		.collect(toList()); // 스트림을 리스트로 변환. 최종 연산

```

filter, map, limit는 서로 연결되어 파이프라인을 형성한다.

collect로 파이프라인을 실행한 다음에 닫는다.

연결할 수 있는 스트림 연산을 중간 연산이라고 하며, 스트림을 닫는 연산을 최종 연산이라고 한다. 왜 스트림의 연산을 두 가지로 구분하는 것일까?

**중간 연산**

filter나 sorted 같은 중간 연산은 다른 스트림을 반환한다. 따라서 여러 중간 연산을 연결해서 질의를 만들 수 있다. 중간 연산의 중요한 특징은 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다는 것, 즉 게이르다는 것이다. 중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한 번에 처리하기 때문이다.

```java

// 제품 코드에는 이와 같은 출력 코드를 추가하지 않는게 좋다. 그러나 학습용으로는 매우 좋은 기법이다. 

List<String> names =
        menu.stream()
            .filter(d ->{
                System.out.println("filtering" + d.getName());
                return d.getCalories() > 300;
                })
            .map(d -> {
                System.out.println("mapping" + d.getName());
                return d.getName();
            })
        .limit(3)
        .collect(toList());
System.out.println(names);

filteringpork
mappingpork
filteringbeef
mappingbeef
filteringchicken
mappingchicken
[pork, beef, chicken]
```

스트림의 게으른 특성 덕분에 몇 가지 최적화 효과를 얻을 수 있었다. 첫째, 300칼로리가 넘는 요리는 여러 개지만 오직 처음 3개만 선택되었다. 이는 limit 연산 그리고 쇼트서킷이라 불리는 기법 덕분이다. 둘째, filter와 map은 서로 다른 연산이지만 한 과정으로 병합되었다(이 기법을 루프 퓨전이라고 한다). 

**최종 연산**

최종 연산은 스트림 파이프라인에서 결과를 도출한다. 보통 최종 연산에 의해 List, Integer, void 등 스트림 이외의 결과가 반환된다. 예를 들어 파이프라인에서 forEach는 소스의 각 요리에 람다를 적용한 다음에 void를 반환하는 최종 연산이다. System.out.println을 forEach에 넘겨주면 menu에서 만든 스트림의 모든 요리를 출력한다. `menu.stream().forEach(System.out::println);`

cf) 스트림 파이프라인의 개념은 빌더 패턴과 비슷하다. 

중간 연산

| 연산 | 형식 | 반환형식 | 연산의 인수 | 함수 디스크립터 |
| -- | -- | -- | -- | -- |
| filter | 중간연산 | `Stream<T>` | `Predicate<T>` | T -> boolean |
| map | 중간연산 | `Stream<T>` | `Function<T,R>` | T -> R |
| limit | 중간연산 | `Stream<T>` |  |  |
| sorted | 중간연산 | `Stream<T>` | `Comparator<T>` | (T,T) -> int |
| distinct | 중간연산 | `Stream<T>` |  | |  |
최종 연산

| 연산 | 형식 | 목적 |
| -- | -- | -- |
| forEach | 최종연산 | 스트림의 각 요소를 소비하면서 람다를 적용한다. void를 반환한다. |
| count | 최종연산 | 스트림의 요소 개수를 반환한다. long을 반환한다. |
| collect | 최종연산 | 스트림을 리듀스해서 리스트, 맵, 정수 형식의 컬렉션을 만든다. |

## 5장 - 스트림 활용
### 필터링 슬라이싱 
filter 메서드는 프레디케이트(불린을 반환하는 함수)를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다. 
``` java
// 채식 요리인지 확인하는 메서드 레퍼런스
List<Dish> vegetarianMenu = menu.stream()
	.filter(Dish::isVegetarian)
	.collect(toList());
```
고유 요소 필터링 
``` java
List<Integer> numbers = Arrays.asList(1,2,1,1,3,4,5);
numbers.stream().filter(i -> i%2 == 0).distinct().forEach(System.out::println);
```
스트림 축소
``` java
List<Dish> vegetarianMenu = menu.stream()
	.filter(d -> d.getCalories() > 300)
	.limit(3)
	.collect(toList());
```
요소 건너뛰기 
``` java
// 300칼로리 이상의 처음 두 요리를 건너 뛴 다음에 300칼로리가 넘는 나머지 요리를 반환한다.
List<Dish> vegetarianMenu = menu.stream()
	.filter(d -> d.getCalories() > 300)
	.skip(2)
	.collect(toList());
```

### 매핑
특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행되는 연산이다. 예를 들어 SQL의 테이블에서 특정 열만 선택할 수 있다. 스트림 API의 map과 flatMap 메서드는 특정 데이터를 선택하는 기능을 제공한다.

스트림의 각 요소에 함수 적용하기 
``` java
List<String> dishNames = menu.stream()
	.map(Dish::getName) // map 메서드의 출력 스트림은 Stream<String> 형식을 갖는다. 
	.collect(toList());

List<String> words = Arrays.asList(“java8”, “in”, “action”);
List<Integer> list = words.stream().map(String::length).collect(toList());
```

**스트림 평면화**

메서드 map을 이용해서 리스트의 각 단어의 길이를 반환하는 방법을 확인했다. 이를 응용해서 리스트에서 고유 문자로 이루어진 리스트를 반환해보자. 예를 들어 [“Hello”, “World”] 리스트가 있다면 결과로 [“H”,”e”,”l”,”o”,”W”,”r”,”d”]를 포함하는 리스트가 반환되어야 한다. 다음처럼 문제를 해결할 수 있다.
```java
words.stream()
	.map(word -> word.split(“”))
	.distinct()
	.collect(toList());
```
하지만 위 코드에서 map으로 전달한 람다는 각 단어의 String[](문자열 배열)을 반환한다는 문제가 있다. 따라서 map 메서드가 반환한 스트림의 형식은 `Stream<String[]>`이다. 우리가 원하는 것은 문자열의 스트림을 표현할 `Stream<String>`이다. 다행히 flatMap이라는 메서드를 이용해서 이 문제를 해결할 수 있다.

먼저 각 단어를 개별 문자열로 이루어진 배열로 만든 다음에 각 배열을 별로의 스트림으로 만들어야 한다. 

```java
List<String> words = Arrays.asList("Hello", "World");
List<String> str = words.stream()
        .map(w -> w.split(""))
        .flatMap(Arrays::stream)
        .distinct()
        .collect(toList());
System.out.println(str);
```
flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다. 즉, 생성된 스트림을 하나의 스트림으로 평면화한다. 

### 검색과 매칭
특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용된다. 스트림 API는 AllMatch, anyMatch, noneMatch, findFirst, findAny 등 다양한 유틸리티 메서드를 제공한다. 

```java
// 프레디케이트가 주어진 스트림에서 적어도 한 요소와 일치하는지 확인할 때 anyMatch 메서드를 이용한다. 
if(menu.stream().anyMatch(Dish::isVegetarian) { // anyMatch는 불린을 반환하므로 최종 연산이다. 
	...
}

// 프레디케이트가 모든 요소와 일치하는지 검사
boolean isHealthy = menu.stream().allMatch(d -> d.getCalories() < 1000); 

//noneMatch
boolean isHealthy = menu.stream().noneMatch(d -> d.getCalories() < 1000); 
```
anyMatch, allMatch, noneMatch 세 가지 메서드는 스트림 **쇼트서킷 기법**, 즉 자바의 &&, ||와 같은 연산을 활용한다.

**쇼트서킷 평가**

때로는 전체 스트리믈 처리하지 않았더라도 결과를 반환할 수 있다. 예를 들어 여러 and 연산으로 연결된 커다란 불린 표현식을 평가한다고 가정하자. 표현식에서 하나라도 거짓이라는 결과가 나오면 나머지 표현식의 결과와 상관없이 전체 결과도 거짓이 된다. 이러한 상황을 쇼트서킷이라고 부른다.

allMatch, noneMatch, findFirst, findAny 등의 연산은 모든 스트립의 요소를 처리하지 않고도 결과를 반환할 수 있다. 원하는 요소를 찾았으면 즉시 결과를 반환할 수 있다. 마찬가지로 스트림의 모든 요소를 처리할 필요 없이 주어진 크기의 스트림을 생성하는 limit도 쇼트서킷 연산이다. 특히 무한한 요소를 가진 스트림을 유한한 크기로 줄일 수 있는 유용한 연산이다.

**요소 검색**

findAny 메서드는 현재 스트림에서 임의의 요소를 반환한다.
 
```java
Optional<Dish> dish = menu.stream().filter(Dish::isVegetarian).findAny();
```
그런데 위 코드에 사용된 Optional은 무엇일까? `Optional<T>` 클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스다. 이전 예제에서 findAny는 아무 요소도 반환하지 않을 수 있다. null은 쉽게 에러를 일으킬 수 있으므로 자바8 라이브러리 설계자는 `Optional<T>`라는 기능을 만들었다. 
``` java
isPresent() 는 Optional이 값을 포함하면 참(true)을 반환한고, 값을 포함하지 않으면 거짓(false)를 반환한다.
ifPresent(Consumer<T> block)은 값이 있으면 주어진 블록을 실행한다. 

menu.stream()
	.filter(Dish::isVegetarian)
	.findAny() // Optional<Dish> 반환 
	.ifPresent(d -> System.out.println(d.getName()); // 값이 있으면 출력하고, 없으면 아무 일도 일어나지 않는다. 

T get()은 값이 존재하면 값을 반환하고 값이 없으면 NoSuchElementException을 일으킨다.
T orElse(T other)는 값이 있으면 값을 반환하고, 값이 없으면 기본값을 반환한다.
```

그런데 왜 findFirst와 findAny 두 가지 메서드 모두 필요할까? 바로 병렬성 때문이다. 병렬 실행에서는 첫 번째 요소를 찾기 어렵다. 따라서 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다. 

### 리듀싱
모든 스트림 요소를 처리해서 값으로 도출하는 것을 리듀싱 연산이라고 한다. 

**요소의 합**

`int sum = numbers.stream().reduce(0, (a,b) -> a+b);` reduce를 이용하면 애플리케이션의 반복된 패턴을 추상화할 수 있다. reduce는 두 개의 인수를 갖는다. 초기값 0과 두 요소를 조합해서 새로운 값을 만드는 `BinaryOperator<T>.` 메서드 레퍼런스를 이용해서 이 코드를 좀 더 간결하게 만들 수 있다. 자바 8에서는 Integer 클래스에 두 숫자를 더하는 정적 sum 메서드를 제공한다. `int sum = numbers.stream().reduce(0,Integer::sum);`

초기값을 받지 않도록 오버로드된 reduce도 있다. 그러나 이 reduce는 Optional 객체를 반환한다. `Optional<Integer> sum = numbers.stream().reduce((a,b)->(a+b));` 스트림에 아무 요소도 없는 상황이라면 초기값이 없으므로 reduce는 합계를 반환할 수 없다. 따라서 합계가 없음을 가리킬 수 있도록 Optional 객체로 감싼 결과를 반환한다. 

**최대/최소값**

`Optional<Integer> max = numbers.stream().reduce(Integer::max);`

**reduce 메서드의 장점과 병렬화**

reduce를 이용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 reduce를 실행할 수 있게 된다. 반복적인 합계에서는 sum 변수를 공유해야 하므로 쉽게 병렬화 하기 어렵다.

**스트림 연산: 상태 없음과 상태 있음**

map, filter 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다. 따라서 (사용자가 제공한 람다나 메서드 레퍼런스가 내부적인 가변 상태를 갖지 않는다는 가정 하에) 이들은 보통 상태가 없는, 즉 내부 상태를 갖지 않는 연산이다(stateless operation). 하지만 reduce, sum, max 같은 연산은 결과를 누적할 내부 상태가 필요하다. 스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 한정되어 있다.

sorted나 distinct 같은 연산은 filter나 map처럼 스트림을 입력으로 받아 다른 스트림을 출력하는 것처럼 보일 수 있다. 하지만 스트림의 요소를 정렬하거나 중복을 제거하려면 과거의 이력을 알고 있어야 한다. 따라서 이러한 연산은 내부 상태를 갖는 연산으로 간주할 수 있다. 

```java
실전 연습 
Trader raoul = new Trader("Raoul","Cambridge");
Trader mario = new Trader("Mario","Milan");
Trader alan = new Trader("Alan","Cambridge");
Trader brian = new Trader("Brian","Cambridge");

List<Transaction> transactions = Arrays.asList(
        new Transaction(brian,2011,300),
        new Transaction(raoul,2012,1000),
        new Transaction(raoul,2011,400),
        new Transaction(mario,2012,710),
        new Transaction(mario,2012,700),
        new Transaction(alan,2012,950)
);

//1번 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하라
List<String> list = transactions.stream()
                                    .filter(t -> t.getYear() ==2011)
                                    .map(Transaction::toString)
                                    .sorted()
                                    .collect(toList());
System.out.println("1번"+list);

//2번 거래자가 근무하는 모든 도시를 중복 없이 나열하시오
List<String> cities = transactions.stream()
                                    .map(t -> t.getTrader().getCity())
                                    .distinct()
                                    .collect(toList());
System.out.println("2번"+cities);

//3번 Cambridge에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오
List<String> tradersInCombridege = transactions.stream()
        .filter(t -> t.getTrader().getCity().equals("Cambridge"))
        .map(t -> t.getTrader().getName())
        .sorted()
        .distinct()
        .collect(toList());

System.out.println("3번"+tradersInCombridege);

List<Trader> tradersInCambridege2 = transactions.stream()
        .map(Transaction::getTrader)
        .filter(t -> t.getCity().equals("Cambridge"))
        .sorted(comparing(Trader::getName))
        .distinct()
        .collect(toList());

System.out.println("3번-2 "+tradersInCambridege2);

//4번 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오.
String traders = transactions.stream()
        .map(t ->t.getTrader().getName())
        .sorted()
        .distinct()
        .reduce("",(a,b)->a+b);
System.out.println("4번 "+traders);

String traders2 = transactions.stream()
        .map(t ->t.getTrader().getName())
        .sorted()
        .distinct()
        .collect(joining());
System.out.println("4번-2 "+traders2);

//5번 밀라노에 거래자가 있는가?
boolean milanoTrader = transactions.stream()
        .anyMatch(t -> t.getTrader().getCity().equals("Milan"));
System.out.println("5번"+milanoTrader);

//6번 Cambridge에 거주하는 거래자의 모든 트랜잭션 값을 출력하시오
List<String> transactionValue = transactions.stream()
                                            .map(t -> t.getTrader().getCity())
                                            .distinct()
                                            .collect(toList());
System.out.println("6번" +transactionValue);

//7번 전체 트랜잭션 중 최대값은 얼마인가
int max = transactions.stream().map(i -> i.getValue()).reduce(0,Integer::max);
System.out.println("7번 최대값 :"+max);

//8번 전체 트랜잭션 중 최소값
Optional<Integer> min = transactions.stream().map(i->i.getValue()).reduce(Integer::min);
System.out.println("8번 최소값 :"+min.get());
```
### 숫자형 스트림
스트림 API 숫자 스트림을 효율적으로 처리할 수 있도록 세 가지 기본형 특화 스트림을 제공한다. 박싱 비용을 피할 수 있도록 ‘int 요소에 특화된 IntStream’, ‘double 요소에 특화된 DoubleStream’, ‘long 요소에 특화된 LongStream’을 제공한다. 특화 스트림은 오직 박싱 과정에서 일어나는 효율성과 관련 있으며 스트림에 추가 기능을 제공하진 않는다는 사실을 기억하자. 

**숫자 스트림으로 매핑**

스트림을 특화 스트림으로 변환할 때는 mapToInt, mapToDouble, mapToLong 세 가지 메서드를 가장 많이 사용한다. 이들 메서드는 map과 정확히 같은 기능을 수행하지만, `Stream<T>` 대신 특화된 스트림을 반환한다. 
```java
int calories = menu.stream().mapToInt(Dish::getCalories).sum();
```

mapToInt 메서드는 각 요리에서 모든 칼로리를 추출한 다음에 IntStream(`Stream<Integer>`가 아님)을 반환한다. 따라서 IntStream 인터페이스에서 제공하는 sum 메서드를 이용해서 칼로리 합계를 계산할 수 있다. 스트림이 비어 있으면 sum은 기본값 0을 반환한다. IntStream은 max, min, average 등 다양한 유틸리티 메서드도 지원한다.

**객체 스트림으로 복원하기**

boxed 메서드를 이용해서 특화 스트림을 일반 스트림으로 변환할 수 있다. 
```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); // 스트림을 숫자 스트림으로 변환 
Stream<Integer> stream = intStream.boxed(); // 숫자 스트림을 스트림으로 변환 
```

**기본값 : OptionalInt**

합계 에제에서는 0이라는 기본값이 있었으므로 별 문제가 없었다. 하지만 IntStream에서 최대값을 찾을 때는 0이라는 기본값 때문에 잘못된 결과가 도출될 수 있다. 스트림에 요소가 없는 상황과 실제 최대값이 0인 상황을 어떻게 구별할 수 있을까. OptionalInt, OptionalDouble, OptionalLong 세 가지 기본형 특화 스트림 버전이 있다. 
```java
OptionalInt maxCalories = menu.stream().maxToInt(Dish::getCalories).max(); 
int max = maxCalories.orElse(1); // 값이 없을 때 기본 최대값을 명시적으로 설정 
```

**숫자 범위**

프로그램에서는 특정 범위의 숫자를 이용해야 하는 상황이 자주 발생한다. 자바8의 IntStream과 LongStream에서는 range와 rangeClosed라는 두 가지 정적 메서드를 제공한다. 두 메서드 모두 첫 번째 인수로 시작값을, 두 번째 인수로 종료값을 갖는다. range 메서드는 시작값과 종료값이 결과에 포함되지 않는 반면 rangeClosed는 시작값과 종료값이 결과에 포함된다는 점이 다르다. 
```java
IntStream evenNumbers = IntStream.rangeClosed(1,100).filter(n->n%2==0);
System.out.println(evenNumbers.count()); 
```
### 스트림 만들기
**값으로 스트림 만들기**

임의의 수를 인수를 받는 정적 메서드 Stream.of를 이용해서 스트림을 만들 수 있다. 예를 들어 다음 코드는 Stream.of로 문자열 스트림을 만드는 예제다. 스트림의 모든 문자열을 대문자로 변환한 후 문자열을 하나씩 출력한다.
```java
Stream<String> stream = Stream.of(“java8”,”lambda”,”in”,”action”);
stream.map(String::toUpperCase).forEach(System.out::println);
``` 
다음 처럼 empty 메서드를 이용해서 스트림을 비울 수 있다. `Stream<String> emptyStream = Stream.empty();`

**배열로 스트림 만들기**

배열을 인수로 받는 정적 메서드 Arrays.stream을 이용해서 스트림을 만들 수 있다. 
```java
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum();
```

**파일로 스트림 만들기**

파일을 처리하는 I/O 연산에 사용하는 자바의 NIO API도 스트림 API를 활용할 수  있도록 엡테이트되었다. java.nio.file.Files의 많은 정적 메서드가 스트림을 반환한다. 예를 들어 Files.lines는 주어진 파일의 행 스트림을 문자열로 반환한다. 
```java
long uniqueWords =0;

try(Stream<String> lines = 
	Files.lines(Paths.get(“data.txt”), Charset.defaultCharset())) { // 스트림은 자원을 자동으로 해제할 수 있는 AutoClosable이다.
		uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(“ “))) // 단어 스트림 생성 
			.distinct() // 중복 제거
			.count(); // 고유 단어 수 계산
}
catch(IOException e){ } // 파일을 열다가 예외가 발생하면 처리
```
**함수로 무한 스트림 만들기**

스트림 API는 함수에서 스트림을 만들 수 있는 두 개의 정적 메서드 Stream.iterate와 Stream.generate를 제공한다. 두 연산을 이용해서 무한 스트림, 즉 고정된 컬렉션에서 고정된 크기의 스트림을 만들었던 것과는 달리 크기가 고정되지 않은 스트림을 만들 수 있다. iterate와 generate에서 만든 스트림은 요청할 때마다 주어진 함수를 이용해서 값을 만든다. 따라서 무제한으로 값을 계산할 수 있다. 하지만 보통 무한한 값을 출력하지 않도록 limit(n) 함수를 함께 연결해서 사용한다. 

```java
// iterate를 사용하는 방법
Stream.iterate(0, n -> n+2)
	.limit(10)
	.forEach(System.out::println);
```

iterate 메서드는 초기값(예제에서는 0)과 람다(예제에서는 `UnaryOperator<T>` 사용)를 인수로 받아서 새로운 값을 끊임없이 생산할 수 있다. 예제에서는 람다 `n -> n+2` 즉 이전 결과에 2를 더한 값을 반환한다. 결과적으로 짝수 스트림을 생성한다. 기본적으로 기존 결과에 의존해서 순차적으로 연산을 수행한다. iterate는 요청할 때마다 값을 생산할 수 있으며 끝이 없으므로 무한 스트림을 만든다. 이러한 스트림을 **언바운드 스트림**이라고 표현한다. 

```java
// 피보나치 수열
Stream.iterate(new int[]{0,1},
		t -> new int[]{t[1], t[0] + t[1]})
	.limit(10)
	.map(t -> t[0])
	.forEach(System.out::println);

```

```java
// generate 사용하는 방법
Stream.generate(Math::random)
	.limit(5)
	.forEach(System.out::println);
```
iterate와 비슷하게 generate도 요구할 때 값을 계산하는 무한 스트림을 만들 수 있다. 하지만 iterate와 달리 generate는 생산된 각 값을 연속적으로 계산하지 않는다. generate는 `Supplier<T>`를 인수로 받아서 새로운 값을 생산한다.