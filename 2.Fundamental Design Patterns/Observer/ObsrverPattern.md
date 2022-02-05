# :mortar_board: Observer 패턴

> Combine 프레임워크를 이용하여 Observer 패턴에 대해서 알아보자!

Observer 패턴은 하나의 개체가 다른 개체의 **변경점을 관찰** 하는것이다.

![Observer](/2.Fundamental%20Design%20Patterns/Observer/observer.png)

이 패턴은 아래 세가지를 포함한다.

1. `Subscriber`는 관찰자 개체로써 변경점을 전달 받는다.

2. `Publisher`는 관찰가능한 개체로써 변경점을 전달한다. **(발행자)**

3. `Value`는 변경 대상 개체이다.

> 변경 대상의 변화를 감지함으로 이벤트를 전달받아 화면 및 각종 이벤트로 처리 가능하다 => 반응형 프로그래밍

## 언제 사용할 수 있을까?

다른 개체의 변경점을 전달받도록 하고 싶을때 Observer 패턴을 사용한다.

View Controller에 Subscriber가 있고 model에 publisher가 있는 MVC 패턴에서 종종 사용된다. 이렇게 하면 view controller의 타입에 대해서 알 필요 없이 model의 변경점을 view controller에 전달 할수있다.

그래서, 각각의 다른 view controller들이 같은 모델의 변경점을 사용하고 관찰가능하다.

> 관찰 대상인 model은 view와 controller에 대해서 모르기 때문에 코드분리에 용이하다

## 예제 코드

```swift
import Combine

public class User {
    
    // @Published: 관찰 가능한 객체 (Rx의 Observable Object)
    @Published var name: String
    
    public init(name: String) {
        self.name = name
    }
}

let user = User(name: "Ray")

// 관찰 객체 접근시 $ 사용
let publisher = user.$name

// sink: Rx의 bind 역할
// AnyCancellable: Disposable
var subscriber: AnyCancellable? = publisher.sink() {
    print("User's name is \($0)")
}

// onNext
user.name = "Vicki"

```

## 사용시 유의할 점?

Observer 패턴을 구현하기 전에 변경점에 대한 내용 및 조건에 대해서 정의해야 한다. 만약 개체 또는 프로퍼티의 변경 될 필요가 없다면 `var` 또는 `@Published`를 선언하는 대신에 `let`으로 선언하는 것이 좋다.

## 핵심요약

* Observer 패턴을 통해 하나의 개체가 다른 개체의 변경점을 관찰할 수 있으며 `Subscriber(구독자)`, `Publisher(발행자)`, `Value(변경 대상)` 의 세가지 유형을 포함한다.

***
##### Artwork/images/designs: from Design Patterns by Tutorials, available at www.raywenderlich.com