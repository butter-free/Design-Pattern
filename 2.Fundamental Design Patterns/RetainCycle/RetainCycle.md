# :mortar_board: 순환 참조

> 참조란? 순환 참조란? 참조에 대해서 먼저 알아보자!

## 참조(Reference)

> 클래스가 생성자로 생성이 되어 메모리 주소를 할당받게 된 상태를 인스턴스라 한다.
> 인스턴스는 Heap 영역에 생성되고 스택영역의 인스턴스 변수가 주소값을 가리키게 되며 이러한 **관계**를 참조라 한다.
>> 스택 영역에 존재하는 변수는 힙 영역의 주소값을 가지고 있다.

* Understanding Swift Performance(wwdc2016)
![Allocation](/2.Fundamental%20Design%20Patterns/RetainCycle/Allocation.png)

그래서 순환 참조가 뭐지? 알아보기 위해서 메모리 관리 기법에 대해 알아보자.

## Memory Management

> iOS에서는 Retain count로 메모리 관리를 한다. 참조 갯수로 참조가 늘어나면 1씩 증가하며 0이 되면 해제 된다.

* 메모리 관리 과정
![MRR](/2.Fundamental%20Design%20Patterns/RetainCycle/MRR.png)

## MRR(Menual Retain Release)

> retain(메모리 할당), release(메모리 해제)를 개발자가 직접 해주어야 한다. 런타임에서 진행된다.

## ARC(Auto Reference Cycle)

> MRR과 방식은 동일 하지만 컴파일 단계에서 메모리 할당 해제 코드가 추가 된다. 개발자가 메모리 관리에 대해서 신경쓰지 않아도 되지만 해제되지 않는 경우가 있어 유의 해야 한다.

![ARC](/2.Fundamental%20Design%20Patterns/RetainCycle/ARC_Illustration.jpg)

메모리 처리에 대해서 일일이 코드 처리를 해주지 않아도 되기때문에 편리하면서 코드량도 감소 하지만 사용되지 않는 객체에 대해서 메모리 해제가 되지않는 메모리 누수가 발생할 수 있기 때문에 유의 해야 한다.

## 메모리 누수(Momory Leak)

> ARC에서 순환참조로 인해 객체가 서로 참조하는 경우에 더이상 사용하지 않지만 메모리 해제되지 않는 현상이 발생한다.

예제코드로 메모리 누수가 발생하는 상황을 알아보자.

* 예제코드

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}
```

* 레퍼런스 변수 초기화

```swift
var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")
```

![referenceCycle1](/2.Fundamental%20Design%20Patterns/RetainCycle/referenceCycle1.png)

* 순환 참조

```swift
john!.apartment = unit4A
unit4A!.tenant = johnd
```

![referenceCycle1](/2.Fundamental%20Design%20Patterns/RetainCycle/referenceCycle2.png)

* 메모리 누수 발생

```swift
john = nil
unit4A = nil
```

![referenceCycle1](/2.Fundamental%20Design%20Patterns/RetainCycle/referenceCycle3.png)

메모리 누수가 발생되면 해당 메모리를 해제 할 방법이 없다. 메모리 누수가 많이 발생한다면 메모리 사용량이 계속 증가하여 결국 앱이 강제 종료 될수도 있다!

## 순환 참조 해결방안

> weak, unowned, 약한 참조, 강한 참조

## 참고

* Advanced Swift(objc.io)
* https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html
* https://developer.apple.com/library/archive/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html
* https://blog.indoorway.com/swift-vs-kotlin-the-differences-in-memory-management-860828edf8 (JVM)
