# :mortar_board: Strategy 패턴

> Strategy(전략) 패턴이 어떤 패턴인지 알아보자!

Strategy 패턴은 상호 교환이 가능한 객체들의 집합으로 **런타임에 교체 및 설정**할 수 있다.

![strategy](/2.Fundamental%20Design%20Patterns/Strategy/strategy.png)

* iOS 앱 개발에서 Strategy 패턴이 사용될 때 대부분 사용 대상이 뷰 컨트롤러인 경우가 대부분 이지만 기술적으로 상호 교환이 가능한 **동작이 필요한 모든 종류의 객체일 수 있다.**

* Strategy 프로토콜은 모든 전략이 반드시 구현해야하는 메서드들로 정의한다.

* 전략들은 전략 프로토콜을 준수하는 객체이다.

## 언제 사용될 수 있을까?

두가지 혹은 그 이상의 다른 상호 교환이 가능한 동작들이 있을때, Strategy 패턴을 사용 할 수 있다.
  
Strategy 패턴은 [Delegation 패턴](/2.Fundamental%20Design%20Patterns/Delegation/DelegationPattern.md)과 유사하다. 두 패턴 모두 구체적인 객체 대신에 프로토콜에 의존하기 때문에 코드 유연성이 증가된다. 결과적으로 전략 프로토콜을 구현하는 모든 객체는 런타임에 **전략적으로 사용가능하다.**
  
Delegation 패턴과는 다르게 Strategy 패턴은 **객체들의 집합**을 사용한다.
  
Delegate들은 대게 런타임에 고정적이다. 예를 들면, UITableView를 위한 dataSource와 delegate는 Interface Builder로 부터 설정이 가능하고, 이것은 런타임 동안 거의 변화하지 않는다.
  
그러나 각 전략들은 런타임 동안 쉽게 상호 교환이 이루어지는 경향이 있다.

> Delegation 패턴과 비교해 보았다. 두가지 패턴 모두 프로토콜을 의존하기 때문에 코드 유연성이 증가 한다. **(결합도 낮음)**
>  
> Strategy 패턴은 런타임 동안 전략들이 교체되기가 쉽다고 한다. 패턴의 세가지 범주에서는 행동적인 디자인 패턴에 속하는 이유가 이것이 아닐까? 코드 예제를 통해 계속 알아보자.

## 코드 예제

* MovieRatingStrategy 프로토콜

```swift
import UIKit

public protocol MovieRatingStrategy {
  // 평점을 제공하는 Service의 이름를 보여준다.
  var ratingServiceName: String { get }
  
  // 비동기적으로 영화 평점을 가져온다.
  func fetchRating(for movieTitle: String,
    success: (_ rating: String, _ review: String) -> ())
}
```

* MovieRatingStrategy를 구현한 RottenTamotoesClient

```swift
public class RottenTomatoesClient: MovieRatingStrategy {
  public let ratingServiceName = "Rotten Tomatoes"
  
  public func fetchRating(
    for movieTitle: String,
    success: (_ rating: String, _ review: String) -> ()) {
    
    // 실제 서비스에서는 네트워크를 통해 가져온 값으로 지금은 더미값으로 대체한다.
    let rating = "95%"
    let review = "It rocked!"
    success(rating, review)
  }
}
```

* MovieRatingStrategy를 구현한 IMDbClient

```swift
public class IMDbClient: MovieRatingStrategy {
  public let ratingServiceName = "IMDb"
  
  public func fetchRating(
    for movieTitle: String,
    success: (_ rating: String, _ review: String) -> ()) {
    
    let rating = "3 / 10"
    let review = """
      It was terrible! The audience was throwing rotten
      tomatoes!
      """
    success(rating, review)
  }
}
```

두가지 Client 모두 MovieRatingStrategy 프로토콜을 준수하고, 이 객체들은 서로에 대해 모르는 상태이며, 대신에 프로토콜만 의존할 수 있다.

> 서로에 대해 모른다! 즉 의존성이 없다. 그러므로 대체 가능하며 유연한 상태이다.

* Strategy 패턴을 사용한 view controller

```swift
public class MovieRatingViewController: UIViewController {
  
  // view controller 생성시 주입을 통해서 유연하게 동작한다.
  // view controller도 대상이 RottenTomatoesClient인지
  // IMDbClient인지 모른다.
  public var movieRatingClient: MovieRatingStrategy!
  
  @IBOutlet public var movieTitleTextField: UITextField!
  @IBOutlet public var ratingServiceNameLabel: UILabel!
  @IBOutlet public var ratingLabel: UILabel!
  @IBOutlet public var reviewLabel: UILabel!
  
  public override func viewDidLoad() {
    super.viewDidLoad()
    ratingServiceNameLabel.text = 
      movieRatingClient.ratingServiceName
  }
  
  @IBAction public func searchButtonPressed(sender: Any) {
    guard let movieTitle = movieTitleTextField.text
      else { return }
    
    movieRatingClient.fetchRating(for: movieTitle) {
      (rating, review) in
      self.ratingLabel.text = rating
      self.reviewLabel.text = review
    }
  }
}
```

## Strategy 패턴 사용시 유의할 점

패턴 사용시 남용하지 않도록 유의 해야한다. 특히 동작이 변경되지 않는 경우에는 뷰 컨트롤러 또는 객체 컨텍스트에 직접적으로 배치하는편이 좋다. 이 패턴의 비결은 **언제 행동을 해야하는지를 아는것**이다.

> 동작 변경이 없는 경우에는 코드 내부에 직접 배치하도록 하고 남용하지 않도록 유의한다.

## 핵심요약

* Strategy 패턴은 상호 교환가능한 객체들의 집합으로 정의되며, 런타임에 설정되거나 교체 가능하다.
  
* Strategy 패턴은 전략을 사용하는 개체, 전략 프로토콜, 전략의 모임의 세부분으로 구성 된다.

* Strategy 패턴은 Delegation 패턴과 유사하다. 두 패턴 모두 코드 유연성을 위해 프로토콜을 사용한다. Strategy 패턴은 Delegation 패턴과는 다르게 런타임 동안에 교체가 되고, Delegation 패턴은 대부분 고정된다.

***
##### Artwork/images/designs: from Design Patterns by Tutorials, available at www.raywenderlich.com