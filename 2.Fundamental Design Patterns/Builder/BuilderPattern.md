# :mortar_board: Builder 패턴

> Builder 패턴에 대해서 알아보자!

Builder 패턴은 initializer를 통해 모든 입력을 미리 요구하는 대신 단계별로 입력을 제공하여 복잡한 개체를 만들 수 있다.

![Observer](/2.Fundamental%20Design%20Patterns/Builder/builder.png)

Builder 패턴은 아래의 3가지의 주요 유형을 포함한다.

1. `Director`는 입력을 받아들이고 `Builder`에게 전달한다. 대게 뷰 컨트롤러에서 사용되는 뷰 컨트롤러나 helper 클래스이다.
  
2. `Product`는 만들어진 복잡한 개체 이다. 사용되기 원하는 형태에 따라서 구조체 또는 클래스가 될 수 있으며, 대부분 모델이지만 **유스케이스(UseCase)** 에 따라서 어떠한 유형으로도 가능하다.
  
3. `Builder`는 단계별로 입력을 전달 받고 Product의 생성을 처리한다. 대게 클래스의 형태이므로 레퍼런스로 **재사용이 가능**하다.

> 개체를 만들기 위해서 필요한 내용을 Director에서 입력을 받아 Builder로 전달하여 Product를 만드는 과정으로 보인다.

## 언제 사용될 수 있을까?

일련의 단계를 거쳐 복잡한 개체를 만들때에 Builder 패턴을 사용한다.

여러개의 입력이 필요한 개체인 경우에 이 패턴은 특히 유용하다. `Builder`는 `Product`를 만들기 위해서 여러 입력들이 어떻게 사용될지 추상화하며, `Director`가 제공하는 순서에 상관없이 받아들인다.

예를 들어, Builder 패턴을 사용하여 `hamburder builder` 를 구현한다고 하자. `Product`는 hamburger 모델이 될수 있으며 고기 선택, 토핑, 소스 등의 입력을 가지고 있다. `Director`는 어떻게 햄버거를 만들어야 하는지 알고 있는 직원 개체가 될 수 있고 사용자로 부터 입력을 받는 뷰 컨트롤러가 될 수 있다.

따라서 `hamburger builder`는 고기 선택, 토핑, 소스를 입력 받을수 있고, 이러한 요청에 따라서 hamburger를 만든다.

> 햄버거를 예를 들어 설명하였다. Builder 패턴은 개체를 만들기 위해 여러가지 입력이 필요한 경우에 유용해 보인다.

## 예제 코드

* Product

```swift
import Foundation

// MARK: - Product
// meat, sauce, toppings가 포함된 Hamburger 모델
public struct Hamburger {
  public let meat: Meat
  public let sauce: Sauces
  public let toppings: Toppings
}

extension Hamburger: CustomStringConvertible {
  public var description: String {
    return meat.rawValue + " burger"
  }
}

// 고기의 종류
public enum Meat: String {
  case beef
  case chicken
  case kitten
  case tofu
}

// 소스 종류
public struct Sauces: OptionSet {
  public static let mayonnaise = Sauces(rawValue: 1 << 0)
  public static let mustard = Sauces(rawValue: 1 << 1)
  public static let ketchup = Sauces(rawValue: 1 << 2)
  public static let secret = Sauces(rawValue: 1 << 3)

  public let rawValue: Int
  public init(rawValue: Int) {
    self.rawValue = rawValue
  }
}

// 토핑 종류
public struct Toppings: OptionSet {
  public static let cheese = Toppings(rawValue: 1 << 0)
  public static let lettuce = Toppings(rawValue: 1 << 1)
  public static let pickles = Toppings(rawValue: 1 << 2)
  public static let tomatoes = Toppings(rawValue: 1 << 3)

  public let rawValue: Int
  public init(rawValue: Int) {
    self.rawValue = rawValue
  }
}
```

> OptionSet 이란?
> 소스를 예를들어 머스타드와 캐첩을 동시에 햄버거에 넣어야 하는 경우 두개 이상의 값을 처리해야 할때에는 배열을 만들게 되는데 배열이 아닌 하나의 값으로 처리 가능하다.
> [OptionSet](https://developer.apple.com/documentation/swift/optionset)

* Builder

```swift
// MARK: - Builder
public class HamburgerBuilder {

  public enum Error: Swift.Error {
    case soldOut
  }

  // 햄버거에 필요한 재료
  public private(set) var meat: Meat = .beef
  public private(set) var sauces: Sauces = []
  public private(set) var toppings: Toppings = []

  private var soldOutMeats: [Meat] = [.kitten]

  // 외부로 부터 필요한 재료를 입력받아 저장한다.
  public func addSauces(_ sauce: Sauces) {
    sauces.insert(sauce)
  }

  public func removeSauces(_ sauce: Sauces) {
    sauces.remove(sauce)
  }

  public func addToppings(_ topping: Toppings) {
    toppings.insert(topping)
  }

  public func removeToppings(_ topping: Toppings) {
    toppings.remove(topping)
  }

  public func setMeat(_ meat: Meat) throws {
    guard isAvailable(meat) else { throw Error.soldOut }
    self.meat = meat
  }

  public func isAvailable(_ meat: Meat) -> Bool {
    return !soldOutMeats.contains(meat)
  }

  // 전달받은 재료로 햄버거를 만든다!
  public func build() -> Hamburger {
    return Hamburger(meat: meat,
                     sauce: sauces,
                     toppings: toppings)
  }
}
```

* Director

```swift
// MARK: - Director
public class Employee {

  public func createCombo1() throws -> Hamburger {
    let builder = HamburgerBuilder()
    try builder.setMeat(.beef)
    builder.addSauces(.secret)
    builder.addToppings([.lettuce, .tomatoes, .pickles])
    return builder.build()
  }

  public func createKittenSpecial() throws -> Hamburger {
    let builder = HamburgerBuilder()
    try builder.setMeat(.kitten)
    builder.addSauces(.mustard)
    builder.addToppings([.lettuce, .tomatoes])
    return builder.build()
  }
}

```

> Employee 클래스는 Builder를 사용하는 클래스로 필요 재료를 입력하고 생성된 햄버거 객체를 전달 받는다.

* Example

```swift
// MARK: - Example
let burgerFlipper = Employee()

if let combo1 = try? burgerFlipper.createCombo1() {
  print("Nom nom " + combo1.description)
}

if let kittenBurger = try?
  burgerFlipper.createKittenSpecial() {
  print("Nom nom nom " + kittenBurger.description)

} else {
  print("Sorry, no kitten burgers here... :[")
}
```

> Employee를 이용하여 각 햄버거 객체를 가져오는 예제코드

## 사용시 유의할 점은 무엇일까?

Builder 패턴은 일련의 단계를 거쳐 여러개의 입력을 요구하는 복잡한 개체를 만드는데에 가장 적합하다. 만약 Product를 만들때 여러개의 입력이나 단계별로 작성할 수 없는 경우에 Builder 패턴은 문제가 될 수 있다.

이러한 경우, convenience 이니셜라이저를 이용하여 Product를 만드는 것을 고려해보자.

> 남용은 좋지 않다 패턴이 필요한지 생각해보고 그렇지 않다면 이니셜라이저를 활용하자.

## 핵심 요약

* Builder 패턴은 단계별로 복잡한 개체를 만드는데에 적합하며 `Director`, `Product`, `Builder`인 세가지 객체로 구성되어있다.

* Director는 입력을 받아들이고 Builder로 전달한다. Product는 만들어진 복잡한 객체이다. Builder는 단계별 입력들로 객체를 만든다.

> 그룹이나 모임을 만들어야 하는 서비스에서 유용하게 사용할 수 있을것 같다.
