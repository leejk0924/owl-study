

# 아이템1. 생성자 대신 정적 팩터리 메서드를 고려하라
## ❓ 정적 팩터리 메서드란?
정적 팩터리 메서드라는 말이 낯설게 느껴질 수도 있겠지만 사실은 꽤 친숙한 메서드일 것이다. **객체를 생성할 때, 생성자가 아니라 정적 Static 메서드를 사용하는 것**을 정적 팩토리 메서드라고 한다. 

> 여기서 팩터리는 객체를 생성하는 역할을 분리하겠다는 의미로 [GoF 디자인 패턴]() 중 [팩토리 패턴]()에서 이름만 따왔을 뿐 크게 연관성은 없다.

### 🤓 예시 코드
#### (1) LocalTime 클래스의 of 메소드 (Static 메서드)

```java
public static LocalTime of(int hour, int minute) {
  ChronoField.HOUR_OF_DAY.checkValidValue((long)hour);
  if (minute == 0) {
    return HOURS[hour];
  } else {
    ChronoField.MINUTE_OF_HOUR.checkValidValue((long)minute);
    return new LocalTime(hour, minute, 0, 0);
  }
}

```

#### (2) `hour`, `minutes`을 인자로 받은 뒤 `LocalTime` 객체(9시 30분)를 반환

```java
LocalTime openTime = LocalTime.of(9, 30);
```

`LocalTime`의 `of` 메소드처럼 생성자를 통해 직접 객체를 생성하는 것이 아니라 **_메서드로 객체를 생성하는 것_** 을 정적 팩터리 메서드 라고 한다.

## 🤔 왜 생성자 대신 정적 팩터리 메서드를 고려해야할까?
### 🖐️ 다섯가지 이유
#### 1️⃣ 첫번째 : "이름을 가질 수 있다."

프로그래밍에서 다른 사람이 이해하기 쉽도록 코드를 짜는 것은 중요한 요소 중에 하나다. 
`new` 키워드를 사용해 객체를 만드는 생성자는 생성자의 결국 내부 구조를 잘 알고 있어야만 목적에 알맞게 객체를 생성할 수 있다. 
반면 정적 팩터리 메서드는 이름을 가질 수 있으므로 네이밍을 통해 어떤 값을 이용해 객체를 만드려고 하는지 등 읽는 이에게 코드의 설계 의도를 좀 더 쉽게 전달할 수 있다. 

#### 🤓 비교 1. 생성자

```java
class Phone {
    private String brand;

    public Phone(String brand) {
        this.brand = brand;
    }
}
```

```java
Public class Application {
    public static void main(String[] args) {
        Phone phone = new Phone("Samsung");
    }
}

```
> 위와 같이 생성자로 Phone 객체 생성할 때, 생성자의 내부 구조를 모르고 있다면 new 키워드만으로는 "Samsung" 어떤 정보인지 이해하는 것이 어렵다.

#### 🤓 비교 2. 정적 팩토리 메서드


```java
class Phone {
    private String brand;

    private Phone(String brand) {
        this.brand = brand;
    }
    
    public static Phone brandOf(String brand) {
        return new Phone(brand);
    }
}
```
```java
public class Application {
    public static void main(String[] args) {
        Phone phone = Phone.brandOf("Samsung");
    }
}
```
> 반면 정적 팩토리 메서드로 객체를 생성하는 경우 메서드의 이름을 통해 직관적으로 객체에 대한 정보를 단번에 이해할 수 있다. 이처럼 정적 팩토리 메서드는 객체 생성 시 목적에 알맞은 이름을 표현함으로써 코드의 가독성이 좋아지는 효과가 있다.

또, 같은 시그니처를 가지는 생성자 여러 개가 필요할 때에도 **정적 팩터리 메서드**를 이용할 수 있다.

#### 🤓 비교 1. 생성자

```JAVA
public class Student {
    private String name;
    private String id;

    // 컴파일 오류 발생
    public Student(String name){
        this.name = name;
    }

    public Student(String id){
        this.id = id;
    }
}
```

> 일반적인 생성자 방식으로는 매개변수의 이름만 다른 두 개의 생성자를 만들 수 없다.

#### 🤓 비교 2. 정적 팩토리 메서드

```JAVA
public class StaticStudent {
    private String name;
    private String id;

    public static StaticStudent name(String name){
        StaticStudent staticStudent = new StaticStudent();
        staticStudent.name = name;
        return staticStudent;
    }

    public static StaticStudent id(String id){
        StaticStudent staticStudent = new StaticStudent();
        staticStudent.id = id;
        return staticStudent;
    }
}
```
> 하지만 정적 팩터리 메서드로 바꿔준다면 같은 시그니처를 가지는 생성자 여러개를 만들 수 있다.

#### 2️⃣ 두번째 : "호출할 때마다 인스턴스를 새로 생성할 필요가 없다."
인스턴스의 내부 값을 수정할 수 없는 클래스를 [불변 클래스]()라고 한다. 대표적인 불변 클래스 Boolean 처럼 자주 사용하는 인스턴스(`true`, `false`)의 개수가 정해져있다면 해당하는 숫자만큼 미리 인스턴스를 생성해놓고 조회(캐싱)하여 재활용하는 구조로 만들수 있다. 이렇게 정적 팩터리 메서드와 캐싱 구조를 함께 사용하면 매번 새로운 객체를 생성할 필요가 없어져 불필요한 객체 생성을 피할 수 있다. 특히나 **규모가 커서 생성 비용도 큰 객체가 자주 요청되는 상황이라면** 성능이 저하될 가능성이 있는데, 정적 팩토리 메서드를 사용하면 **성능을 상당히 끌어올릴 수 있는 것이다.**

#### 🤓 예시 코드

```java
public final class Boolean implements java.io.Serializable,Comparable<Boolean> {
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
}
```

> 위의 Boolean 클래스는 `TRUE`, `FALSE`를 상수로 정의해놓고 `valueOf()` 메소드에서 기본 타입인 boolean 값을 받아 Boolean 객체 참조로 변환해주고 있다. `valueOf()` 메소드는 객체를 아예 생성하지 않고 상수를 반환하는 것이다. 이렇게 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피하고, 객체 생성 비용이 큰 객체가 자주 요청되는 상황에서 성능을 끌어올릴 수 있다.

 
#### 🚶‍♂️ 더 나아가기
이렇게 호출마다 인스턴스를 생성하는 것이 아니라, 같은 요청에 같은 인스턴스를 반환하는 방식으로 인스턴스의 생성과 파괴를 철저하게 통제하는 클래스를 [인스턴스 통제 클래스]() 라고 한다. 또 인스턴스가 단 하나뿐임을 보장하며 인스턴스를 통제하는 것은 [플라이웨이트 패턴]()의 근간이 된다.

#### 3️⃣ 세번째 : "하위 타입의 객체를 반환할 수 있다."
생성자를 사용하여 객체를 생성하는 경우 객체의 클래스가 하나로 제한되지만, 정적 팩터리 메서드를 사용하면 원하는 객체를 리턴할 수 있다. 자바 다형성의 특징을 이용하여 높은 자유도와 유연성을 제공하는 것이다. 인터페이스를 구현하고 있는 클래스를 노출시키지 않을 수 있으니 사용자 입장에서도 반환 된 클래스가 어떤 클래스인지 굳이 찾아보지 않아도 되는 장점까지 있다.
#### 🤓 예시 코드
```java
public interface Customer {
    static Customer VIP() {
        return new VIP();
    }

    static Customer MVP() {
        return new MVP();
    }
}

class VIP implements Customer {
}

class MVP implements Customer {
}
```
#### 4️⃣ 네번째 : "매개변수에 따라 다른 클래스의 객체를 반환할 수 있다."
위에서 설명한 세번째 장점과 유사한 맥락으로, 반환 타입의 하위타입이기만 하면 어떤 클래스의 객체를 반환하든 상관 없다. 

#### 🤓 예시 코드
```java
class Customer {
    static Customer upgradeLevel(int totalPrice) {
        if (totalPrice < 500000) {
            return new VIP();
        }

        if (totalPrice < 1000000) {
            return new MVP();
        }

        return new Customer();
    }
```
#### 5️⃣ 다섯번째 : "작성 시점에 반환할 객체의 클래스가 없어도 된다."

한마디로 클라이언트를 구현체로부터 분리해주는 것이다. JDBC의 서비스 접근 API인 `DriverManager.getConnection()`을 예로 들을 수 있다. 
  
~~~JAVA
// JDBC 1단계 : 드라이버 객체 로딩			
DriverManager.registerDriver(new org.h2.Driver());

// JDBC 2단계 : 커넥션 연결
conn = DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test", "sa", "");
~~~

여기 2단계인 `getConnection()`에서 반환되는 구현체는 DBMS 의 종류에 따라 달라진다. 클라이언트는 세부적인 구현 내용을 몰라도 서비스를 이용 할 수 있다. `Connection` 타입은 인터페이스이며, 실제로 구현하는 클래스가 존재하지 않아도 되고, 구현하는 클래스가 계속 추가될 수도 있다.

## 😓 장점이 있으면 단점도 있는 법.
### ✌️ 두가지 단점
#### 1️⃣ 첫번째 : "하위 클래스를 만들 수 없다."
상속을 하려면 public 혹은 protected 생성자가 필요하기때문에, 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다. 인스턴스 통제 클래스를 구현하기 위해서는 사용자가 new 키워드를 사용하여 임의로 객체를 생성함을 막아야한다. 이를 위해 생성자의 접근 제어자를 private 로 설정해야하는데, 생성자가 private 인 클래스는 상속을 할 수 없다. 즉, 부모 클래스가 될 수 없다. 그러나 이 제약은 상속보다 [컴포지션]()을 사용하도록 유도하고, [불변타입]()으로 만들기 위해서는 이 제약을 지켜야 한다고 한다. 따라서 이 단점은 장점으로도 작용할 수도 있다.  


#### 🤓 예시 코드

```java
public class Fruit {

  public class Furniture() { }    // 기본 생성자

  public static Table createTable() {
    return new Table();
  }
}
```

```java
public class Table extends Furniture {
  public Table(){
    super();      // 컴파일 에러 발생
  }
}
``` 

#### 2️⃣ 두번째 : "프로그래머가 찾는 것이 어렵다."
java doc에서 클래스의 생성자는 잘 표시되어있어 쉽게 찾을 수 있으나, 정적 팩터리 메서드는 일반 메서드이기 때문에 개발자가 직접 문서를 찾아야하는 어려움이 있다. 따라서 API 문서를 잘 작성하고 널리 알려진 메서드 네이밍 컨벤션을 사용해서 클라이언트가 잘 사용할 수 있도록 작성해야 한다. 아래는 정적 팩터리 메서드에서 흔히 사용하는 명명 방식이다.

#### 📛 정적 팩터리 메서드 명명방식
- **`from`** : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
- > 
  ```java
  Date d = Date.from(instant);
  ```
- **`of`** : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
  > 
  ```java 
  Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
  ```
  
- **`valueOf`** : `from`과 `of`의 더 자세한 버전
  > 
  ```java 
  BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
  ```
  
- **`instace` 혹은 `getInstance`** : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하나, 같은 인스턴스임을 보장하지 않음.
  > 
  ```java 
  StackWalker luke = StackWalker.getInstance(options);
  ```
  
- **`create` 혹은 `newInstance`** : `instance` 혹은 `getInstance`와 같지만 매번 새로운 인스턴스를 생성해 반환함을 보장.
  > 
  ```java 
  Object newArray = Array.newInstance(classObject, arrayLen);
  ```
  
- **`getType`** : `getInstance`와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용. `Type`은 팩터리 메서드가 반환할 객체의 타입.
  > 
  ```java 
  FileStore fs = files.getFileStore(path);
  ```
  
- **`newType`** : `newInstance`와 같으나 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용. `Type`은 팩터리 메서드가 반환할 객체의 타입.
  > 
  ```java 
  BufferedReader br = Files.newBufferedReader(path);
  ```
  
- **`type`** : `getType`과 `newType`의 간결한 버전
  > 
  ```java 
  List<Complaint> litany = Collections.list(legacyLitany);
  ```
  

## 🙆‍♀️ 정리!
> 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 하지만 대부분의 경우 정적 팩터리를 사용하는 편이 유리하므로 무작정 public 생성자를 사용했다면 습관을 고치는 것이 좋다.

 # 👼 Reference
 - https://tecoble.techcourse.co.kr/post/2020-05-26-static-factory-method/
 - https://7942yongdae.tistory.com/147
 - https://devlog-wjdrbs96.tistory.com/256
 - https://catsbi.oopy.io/d7f3a636-b613-453b-91c7-655d71fda2b1
 - https://coder-in-war.tistory.com/entry/Java-28-%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C
 - https://hudi.blog/effective-java-static-factory-method/
