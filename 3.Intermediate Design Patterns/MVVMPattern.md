# :mortar_board: MVVM 패턴

> RxSwift와 잘 어울려 함께 많이 사용하는 패턴인 Model-View-ViewModel 패턴에 대해서 알아보자!

Model-View-ViewModel(MVVM)은 객체를 3개의 개별 그룹으로 구분하는 구조적인 디자인 패턴으로 각 객체는 아래와 같다.

![mvvm](/3.Intermediate%20Design%20Patterns/mvvm.png)
  
* Model은 앱이 가지고 있는 데이터로 대게 간단한 구조체나 클래스로 구성한다.

* View는 화면에 보이는 시각적인 요소로 일반적으로 UIView의 서브클래스이다.

* ViewModel은 Model 정보를 화면에 표시되는 값으로 변환한다.

> 시각적인 요소인 화면(View)과 비즈니스 로직을 처리하는 모델(Model) 모델을 가공하여 화면에 보여주는 값으로 변경하는 뷰 모델(ViewModel)로 구성하여 화면과 비즈니스 로직을 분리한다.
  
> Model은 Entity, UseCase, Repository 등 도메인(개발 목적, 요구사항)에 따라 좀더 세분화 된다.
> https://tech.olx.com/clean-architecture-and-mvvm-on-ios-c9d167d9f5b3

## 언제 사용될 수 있을까?

View에 표시하기 위해 Model을 변경할 필요가 있을때 사용된다. 예를들면, Date를 날짜 문자열로 변환하거나, 숫자를 통화 형식의 문자열로 변경하는 등의 변환을 ViewModel을 이용하여 유용하게 처리할수 있다.
  
특히 View Controller의 코드가 방대해지기 쉬운 MVC 패턴에 대해서 MVVM 패턴으로의 전환은 좋은 해결책이다.

## 예제 코드

* Pet

```swift
import PlaygroundSupport
import UIKit

// MARK: - Model
public class Pet {
  public enum Rarity {
    case common
    case uncommon
    case rare
    case veryRare
  }
  
  public let name: String
  public let birthday: Date
  public let rarity: Rarity
  public let image: UIImage
  
  public init(
    name: String,
    birthday: Date,
    rarity: Rarity,
    image: UIImage
  ) {
    self.name = name
    self.birthday = birthday
    self.rarity = rarity
    self.image = image
  }
}
```

* PetViewModel

```swift
// MARK: - ViewModel
public class PetViewModel {
  
  // 변환이 필요한 Entiy
  private let pet: Pet
  private let calendar: Calendar
  
  public init(pet: Pet) {
    self.pet = pet
    self.calendar = Calendar(identifier: .gregorian)
  }
  
  public var name: String {
    return pet.name
  }
  
  public var image: UIImage {
    return pet.image
  }
  
  public var ageText: String {
    let today = calendar.startOfDay(for: Date())
    let birthday = calendar.startOfDay(for: pet.birthday)
    let components = calendar.dateComponents(
      [.year],
      from: birthday,
      to: today
    )
    let age = components.year!
    return "\(age) years old"
  }
  
  public var adoptionFeeText: String {
    switch pet.rarity {
    case .common:
      return "$50.00"
    case .uncommon:
      return "$75.00"
    case .rare:
      return "$150.00"
    case .veryRare:
      return "$500.00"
    }
  }
}

extension PetViewModel {
  public func configure(_ view: PetView) {
    view.nameLabel.text = name
    view.imageView.image = image
    view.ageLabel.text = ageText
    view.adoptionFeeLabel.text = adoptionFeeText
  }
}
```

> Model(Entity) -> View에 보여지는 값 변환하는 역할을 담당한다 그래서 ViewModel!

* PetView
  
```swift
// MARK: - View
public class PetView: UIView {
  public let imageView: UIImageView
  public let nameLabel: UILabel
  public let ageLabel: UILabel
  public let adoptionFeeLabel: UILabel
  
  public override init(frame: CGRect) {
    
    var childFrame = CGRect(
      x: 0, y: 16,
      width: frame.width,
      height: frame.height / 2
    )
    imageView = UIImageView(frame: childFrame)
    imageView.contentMode = .scaleAspectFit
    
    childFrame.origin.y += childFrame.height + 16
    childFrame.size.height = 30
    nameLabel = UILabel(frame: childFrame)
    nameLabel.textAlignment = .center
    
    childFrame.origin.y += childFrame.height
    ageLabel = UILabel(frame: childFrame)
    ageLabel.textAlignment = .center
    
    childFrame.origin.y += childFrame.height
    adoptionFeeLabel = UILabel(frame: childFrame)
    adoptionFeeLabel.textAlignment = .center
    
    super.init(frame: frame)
    
    backgroundColor = .white
    addSubview(imageView)
    addSubview(nameLabel)
    addSubview(ageLabel)
    addSubview(adoptionFeeLabel)
  }
  
  @available(*, unavailable)
  public required init?(coder: NSCoder) {
    fatalError("init?(coder:) is not supported")
  }
}
```

* Playground

```swift
// MARK: - Example
let birthday = Date(timeIntervalSinceNow: (-2 * 86400 * 366))
let image = UIImage(named: "stuart")!
let stuart = Pet(
  name: "Stuart",
  birthday: birthday,
  rarity: .veryRare,
  image: image
)

let viewModel = PetViewModel(pet: stuart)

let frame = CGRect(x: 0, y: 0, width: 300, height: 420)
let view = PetView(frame: frame)

viewModel.configure(view)

PlaygroundPage.current.liveView = view
```

> 순수 데이터(Model)와 화면(View)에 표현될 값으로 변환하는 역할을 하는 ViewModel

## 사용시 유의할 점?

MVVM패턴은 앱에서 Model을 View에서 필요한 값으로 변환하는 경우가 많은 경우에 유용하지만, 모든 Model 객체가 이러한 변환이 필요한 것은 아니기 때문에 적절히 다른 패턴과 함께 사용해야 한다.

> 모든 Model 객체가 ViewModel을 이용한 변환이 필요한 상태는 아니기 때문에 불필요한 코드 작성이 될 수 있다는 것으로 보인다.

## 핵심 요약

* MVVM은 MVC에 비해서 View Controller의 코드를 줄이는 데에 유용하다.
  
* ViewModel은 객체를 다른 객체로 변환하는 클래스이며, View Controller에 전달되어 View에 표시될 수 있다.
  
* 하나의 View에 대해서 두개 이상의 ViewModel을 사용하는 경우 View를 구성하는 코드를 각자의 View로 구분하는게 더 간단할 수 있다.

***
##### Artwork/images/designs: from Design Patterns by Tutorials, available at www.raywenderlich.com