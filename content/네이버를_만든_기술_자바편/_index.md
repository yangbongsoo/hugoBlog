+++
title = "네이버를 만든 기술 자바편"
pre ="<i class='fa fa-coffee' ></i> "
+++

### 1. 자바의 날짜와 시간 API
2014년에 최종 배포된 JDK8에는 JSR-310이라는 표준 명세로 날짜와 시간에 대한 새로운 API가 추가됐다. 스프링 프레임워크4.0에서는 JSR-310을 기본으로 지원한다.

```java
public class Jsr310Test {
	@Test
	public void shouldGetAfterOneDay() {
		LocalDate theDay = IsoChronology.INSTANCE.date(1582, 10, 4);
		DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy.MM.dd");
		assertThat(theDay.format(formatter)).isEqualTo("1582.10.04");

		LocalDate nextDay = theDay.plusDays(1);
		assertThat(nextDay.format(formatter)).isEqualTo("1582.10.05");
	}

	@Test
	public void shouldGetAfterOneHour() {
		ZoneId seoul = ZoneId.of("Asia/Seoul");
		ZonedDateTime theTime = ZonedDateTime.of(1988, 5, 7, 23, 0, 0, 0, seoul);
		DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy.MM.dd HH:mm");
		assertThat(theTime.format(formatter)).isEqualTo("1988.05.07 23:00");
		ZoneRules seoulRules = seoul.getRules();
		assertThat(seoulRules.isDaylightSavings(Instant.from(theTime))).isFalse();

		ZonedDateTime after1Hour = theTime.plusHours(1);
		assertThat(after1Hour.format(formatter)).isEqualTo("1988.05.08 01:00");
		assertThat(seoulRules.isDaylightSavings(Instant.from(after1Hour))).isTrue();
	}

	@Test
	public void shouldGetAfterOneMinute() {
		ZoneId seoul = ZoneId.of("Asia/Seoul");
		ZonedDateTime theTime = ZonedDateTime.of(1961, 8, 9, 23, 59, 59, 0, seoul);
		DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy.MM.dd HH:mm");
		assertThat(theTime.format(formatter)).isEqualTo("1961.08.09 23:59");

		ZonedDateTime after1Minute = theTime.plusMinutes(1);
		assertThat(after1Minute.format(formatter)).isEqualTo("1961.08.10 00:30");
	}

	@Test
	public void shouldGetAfterTwoSecond() {
		ZoneId utc = ZoneId.of("UTC");
		ZonedDateTime theTime = ZonedDateTime.of(2012, 6, 30, 23, 59, 59, 0, utc);
		DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy.MM.dd HH:mm:ss");
		assertThat(theTime.format(formatter)).isEqualTo("2012.06.30 23:59:59");

		ZonedDateTime after2Seconds = theTime.plusSeconds(2);
		assertThat(after2Seconds.format(formatter)).isEqualTo("2012.07.01 00:00:01");
	}

	@Test(expected=ZoneRulesException.class)
	public void shouldThrowExceptionWhenWrongTimeZoneId(){
		ZoneId.of("Seoul/Asia");
	}

	@Test
	public void shouldGetDate() {
		LocalDate theDay = LocalDate.of(1999, 12, 31);

		assertThat(theDay.getYear()).isEqualTo(1999);
		assertThat(theDay.getMonthValue()).isEqualTo(12);
		assertThat(theDay.getDayOfMonth()).isEqualTo(31);
	}

	@Test(expected=DateTimeException.class)
	public void shouldNotAcceptWrongDate() {
		LocalDate.of(1999, 13, 31);
	}


	@Test
	public void shouldGetDayOfWeek() {
		LocalDate theDay = LocalDate.of(2014, 1, 1);

		DayOfWeek dayOfWeek = theDay.getDayOfWeek();
		assertThat(dayOfWeek).isEqualTo(DayOfWeek.WEDNESDAY);
	}
}
```

JDK7에서도 백포트 모듈을 통해 JSR-310을 쓸 수 있다.
```xml
<dependency>
    <groupId>org.threeten</groupId>
    <artifactId>threetenbp</artifactId>
    <version>0.8.1</version>
</dependency>

```

**오늘 날짜 구하기**

```java
LocalDate date = LocalDate.now();
LocalTime time = LocalTime.now();

System.out.println(date.getYear());
System.out.println(date.getMonthValue());
System.out.println(date.getDayOfMonth());
System.out.println(time.getHour());
System.out.println(time.getMinute());
System.out.println(time.getSecond());
```

### 2. 자바의 HashMap은 어떻게 작동하는가?
HashMap은 보조 해시 함수를 사용하기 때문에 보조 해시 함수를 사용하지 않는 HashTable에 비해 해시 충돌 발생이 덜하다. 그리고 HashMap은 null을 키로 사용할 수 있다.

**해시 충돌처리**

1. 개방 주소법(open addressing)
    1. 선형탐색법 : 해당 위치가 포화상태면 한자리씩 옮기면서 빈자리 찾아감
    2. 2차탐색법 : 해당 위치가 포화상태면 그 자리의 주소 + m^2 값으로 옮기면서 빈자리 찾아감
2. 연쇄방법
    1. 합병연쇄 : 빈 버켓에 충돌을 일으키는 레코드를 삽입하고 그 위치를 포인터로서 기억시킴
    ![](/navertechjava8.jpg)
    2. 분리연쇄(seperate chaining) : 각 버켓을 주소로 하는 레코드들을 연결리스트로 연결하고 그 헤드 포인터를 해시 테이블에 저장![](/navertechjava9.jpg)



자바 HashMap에서 사용하는 방식은 seperate chaining이다. open addressing은 연속된 공간에 데이터를 저장하기 때문에 separate chaining보다 캐시 효율이 높다. 따라서 데이터 개수가 충분히 적다면 open addressing이 separate chaining보다 성능이 더 좋다. 하지만 배열의 크기가 커질수록 캐시 효율이 높다는 open addressing의 장점은 사라진다.

자바8에서는 데이터 개수가 많아지면 separate chaining에서 연결 리스트 대신 레드블랙 트리를 사용한다. 즉 하나의 해시 버킷에 8개의 키-값 쌍이 모이면 연결리스트를 트리로 변경한다. 만약 해당 버킷에 있는 데이터를 삭제해 개수가 6개에 이르면 다시 연결 리스트로 변경한다.

**보조 해시 함수**

HaspMap은 키-값 쌍 데이터 개수가 일정 개수 이상이면 해시 버킷의 개수를 두배로 늘린다.(버킷의 최대 개수는 2^30개) 그런데 이렇게 해시 버킷 크기를 두 배로 확장하는 것에는 결정적인 문제가 있다. 해시 버킷의 개수 M이 2^a 형태가 되기 때문에 'index = X.hashCode() % M'을 계산할때 X.hashCode()의 하위 a개의 비트만 사용하게 된다는 것이다. 즉 해시 함수가 32비트 영역을 고르게 사용하도록 만들었다 하더라도 해시 값을 2의 승수로 나누면 해시 충돌이 쉽게 발생할 수 있다. 이 때문에 보조 해시 함수가 필요하다.

'index = X.hashCode() % M'을 계산할 때 사용하는 M값은 소수일때 index값의 분포가 가장 균동할 수 있다. 그러나 M값이 소수가 아니기 때문에 별도의 보조 해시 함수를 이용해 index 값 분포가 가급적 균등할 수 있게 해야 한다.

보조 해시 함수의 목적은 '키'의 해시 값을 변형해 해시 충돌 가능성을 줄이는 것이다.

```java
static final int hash(Object key){
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
자바 8에서는 자바 5~7과는 다르게 새로운 방식의 보조 해시 함수를 사용하고 있다.

자바 8의 HashMap 보조 해시 함수는 상위 16비트 값을 XOR로 연산하는 매우 단순한 형태의 보조 해시 함수를 사용한다. 이유는 두가지다. 첫째, 자바 8에서는 해시 충돌이 많이 발생하면 연결 리스트 대신 레드블랙 트리를 사용하므로 해시 충돌 시 발생할 수 있는 성능 문제가 완화됐다. 둘째, 최근 해시 함수는 균등분포가 잘 되게 만들어지는 경향이 있어 자바7까지 사용했던 보조 해시 함수의 효과가 크지 않다. 두번째 이유가 좀 더 결정적인 원인이 되어 자바 8에서 보조 해시 함수 구현을 바꿨다.

**String 객체에 대한 해시 함수**

String 객체의 해시 함수에 31을 사용하는 이유는 31이 소수고 어떤 수에 31을 곱하는 것은 빠르게 계산할 수 있기 때문이다. '31N = 32N-N'인데, 32는 2^5이니 어떤 수에 대한 32를 곱한 값은 시프트 연산으로 쉽게 구현할 수 있다. 따라서 N에 31을 곱한 값은 `(N<<5)-N` 과 같다. 31을 곱하는 연산은 이렇게 최적화된 머신 코드로 생성할 수 있기 때문에 String 클래스에서 해시 값을 계산할 때는 31을 승수로 사용한다.

### 4. 람다가 이끌어 갈 모던 자바
람다 표현식은 함수를 간결하게 표현한다. 프로그래밍 언어의 개념으로는 단순한 익명 함수 생성 문법이라 이해할만 하다.

클로저 : 자신을 감싼 영역에 있는 외부 변수에 접근하는 함수다. 클로저에서 접근하는 함수 밖의 변수를 자유 변수라 한다. 이 정의에 따르면 람다 표현식으로 정의한 익명 함수 가운데 일부는 클로저고 일부는 클로저가 아니다.

자바8 인 액션에서 설명한 클로저 : 원칙적으로 클로저란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킨다. 예를 들어 클로저를 다른 함수의 인수로 전달할 수 있다. 클로저는 클로저 외부에 정의된 변수의 값에 접근하고, 값을 바꿀수 있다. 자바8의 람다와 익명 클래스는 모두 메서드의 인수로 전달될 수 있으며 자신의 외부 영역의 변수에 접근할 수 있지만 람다가 정의한 메서드의 지역변수 값은 바꿀수 없다.

**람다 표현식의 내부 구현**

자바의 람다는 앞에서 본 다른 JVM 언어가 그랬듯(groovy, scala, kotlin 일부 함수) 익명 클래스의 생성을 축약한 것처럼 보인다. 그러나 람다는 익명 클래스를 생성하지 않는다. 컴파일된 소스 폴더나 역컴파일을 해도 익명 클래스의 흔적은 없다.

우선 람다 표현식이 익명 클래스가 아니기 때문에 언어를 쓰는 사용자에게 가장 드러나는 차이는 this 키워드의 의미다.
```java
public class ThisDifference {
    //네이버를 만든 기술 - 자바편 p107
    public static void main(String[] args) {
        new ThisDifference().print();
    }

    public void print() {
        Runnable anonClass = new Runnable() {
            @Override
            public void run() {
                verifyRunnable(this);
            }
        };

        anonClass.run();

        Runnable lambda = () -> verifyRunnable(this);
        lambda.run();
    }

    private void verifyRunnable(Object obj) {
        System.out.println(obj.getClass());
    }
}

결과
class java8.ThisDifference$1 //익명 클래스
class java8.ThisDifference
```

익명 클래스 내부에서 전달한 this는 Runnable을 구현한 익명 클래스 그 자체인데 반해 람다 표현식을 썼을 때는 익명 클래스가 아닌 것이다. 람다 표현식 안에서 선언한 this의 타입은 이를 생성한 클래스인 ThisDifference다. 익명 클래스 안에서 이를 생성한 객체를 전달하려면 'ThisDifference.this' 처럼 직접 타입을 지정하면 된다.

이렇듯 람다 표현식은 익명 클래스와 다르다. 다른 JVM 언어처럼 람다 표현식을 익명 클래스로 바꾸는 것이 적절하고 쉬운 방법으로 보인다. 하지만 익명 클래스로는 매번 새로운 인스턴스를 생성하고 단순한 바이트코드 명세로 표현되지 않는 등의 단점이 있다. 결과적으로 자바8에서 람다 표현식은 invokedynamic이라는 바이트코드로 변환된다.

원래 invokedynamic은 자바 언어가 아닌 JRuby, Jython, Groovy와 같은 동적 타입 언어를 위한 명세였다. 동적 타입 언어는 컴파일 시점에 타입이 확정되지 않은 메서드를 런타임에 호출할 수 있는데 이를 효율적으로 지원하기 위해 자바7부터 invokedynamic 명세가 포함됐다.

cf) 람다 표현식을 쓴 코드를 자바6과 자바7에서도 실행 할 수 있도록 컴파일 하는 Retrolambda라는 프로젝트도 있다. 메이븐이나 그래들의 플러그인으로 설정하면 익명 클래스를 생성하는 방식으로 람다 표현식을 컴파일한다. 자바8의 문법을 지원하지 않는 안드로이드 환경에서도 활용할 만하다.

### 5. JVM 이해하기
JVM의 역할은 자바 애플리케이션을 클래스 로더를 통해 읽어 들여 자바 API와 함께 실행하는 것이다.

가상머신이란 프로그램을 실행하기 위해 물리적 머신(컴퓨터)과 유사한 머신을 소프트웨어로 구현한 것이라고 정의한다.

**JVM의 특징**

스택 기반의 가상 머신(대표적인 컴퓨터 아키텍처인 인텔x86 아키텍처나 ARM 아키텍처가 레지스터 기반으로 작동하는데 비해 JVM은 스택 기반으로 작동한다).

심벌릭 레퍼런스

가바지 컬렉션

기본 자료형을 명확하게 정의해 플랫폼 독립성 보장한다(플랫폼에 따라 기본 자료형 크기가 변하지 않는다).

네트워크 바이트 순서(자바 클래스 파일은 네트워크 바이트 순서를 사용한다).

안드로이드에 탑재된 달빅 VM은 JVM이긴 하지만 JVM 명세를 따르지 않는다. 스택 머신인 다른 JVM과 달리 달빅 VM은 레지스터 머신이다.

**JVM 구조**

![](/jvm1.PNG)

클래스 로더가 컴파일된 자바 바이트코드를 런타임 데이터 영역에 로드하고 실행엔진이 자바 바이트코드를 실행한다.

**클래스 로더 :** 자바는 동적코드, 컴파일 타임이 아니라 런타임에 클래스를 처음으로 참조할 때 해당 클래스를 로드하고 링크하는 특징이 있다. 이 동적 코드를 담당하는 부분이 JVM의 클래스 로더다(WAS 제조사마다 조금씩 다른 형태의 계층 구조를 갖고 있다).

**런타임 데이터 영역 :** JVM이라는 프로그램이 OS위에서 실행되면서 할당받는 메모리 영역.

![](/runtimedataarea.PNG)

Method Area와 Heap은 모든 스레드가 공유해서 사용한다.

**실행엔진 :** 클래스로더를 통해 JVM내의 런타임 데이터 영역에 배치된 바이트코드는 실행엔진에 의해 실행된다.

바이트 코드 --> 기계어

인터프리터 : 바이트코드 명령어를 하나씩 읽어서 해석하고 실행. 그래서 느려.

JIT 컴파일러 : 느림을 보완하기 위해 인터프리터 방식으로 실행하다가 적절한 시점에 바이트코드 전체를 컴파일해 네이티브 코드로 변경하고 직접 실행하는 방식(네이티브 코드를 캐시에 보관하기 때문에 빠름).

**한번만 실행되는 코드라면 컴파일 하지 않고 인터프리팅하는게 훨씬 유리**

### 10. 자바 가비지 컬렉션의 작동과정
**stop the world :** 가비지컬렉션을 실행하기 위해 JVM이 애플리케이션 실행을 멈추는것.(GC 실행하는 스레드 말고 나머지 스레드 작업 멈춰) GC 튜닝이랑 stop the world 시간을 줄이는것.

오라클 JVM인 HOTSPOT VM 에서는 물리적 공간을 둘로 나눔

![](/gcstructure.PNG)

**Young 영역의 구성**

새로 생성한 대부분의 객체는 Eden에 쌓인다. GC 한번 발생 후 살아남은 객체들은 Survivor로 옮겨가고 가득 차면 비어있는 Survivor로 이동한다. 그리고 가득찼던 Survivor 영역은 아무 데이터도 없는 상태가 된다. 이 과정을 반복하다 계속해서 살아남는 객체는 old 영역으로 이동한다. Survivor 영역 중 하나는 반드시 비어 있는 상태로 남아 있어야 한다. 만약 두 Survivor 영역에 모두 데이터가 존재하거나 두 영역 모두 사용량이 0이라면 시스템이 정상적인 상황이 아니라고 생각하면 된다.

참고로 HotSpot VM에서는 더욱 빠른 메모리 할당을 위해 두 가지 기술을 사용한다.

bump the pointer : Eden영역에 할당된 마지막 객체를 추적해서 빠르게 메모리 할당 이루어지게 하는 방법.

TLABS : 멀티스레드에서는 객체를 Eden에 넣으려면 스레드 세이프하기 위해 락이 발생할 수 밖에 없고, 락으로 인한 경합 때문에 성능이 떨어진다. 이를 해결 하기 위해, 스레드가 각각의 몫에 해당하는 Eden 영역의 작은 덩어리를 가질 수 있게 하는 기술이다.

간단하게 Young 영역에 대한 GC를 알아봤다. 위에서 이야기한 두 가지 기술(bump the pointer, TLABs)을 반드시 기억하고 있을 필요는 없다. 그러나 Eden 영역에 최초로 객체가 만들어지고, Survivor 영역을 통해 Old 영역으로 오래 살아남은 객체가 이동한다는 사실은 꼭 기억하기 바란다.

**Old 영역 GC(JDK 7기준)**

Old 영역은 기본적으로 데이터가 가득 차면 GC를 실행한다.

1. Serial GC : 운영서버에서 사용하면 안된다(데스크톱의 CPU 코어가 하나 만 있을 때 사용하기 위해 만든 방식이다). 이 방식은 mark-sweep-compact라는 알고리즘을 사용한다. 이 알고리즘의 첫 단계는 Old 영역에 살아 있는 객체를 식별(mark)하는 것이다. 그 다음에는 힙(heap)의 앞부분부터 확인해 살아 있는 것만 남긴다(sweep). 마지막 단계에서는 각 객체들이 연속으로 쌓이도록 힙의 맨 앞부분부터 채워서 객체가 존재하는 부분과 객체가 없는 부분으로 나눈다(compact). Serial GC는 적은 메모리와 CPU 코어 개수가 적을 때 적합한 방식이다.
2. Parallel GC : Serial GC와 기본적인 알고리즘은 같다. 그러나 Serial GC가 가비지 컬렉션을 처리하는 스레드가 하나인 것에 비해 Parallel GC는 가비지 컬렉션을 처리하는 스레드가 여러개다.
3. CMS GC : 초기 Initial Mark 단계에서는 클래스 로더에서 가장 가까운 객체 중 살아 있는 객체만 찾는 것으로 끝낸다. 따라서 멈추는 시간이 매우 짧다. 그리고 Concurrent Mark 단계에서는 방금 살아있다고 확인한 객체에서 참조하고 있는 객체를 따라가면서 확인하는데 이 단계의 특징은 다른 스레드가 실행 중인 상태에서 동시에 진행된다는 점이다. 그 다음 Remark 단계에서는 Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다. 마지막으로 Concurrent Sweep 단계에서는 쓰레기를 정리하는 작업을 실행한다. 이 작업도 다른 스레드가 실행되고 있는 상황에서 진행한다.
 이러한 단계로 진행되기 때문에 stop the world 시간이 매우 짧지만 다른 GC 방식보다 메모리와 CPU를 더 많이 사용하고 Compaction 단계가 기본적으로 제공되지 않는다. 따라서 조각난 메모리가 많아 Compaction 작업을 실행하면 오히려 stop the world 시간이 길어지기 때문에 신중하게 사용해야 한다.

4. G1 GC : 바둑판의 각 영역에 객체를 할당하고 GC를 실행. 그러다가 해당 영역이 꽉 차면 다른 영역에 객체를 할당하고 GC를 실행한다. Young->Old로 가는 이동단계가 사라진 방식

자바7부터 공식적으로 사용하게 된 G1 GC는 기존의 GC 개념을 획기적으로 발전시켜 놓았다. G1 GC는 멀티 프로세서 환경에서 대용량의 메모리가 사용되는 현대 애플리케이션을 목표로 새롭게 개발된 GC 메커니즘으로 간단히 말해 메모리 영역을 작은 단위(Region)로 나눠 CMS를 수행한다. 즉, 작은 영역별로 나뉘어진 메모리에 병렬로 GC를 수행해 지연이 최소화된 최적의 중단 없는 고성능 자바 실행 환경을 제공한다.

### 13. 자바의 Reference 클래스와 가비지 컬렉션
자바의 가비지 컬렉터는 작동 방식에 따라 종류가 다양하지만 공통적으로 크게 다음 두 가지 작업을 실행한다고 볼 수 있다.
1. 힙(heap) 내의 객체 중에서 가비지(garbage)를 찾아낸다.
2. 찾아낸 가비지를 처리해 힙의 메모리를 회수한다.

최초 버전의 자바는 이러한 가비지 컬렉션 작업에 사용자 코드가 관여하지 않도록 구현됐다. 하지만 JDK1.2부터는 java.lang.ref 패키지를 추가해 제한적이나마 사용자 코드와 가비지 컬렉터가 상호작용할 수 있게 했다.

java.lang.ref 패키지는 전형적인 객체 참조인 strong reference 외에도 soft reference, weak reference, phantom reference라는 3가지 새로운 참조 방식을 각 Reference 클래스로 제공한다. 이 3가지 Reference 클래스를 애플리케이션에 사용하면 가비지 컬렉션에 일정 부분 관여할 수 있고, LRU(least recently used) 캐시같이 특별한 작업을 하는 애플리케이션을 더 쉽게 작성 할 수 있다.

![](/navertechjava1.jpg)

**가비지 컬렉션과 접근 가능성**

가비지 컬렉터는 객체가 가비지인지 판별하기 위해 접근 가능성(reachability)이라는 개념을 사용한다. 어떤 객체에 유효한 참조가 있으면 접근 가능한(reachable) 상태로, 유효한 참조가 없으면 접근 불가능한(unreachable) 상태로 구별하고, 접근 불가능한 객체를 가비지로 간주해 가비지 컬렉션을 실행한다. 한 객체는 여러 다른 객체를 참조하고 참조된 다른 객체도 마찬가지로 또 다른 객체를 참조할 수 있으므로 객체들은 참조 사슬을 이룬다. 이런 상황에서 유효한 참조 여부를 파악하려면 항상 유효한 최초의 참조가 있어야 하는데 이를 객체 참조의 루트 세트(root set)라 한다.

![](/navertechjava2.jpg)
JVM에서 런타임 데이터 영역의 구조를 단순화하면 위와 같다. 런타임 데이터 영역은 스레드가 차지하는 영역과 객체를 생성 및 보관하는 하나의 큰 힙 영역, 클래스 정보가 차지하는 메서드 영역으로 크게 3부분으로 나눌 수 있다. 위 그림에서 객체에 대한 참조는 화살표로 표시된다.

**힙에 있는 객체에 대한 참조는 다음 4가지 종류 중 하나다.**

1. 힙 내의 다른 객체에 의한 참조
2. 자바 스택, 즉 자바 메서드 실행 시 사용하는 지역 변수와 파라미터에 의한 참조
3. 네이티브 스택, 즉 JNI(java native interface)에 의해 생성된 객체에 대한 참조
4. 메서드 영역의 정적 변수에 의한 참조

이들 중 힙 내의 다른 객체에 의한 참조를 제외한 나머지 세 개가 루트 세트로, 접근 가능성을 판가름하는 기준이 된다. 접근 가능성을 더 자세히 설명하기 위해 루트 세트와 힙 내의 객체를 중심으로 다음과 같이 재구성했다.

![](/navertechjava3.jpg)
위에서 볼 수 있듯이 루트 세트에서 시작한 참조 사슬에 속한 객체는 접근 가능한 객체다. 이 참조 사슬과 무관한 객체가 접근 불가능한 객체로 가비지 컬렉션 대상이다. 오른쪽 아래 객체처럼 접근 가능한 객체를 참조하더라도 다른 접근 가능한 객체가 이 객체를 참조하지 않는다면 이 객체는 접근 불가능한 객체다. 참고로 이 그림에서 참조는 모두 java.lang.ref 패키지를 사용하지 않는 일반적인 참조로 이를 흔히 strong reference라 한다.

**soft reference, weak reference, phantom reference**

java.lang.ref 패키지는 soft reference, weak reference, phantom reference를 클래스 형태로 제공한다. 예를 들면, java.lang.ref.WeakReference 클래스는 참조 대상인 객체를 캡슐화한 WeakReference 객체를 생성한다.
```java
public class WeakReference<T> extends Reference<T> {

    public WeakReference(T referent) {
        super(referent);
    }

    public WeakReference(T referent, ReferenceQueue<? super T> q) {
        super(referent, q);
    }

}
```
이렇게 생성된 WeakReference 객체는 다른 객체와 달리 가비지 컬렉터가 특별하게 취급한다. 캡슐화된 내부 객체는 weak reference에 의해 참조된다.

다음은 WeakReference 클래스가 객체를 생성하는 예다.

```java
WeakReference<Sample> wr = new WeakReference<>(new Sample());
Sample ex = wr.get();;
...
ex = null;
```
코드의 첫 번째 줄에서 생성한 WeakReference 클래스의 객체는 new() 메서드로 생성된 Sample 객체를 캡슐화한 것이다. 참조된 Sample 객체는 두 번째 줄에서 get() 메서드를 통해 다른 참조에 대입된다. 이 시점에서는 WeakReference 객체 내의 참조와 ex 참조, 두 개의 참조가 처음 생성된 Sample 객체를 가리킨다.

![](/navertechjava4.jpg)
앞 코드의 마지막 줄에서 ex 참조에 null을 대입하면 처음 생성한 Sample 객체는 오직 WeakReference 내부에서만 참조된다. 이 상태의 객체를 weakly reachable 객체라고 한다.

![](/navertechjava5.jpg)
자바 명세에는 SoftReference, WeakReference, PhantomReference라는 3가지 클래스에 의해 생성된 객체를 참조 객체(reference object)라 한다. 이는 흔히 strong reference로 표현되는 일반적인 참조나 다른 클래스의 객체와는 달리 위 3가지 Reference 클래스의 객체에 대해서만 사용하는 용어다. 또한 이들 참조 객체에 의해 참조된 객체는 'referent'라 한다. 위의 소스 코드에서 new WeakReference() 생성자로 생성된 객체는 참조 객체고, new Sample() 생성자로 생성된 객체는 referent다.

**Reference 클래스와 접근 가능성**

원래 가비지 컬렉션 대상인지는 접근 가능성 여부로만 판단했고 이를 사용자 코드에서는 관여할 수 없었다. 그러나 java.lang.ref 패키지를 이용해 접근 가능한 객체를 strongly reachable, softly reachable, weakly reachable, phantomly reachable로 더 자세히 구별해 가비지 컬렉션이 실행될 때의 동작을 다르게 지정할 수 있게 됐다. 다시 말해, 가비지 컬렉션 대상 여부를 판별하는 부분에 사용자 코드가 개입할 수 있게 됐다.

![](/navertechjava6.jpg)
녹색으로 표시한 중간의 두 객체는 WeakReference로만 참조된 weakly reachable 객체이고, 파란색 객체는 strongly reachable 객체이다. GC가 동작할 때, unreachable 객체뿐만 아니라 weakly reachable 객체도 가비지 객체로 간주되어 메모리에서 회수된다. root set으로부터 시작된 참조 사슬에 포함되어 있음에도 불구하고 GC가 동작할 때 회수되므로, 참조는 가능하지만 반드시 항상 유효할 필요는 없는 LRU 캐시와 같은 임시 객체들을 저장하는 구조를 쉽게 만들 수 있다.

위 그림에서 WeakReference 객체 자체는 weakly reachable 객체가 아니라 strongly reachable 객체다. 또한 그림에서 A로 표시한 객체와 같이 WeakReference에 의해 참조되면서 동시에 루트 세트에서 시작한 참조 사슬에 포함된 경우에는 weakly reachable 객체가 아니라 strongly reachable 객체다.

가비지 컬렉터가 작동해 어떤 객체를 weakly reachable 객체로 판명하면 가비지 컬렉터는 WeakReference 객체에 있는 weakly reachable 객체에 대한 참조를 null로 설정한다. 이에 따라 weakly reachable 객체는 접근 불가능한 객체와 마찬가지 상태가 되고, 가비지로 판명된 다른 객체와 함께 메모리 회수 대상이 된다.

**접근 가능성의 세기**

접근 가능성 정도는 가비지 컬렉터가 객체를 처리하는 기준이 되는데, 자바 명세에서는 이를 접근 가능성의 세기(strengths of reachability)라 하고 정도에 따라 5가지 상태로 분류한다.

1. strongly reachable : 루트 세트에서 시작해서 어떤 참조 객체도 중간에 끼지 않은 상태로 참조 가능한 상태.
2. softly reachable : strongly reachable 객체가 아닌 객체 중에서 weak reference, phantom reference 없이 soft reference만 통과하는 참조 사슬이 하나라도 있는 상태.
3. weakly reachable : strongly reachable 객체도, softly reachable 객체도 아닌 객체 중에서 phantom reference 없이 weak reference만 통과하는 참조 사슬이 하나라도 있는 상태.
4. phantomly reachable : strongly reachable 객체, softly reachable 객체, weakly reachable 객체 모두 해당되지 않는 상태. 이 상태는 파이널라이즈(finalize)됐지만 아직 메모리가 회수되지 않은 상태다.
5. unreachable : 루트 세트에서 시작되는 참조 사슬로 참조되지 않는 상태

위에서 예로 든 WeakReference 외에도 SoftReference나 PhantomReference 등을 이용해서도 접근 가능성을 지정할 수 있고 이에 따라 각 객체의 가비지 컬렉션 여부는 달라진다. 하나의 객체에 대한 참조의 개수나 참조 형태는 아무런 제한이 없으므로 하나의 객체는 다양한 조합으로 참조될 수 있다.

가비지 컬렉터는 루트 세트에서 시작해 객체에 대한 모든 경로를 탐색하고 그 경로에 있는 참조 객체를 조사해 그 객체에 대한 접근 가능성을 결정한다. 다양한 참조 관계의 결과 하나의 객체는 앞서 언급한 5가지 접근 가능성 중 하나가 된다.

![](/navertechjava7.jpg)

위 그림에서 객체 B의 접근 가능성은 softly reachable이다. 루트 세트에서 SoftReference를 통해 B를 참조할 수 있기 때문이다. 만약 루트 세트의 SoftReference에 대한 참조가 없다면 객체 B는 phantomly reachable이 된다.

**softly reachable과 SoftReference**

softly reachable 객체, 즉 strong reachable이 아니면서 오직 SoftReference 객체로만 참조된 객체는 힙에 남아 있는 메모리 크기와 해당 객체의 사용 빈도에 따라 가비지 컬렉션 여부가 결정된다. 그래서 softly reachable 객체는 weakly reachable 객체와는 달리 가비지 컬렉터가 작동할 때마다 회수되지 않으며 자주 사용될수록 더 오래 살아남는다.

softly reachable 객체를 가비지 컬렉션하기로 결정하면 앞서 설명한 WeakReference 경우와 마찬가지로 참조 사슬에 존재하는 SoftReference 객체 내의 softly reachable 객체에 대한 참조가 null로 설정되며, 이후 이 softly reachable 객체는 접근 불가능한 객체로 취급되어 가비지 컬렉터에 의해 메모리가 회수된다.

softly reachable 객체는 힙에 남아 있는 메모리가 많을수록 회수 가능성이 낮기 때문에 다른 비지니스 로직 객체를 위해 어느 정도 비워 둬야 할 힙 공간이 softly reachable 객체에 의해 일정 부분 점유된다. 따라서 전체 메모리 사용량이 높아지고 가비지 컬렉션이 더 자주 일어나며 가비지 컬렉션에 걸리는 시간도 상대적으로 길어지는 문제가 있다.

**weakly reachable과 WeakReference**

weakly reachable 객체는 특별한 정책에 의해 가비지 컬렉션 여부가 결정되는 softly reachable 객체와는 달리 가비지 컬렉션을 실행할 때마다 회수 대상이 된다. 앞서 설명한 것처럼 WeakReference 내의 참조가 null로 설정되고 weakly reachable 객체는 접근 불가능한 객체와 같은 상태가 돼 가비지 컬렉터에 의해 메모리가 회수된다. 그러나 가비지 컬렉션 알고리즘에 따라 객체 회수 시기가 결정되므로 가비지 컬렉션이 실행될 때마다 메모리까지 회수되지는 않을 수 있다. 이는 softly reachable 객체는 물론 접근 불가능한 객체도 마찬가지다. 가비지 컬렉터가 가비지 컬렉션 대상인 객체를 찾는 작업과 가비지 컬렉션 대상인 객체를 처리해 메모리를 회수하는 작업은 즉각적인 연속 작업이 아니며, 가비지 컬렉션 대상 객체의 메모리를 한 번에 모두 회수하지도 않는다.

LRU 캐시와 같은 애플리케이션에서는 softly reachable 객체보다는 weakly reachable 객체가 유리하므로 LRU 캐시를 구현할 때는 대체로 WeakReference를 사용한다.

**ReferenceQueue**

phantomly reachable 객체의 동작 방식과 PhantomReference를 이해하려면 java.lang.ref 패키지에서 제공하는 ReferenceQueue 클래스를 알아야 한다.

앞서 SoftReference 객체나 WeakReference 객체가 참조하는 객체가 가비지 컬렉션 대상이 되면 SoftReference 객체와 WeakReference 객체 내의 참조는 null로 설정된다고 설명했다. 이때 SoftReference 객체와 WeakReference 객체 자체는 ReferenceQueue의 큐에 추가된다.

ReferenceQueue의 큐에 추가하는 작업은 가비지 컬렉터에 의해 자동으로 실행된다. ReferenceQueue의 poll() 메서드나 remove() 메서드를 이용해 ReferenceQueue에 이들 참조 객체가 큐에 추가됐는지 확인하면 softly reachable 객체나 weakly reachable 객체의 가비지 컬렉션 여부를 파악할 수 있고, 이에 따라 관련 리소스나 객체에 대한 후속 작업을 진행할 수 있다. **어떤 객체가 더 이상 필요 없게 됐을 때 이와 관련한 후처리가 필요한 애플리케이션에서 이 ReferenceQueue를 유용하게 사용할 수 있다.** 자바의 Collections 클래스 중에서 간단한 캐시를 구현하는 용도로 자주 사용되는 WeakHashMap 클래스는 ReferenceQueue와 WeakReference를 사용해 구현했다.

Reference 추상 클래스 생성자 코드
```java
    /* -- Constructors -- */

    Reference(T referent) {
        this(referent, null);
    }

    Reference(T referent, ReferenceQueue<? super T> queue) {
        this.referent = referent;
        this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
    }
```
SoftReference와 WeakReference는 ReferenceQueue를 사용할 수도 있고 사용하지 않을 수도 있다. 이는 이들 클래스의 생성자 중에서 ReferenceQueue를 받는 파라미터로 받는 생성자를 사용할지 여부로 결정한다. 그러나 PhantomReference는 반드시 ReferenceQueue를 사용해야 한다. PhantomReference의 생성자는 단 하나이며 항상 ReferenceQueue를 파라미터로 받기 때문이다.

SoftReference와 WeakReference는 객체 내부의 참조가 null로 설정된 이후에 ReferenceQueue의 큐에 추가되지만 PhantomReference는 객체 내부의 참조를 null로 설정하지 않고 참조된 객체를 phantomly reachable 객체로 만든 이후에 ReferenceQueue의 큐에 추가된다. 이를 통해 애플리케이션은 객체의 파이널라이즈 이후에 필요한 작업을 처리할 수 있게 된다.

**phantomly reachable과 PhantomReference**

phantomly reachable 객체의 작동 방식은 softly reachable 객체와 weakly reachable 객체의 작동 방식과는 많이 다르다. phantomly reachable 객체의 작동 방식을 이해하려면 먼저 가비지 컬렉션의 작동 방식을 알아야 한다. 가비지 컬렉션의 대상 객체를 찾는 작업과 가비지 컬렉션의 대상 객체를 처리하는 작업이 연속적이지 않듯이 가비지 컬렉션의 대상 객체를 처리하는 작업과 할당된 메모리를 회수하는 작업도 연속된 작업이 아니다. 메모리 회수 작업은 가비지 컬렉션의 대상 객체를 처리하는 작업, 즉 객체의 파이널라이즈 이후에 가비지 컬렉션의 알고리즘을 기반으로 실행된다.

가비지 컬렉션 대상 여부를 결정하는 부분에 관여하는 softly reachable, weakly reachable과는 달리 phantomly reachable은 파이널라이즈와 메모리 회수 사이에 관여한다. strongly reachable, softly reachable, weakly reachable에 해당하지 않고 PhantomReference로만 참조되는 객체는 먼저 파이널라이즈된 이후에 phantomly reachable로 간주된다. 다시 말해, 객체에 대한 참조가 PhantomReference만 남게 되면 해당 객체는 바로 파이널라이즈된다. 가비지 컬렉터가 객체를 처리하는 순서는 항상 다음과 같다.

1. soft reference
2. weak reference
3. 파이널라이즈
4. phantom reference
5. 메모리 회수

즉, 어떤 객체에 대해 가비지 컬렉션 여부를 판별할 때 먼저 대상 객체의 접근 가능성을 strongly, softly, weakly 순서로 판별한다. 판별 과정을 모두 거쳐도 해당되는 접근 가능성이 없으면 가비지 컬렉션 대상이 돼 위의 순서에 의해 처리된다. 가비지 컬렉터는 해당 객체에 대해 파이널라이즈를 진행한 이후에 phantomly reachable 여부를 판별한다. 대상 객체를 참조하는 PhantomReference가 있다면 phantomly reachable로 간주해 PhantomReference를 ReferenceQueue에 넣고 파이널라이즈 이후 작업을 애플리케이션이 실행하게 하고 메모리 회수를 지연시킨다.

이처럼 PhantomReference를 사용하면 어떤 객체가 파이널라이즈된 이후에 할당된 메모리가 회수되는 시점에 사용자 코드가 관여할 수 있게 된다. 파이널라이즈 이후에 처리해야 하는 리소스 정리 등의 작업이 있다면 유용하게 사용할 수 있다.