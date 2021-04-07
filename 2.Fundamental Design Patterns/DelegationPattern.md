# :mortar_board: Delegation 패턴

> iOS 개발을 한다면 UIKit에서 자주 사용되기 때문에 모를수가 없는 패턴(?)

Delegation 패턴은 자기 자신이 작업을 수행하지 않고, 대리자가 작업을 대신 수행할 수 있도록 해준다.

![delegate](/2.Fundamental%20Design%20Patterns/delegate.png)

* Delegation 패턴의 세가지 특징
  * delegate는 대게 순환 참조를 피하기 위해서 약한 참조의 프로퍼티로 가지고 있다.

  * delegate 프로토콜은 대리자가 구현하거나 구현해야 하는 메서드를 정의한다.
  
  * delegate는 대리자의 프로토콜을 구현을 도와주는 개체이다.

> delegate 프로퍼티로 객체레퍼런스를 직접 전달 받기 때문에 weak 키워드를 프로퍼티에 사용하여 약한 참조로 처리한다.

구체적인 객체 대신 delegate 프로토콜에 의존한다면 좀더 유연한 구현이 가능하다. 프로토콜을 구현하는 모든 객체를 대리자로 사용할 수 있다.

> 유연한 구현..?

## 언제 사용될 수 있을까?

거대한 클래스 코드의 분리나 재사용 가능한 컴포넌트 생성에 사용 될 수 있다. 자주 보았던 UIKit에서 DataSource와 Delegate 이름으로 지정된 객체들은 각각 다른 개체에게 데이터를 제공하거나 작업을 수행하도록 요청하기 때문에 Delegation 패턴을 따르고 있다.

> TableView, CollectionView 에서의 dataSource는 보여지는 내용에 대한 처리(데이터 제공), delegate는 이벤트에 대해서 처리한다(이벤트 작업 수행).

## 예제 코드

* tableView와 사용될 item 생성
  
```swift
import UIKit

public class MenuViewController: UIViewController {

  public weak var delegate: MenuViewControllerDelegate?
  
  @IBOutlet public var tableView: UITableView! {
    didSet {
      tableView.dataSource = self
      tableView.delegate = self
    }
  }
  
  private let items = ["Item 1", "Item 2", "Item 3"]
}
```

* Delegate 프로토콜

```swift
public protocol MenuViewControllerDelegate: class {
  func menuViewController(
    _ menuViewController: MenuViewController,
    didSelectItemAtIndex index: Int)
}
```

* DataSource와 Delegate

```swift
// MARK: - UITableViewDataSource
extension MenuViewController: UITableViewDataSource {
  
  public func tableView(_ tableView: UITableView,
                 cellForRowAt indexPath: IndexPath)
    -> UITableViewCell {
      let cell =
        tableView.dequeueReusableCell(withIdentifier: "Cell",
                                      for: indexPath)
      cell.textLabel?.text = items[indexPath.row]
      return cell
  }
  
  public func tableView(_ tableView: UITableView,
                 numberOfRowsInSection section: Int) -> Int {
    return items.count
  }
}

// MARK: - UITableViewDelegate
extension MenuViewController: UITableViewDelegate {
  
  public func tableView(_ tableView: UITableView,
                 didSelectRowAt indexPath: IndexPath) {
    delegate?.menuViewController(self, didSelectItemAtIndex: indexPath.row)
  }
}
```

## 유의할점

Delegate는 매우 유용하지만 남용하기 쉽고, 너무 많은 delegate를 만들지 않도록 유의해야한다.

객체가 여러개의 delegate를 필요로 하는 경우에는 너무 많은 작업을 수행하고 있을 수 있다. 이러한 경우에 객체의 기능을 분리하는 방향으로 고려하는 것이 좋다!

또한 코드 동작을 파악할 때에 클래스간 이동이 잦다면 코드가 너무 많이 분리된 상태일 것이며 적절하게 조절 해야한다.

순환 참조를 만들지 않도록 delegate 프로퍼티는 대부분 약한 참조여야 하고, !를 이용하여 강제 언래핑 하지않도록 한다.(크래시!!)

강한 참조의 delegate를 만들어야하는 상황에서는 다른 패턴사용도 고려 해보자. 예를들어 전략 패턴 사용 등등..

## 핵심요약

> 핵심 내용만 나열 해보자.

* Delegation 패턴의 3가지 특징??

* 거대한 클래스 코드의 분리나 재사용 가능한 컴포넌트 생성에 사용 될 수 있다.

* delegate는 대부분 약한 참조로 사용된다.