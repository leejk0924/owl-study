# 아이템6. 불필요한 객체 생성을 피하라

## 🙆🏻‍♀️ 핵심 정리
> 1. 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하라.
> 2. 생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피하라.
> 3. 박싱된 기본 타입 보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하라
  
<br>

## 불필요한 String 인스턴스 생성을 피하라.

### new String으로 불필요한 객체를 생성하는 경우
~~~java
        String hello1 = "hello";
        String hello2 = new String("hello");
        String hello3 = "hello";
~~~

문자열 hello1, hello2, hello3은 모두 "hello" 라는 문자열을 갖게 되지만,  
이 hello2가 참조하는 주소 값은 다르기 때문에 동일한 데이터에 대해 서로 다른 메모리를 할당하는 낭비가 발생한다

![](/contents/imgs/item_6_StringPool.png)

그래서 문자열을 선언할 때는 new 키워드를 쓸 것이 아니라 리터럴로 선언해야 한다.
아래는 위 코드에 대한 결과값이다.

~~~java
hello1 == hello2      : false
hello1.equals(hello2) : true

hello1 == hello3      : true
hello1.equals(hello3) : true
~~~

<br>

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많다.  
재사용은 빠르고 세련되며, 특히 불변 객체는 언제든 재사용할 수 있다.  
아래에 **안좋은 예시**와 **개선된 예시**를 한번 비교해 보자.

### 👎 안좋은 예시
~~~java
    for (int i = 0; i < 100000; i++) {
        String s = new String("BAD");
    }
~~~

**걸린 시간**

~~~
4ms
~~~

> ⚠️ `걸린 시간`은 PC 환경에 따라 상이할 수 있습니다.


이 문장은 실행될 때마다 String 인스턴스를 새로 만들기 때문에 **완전히 쓸데없는 행위**이다.

<br>

### 👍 개선된 예시
~~~java
    for (int i = 0; i < 100000; i++) {
        String s = "java";
    }
~~~

**걸린 시간**

~~~
1ms
~~~

> ⚠️ `걸린 시간`은 PC 환경에 따라 상이할 수 있습니다.

이 코드는 새로운 인스턴스를 매번 만드는 대신 **하나**의 String 인스턴스를 사용한다.
같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 **같은 객체를 재사용함이 보장**된다.

<br>

---

## 정적 팩터리 메서드를 이용해 불필요한 객체 생성을 피하라.

생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스에서는 **정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다.**  


### 👎 안좋은 예시
~~~java
public class EmailValidate {

    static boolean email(String email){
        return email.matches("^[a-zA-Z0-9+-\\_.]+@[a-zA-Z0-9-]+\\.[a-zA-Z0-9-.]+$");
    }
}
~~~

이 방식의 문제는 String.matches 메서드를 사용한다는 데 있다.  
String.matches는 정규표현식으로 문자열 형태를 확인하는 **가장 쉬운 방법**이지만,
**성능이 중요한 상황**에서 반복해 사용하기엔 **적합하지 않다.**  
위 코드의 성능을 개선하기 위해서는 정적 팩터리 메서드를 이용해 **인스턴스를 재사용**하게 만들면 된다.


### 👍 개선된 예시

~~~java
public class StaticEmailValidate {

    private static final Pattern EMAIL = Pattern.compile(
        "^[a-zA-Z0-9+-\\_.]+@[a-zA-Z0-9-]+\\.[a-zA-Z0-9-.]+$"
    );

    static boolean isEmail(String email){
        return EMAIL.matcher(email).matches();
    }
}
~~~

이렇게 Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱 후, 나중에 **isEmail** 메서드가 호출될 때 마다 **인스턴스를 재사용**하게 만들면 성능을 상당히 끌어올릴 수 있다.

<br>

**걸린 시간** (10만번 반복)
-  안좋은 예시
~~~
198ms
~~~

- 개선된 예시
~~~
77ms
~~~

> ⚠️ `걸린 시간`은 PC 환경에 따라 상이할 수 있습니다.

성능만 좋아진 것이 아니라, 코드도 더 명확해졌다.  
개선 전에서는 존재조차 몰랐던 **Pattern** 인스턴스를 static final 필드로 끄집어 내고 이름을 지어주어 코드의 의미가 훨씬 잘 드러난다.

<br>

---

## 불필요한 객체를 만들어 내는 오토박싱을 피하라.

오토박싱은 JDK 1.5부터 추가된 기능으로, 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자바 컴파일러가 자동으로 상호 변환해주는 기술이다.

|기본타입|박싱된 기본타입|
|---|---|
|byte|Byte|
|short|Short|
|int|Integer|
|long|Long|
|float|Float|
|double|Double|
|char|Character|
|boolean|Boolean|
|void|Void|

<br>

의미상으로는 별 다를 것 없지만 성능에서는 **그렇지 않다.**  
다음은 **박싱된 기본 타입**을 사용하여 의도치않은 **오토박싱**이 숨어 들었을 때와 **기본타입**을 사용했을 때의 걸린 시간을 비교한 예시이다.

### 👎 박싱된 기본타입을 사용하였을 때
~~~java
    private static long sum() {
        Long sum = 0L;
        for (long i = 0; i < Integer.MAX_VALUE; i++) {
            sum += i;
        }
        return sum;
    }
~~~
**걸린 시간**
~~~
6243.3131ms
~~~

> ⚠️ `걸린 시간`은 PC 환경에 따라 상이할 수 있습니다.
> 
### 👍 기본타입을 사용하였을 때
~~~java
    private static long sum() {
        long sum = 0L;
        for (long i = 0; i < Integer.MAX_VALUE; i++) {
            sum += i;
        }
        return sum;
    }
~~~
**걸린 시간**
~~~
1005.2701ms
~~~

> ⚠️ `걸린 시간`은 PC 환경에 따라 상이할 수 있습니다.


두 예시를 비교해 보면 걸린시간의 차이가 약 6배나 나는것을 볼 수 있다.  
이 예시로 보는 교훈은 명확하다.  
**박싱된 기본 타입 보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.**


## 👼🏻 Reference

