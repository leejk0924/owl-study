# 아이템2.생성자에 매개변수가 많다면 빌더를 고려하라
## ❓ 빌더란?

### 대표적인 객체 생성 디자인패턴
- 점층적 생성자(Telescoping Constructor) 패턴
- 자바 빈(JavaBean) 패턴
- 빌더(Builder) 패턴

생성자(Constructor)로 인스턴스를 생성할 때 겪는 불편함에 대한 방안으로 고안 된 디자인 패턴이다. Java에서 클래스를 객체화하는 패턴으로는 [점층적 생성자 패턴]()과 [자바 빈즈 패턴](), [빌더 패턴]()이 존재하는데, [점층적 생성자 패턴]()이 가진 안전성과 [자바 빈즈 패턴]()의 가독성이 합쳐진 것이 [빌더 패턴]()이다. 빌더 패턴에 대해서 더 자세히 알아보기 위해서는 결국 나머지 두 패턴을 이해하고 비교해보아야 한다.


## 🤔 왜 하필 빌더를 고려해야할까?

### 빌더 패턴의 장점
1. 매개변수가 많아도 쉽고 안전하게 객체를 생성할 수 있다.
2. 매개변수의 순서와 상관없이 유연하게 객체를 생성할 수 있다.
3. 적절한 이름을 부여하여 가독성을 높일 수 있다.

결론부터 말하자면 빌더패턴은 생성자(Constructor)가 많을 경우 또는 오브젝트 생성 후 변경 불가능한 불변 오브젝트가 필요한 경우, 불변 오브젝트를 생성하여 오브젝트의 일관성(Consistency),변경 불가능(immutable)을 실현하여 코드의 가독성과 불변성,일관성을 유지하도록 하는 장점이 있다. 앞에서 설명했듯 이런 빌더 패턴의 장점은 다른 패턴과의 비교를 통해서 더 깊게 이해할 수 있을 것이다.

## 점층적 생성자 패턴

## 자바 빈즈 패턴

## 🙆🏻‍♀️  핵심정리
> 생성자나 정적 팩터리가 처리해야할 매개변수가 많다면 빌더 패턴을 선택하는 편이 더 좋다. 매개변수 중 다수가 필수가 아니거나, 같은 타입이라면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고 자바빈즈보다 훨씬 안전하다.

# 👼 Reference
- [Java 빌더 패턴 (Builder Pattern)이란? by Jan92](https://wildeveloperetrain.tistory.com/30)
- [[JAVA] 빌더 패턴(Builder Pattern)에 대해 알아보자 by demonic](https://lemontia.tistory.com/483)
- [[Java] 빌더 패턴(Builder Pattern)을 사용해야 하는 이유 by 망나니개발자](https://mangkyu.tistory.com/163)
- [Builder 기반으로 객체를 안전하게 생성하는 방법 by yun](https://cheese10yun.github.io/spring-builder-pattern/)
- [[Java] 빌더 패턴(Builder Pattern) by Gyun's 개발일지](https://devlog-wjdrbs96.tistory.com/207)
- [Builder Pattern(빌더 패턴 by Effective Java) by 웅이삼촌 개발블로그](https://velog.io/@hero6027/Builder-Pattern%EB%B9%8C%EB%8D%94-%ED%8C%A8%ED%84%B4-by-Effective-Java)
