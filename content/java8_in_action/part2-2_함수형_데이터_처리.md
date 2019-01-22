+++
title = "Part2-2 함수형 데이터 처리"
+++

## 6장 - 스트림으로 데이터 수집

4장과 5장에서는 스트림에서 최종 연산 collect를 사용하는 방법을 확인했다. 하지만 toList로 스트림 요소를 항상 리스트로만 변환했다. 이 장에서는 reduce가 그랬던  것처럼 collect 역시 다양한 요소 누적 방식을 인수로 받아서 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행할 수 있음을 설명한다.

```java
// 통화별로 트랜잭션을 그룹화한 코드 - 명령형 버전
Map<Currency, List<Transaction>> transactionByCurrencies = new HashMap<>();

for(Transaction transaction : transactions){
  Currency currency = transaction.getCurrency();
  List<Transaction> transactionForCurrency = transactionByCurrencies.get(currency);

  if(transactionForCurrency == null){
    transactionForCurrency = new ArrayList<>();
    transactionByCurrencies.put(currency, transactionForCurrency);
  }

  transactionForCurrency.add(transaction);
}
```

통화별로 트랜잭션 리스트를 그룹화하기 위해 위와 같은 방법도 있지만 자바8에서는 더 간결한 구현이 가능하다.

```java
Map<Currency, List<Transaction>> transactionByCurrencies = 
  transactions.stream().collect(groupingBy(Transaction::getCurrency));
```

### 컬렉터란 무엇인가?

Collector 인터페이스 구현은 스트림의 요소를 어떤 식으로 도출할지 지정한다. 5장에서는 '각 요소를 리스트로 만들어라'를 의미하는 toList를 Collector 인터페이스의 구현으로 사용했다. 여기서는 groupingBy를 이용해서 '각 키(통화) 버킷 그리고 각 키 버킷에 대응하는 요소 리스트를 값으로 포함하는 맵을 만들라'는 동작을 수행한다.

collect 메서드로 Collector 인터페이스 구현을 전달한다. 스트림에 collect를 호출하면 스트림의 요소에 내부적으로 리듀싱 연산이 수행된다. 통화 예제에서 보여주는 것처럼 Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할지 결정된다. Collectors 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 정적 팩토리 메서드를 제공한다. ex) toList(), counting()

**리듀싱과 요약**

첫 번째 예제로 counting()이라는 팩토리 메서드가 반환하는 컬렉터로 메뉴에서 요리 수를 계산한다.

`long howMayDishes = menu.stream().collect(counting());`

두 번째는 메뉴에서 칼로리가 가장 높은 요리를 찾는다고 해보자. Collectors.maxBy, Collectors.minBy 두 개의 메서드를 이용해서 스트림의 최댓값과 최솟값을 계산할 수 있다. 두 컬렉터는 스트림의 요소를 비교하는데 사용할 Comparator를 인수로 받는다.

```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```

또한 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산에도 리듀싱 기능이 자주 사용된다. 이러한 연산을 **요약 연산**이라 부른다.

다음은 메뉴 리스트의 총 칼로리를 계산하는 코드다.

```java
int totalCalories = menu.stream.collect(summingInt(Dish::getCalories));
```

summingInt 뿐만 아니라 summingLong, summingDouble, averagingInt, averagingLong, averagingDouble 등 다양한 형식이 존재한다.

```java
double avgCalories = menu.stream.collect(averagingInt(Dish::getCalories));
```

두 개 이상의 연산을 한 번에 수행해야 할 때도 있다. 이런 상황에서는 팩토리 메서드 summarizingInt가 반환하는 컬렉터를 사용할 수 있다. 예를 들어 다음은 하나의 요약 연산으로 메뉴에 있는 요소수, 합계, 평균, min, max 등을 계산하는 코드다.

```java
IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
```

위 코드를 실행하면 IntSummaryStatistics 클래스로 모든 정보가 수집된다.

```java
IntSummaryStatistics { count=9, sum=4300, min=120, average=477.777778, max=800 }
```

마찬가지로 int뿐 아니라 long이나 double에 대응하는 summarizingLong, summarizingDouble 메서드와 관련된 LongSummaryStatistics, DoubleSummaryStatistics 클래스도 있다.

**문자열 연결**

문자열 연결을 위해 joining메서드는 내부적으로 StringBuilder를 이용해서 문자열을 하나로 만든다. 추가적으로 연결된 문자열들 사이에 구분 문자열을 넣을 수 있도록 오버로드된 joining 팩토리 메서드도 있다.

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(","));

//코드 실행 결과
pork, beef, chicken, french fries, rice, season fruit, pizza, prawns, salmon
```

**범용 리듀싱 요약 연산**

지금까지 살펴본 모든 컬렉터는 reducing 팩토리 메서드로도 정의할 수 있다. 즉 범용 Collectors.reducing으로도 구현할 수 있다.

```java
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i,j) -> i+j));
```

reducing은 세 개의 인수를 받는다. 첫 번째 인수는 리듀싱 연산의 시작값이거나 스트림에 인수가 없을 때는 반환값이다. 두 번째 인수는 변환 함수다. 세 번째 인수는 같은 종류의 두 항목을 하나의 값으로 더하는 BinaryOperator다.

다음처럼 한 개의 인수를 가진 reducing 버전을 이용해서 가장 칼로리가 높은 요리를 찾는 방법도 있다.

```java
Optional<Dish> mostCalorieDish = menu.stream().collect(reducing(
  (d1, d2) -> d1.getCaloriees() > d2.getCalories() ? d1 : d2));
```

한 개의 인수를 갖는 reducing 컬렉터는 시작값이 없으므로 빈 스트림이 넘겨졌을 때 시작값이 설정되지 않는 상황이 벌어진다. 그래서 Optional로 받고, 반환함수가 자기 자신이기 때문에(항등 함수) 최종적으로 `Optional<Dish>` 객체를 반환한다.

**컬렉션 프레임워크 유연성: 같은 연산도 다양한 방식으로 수행할 수 있다.**

이전 예제의 람다표현식 대신 Integer 클래스의 sum 메서드 레퍼런스를 이용하면 코드를 좀 더 단순화할 수 있다.

```java
int totalCalories = menu.stream().collect(reducing(
  0, // 초기값
  Dish::getCalories, //변환 함수
  Integer::sum)); // 합계 함수
```

또 컬렉터를 이용하지 않는 방법도 있다.

```java
int totalCalories = menu.stream().map(Dish::getCalories).reduce(Integer::sum).get();
```

reduce(Integer::sum)도 빈 스트림과 관련한 널 문제를 피할 수 있도록 int가 아닌 `Optional<Integer>`를 반환한다. 그리고 get으로 Optional 객체 내부의 값을 추출했다. 요리 스트림은 비어있지 않다는 사실을 알고 있으므로 get을 자유롭게 사용할 수 있다. 마지막으로 스트림을 IntStream으로 매핑한 다음에 sum 메서드를 호출하는 방법으로도 결과를 얻을 수 있다.

```java
int totalCalories = menu.stream().mapToInt(Dish::getCalories).sum();
```

마지막 방식이 가독성이 가장 좋고 간결하다. 또한 IntStream 덕분에 자동 언박싱 연산을 수행하거나 Integer를 int로 변환하는 과정을 피할 수 있으므로 성능까지 좋다.

### 그룹화

메뉴를 그룹화한다고 가정하자. 예를 들어 고기를 포함하는 그룹, 생선을 포함하는 그룹, 나머지 그룹으로 메뉴를 그룹화할 수 있다. 다음처럼 팩토리 메서드 Collectors.groupingBy를 이용해서 쉽게 메뉴를 그룹화할 수 있다.

```java
Map<Dish.Type, List<Dish>> dishesByType = 
    menu.stream().collect(groupingBy(Dish::getType));
```

다음은 Map에 포함된 결과다.

```
{Fish=[prawns, salmon], OTHER=[french fries, rice], MEAT=[pork, beef, chicken]}
```

스트림의 각 요리에서 Dish.Type과 일치하는 모든 요리를 추출하는 함수를 groupingBy 메서드로 전달했다. 이 함수를 기준으로 스트림이 그룹화되므로 이를 분류 함수라고 부른다.

그런데 위와 같이 단순한 분류 기준이 아닌 복잡한 분류 기준이 필요한 상황에서는 메서드 레퍼런스를 분류 함수로 사용할 수 없다. 예를 들어 400칼로리 이하를 'diet'로, 400~700칼로리를 'normal'로, 700칼로리 초과를 'fat' 요리로 분류한다고 가정하자. Dish 클래스에는 이러한 연산에 필요한 메서드가 없으므로 메서드 레퍼런스를 분류 함수로 사용할 수 없다. 따라서 다음 예제처럼 람다 표현식으로 필요한 로직을 구현해야 한다.

```java
public enum CaloricLevel { DIET, NORMAL, FAT }

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
    groupingBy(dish -> {
        if (dish.getCalories() <= 400)
            return CaloricLevel.DIET;
        else if (dish.getCalories() <= 700)
            return CaloricLevel.NORMAL;
        else
            return CaloricLevel.FAT;
    })
);
```

**다수준 그룹화**

지금까지 메뉴의 요리를 종류 또는 칼로리로 그룹화하는 방법을 살펴봤다. 그러면 요리 종류와 칼로리 두 가지 기준으로 동시에 그룹화할 수 있을까?

두 인수를 받는 팩토리 메서드 Collections.groupingBy를 이용해서 항목을 다수준으로 그룹화할 수 있다.

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = 
    menu.stream().collect(
        groupingBy(Dish::getType,
            groupingBy(dish -> {
                if (dish.getCalories() <= 400)
                    return CaloricLevel.DIET;
                else if (dish.getCalories() <= 700)
                    return CaloricLevel.NORMAL;
                else
                    return CaloricLevel.FAT;
})));
```

```
{MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]},
 FISH={DIET=[prawns], NORMAL=[salmon]},
 OTHER={DIET=[rice, seasonal fruit], NORMAL=[french fries, pizza]}}
```

### 분할

분할은 분할 함수라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능이다. 분할 함수는 불린을 반환하므로 맵의 키 형식은 Boolean이다. 결과적으로 그룹화 맵은 최대 두 개(T/F)의 그룹으로 분류된다. 예를 들어 채식주의자 친구를 저녁에 초대했다고 가정하자. 그러면 이제 모든 요리를 채식 요리와 채식이 아닌 요리로 분류 해야 한다.

```java
Map<Boolean, List<Dish>> partitionedMenu = 
    menu.stream().collect(partitioningBy(Dish::isVegetarian));
```

위 코드를 실행하면 다음과 같은 맵이 반환된다.

```
{false=[pork, beef, chicken, prawns, salmon],
 true=[french fries, rice, season fruit, pizza]}
```

이제 참값의 키로 맵에서 모든 채식 요리를 얻을 수 있다.

```java
List<Dish> vegetarianDishes = partitionedMenu.get(true);
```

물론 이전 예제에서 사용한 프레디케이트로 필터링한 다음에 별도의 리스트에 결과를 수집해도 같은 결과를 얻을 수 있다.

```java
List<Dish> vegetarianDishes = menu.stream().filter(Dish::isVegetarian).collect(toList());
```



