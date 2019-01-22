+++
title = "자바의 정석"
pre ="<i class='fa fa-coffee' ></i> "
+++

### Chapter 1 자바를 시작하기 전에
**동적 로딩(Dynamic Loading)을 지원한다.**<br>
자바로 작성된 애플리케이션은 여러 개의 클래스로 구성되어 있다. 자바는 동적 로딩을 지원하기 때문에 실행 시에 모든 클래스가 로딩되지 않고 필요한 시점에 클래스를 로딩하여 사용할 수 있다는 장점이 있다. 또한 일부 클래스가 변경되어도 전체 애플리케이션을 다시 컴파일 하지 않아도 되며, 애플리케이션의 변경사항이 발생해도 비교적 적은 작업만으로도 처리할 수 있다.

**javac.exe**<br>
자바 컴파일러, 자바소스코드를 bytecode로 컴파일한다.<br>

**java.exe**(JIT 컴파일러)<br>
자바 인터프리터, 컴파일러가 생성한 bytecode를 해석하고 실행한다.<br>

JRE(자바실행환경) = JVM + Java API(클래스 라이브러리)<br>
JDK(자바개발도구) = JRE + 개발에 필요한 실행파일(javac.exe 등)

### Chapter 2 변수
**리터럴**<br>
그 자체로 데이터인 것을 리터럴(literal)이라고 한다. 상수(constant)와 의미가 같지만 프로그래밍에서는 상수를 '값을 한번 저장하면 변경할 수 없는 저장공간' 으로 정의하기 때문에 이와 구분하기 위해 '리터럴'이라는 용어를 사용한다.

### Chapter 3 연산자
**연산자**<br>
값의 범위가 작은 타입에서 큰 타입으로의 형변환은 생략할 수 있다. 그러나 큰 자료형에서 작은 자료형으로의 형변환에 캐스트 연산자를 사용하지 않으면 컴파일시 에러가 발생한다.

long은 8byte고 float은 4byte지만 실수형이 정수형보다 훨씬 더 큰 범위를 갖기 때문에 float이 더 크다고 할 수 있다. 또한 char와 short 모두 2byte지만 char는 음수를 갖지 않으므로 서로 범위가 달라서 자동적으로 형변환이 수행되지 않는다.

**사칙연산**<br>
사칙 연산에서 int형 보다 크기가 작은 자료형은 int형으로 형변환 후에 연산을 수행한다.<br>
byte + short -> int + int -> int<br>
두개의 피연산자 중 자료형의 표현범위가 큰 쪽에 맞춰서 형변환 된 후 연산을 수행한다.<br>
float + long -> float + float -> float <br>

**쉬프트 연산자**<br>
쉬프트 연산자는 정수형 변수에만 사용할 수 있는데 `<<`연산자의 경우, 피연산자의 부호에 상관없이 자리를 왼쪽으로 이동시키며 빈칸을 0으로만 채우면 되지만 `>>`연산자는 오른쪽으로 이동시키기 때문에 음수인 경우 부호를 유지시켜주기 위해 빈자리를 1로 채우게 된다. 반면에 `>>>`연산자는 부호에 상관없이 항상 0으로 빈자리를 채운다. 그렇기 대문에 10진 연산보다는 비트연산(2진 연산)에 주로 사용된다.

### Chapter 4 조건문과 반복문
**이름 붙은 반복문** <br>
```java
Loop1 : for(int i=0; i<9; i++){
            for(int j=1; j<9; j++){
                if(j==5)
                    break Loop1;
            }
        }
```
위와 같이 break를 설정하면 바로 빠져나오게 할 수 있다.

### Chapter 5 배열
**배열의 복사**<br>
System클래스의 arraycopy()를 사용하면 보다 간단히 배열을 복사할 수 있다.
```java
System.arraycopy(arr1, 0, arr2, 0, arr1.length);
```
arr1배열의 0번째부터 arr1.length까지의 내용을 arr2의 0번째부터 복사하겠다는 뜻

### Chapter 6 객체지향 프로그래밍1
**class**<br>
프로그래밍 언어에서 데이터 처리를 위한 저장형태의 발전과정은 다음과 같다.
![](/data.PNG)
그동안 데이터와 함수가 서로 관계가 없는 것처럼 따로 다루어져 왔지만, 사실 함수는 주로 데이터를 가지고 작업을 하기 때문에 많은 경우에 있어서 데이터와 함수는 관계가 깊다. 예를들어 C언어에서는 문자열을 문자의 배열로 다루지만, 자바에서는 String이라는 클래스로 문자열을 다룬다. 문자열을 단순히 문자의 배열로 정의하지 않고 클래스로 정의한 이유는 문자열과 문자열을 다루는데 필요한 함수들을 함께 묶기 위해서이다.

**parameter**<br>
메서드의 매개변수가 있는 경우, 본격적인 작업을 시작하기에 앞서 넘겨받은 매개변수의 값이 유효한 것인지 꼭 확인 하는것이 중요하다.

**return**<br>
아래 왼쪽같이 메서드 내에서 return문을 여러 번 쓰는 것보다 가능하면 오른쪽처럼 마지막에 한번만 사용하는 것이 좋다.
```java
int max(int a, int b)            int max(int a, int b)
{                                {
    if(a>b)                          int result = 0;
        return a;                    if(a>b)
    else                                result = a;
        return b;                    else
}                                       result = b;

                                     return result;
                                  }
```

**리턴값이 있는 메서드를 리턴값이 없는 메서드로 바꾸는 방법**<br>
```java
public static void main(String[] args)
{
    ReturnTest r = new ReturnTest();

    int[] result = {0};
    r.add(3,5,result);
    System.out.println(result[0]);
}

void add(int a, int b, int[] result)
{
    result[0] = a + b;
}
```
메서드는 단 하나의 값만을 리턴할 수 있지만 이것을 응용하면 여러 개의 값을 리턴받는 것과 같은 효과를 얻을 수 있다. (임시적으로 간단히 처리할 때는 별도의 클래스를 선언하는 것보다 이처럼 배열을 이용하는 것이 좋다)

**JVM 메모리 구조**<br>
![](/jvm.PNG)
1.메소드 영역(method area)

    - 프로그램 실행 중 어떤 클래스가 사용되면, JVM은 해당 클래스의 클래스파일(*.class)을 읽어서 분석하여 클래스에 대한 정보(클래스 데이터)를 이곳에 저장한다. 이때 그 클래스의 클래스변수도 이 영역에 함께 생성된다.

2.호출스택(call stack)

    - 호출스택은 메서드 작업에 필요한 메모리 공간을 제공한다. 메서드가 호출되면 호출스택에 메모리가 할당되며 이 메모리는 메서드가 작업을 수행하는 동안 지역변수(매개변수 포함) 연산의 중간결과 등을 저장하는데 사용된다. 그리고 메서드가 작업을 마치면 할당되었던 메모리공간은 반환되어 비워진다.

3.힙(heap)

    - 인스턴스가 생성되는 공간. 프로그램 실행 중 생성되는 인스턴스는 모두 이곳에 있다.

**초기화 블럭**<br>
클래스 초기화 블럭 - 클래스 변수의 복잡한 초기화에 사용된다.<br>
인스턴스 초기화 블럭 - 인스턴스 변수의 복잡한 초기화에 사용된다.<br>

생성자 보다 인스턴스 초기화 블럭이 먼저 수행된다. 클래스 초기화는 클래스가 처음 로딩될 때 클래스 변수들이 자동으로 메모리에 만들어지고, 바로 클래스 초기화 블럭에서 클래스 변수들을 초기화하게 되는 것이다.
```java
class InitBlack
{
    static
    {
        //클래스 초기화 블럭
    }

    {
        //인스턴스 초기화 블럭
    }
}

```
초기화 블럭 내에는 메서드 내에서와 같이 조건문, 반복문, 예외처리 구문 등을 자유롭게 사용할 수 있으므로, 초기화 작업이 복잡하여 명시적 초기화만으로는 부족한 경우 초기화 블럭을 사용한다.

### Chapter 7 객체지향 프로그래밍2
**final**<br>
생성자를 이용한 final 멤버변수 초기화<br>
final이 붙은 변수는 상수이므로 일반적으로 선언과 초기화를 동시에 하지만, 인스턴스 변수의 경우 생성자에서 초기화 되도록 할 수 있다.
```java
class Card{
    final int NUMBER;
    final String KIND;

    Card(int num, String kind){
        NUMBER = num;
        KIND = kind;
    }
}
```
이 기능을 활용하면 각 인스턴스마다 final이 붙은 멤버변수가 다른 값을 갖도록 하는것이 가능하다.

**생성자의 접근 제어자**<br>
생성자에 접근 제어자를 사용함으로써 인스턴스의 생성을 제한할 수 있다. 보통 생성자의 접근 제어자는 클래스의 접근 제어자와 같지만, 다르게 지정할 수도 있다.

생성자의 접근 제어자를 private으로 지정하면, 외부에서 생성자에 접근 할 수 없으므로 인스턴스를 생성할 수 없게 된다. 그래도 클래스 내부에서는 인스턴스의 생성이 가능하다.

```java
class Singleton{

    private static Singleton s = new Singleton();

    private Singleton(){
        //...
    }

    //인스턴스를 생성하지 않고도 호출할 수 있어야 하므로 static이어야 한다.
    public static Singleton getInstance(){
        return s;
    }
}
```
위의 경우처럼 생성자를 통해 직접 인스턴스를 생성하지 못하게 하고 public 메서드를 통해 인스턴스에 접근하게 함으로써 사용할 수 있는 인스턴스의 개수를 제한 할 수 있다.

또한 생성자가 private인 클래스는 다른 클래스의 조상이 될 수 없으므로 클래스 앞에 final을 더 추가하여 상속할 수 없는 클래스라는 것을 알리는 것이 좋다.

```java
public final class Math{
    private Math(){
        //...
    }
}
```

**정리**<br>
접근지정을 할 수 있는 곳은 클래스, 멤버변수, 메서드, 생성자 4곳이다.

**클래스**<br>
private, protected은 내부 클래스만 가능.<br>
클래스 접근 지정자가 default일 경우에는, 다른 패키지에서 객체 생성이 안된다.<br>
**멤버변수와 메서드**<br>
private : 같은 클래스만 접근 가능<br>
default : 같은 패키지만 접근 가능<br>
protected : 같은 패키지와 상속관계에서는 접근 가능<br>
public : 어디서나 접근 가능<br>
**생성자**<br>
private : 외부에서 객체 생성을 못함. 상속도 불가능.<br>
default : 같은 패키지면 객체생성, 상속 가능하지만 다른 패키지면 객체생성, 상속 못함<br>
protected : 같은 패키지면 객체생성, 상속 가능하지만 다른 패키지면 상속은 가능한데 객체생성은 못함<br>
public : 같은, 다른 패키지 객체생성, 상속 가능<br>

**참조변수와 인스턴스의 연결**<br>
조상 클래스에 선언된 멤버변수와 같은 이름의 인스턴스변수를 자손 클래스에 중복으로 정의했을 때,
1. 조상 타입의 참조변수를 사용했을 때는 조상 클래스에 선언된 멤버변수가 사용되고, 자손타입의 참조변수를 사용했을 때는 자손 클래스에 선언된 멤버변수가 사용된다.
2. 메서드의 경우 참조 변수의 타입에 관계없이 항상 실제 인스턴스의 메서드(오버라이딩된 메서드)가 호출된다.

```java
public static void main(String[] args){

    Parent p = new Child();
    Child c = new Child();

    System.out.println("p.x = "+p.x);
    p.method();

    System.out.println("c.x = "+c.x);
    c.method();
}

class Parent{
    int x = 100;

    void method(){
        System.out.println("Parent Method");
    }
}

class Child extends Parent{
    int x = 200;

    void method(){
        System.out.println("Child Method");
    }
}
```
실행결과
```
p.x = 100
Child Method
c.x = 200
Child Method
```
그러나 멤버변수들은 주로 private으로 접근을 제한하고, 외부에서는 메서드를 통해서만 멤버변수에 접근할 수 있도록 하지, 다른 외부 클래스에서 참조변수를 통해 직접적으로 인스턴스변수에 접근할 수 있게 하지 않는다.

**instanceof 연산자**<br>
instanceof를 이용한 연산결과로 실제 인스턴스와 같은 타입 뿐만 아니라 조상타입에도 true를 얻는다. 즉, 참조변수가 검사한 타입으로 형변환이 가능하다는 것을 뜻한다.

**인터페이스의 상속**<br>
인터페이스는 인터페이스로부터만 상속받을 수 있으며, 클래스와는 달리 다중상속, 즉 여러개의 인터페이스로부터 상속을 받는 것이 가능하다.

```java
interface Movable{
    void move(int x, int y);
}

interface Attackable{
    void attack(Unit u);
}

interface Fightable extends Movable, Attackable{

}

```
Fightable 자체에는 정의된 멤버가 하나도 없지만 조상 인터페이스로부터 상속받은 두개의 추상 메서드를 멤버로 갖게 된다.

**인터페이스의 이해**<br>
인터페이스의 규칙이나 활용이 아닌 본질적인 측면에 대해서 살펴보자.

- 클래스는 사용하는 쪽(User)과 클래스를 제공하는 쪽(Provider)이 있다.
- 메서드를 사용(호출)하는 쪽(User)에서는 사용하려는 메서드(Provider)의 선언부만 알면 된다.(내용은 몰라도 된다.)

```java
class A{
    public void methodA(B b){
        b.methodB();
    }
}

class B{
    public void methodB(){
        System.out.println("methodB()");
    }
}

class InterfaceTest{
    public static void main(String args[]){
        A a = new A();
        a.methodA(new B());
    }
}
```
위와 같이 클래스 A와 클래스 B가 있을 때 클래스 A(User)는 클래스  B(Provider)의 인스턴스를 생성하고 메서드를 호출한다. 이 두 클래스는 서로 직접적인 관계에 있다. 이것을 간단히 'A-B'라고 표현하자.

![](/interface1.PNG)

이경우 클래스 A를 작성하기 위해서는 클래스 B가 이미 작성되어 있어야 한다. 그리고 클래스 B의 methodB()의 선언부가 변경되면, 이를 사용하는 클래스 A도 변경되어야 한다. 즉 직접적인 관계의 두 클래스는 한 쪽(Provider)이 변경되면 다른 한 쪽(User)도 변경되어야 한다는 단점이 있다.

그러나 클래스 A가 클래스 B를 직접 호출하지 않고 인터페이스를 매개체로 해서 클래스 A가 인터페이스를 통해서 클래스 B의 메서드에 접근하도록 하면, 클래스 B에 변경사항이 생기거나 클래스 B와 같은 기능의 다른 클래스로 대체 되어도 클래스 A는 전혀 영향을 받지 않도록 하는 것이 가능하다.

두 클래스간의 관계를 직접적으로 변경하기 위해서는 먼저 인터페이스를 이용해서 클래스 B(Provider)의 선언과 구현을 분리 해야한다.

먼저 다음과 같이 클래스 B에 정의된 메서드를 추상메서드로 정의하는 인터페이스 I를 정의한다.
```java
interface I{
    public abstract void methodB();
}
```
그 다음에는 클래스 B가 인터페이스 I를 구현하도록 한다.
```java
class B implements I{
    public void methodB(){
        System.out.println("methodB in B class");
    }
}
```
이제 클래스 A는 클래스 B 대신 인터페이스 I를 사용해서 작성할 수 있다.
```java
class A{
    public void methodA(I i){
        i.methodB();
    }
}
```

클래스 A를 작성하는데 있어서 클래스 B가 사용되지 않았다는 점에 주목하자. 이제 클래스 A와 클래스 B는 'A-B'의 직접적인 관계에서 'A-I-B'의 간접적인 관계로 바뀐 것이다.

![](/interface2.PNG)

결국 클래스 A는 여전히 클래스 B의 메서드를 호출하지만, 클래스 A는 인터페이스 I하고만 직접적인 관계에 있기 때문에 클래스 B의 변경에 영향을 받지 않는다. 클래스 A는 인터페이스를 통해 실제로 사용하는 클래스의 이름을 몰라도 되고 심지어는 실제로 구현된 클래스가 존재하지 않아도 문제되지 않는다. 클래스 A는 오직 직접적인 관계에 있는 인터페이스 I의 영향만 받는다.

![](/interface3.PNG)


인터페이스 I는 실제구현 내용(클래스 B)을 감싸고 있는 껍데기이며, 클래스 A는 껍데기 안에 어떤 알맹이(클래스)가 들어 있는지 몰라도 된다.

### Chapter 8 예외처리

예외 클래스들은 다음과 같이 두 개의 그룹으로 나눠질 수 있다. (모든 예외의 최고 조상은 Exception 클래스이다.)
- RuntimeException 클래스와 그 자손클래스들 (프로그래머의 실수로 발생하는 예외)
    - ArithmeticException
    - ClassCastException
    - NullPointerException
    - IndexOutOfBoundsException
- Exception 클래스와 그 자손 클래스들 (사용자의 실수와 같은 외적인 요인에 의해 발생하는 예외)
    - IOException
    - ClassNotFoundException
    - FileNotFoundException
    - DataFormatException

RuntimeException 클래스들 그룹에 속하는 예외가 발생할 가능성이 있는 코드에는 예외처리를 해주지 않아도 컴파일 시에 문제가 되지 않지만, Exception 클래스들 그룹에 속하는 예외가 발생할 가능성이 있는 예외는 반드시 처리를 해주어야 하며, 그렇지 않으면 컴파일 시에 에러가 발생한다.

**System.err**<br>
System.err는 setErr 메서드를 이용해서 출력방향을 바꾸지 않는 한 err에 출력하는 내용은 모두 화면에 나타나게 된다.
```java
PrintStream ps =null;
FileOutputStream fos = null;

try{
    //error.log파일에 출력할 준비를 한다.
    fos = new FileOutputStream("error.log",true);
    ps = new PrintStream(fos);
    //error의 출력을 화면이 아닌, error.log 파일로 변경한다.
    System.setErr(ps);
}
```
출력 방향이 변경되었기 때문에 `System.err.println("")`을 이용해서 출력하는 내용은 error.log파일에 저장된다.

**예외 되던지기(exception re-throwing)**<br>
단 하나의 예외에 대해서 예외가 발생한 메서드와 호출한 메서드 양쪽에서 처리할 수 있다.
```java
class ExceptionEx23{
    public static void main(String[] args){
        try{
            method1();
        }catch(Exception e){
            System.out.println("main메서드에서 예외가 처리되었습니다.");
        }
    }

    static void method1() throws Exception{
        try{
            throw new Exception();
        }catch(Exception e){
            System.out.println("method1메서드에서 예외가 처리되었습니다.");
            throw e; // 다시 예외를 발생시킨다.
        }
    }
}
```
method1()의 catch블럭에서 예외를 처리하고도 throw문을 통해 다시 예외를 발생시켰다. 그리고 이 예외를 method1을 호출한 main 메서드에서 한번 더 처리하였다.

### Chapter 9 java.lang 패키지
**String 클래스**
String클래스에는 문자열을 저장하기 위해서 문자형 배열 변수(char[]) value를 인스턴스 변수로 정의해놓고 있다.

한번 생성된 String 인스턴스가 갖고 있는 문자열은 읽어 올 수만 있고, 변경할 수는 없다.<br>
예를 들어 ‘+’연산자를 이용해서 문자열을 결합하는 경우 인스턴스내의 문자열이 바뀌는 것이 아니라 새로운 문자열이 담긴 String인스턴스가 생성되는 것이다. 이처럼 덧셈연산자(+)를 사용해서 문자열을 결합하는 것은 매 연산 시마다 새로운 문자열을 가진 String인스턴스가 생성되어 메모리공간을 차지하게 되므로 가능한 한 결합횟수를 줄이는 것이 좋다. 그래서 문자열간의 결합이나 추출 등 문자열을 다루는 작업이 많이 필요한 경우에는 String클래스 대신 StringBuffer클래스를 사용하는 것이 좋다.  String인스턴스와는 달리 StringBuffer인스턴스에 저장된 문자열은 변경이 가능하므로 하나의 StringBuffer인스턴스만으로도 문자열을 다루는 것이 가능하다.

cf) StringBuilder가 JDK1.5부터 새롭게 추가되었다. StringBuffer와 완전히 동일한 클래스이다. 다만 동기화 처리를 하지 않기 때문에 멀티스레드 프로그래밍에서는 사용하면 안되지만 그 외의 경우에는 StringBuffer보다 빠른 성능을 보장한다.

문자열을 만들 때 두 가지 방법, 문자열 리터럴을 지정하는 방법과 String클래스의 생성자를 사용해서 만드는 방법이 있다.
```java
String str1 = “str1”;
String str2 = “str1”;
String str3 = new String(“str1”);
```
equals(String s)를 사용했을 때는 문자열간의 내용으로 비교하기 때문에 str1,str2,str3 비교했을 때 모두 true로 나온다. 하지만 각 String인스턴스의 주소값을 등가비교연산자(==)로 비교했을 때는 결과가 다르다. 리터럴로 문자열을 생성했을 경우, 같은 내용의 문자열들은 모두 하나의 String인스턴스를 참조하도록 되어 있다. 그러나 String클래스의 생성자를 이용한 String인스턴스의 경우에는 new연산자에 의해서 메모리할당이 이루어지기 때문에 항상 새로운 String인스턴스가 생성된다.

일반적으로 문자열들을 비교하기 위해서 equals메서드를 사용하지만, equals메서드로 문자열의 내용을 비교하는 것보다는 등가비교연산자(==)를 이용해서 주소(4byte)를 비교하는 것이 더 빠르다. 그래서 비교해야할 문자열의 개수가 많은 경우에는 보다 빠른 문자열 검색을 위해서 intern메서드와 등가비교연산자(==)를 사용하기도 한다.

String클래스의  intern()은 String인스턴스의 문자열을 ‘constant pool’에 등록하는 일을 한다. 등록하고자 하는 문자열이 ‘constant pool’에 이미 존재하는 경우에는 그 문자열의 주소값을 반환한다.

**String클래스의 생성자와 메서드**<br>
```java
String file = “Hello.txt”;
boolean b = file.endsWith(“txt”);  //true
```
```java
String s = “java.lang.Object”;
boolean b = s.startsWith(“java”); // true
boolean b2 = s.startWith(“lang”);// false
```

```java
boolean equalsIgnoreCase(String str)
//대소문자 구분없이 비교한다.
```

```java
String[] split(String regex)
```

기본형 -> 문자열
```java
String a = String.valueof(‘a’);
String b = String.valueof(100);
String c = String.valueof(true);
```

문자열 -> 기본형
```java
boolean Boolean.getBoolean(String s)
byte Byte.parseByte(String s)
short Short.parseShort(String s)
int Integer.parseInt(String s)
long Long.parseLong(String s)
float Float.parseFloat(String s)
double Double.parseDouble(String s)
```
### Chapter 10 내부 클래스
내부 클래스는 마치 변수를 선언하는 것과 같은 위치에 선언할 수 있으며, 변수의 선언위치에 따라 인스턴스변수, 클래스변수, 지역변수로 구분되는 것과 같이 내부 클래스도 선언위치에 따라 인스턴스 클래스, 스태틱 클래스, 지역 클래스로 나뉜다.
```java
class Outer{
	class InstanceInner{}
	static class StaticInner {}

	void myMethod(){
		class LocalInner{}
	}
}
```

### Chapter 11 컬렉션 프레임워크와 유용한 클래스
![](/collection.PNG)
List : 순서가 있는 데이터의 집합. 데이터의 중복을 허용한다.(ArrayList, LinkedList, Stack, Vector 등)<br>
Set : 순서를 유지하지 않는 데이터의 집합. 데이터의 중복을 허용하지 않는다.(HashSet, TreeSet 등)<br>
Map : 키와 값의 쌍으로 이루어진 데이터의 집합. 순서는 유지되지 않으며, 키는 중복을 허용하지 않고, 값은 중복을 허용한다. (HashMap, TreeMap, Hashtable, Properties 등)<br>

**Map.Entry 인터페이스**<br>
Map.Entry 인터페이스는 Map 인터페이스의 내부 인터페이스이다. 내부 클래스와 같이 인터페이스도 이처럼 인터페이스 안에 인터페이스를 정의할 수있다.
```java
public interface Map{
	…
	interface Entry{
		Object getKey();
		Object getValue();
		...
	}
}
```

cf) Map 인터페이스 메서드중 Set entrySet()이 있다. Map에 저장되어 있는 key-value쌍을 Map.Entry타입의 객체로 저장한 Set으로 반환한다.

**동기화**<br>
멀티스레드 프로그래밍에서는 하나의 객체를 여러 쓰레드가 동시에 접근할 수 있기 때문에 데이터의 일관성을 유지하기 위해서는 동기화가 필요하다.
Vector와 Hashtable과 같은 구버전(JDK1.2 이전)의 클래스들은 자체적으로 동기화 처리가 되어 있는데, 멀티스레드 프로그래밍이 아닌 경우에는 불필요한 기능이 되어 성능을 떨어트리는 요인이 된다.
그래서 새로 추가된 ArrayList와 HashMap과 같은 컬렉션은 동기화를 자체적으로 처리하지 않고 필요한 경우에만 java.util.Collections 클래스의 동기화 메서드를 이용해서 동기화처리가 가능하도록 변경하였다.
이들을 사용하는 방법은 다음과 같다.
```java
List list = Collections.synchronizedList(new ArrayList(…));
```

**ArrayList**
```java
List list = new ArrayList(10);
```
ArrayList를 생성 할 때, 저장할 요소의 갯수를 고려해 초기화 시키는게 좋다. 생성할 때 지정한 크기보다 더 많은 객체를 저장하면 자동적으로 크기가 늘어나기는 하지만 이 과정에서 처리시간이 많이 소요된다.
다시 말해 배열은 크기를 변경할 수 없으므로 새로운 배열을 생성해 데이터를 복사해야하기 때문에 효율이 많이 떨어진다.

ArrayList remove를 통해 객체를 삭제할때는
```java
for(i = list2.size2()-1; i>=0; i--){
    if(list1.contains(list2.get(i)))
        list2.remove(i);
}
```
위와 같이 끝에서부터 인덱스를 감소시키면서 삭제를 진행한다. 그 이유는 배열의 한 요소가 삭제될때마다 나머지 요소들이 한칸씩 앞으로 자리이동을 해야되는데 뒤에서부터하면 이동을 최소화 시킬수 있다.

실제 ArrayList remove 메서드 일부
```java
private void fastRemove(int index){
    modCount++;
    int numMoved = size - index -1;
    if(numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null;
}
```
내가 생성한 객체 size가 5이고 지우려고하는 인덱스가 마지막이라면(4) numMoved는 0이되므로 System.arraycopy를 호출하지 않아 작업시간이 짧아진다. 배열의 중간에 위치한 객체를 추가하거나 삭제하는 경우 System.arraycopy()를 호출해서 다른 데이터의 위치를 이동시켜 줘야하기 때문에 다루는 데이터의 개수가 많을수록 작업시간이 오래걸린다.

**LinkedList**<br>
실제로 LinkedList 클래스는 이름과 달리 더블 원형 링크드리스트로 구현되어있다.

순차적으로 추가/삭제하는 경우에는 ArrayList가 LinkedList보다 빠르다. 중간 데이터를 추가/삭제 하는 경우에는 LinkedListk가 ArrayList보다 빠르다. 두 클래스의 장점을 이용해서 두 클래스를 혼합해서 사용하는 방법도 있다.
```java
ArrayList al = new ArrayList(1000000);
for(int i=0; i<100000;i++)
    al.add(i+"");

LinkedList ll = new LinkedList(al);
for(int i=0; i<1000; i++)
    ll.add(500, "X");
```
컬렉션 프레임워크에 속한 대부분의 컬렉션 클래스들은 이처럼 서로 변환이 가능한 생성자를 제공한다.

**Stack & Queue**<br>
순차적으로 데이터를 추가하고 삭제하는 스택에는 ArrayList와 같은 배열기반의 컬렉션 클래스가 적합하지만, 큐는 데이터를 꺼낼 때 항상 첫번째 저장된 데이터를 삭제하므로, ArrayList와 같은 배열기반의 컬렉션 클래스를 사용한다면 데이터를 꺼낼 때마다 빈 공간을 채우기 위해 데이터의 복사가 발생하므로 비효율적이다. 그래서 큐는 ArrayList보다 데이터의 추가/삭제가 쉬운 LinkedList로 구현하는 것이 더 적합하다.

**HashMap**<br>
```java
HashMap map = new HashMap();
map.put("김자바", new Integer(90));
map.put("김자바", new Integer(100));
map.put("이자바", new Integer(100));
map.put("강자바", new Integer(80));
map.put("안자바", new Integer(90));
```
Map에서 키가 중복되면 기존의 값을 덮어쓴다.

```java
Set set = map.entrySet();
Iterator it = set.iterator();

while(it.hasNext()){
    Map.Entry e = (Map.Entry)it.next();
    System.out.println(e.getKey() + e.getValue());
}
```
Map은 Iterator가 없기 때문에 entrySet() 메서드를 통해 Set에 key와 value 결합한 형태로 저장시켜. iterator를 통해 하나씩 읽어온다. 그리고  Map의 내부 인터페이스인 Map.Entry를 통해 key와 value를 얻어온다.

cf) entrySet()을 이용해서 key와 value를 함께 읽어 올수도 있고 keySet()이나 values()를 이용해서 키와 값을 따로 읽어 올 수 있다. (key는 중복을 허용하지 않으니까 Set타입으로 반환하고 value는 중복을 허용하니까 Collection 타입으로 반환한다.)
```java
    Set set = map.entrySet();
    Iterator iterator = set.iterator();
    while(iterator.hasNext()){
        Map.Entry e = (Map.Entry)iterator.next();
        System.out.println(e.getKey() + " : " + e.getValue());
    }

    Set set = map.keySet();
    Iterator iterator = set.iterator();
    while(iterator.hasNext()){
        System.out.println(iterator.next());
    }

    Collection values = map.values();
    Iterator iterator = values.iterator();
    while(iterator.hasNext()){
        System.out.println(iterator.next());
    }
```

**HashSet**<br>
HashSet은 내부적으로 HashMap을 이용해서 만들어졌으며, HashSet이란 이름은 해싱을 이용해서 구현했기 때문에 붙여진 것이다.

저장한 순서를 유지하고자 한다면 LinkedHashSet을 사용해야한다.

String클래스는 문자열의 내용으로 해시코드를 만들어 내기 때문에 내용이 같은 문자열에 대한 hashCode()호출은 항상 동일한 해시코드를 반환한다. 반면에 Object클래스는 객체의 주소로 해시코드를 만들어 내기 때문에 실행 할 때마다 해시코드 값이 달라질 수 있다.

**Comparator와 Comparable**<br>
Comparable - 기본 정렬기준을 구현하는데 사용<br>
Comparator - 기본 정렬기준 외에 다른 기준으로 정렬하고자 할 때 사용<br>

둘다 인터페이스로 Comparable을 구현하고 있는 클래스들은 같은 타입의 인스턴스끼리 서로 비교할 수 있다. 그래서 Comparable을 구현한 클래스는 정렬이 가능하다는 것을 의미한다.

**해싱**<br>
해싱이란 해시함수를 이용해서 데이터를 해시테이블에 저장하고 검색하는 기법을 말한다. 해시함수는 데이터가 저장되어 있는 곳을 알려 주기 때문에 다량의 데이터 중에서도 원하는 데이터를 빠르게 찾을 수 있다.

**제네릭**<br>
```java
List myIntList = new LinkedList();
myIntList.add(new Integer(0));
Integer i = (Integer)myIntList.iterator().next();
```
리스트에 어떤 타입이 들어갈지 알고 있지만 캐스팅은 필수다. 컴파일러는 iterator에 의해 Object가 리턴될것이라는 것까지 밖에 보장못한다. 그래서 리스트에 특정한 타입만 들어갈 수 있도록 강제해서 캐스팅도 안해도 되는 제네릭 !!
```java
List<Integer> myIntList = new LinkedList<>();
```
myIntList 변수에 타입에 대한 정의를 해서 인자의 정확성을 컴파일 타임에 알 수 있게 되었다.
```java
List<String> ls = new ArrayList<String>(); // 1
List<Object> lo = ls; // 2
lo.add(new Object()); // 3
```
lo에 String 리스트를 대입하고 새로운 Object를 추가시켰다. 그러니 lo에는 이제 String만 있는게 아니니까 이런 문제를 방지하기 위해 컴파일러는 2에서 에러를 발생시킨다.

컬렉션에 저장될 객체에도 다형성이 필요하다 => 와일드 카드 '?'
```java
public static void printAll(ArrayList<? extends Product> list, ArrayList<? extends Product> list2){
...
}

//타입을 별도로 선언함으로써 코드 간략화
public static <T extends Product> void printAll(ArrayList<T> list, ArrayList<T> list2){
...
}
```
T라는 타입이 Product의 자손 타입이라는 것을 미리 정의해놓고 사용.

cf) Product가 인터페이스임에도 키워드를 extends로 사용함에 주의해라.

`<? extends T>` - T또는 T의 자손타입을 의미
`<? super T>` - T또는 T의 조상타입을 의미

Collections.sort()에 적용된 제네릭
`public static <T extends Comparable<? super T>> void sort(List<T> list)`

1. List 인터페이스를 구현한 컬렉션을 매개변수의 타입으로 정의하고 있다. 그리고 그 컬렉션에는 'T'라는 타입의 객체를 저장하도록 선언되어 있다.
2. 'T'는 Comparable 인터페이스를 구현한 클래스의 타입이어야 하고, 그 Comparable 인터페이스는 'T'또는 그 조상타입을 비교하는 Comparable이어야 한다.

Comparable과 Comparator에 대해서 정확히 알자

**Comparable 인터페이스 :** 자기 자신과 같은 클래스 타입의 다른 객체와 비교하기 위해 사용
```java
public interface Comparable<T>{
	public int compareTo(T o);
}
```
객체 비교를 위해서 compareTo 메서드를 오버라이딩해서 비교한다.

현재의 객체가 다른 객체와 비교하여 최종 반환되는 값이 0이면 순서가 같은것, 음수면 순서가 앞에 있는것, 양수면 순서가 뒤에 있는것.

**Comparator 인터페이스 :** 다른 두개의 객체를 비교, 즉 Comparator를 구현한 객체가 다른 두개의 객체를 비교하는 기준이 되어 두개의 객체를 비교한다.
```java
public interface Comparator<T>{
	int compare(T o1, T o2); // o1이 비교하는 주체자, o2가 비교대상자
}
```
### Chapter 12 스레드
모든 스레드는 독립적인 작업을 수행하기 위해 자신만의 호출스택을 필요로 하기 때문에, 새로운 스레드를 생성하고 실행시킬 때마다 새로운 호출스택이 생성되고 스레드가 종료되면 작업에 사용된 호출스택은 소멸한다.

**스레드 그룹**<br>
모든 스레드는 반드시 스레드 그룹에 포함되어 있어야 하기 때문에, 스레드 그룹을 지정하는 생성자를 사용하지 않은 스레드는 기본적으로 자신을 생성한 스레드와 같은 스레드 그룹에 속하게 된다.

자바 애플리케이션이 실행되면, JVM은 main과 system이라는 스레드 그룹을 만들고 JVM운영에 필요한 스레드들을 생성해서 이 스레드 그룹에 포함시킨다. 예를 들어 main 메서드를 수행하는 main이라는 이름의 스레드는 main스레드 그룹에 속하고, 가비지컬렉션을 수행하는 Finalizer스레드는 system스레드 그룹에 속한다. 우리가 생성하는 모든 스레드 그룹은 main스레드 그룹의 하위 스레드 그룹이 되며, 스레드 그룹을 지정하지 않고 생성한 스레드는 자동적으로 main스레드 그룹에 속하게 된다.

**데몬스레드**<br>
다른 일반 스레드의 작업을 돕는 보조적인 역할을 수행하는 스레드이다. 데몬 스레드는 무한루프와 조건문을 이용해서 실행 후 대기하고 있다가 특정 조건이 만족되면 작업을 수행하고 다시 대기하도록 작성한다. 데몬 스레드의 예로는 가비지 컬렉터, 워드프로세서의 자동저장, 화면자동갱신 등이 있다.

데몬스레드는 일반 스레드의 작성방법과 실행방법이 같으며 다만 스레드를 생성한 다음 실행하기 전에 setDaemon(true)를 호출하기만 하면 된다. 그리고 데몬 스레드가 생성한 스레드는 자동적으로 데몬 스레드가 된다.

```java
Thread t = new Thread(new ThreadEx8());

t.setDaemon(true);
t.start();
```
setDaemon 메서드는 반드시 start()를 호출하기 전에 실행되어야 한다.

**스레드 실행제어 / 동기화**<br>
suspend(), resume(), stop()은 스레드를 교착상태에 빠뜨릴 가능성이 있기 때문에 deprecated되었으므로 사용하지 않는 것이 좋다.(suspend() 대신 wait()을, resume() 대신 notify()를 사용한다.)

wait()과 notify()는 동기화 블록 내에서만 사용할 수 있으며 notify()는 객체의 waiting pool에 있는 스레드 중의 하나만 깨우고 notifyAll()은 모든 스레드를 깨운다. 어차피 한 번에 하나의 스레드만 객체를 사용할 수 있기 때문에 notify()를 사용하나 notifyAll()을 사용하나 별차이는 없다. 그러나 notify()에 의해 어떤 스레드가 깨워지게 될지는 알수 없기 때문에 다시 객체의 waiting pool에 들어가더라도 notifyAll()을 이용해서 모든 스레드를 깨워놓고 JVM의 스레드 스케줄링에 의해서 처리되도록 하는 것이 안전하다.

### Chapter 14 입출력(I/O)
직렬화 : 객체를 데이터 스트림으로 만드는 것. 다시 얘기하면 객체에 저장된 데이터를 스트림에 쓰기위해 연속적인 데이터로 변환하는 것.
역직렬화 : 스트림으로부터 데이터를 읽어서 객체를 만드는것

객체는 클래스에 정의된 인스턴스변수의 집합이다. 객체에는 클래스변수나 메서드가 포함되지 않는다. 객체는 오직 인스턴스변수들로만 구성되어 있다. 인스턴스변수는 인스턴스마다 다른 값을 가질 수 있어야하기 때문에 별도의 메모리공간이 필요하지만 메서드는 변하는 것이 아니라서 메모리를 낭비해 가면서 인스턴스마다 같은 내용의 코드를 포함시킬 이유는 없다.

직렬화(객체 --> 스트림) : 객체를 스트림에 출력 ObjectOutputStream
역직렬화(스트림 --> 객체) : 스트림으로부터 객체에 입력 ObjectInputStream

**직렬화가 가능한 클래스 만들기**<br>
직렬화하고자 하는 클래스가 java.io.Serializable인터페이스를 구현하도록 하면 된다.
```java
public class UserInfo implements java.io.Serializable{
    String name;
    String password;
    int age;
}
```

아래는 직렬화할 수 없는 클래스의 객체를 인스턴스변수가 참조하고 있어서 직렬화에 실패한다. 모든 클래스의 최고조상인 Object는 Serializable을 구현하지 않았기 때문에 직렬화할 수 없다.
```java
public class UserInfo implements java.io.Serializable{
    String name;
    String password;
    int age;

    Object obj = new Object();
}
```

아래와 같은 경우는 직렬화가 가능하다. 인스턴스변수 obj의 타입이 직렬화가 안되는 Object이긴 하지만 실제로 저장된 객체는 직렬화가 가능한 String인스턴스이기 때문에 직렬화가 가능하다. 인스턴스변수의 타입이 아닌 실제로 연결된 객체의 종류에 의해서 결정된다.
```java
public class UserInfo implements java.io.Serializable{
    String name;
    String password;
    int age;

    Object obj = new String("abc");
}
```

제어자 transient를 붙여서 직렬화 대상에서 제외되도록 할 수 있다. 또한 password와 같이 보안상 직렬화되면 안되는 값에 대해 transient를 사용할 수 있다.
```java
public class UserInfo implements java.io.Serializable{
    String name;
    transient String password;
    int age;

    transient Object obj = new Object();
}
```


---
# 추가적인 내용
**JDK7에서 새롭게 소개된 Invokedynamic**<br>
람다는 내부적으로 Invokedynamic을 활용한다. 자바는 static type 언어라고 불리며, 이는 컴파일 타임에서 이미 멤버 변수들이나 함수 변수들의 타입이 반드시 명시적으로 지정돼야 함을 의미한다. 그에 반해 루비나 자바스크립트는 이른바 ‘duck-typing’이라고 하는 타입 시스템을 사용함으로써 컴파일 타임에서의 타입을 강제하지 않는다. Invokedynamic은 이러한 duck-typing을 JVM레벨에서 기본적으로 지원하면서 자바 외에 다른 언어들이 JVM이라는 플랫폼 위에서 최적화된 방식으로 실행될 수 있는 토대를 제공한다.

초창기 JVM은 항상 바이트코드를 해석했다. 따라서 루프문과 같이 코드 실행 과정에서 반복이 많은 경우 불필요한 해석 시간이 늘어나므로 C와 같은 프로그래밍 언어를 사용해 개발한 애플리케이션과의 성능 격차가 컸다. 이런 성능 문제를 해결하기 위해 바이트코드를 기계어 코드로 컴파일하는 방법을 사용해 실행 시간을 대폭적으로 높이는 JIT 컴파일이라 알려진 기술이 등장했다. 초기의 JIT 컴파일러는 모든 바이트코드를 한번에 기계어로 변환했기 때문에 컴파일 비용이 높았다. 하지만 연이어 등장한 핫스팟은 성능 향상을 위한 동적 변환 기능을 탑재해 자주 실행되는 코드나 반복되는 코드를 분석해 핫 스팟을 찾아낸다. 그리고 해당 부분만 기계어로 변환해 컴파일 비용을 줄이고 동적으로 상황에 맞춰 최적화를 할 수 있게 됐다. 즉, 성능이 덜 중요한 코드의 경우에는 변환에 따른 비용을 줄인다는 의미다.

**자바9**<br>
자바9의 변화 중에서도 가장 손꼽을 만한 변화가 바로 프로젝트 직소다. 단어 그대로 직소 퍼즐처럼 자바의 내부 API들을 모듈화해서 관리한다는 의미다. 이러한 모듈은 jar파일이 아닌 새롭게 소개된 jimage 포맷 안에 담긴다. SE9의 기본  API들은 bootmodules.jimage, extmodules.jimage, appmodules.jimage 등의 이미지로 배포된다. jimage 파일 포맷은 jar 포맷을 장기적으로 대체하기 위한 기술이다. 대략 클래스를 읽는 속도만 5배 향상됐다. 용량으로 비교하면 zip보다 15~16%더 작다고 한다. 내부적으로 해시 충돌을 막기 위한 새로운 해시함수를 사용한다.

**자바 환경변수**<br>
JDK에서 제공하는 명령어들을 환경 변수에 지정하면 시스템의 모든 곳에서 JDK 명령어를 쉽게 실행할 수 있다.

**JAVA_HOME :** JDK가 설치된 홈 디렉토리를 설정하기 위한 환경 변수다. `C:\Program Files\Java\jdk1.7.0_75`<br>
**PATH :** OS에서 명령어를 실행할 때 명령어를 찾아야 하는 폴더의 순위를 설정하는 환경 변수다. 편리한 개발을 위해 Path 환경 변수에 JDK 개발 툴의 경로를 등록한다.<br>
`;%JAVA_HOME%\bin`(bin 디렉토리에는 JDK에서 제공하는 개발 툴 명령어들이 있다.)<br>
**CLASSPATH :** JVM이 시작될 때 JVM의 클래스 로더는 이 환경 변수를 호출한다. 그래서 환경 변수에 설정되어 있는 디렉토리가 호출되면 그 디렉토리에 있는 클래스들을 먼저 JVM에 로드한다. 그러므로 CLASSPATH 환경 변수에는 필수 클래스들이 위치한 디렉토리를 등록하도록 한다.`%JAVA_HOME%\lib\`<br>

### 클래스 로더
JVM에서 클래스가 동작하기 위해서는 클래스 로더가 클래스를 JVM에 올려놓는 과정이 반드시 필요하다. 클래스 로더에 의해서 로딩된 클래스만이 실행 가능한 상태이기 때문이다. 물론 CLASSPATH 경로에 위치한 클래스를 자동으로 로딩할 수 있지만, 개발자가 클래스를 직접 로딩할 수도 있다.

**자바는 동적으로 클래스를 읽어온다**<br>
실행 시 모든 클래스 파일들이 한 번에 JVM 메모리에 로딩되지 않고 요청되는 순간 로딩된다. 자바의 클래스 로더가 이런 역할을 수행한다. 클래스 로더란 '.class' 바이트 코드를 읽어 들여 class 객체를 생성하는 역할을 담당한다.

클래스 로더에 의해서 분석된 클래스의 정보를 개발자가 실행하거나 참조할 수 있다. 이와 같이 클래스의 내부 정보를 보는 기법을 리플렉션이라고 한다.
### 리플렉션
리플렉션의 핵심은 클래스 정보 확인이 아니라 동적인 프로그래밍이다. 리플렉션 기법이 사용된 가장 대표적인 예로는 이클립스 개발 도구의 자동 완성 기능이다.

지금까지 배운 내용에 의하면 객체를 생성하기 위해서는 클래스를 프로그래밍하는 시점에서 new 키워드와 함께 생성자를 호출해야 한다. 즉 클래스를 인스턴스하기 전에 사람이 직접 클래스를 코딩해서 컴파일 시점에서만 객체를 생성할 수 있음을 의미한다. 하지만 **리플렉션은 런타임 시점에서 동적으로 클래스의 정보를 추출하고 실행하는 환경을 제공해줄 수 있다.**

`java.lang.Class`클래스를 통해서 클래스들의 정보를 얻을 수 있다. 보통의 클래스와 달리 Class 클래스에서는 Class 객체를 생성하는 생성자가 없다. 그러므로 new 키워드를 사용해서 객체를 생성할 수 없고, 정보를 얻고자 하는 대상으로부터 Class 객체를 생성해야 한다. 리플렉션의 단어 뜻처럼, 거울로 비춰보기 위한 원래 대상 객체가 필요하다.

**Class 객체 생성하는 3가지 방법**<br>
```java
    //1. 대상 객체에서 제공하는 getClass() 메서드 이용하는 방법
    String str = new String();
    Class class1 = str.getClass();

    //2. 대상 클래스를 이용하여 Class 객체 받아오는 방법
    Class class2 = String.class;

    //3. 대상 클래스의 이름을 이용하여 Class 객체 받아오는 방법
    try{
        Class class3 = Class.forName("java.lang.String"); //반드시 패키지 경로 포함
    }cathch(ClassNotFoundException e){}
```
1번 2번이 명시적인 방법이라면 3번은 forName() 메서드를 동적으로 처리할 수 있다는 장점이 있다.

다음과 같은 리플렉션 기법을 통해 클래스의 생성자 정보, 메서드정보, 필드 정보를 얻을 수 있다.

```java
	Bong bong = new Bong();
	Class class1 = bong.getClass();

	//생성자 정보 얻기
	//Constructor[] constructors = class1.getDeclaredConstructors(); //명시적으로 선언된 생성자를 받는 메서드
	Constructor[] constructors = class1.getConstructors();
	for(int i=0; constructors != null && i< constructors.length; i++){
		System.out.println(constructors[i].toString());
	}

	//메서드 정보 얻기
	//Method[] methods = class1.getDeclaredMethods();
	Method[] methods = class1.getMethods(); //상속 받은 메서드도 알 수 있음.
	for(int i=0; methods != null && i<methods.length; i++){
		System.out.println(methods[i].toString());
	}

	//필드 정보 얻기
	Field[] fields = class1.getDeclaredFields();
	for(int i=0; fields != null && i< fields.length;i++){
		System.out.println(fields[i].toString());
	}

```

**1. 생성자를 의미하는 Constructor 클래스**<br>
```java
    Class<Bong> clazz = Bong.class;
	try {
	    //String형 매개변수와 int형 매개변수를 받는 생성자를 의미하는 Constructor객체 받아온다.
		Constructor<Bong> paramCons = clazz.getConstructor(String.class, Integer.TYPE);

		//newInstacne() 메서드에 매개변수로 넣기 위한 배열
		Object[] params = new Object[]{
				new String("Lodoss"), Bong.NUMBER
			};

		try {
			Bong bongVo = paramCons.newInstance(params);
			System.out.println(bongVo.toString());

		} catch (InstantiationException | IllegalAccessException
				| IllegalArgumentException | InvocationTargetException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	} catch (NoSuchMethodException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (SecurityException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
```
직접 new 키워드를 사용해서 객체를 생성하지 않고 리플렉션 기법으로 동적으로 객체를 생성했다.

**2. 특정 메서드를 가리키는 Method 클래스**<br>
```java
WorkerValue benVo = new WorkerValue("Benjamin Kim", WorkerValue.POSITION_MANAGER);
WorkerValue hamVo = new WorkerValue("Hamzani", WorkerValue.POSITION_MANAGER);

Class<WorkerValue> clazz1 = WorkerValue.class;

try{
    Method m1 = clazz1.getMethod("getName", new Class[] {});

    Object rt1 = m1.invoke(benVo, new Obejct[] {});
    Object rt1 = m1.invoke(hamVo, new Obejct[] {});
}
```
invoke 메서드가 두번 호출 됐는데 첫번째는 benVo 객체의 getName() 메서드가 실행되어 그 값이 Object rt1에 대입된다. 두번째는 hamVo 객체의 getName() 메서드가 실행된다.

**3. 클래스 변수를 의미하는 Field 클래스**<br>
```java
WorkerValue benVo = new WorkerValue("Benjamin Kim", WorkerValue.POSITION_MANAGER);
Class<WorkerValue> clazz1 = WorkerValue.class;

try{
    Field f = clazz.getDeclaredField("position"); //position은 필드명
    f.setAccessible(true);
    Object vlu = f.get(benVo);
}
```
필드는 보통 private 제어자로 외부에서 접근할 수 없다. 그래서 getDeclared* 로 시작하는 메서드를 사용해야 해당 자원에 대한 정보를 획득할 수 있다.

자바 수정자(제어자) :<br>
접근 제어자 : public protected default, private<br>
그외 : static, final, synchronized, transient, volatile, native abstract 등등<br>

transient는 직렬화 가능한 객체에서 멤버변수 중 직렬화 대상에 포함시키지 않을 때, 기본값으로 직렬화 되게 한다.(객체 직렬화시에 데이터의 보존여부를 결정.  멤버변수만 사용 가능하다.) ex) password

**volatile과 synchronized**<br>
http://kwanseob.blogspot.kr/2012/08/java-volatile.html

조금 더 엄밀히 정의를 하자면, (1) 특정 최적화에 주의해라, (2) 멀티 쓰레드 환경에서 주의해라, 정도의 의미를 준다고 보면 된다.

Java에서는 어떤 의미를 가질까? volatile을 사용한 것과 하지 않은것의 차이는 뭘까?<br>
volatile을 사용하지 않은 변수: 마구 최적화가 될 수 있다. 재배치(reordering)이 될 수있고, 실행중 값이 캐쉬에 있을 수 있다.<br>
volatile을 사용한 변수 (1.5미만): 그 변수 자체에 대해서는 최신의 값이 읽히거나 쓰여진다.<br>
volatile을 사용한 변수 (1.5이상): 변수 접근까지에 대해 모든 변수들의 상황이 업데이트 되고, 변수가 업데이트된다.<br>
synchronziation을 사용한 연산: synch블락 전까지의 모든 연산이 업데이트 되고, synch안의 연산이 업데이트된다.<br>