# Generic

이번에는 Generic에 대해서 알아보겠습니다. Genrice의 의미를 영어사전에서 찾아보면 `일반적` 이라는 뜻을 가지고 있습니다. 이처럼 자바에서 Generic은 타입을 일반화한다는 것을 의미하며 컴파일 시점에 개발자가 선언한 타입으로 지정합니다.

## 자바에 Genric이 등장한 이유

Generic은 JDK 1.5버전에 등장하였는데 등장한게 된 가장 큰 이유는 타입 안정성을 보장하기 위함입니다. Genenric이 등장하기 이전에는 Collection Framework와 같은 자료구조를 사용할 때 내부 객체를 다루는 과정에서 형변환이 이루어졌습니다.  이로 인해서 에러가 컴파일 시점에서 발견되는 것이 아니라 런타임 시점에서 발견되기 때문에 에러를 발견하는 것이 쉽지 않았습니다.

Generic이 있지 않았던 상황의 예시를 들어보겠습니다.

```java
import java.util.ArrayList;
import java.util.List;

public class BeforeGeneric {

    public static void main(String[] args) {
        
        List<Object> numbers = new ArrayList<>();
        numbers.add(15);
        // 숫자가 아닌 문자도 들어올 수 있다.
        numbers.add("숫자");
        
        for (Object number : numbers) {
            // 형변환을 직접 해주어야 한다.
            int plus10Number = (int) number + 10;
            System.out.println(plus10Number);
        }
    }
}
```

Generic 이전에는 타입선언이 불가능했기에 예시는 List<Object>를 활용했습니다. 저는 이 코드를 이용하면서 불편한 점을 3가지 느꼈습니다.

1. List<Object>이기 때문에 여러 타입의 객체가 들어올 수 있어 휴먼에러를 발생할 수 있는 가능성이 높다.
2. 특정 연산이나 메소드를 활용하기 위해서는 형변환을 해주어야 하는 것이 불편했습니다.
3. 현재 예러가 발생할 수 있는 코드가 있더라도 컴파일 시점에 잡아주지 못하고 있습니다.

지금은 사실 10줄도 안되는 코드여서 쉽게 눈으로도 예외를 발생할 수 있는 지점을 빠르게 파악할 수 있겠지만 100줄 1000줄의 클래스에서 실시간으로 예외 발생 여부를 파악하는 것은 어려울 것입니다.

이처럼 Generic이 등장하기 전에는 쉽게 말해 휴먼에러가 발생하기 쉬운 개발환경이었습니다.

## 자바의 공변성, 반공변성, 무공변성

공변성, 반공변성 그리고 무공변성은 변성이라는 개념에 포함됩니다. 변성은 타입의 상속 관계에서 다른 타입으로 변경되었을 때 어떤 관계를 가지는지 나타내는 것입니다. 이를 간단하게 표로 먼저 살펴보고 에시를 통해 알아보겠습니다.

|  |  |
| --- | --- |
| 공변성(covariant) | A가 B의 슈퍼타입이면 List<A>가 List<B>에 슈퍼타입이다. |
| 반공변성(contravariant) | A가 B의 슈퍼타입이면 List<B>가 List<A>에 슈퍼타입이다. |
| 무공변성(invariant) | A와 B가 서로 아무런 관계가 없으면 List<A>와 List<B>는 아무런 연관이 없습니다. |

***자바 Generic은 기본적으로 무공변성을 띄고 있습니다.***

### List 에서 공변이 허용되는 경우 발생할 수 있는 문제점

공변성은 두 객체 사이에 SuperType, SubType관계가 존재할 때 다른 타입으로 변경되어도 SuperType, SubType 관계가 동일하게 적용되는 변성입니다.

```java
public class player { }
public class pitcher extends player { }
public class batter extends player { }

List<Player> players = new ArrayList<>();
players.add(new pitcher()); // 가능하다.
players.add(new batter()); // 가능하다.
-----------------------------------------------------
List<Player> players = new ArrayList<>();
List<Pitcher> pitchers = new ArrrayList<>();

players = pitchers; // 불가능하다 만약 이것이 허용된다면
players.add(new Batter()); // Batter는 Player이므로 가능한다
Pitcher pitcher  = players.get(0); // Batter가 반환되는 것인 아닌 Pitcher가 반환된다.
```

위의 예시처럼 Generic에서 공변성을 허용하게 되면 Batter == Pitcher가 되는 모순적인 상황이 나타납니다. 하지만 제네릭에서도 공변이 필요한 상황이 존재하는데요 예를 들면 팀이 우승을 해서 선수들에게 상금을 준다고 하겠습니다.

```java
public void giveWinningPrize (List<Player> players) {
		// 선수들에게 상금을 주는 로직
}
```

이와 같은 메소드가 있을 때 우리는 타자, 투수 모두에게 상금을 주어야 하기 때문에 giveWinningPrize 메소드에 List<Pitcher>와 List<Batter>를 파라미터에 넣어서 이용하려고 할 것이지만 이 때 예외가 발생하게 됩니다.

```java
List<Batter> batters = new ArrayList<>();
List<Pitcher> pitchers = new ArrrayList<>();

giveWinningPrize(batters); // 컴파일 에러 발생
giveWinningPrize(pitchers); // 컴파일 에러 발생
```

즉 위의 상황과 같이 Batter 들과 Pitcher들에게 상금을 주려면 아래와 같이 메소드를 오버로딩에서 구현해야합니다.

```java
public void giveWinningPrize(List<Player> players) {
		// 선수들에게 상금을 주는 로직
}

public void giveWinningPrize(List<Batter> batters) {
		// 타자들에게 상금을 주는 로직
}

public void giveWinningPrize(List<Pitcher> pitchers) {
		// 투수들에게 상금을 주는 로직
}
```

이와 같이 Generic에서도 공변처럼 동작을 해야하는 상황이 필요한데 이 때 `와일드 카드`를 이용합니다.

## 와일드 카드

앞선 예시와 같이 제네릭이 등장했지만 실용성이 떨어지는 상황이 발생했습니다. 이를 해결하기 위해서 모든 타입을 대신할 수 있는 와일드카드 타입 “?”가 등장을 했습니다.

즉 위의 상금을 주는 코드를 아래와 같이 변경할 수 있습니다.

```java
public void giveWinningPrize(List<?> players) {
    // 선수들에게 상금을 주는 로직
}

List<Batter> batters = new ArrayList<>();
List<Pitcher> pitchers = new ArrrayList<>();

giveWinningPrize(batters);
giveWinningPrize(pitchers);
```

이렇게 되면 Batter와 Pitcher 리스트가 들어오더라도 문제가 발생하지 않습니다. 하지만 이와 같이 메소드를 만들게 되면 Batter 및 Pitcher 리스트 외에도 다양한 형태의 리스트가 들어올 수 있고 이는 컴파일 시점에서 예외를 파악하는 것이 아닌 런타임 시점에서 예외를 파악할 수 있습니다. 그래서 우리는 와일드 카드를 이용하데 이를 한정적으로 이용할 수 있도록 제약을 주어야 합니다.

## 한정적 와일드 카드

한정적 와일드 카드는 특정 타입을 기준으로 `extends` 와 `super` 를 통해서 와일드 카드의 상한 범위와 하한 범위를 한정 지어주는 것입니다.

### 상한 경계 와일드카드

상한 경계 와일드카드는 `extends` 키워드를 통해서 최상위 타입을 정의해주는 것입니다. 앞선 예시를 보면 Batter와 Pitcher는 Player에서 상속을 받았기에 Player가 Batter와 Pitcher의 상위 객체입니다. 그래서 상금을 주는 코드를 아래와 같이 변경할 수 있습니다.

```java
public void giveWinningPrize(List<? extends Player> players) {
    // 선수들에게 상금을 주는 로직
}

List<Batter> batters = new ArrayList<>();
List<Pitcher> pitchers = new ArrrayList<>();

giveWinningPrize(batters);
giveWinningPrize(pitchers);
```

이렇게 코드가 수정이 되면 Batter들과 Pitcher들 모두 giveWinningPrize를 이용 가능해지고 Player를 상속받지 않은 다른 객체 리스트는 들어올 수 없게 막을 수 있어 컴파일 시점에서 예외를 파악할 수 있습니다.

### 하안 경계 와일드카드

하한 경계 와일드카드는 super를 통해서 와일드 카드의 최하위의 타입을 정의해주는 방식입니다.

## 참고자료

- https://www.tcpschool.com/java/java_generic_concept
- [https://inpa.tistory.com/entry/JAVA-☕-제네릭-와일드-카드-extends-super-T-완벽-이해](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A0%9C%EB%84%A4%EB%A6%AD-%EC%99%80%EC%9D%BC%EB%93%9C-%EC%B9%B4%EB%93%9C-extends-super-T-%EC%99%84%EB%B2%BD-%EC%9D%B4%ED%95%B4)
- https://www.youtube.com/watch?v=PtM44sO-A6g
- https://mangkyu.tistory.com/241
