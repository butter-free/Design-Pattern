# :mortar_board: Singleton 패턴

> 자주 사용되며 남용하기 쉬운 Singleton 패턴에 대해서 알아보자!

**Singleton** 패턴은 클래스를 오직 하나의 인스턴스로만 접근하도록 제한한다. 클래스에 대한 모든 참조는 **같은 인스턴스를 참조**한다. 

![singleton](/2.Fundamental%20Design%20Patterns/Singleton/singleton.png)

**Singleton plus** 패턴 또한 다른 인스턴스도 만들 수 있는 공유 singleton 인스턴스를 제공하는 일반적인 패턴이다.

## 언제 사용될 수 있을까?

클래스의 인스턴스가 하나 이상 있을 경우에 문제가 발생하거나 논리적이지 않을 경우 singleton 패턴을 사용한다.
  
공유 인스턴스가 대부분 유용하지만 또한 사용자 정의 인스턴스도 만들 수 있도록 허용하려면 singleton plus 패턴을 사용한다.
  
모든 파일 시스템 접근에 대해서 제어하는 FileManager를 예를들면 default 인스턴스는 singleton 혹은 직접 생성 할 수 있다. 대게 직접 생성이되면 **background thread**에서 사용된다.

> Singleton plus..! 처음 들어보는 패턴의 이름이다. 내용으로는 보통은 static 프로퍼티로 하나의 인스턴스만 접근하도록 처리를 하는데 Singleton plus는 직접 생성으로 새로운 인스턴스를 만들도록..? 추가 처리가 진행이 되는것 같다.
> background thread에서 사용될때 주로 이러한 처리를 한다고 하니 흥미롭다..

## 예제 코드

디자인 패턴 분류에서 Singleton 패턴은 생성 패턴에 속해있다. 그 이유는 Singleton은 공유 인스턴스 생성에만 사용되기 때문이다.

* UIApplication

```swift
let app = UIApplication.shared
let app2 = UIApplication() // Error!
```

> Apple framework 중 하나인 UIApplication 객체가 singleton으로 구성되어 있다는 것을 보여준다.
> app2의 경우 생성자가 private로 지정되어 있어 에러가 발생하게 된다.

* Singleton

```swift
public class MySingleton {
    static let shared = MySingleton()

    private init() { }
}

let mySingleton = MySingleton.shared
```

> 단일 인스턴스 접근을 위한 static 프로퍼티와 private로 지정된 생성자로 Singleton 패턴의 정석적인 형태이다.

* FileManager

```swift
let defaultFileManager = FileManager.default
let customFileManager = FileManager()
```

> FileManager는 singleton사용과 새로운 인스턴스 생성을 모두 지원하는 형태의 Singleton Plus이다.
> 이처럼 사용되기 위해서는 생성자를 private 접근 지정자를 사용해서는 안된다.
> 어떤 이점이 있을까..?

* Singleton Plus

```swift
public class MySingletonPlus {

  static let shared = MySingletonPlus()

  public init() { }
}

let singletonPlus = MySingletonPlus.shared

let singletonPlus2 = MySingletonPlus()
```

> Singleton Plus 패턴 예제 코드이다 역시나 일반적인 Singleton 패턴과 다르게 public 접근 지정자를 사용하여 생성자를 정의 하였다.

## 사용시 유의할 점?

Singleton 패턴은 남용하기 매우 쉬우며, 사용할만한 상황에서 먼저 다른 방법으로 해결가능 한지 고려해보자.

예를들어, 단순히 하나의 뷰 컨트롤러에서 다른 객체로 정보를 전달한다고 할때, Singleton의 사용은 적절하지 않으며 생성자나 프로퍼티를 통한 정보 전달을 고려해보자.

> 단순 상태관리를 위해서 사용하였던 경우가 종종 있었던것 같다. 사용하기 전에 다른 처리방법이 있는지 생각해보자.

Singleton이 필요하다고 생각되는 경우에는 Singleton Plus 패턴이 더 나은 선택인지 고려해보자.

> 간혹 Singleton 객체를 초기화 해야하는 경우 보통 release 메서드 등을 작성하여 초기화 코드를 넣어주어서 해결을 하였는데 이러한 경우에 Singleton Plus 패턴이 유용하게 사용될 것 같다.

Singleton이 문제가 되는 가장 일반적인 이유는 테스트하기 어렵다는 것이다. 만약 singleton과 같이 전역으로 저장되는 상태를 가지고 있다면 상태 변화 및 mock 데이터 처리가 곤란할 것이다.

> 테스트 코드 작성에서 Singleton Mock 데이터를 작성 할때에 상태가 다양하며 접근 코드가 매우 많다면 생각만으로 테스트가 매우 힘들것 같다. 항상 테스트를 고려하여 코드가 작성되어야 할것 같다..

## 핵심 요약

* Singleton 패턴은 클래스가 하나의 인스턴스로만 접근하도록 제한한다.

* Singleton Plus 패턴은 "default" 공유 인스턴스 및 직접 인스턴스의 생성도 가능하다.

* 패턴 남용을 유의해야 한다! singleton을 만들기 전에 다른 해결방법이 있는지 고려해보자 singleton을 꼭 적용해야 한다면 singleton plus 패턴도 염두해 두자.
