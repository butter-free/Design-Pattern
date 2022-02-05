# :mortar_board: Coordinator 패턴

> 페이지 이동 처리를 Controller로 부터 분리해보자!

## 흐름 및 구조

![coordinator](/4.Advanced%20Design%20Patterns/Coordinator/coordinator.png)

1. **Coordinator**는 children, router 프로퍼티와 present, dismiss 메서드로 정의된 프로토콜이다. (Router에 이벤트 전달하기 위한 메서드 포함.)

2. **Concrete Coordinator**는 구체적인 ViewController를 만드는 방법과 보여지는 순서가 정의되어 있는 Coordinator 프로토콜의 구현체이다.

3. **Router**는 ViewController가 보여지고 사라지게 하기위한 present, dismiss 메서드 등이 정의된 프로토콜이다. (오로지 화면 처리만 담당한다.)

4. **Concrete Router**는 어떻게 ViewController가 보여지는지 알고 있지만, 정확히 어떤 ViewController가 보여지는지 알지 못한다. 대신에 Coordinator는 Router에 보여지게되는 ViewController를 알려준다.

5. **Concrete View Controllers**들은 서로 다른 ViewController에 대해서 알지 못하지만, 대신에 화면전환이 필요할때마다 Concrete Coordinator가 delegate를 수행 한다.

> 각 컴포넌트의 정의와 흐름은 위와 같다 대략적인 느낌은 이해가 되지만 아직 정확한 내용은 잘 모르겠다. 예제 코드로 알아보자!

## 언제 사용될 수 있을까?

이 패턴을 사용함으로써 ViewController를 서로 분리하며, ViewController에 대해서 직접적으로 알고있는 컴포넌트는 Coordinator뿐이기 때문에 ViewController를 훨씬 더 재사용 가능하다.

> 라고 하는데 솔직히 아직은 잘 이해가 되지않아서 자세히 한번 살펴보자.
> 1. ViewController를 서로 분리한다.
>     
> Coordinator 패턴을 적용하지 않는다면 ViewController를 push나 present 한다고 할때, 어떤 ViewController가 처리되는지 기존에 있던 ViewController는 내용을 모두 알고있게 된다. <br>
> 결국 서로 종속되어 만약 새로운 ViewController로 대체된다면 다시 작성해야 될 것이다. 결국 유연하지 못한 상태가 된다. **(결합도 높음)** <br>
> 하지만 Coordinator 패턴을 사용하면 ViewController의 화면처리는 오로지 delegate를 통하여 coordinator가 진행하게 되므로 유연한 교체가 가능하다. **(결합도 낮음)** <br>
>   
> 2. ViewController를 훨씬 더 재사용 가능하다.
>   
> Coordinator안에서 ViewController를 만들고 화면처리가 진행되기 때문에 특히 ViewController를 생성하는 코드의 재사용성이 많이 올라간다. (흐름에 따라서 이미 정의된 생성 메서드 재사용)
>   
> **결론: Coordinator 패턴은 ViewController의 결합도를 낮추고 재사용성이 올라간다.**

## 예제 코드

* Coordinator

```swift
public protocol Coordinator: class {

    var children: [Coordinator] { get set }
    var router: Router { get }
    
    // Router 전달 목적 메서드
    func present(animated: Bool, onDismissed: (() -> Void)?)
    func dismiss(animated: Bool)
    
    // children 관리 메서드
    func presentChild(_ child: Coordinator, animated: Bool, onDismissed: (() -> Void)?)
}

extension Coordinator {

    public func dismiss(animated: Bool) {
        router.dismiss(animated: animated)
    }

    public func presentChild(_ child: Coordinator, animated: Bool, onDismissed: (() -> Void)? = nil) {
        children.append(child)
        child.present(animated: animated) { [weak self, weak child] in
        guard let self = self, let child = child else { return }
        self.removeChild(child)
        onDismissed?()
        }
    }

    private func removeChild(_ child: Coordinator) {
        guard let index = children.firstIndex(where: { $0 === child }) else { return }
        children.remove(at: index)
    }
}
```

* Concrete Coordinator

```swift
// ViewController 생성 및 delegate 처리
// Router로 전달 하는 역할.
public class HowToCodeCoordinator: Coordinator {

    public var children: [Coordinator] = []
    public var router: Router

    private lazy var stepViewControllers = [
        StepViewController.instantiate(
            delegate: self,
            buttonColor: UIColor(red: 0.96, green: 0, blue: 0.11, alpha: 1),
            text: "When I wake up, well, I'm sure I'm gonna be\n\n" +
                "I'm gonna be the one writin' code for you",
            title: "I wake up"),

        StepViewController.instantiate(
            delegate: self,
            buttonColor: UIColor(red: 0.93, green: 0.51, blue: 0.07, alpha: 1),
            text: "When I go out, well, I'm sure I'm gonna be\n\n" +
                "I'm gonna be the one thinkin' bout code for you",
            title: "I go out"),

        StepViewController.instantiate(
            delegate: self,
            buttonColor: UIColor(red: 0.23, green: 0.72, blue: 0.11, alpha: 1),
            text: "Cause' I would code five hundred lines\n\n" +
                "And I would code five hundred more",
            title: "500 lines"),

        StepViewController.instantiate(
            delegate: self,
            buttonColor: UIColor(red: 0.18, green: 0.29, blue: 0.80, alpha: 1),
            text: "To be the one that wrote a thousand lines\n\n" +
                "To get this code shipped out the door!",
            title: "Ship it!")
    ]

    private lazy var startOverViewController = StartOverViewController.instantiate(delegate: self)

    public init(router: Router) {
        self.router = router
    }

    public func present(animated: Bool, onDismissed: (() -> Void)?) {
        let viewController = stepViewControllers.first!
        router.present(viewController, animated: animated, onDismissed: onDismissed)
    }
}

// Concrete View Controller -> Concrete Coordinator -> Concrete Router
extension HowToCodeCoordinator: StepViewControllerDelegate {
    public func stepViewControllerDidPressNext(_ controller: StepViewController) {
        if let viewController = stepViewController(after: controller) {
            router.present(viewController, animated: true)
        } else {
            router.present(startOverViewController, animated: true)
        }
    }

    private func stepViewController(after controller: StepViewController) -> StepViewController? {
        guard let index = stepViewControllers.firstIndex(where: { $0 === controller }),
                index < stepViewControllers.count - 1 else {
            return nil
        }
        return stepViewControllers[index + 1]
    }
}

extension HowToCodeCoordinator: StartOverViewControllerDelegate {
    public func startOverViewControllerDidPressStartOver(_ controller: StartOverViewController) {
        router.dismiss(animated: true)
    }
}
```

* Router

```swift
// 화면처리만 담당한다!
public protocol Router: class {

    func present(_ viewController: UIViewController, animated: Bool)

    func present(_ viewController: UIViewController, animated: Bool, onDismissed: (() -> Void)?)

    func dismiss(animated: Bool)
}

extension Router {
    public func present(_ viewController: UIViewController, animated: Bool) {
        present(viewController, animated: animated, onDismissed: nil)
    }
}
```

* Concrete Router

```swift
// Coordinator로부터 전달 받음 / ViewController push, pop 처리만 진행한다.
public class NavigationRouter: NSObject { // UINavigationControllerDelegate를 위한 NSObject

    private let navigationController: UINavigationController
    private let routerRootController: UIViewController?
    private var onDismissForViewController: [UIViewController: (() -> Void)] = [:]

    public init(navigationController: UINavigationController) {
        self.navigationController = navigationController
        self.routerRootController = navigationController.viewControllers.first
        super.init()

        navigationController.delegate = self
    }
}

extension NavigationRouter: Router {
    public func present(_ viewController: UIViewController, animated: Bool, onDismissed: (() -> Void)?) {
        onDismissForViewController[viewController] = onDismissed
        navigationController.pushViewController(viewController, animated: animated)
    }

    public func dismiss(animated: Bool) {
        guard let routerRootController = routerRootController else {
            navigationController.popToRootViewController(animated: animated)
            return
        }

        performOnDismissed(for: routerRootController)
        navigationController.popToViewController(routerRootController, animated: animated)
    }

    private func performOnDismissed(for viewController: UIViewController) {
        guard let onDismiss = onDismissForViewController[viewController] else { return }
        onDismiss()
        onDismissForViewController[viewController] = nil
    }
}

extension NavigationRouter: UINavigationControllerDelegate {
    public func navigationController(_ navigationController: UINavigationController, didShow viewController: UIViewController, animated: Bool) {
        guard let dismissedViewController = navigationController.transitionCoordinator?.viewController(forKey: .from),
            !navigationController.viewControllers.contains(dismissedViewController) else {
            return
        }

        performOnDismissed(for: dismissedViewController)
    }
}
```

* Concrete View Controller

```swift
// MARK: - HomeViewController
public class HomeViewController: UIViewController {
  
  // MARK: - Views
  public var onButtonPressed: (() -> Void)?
  
  private var button: UIButton!
  private var label: UILabel!
  
  // MARK: - View Lifecycle
  public override func loadView() {
    setupView()
    setupButton()
  }
  
  private func setupView() {
    view = UIView(frame: CGRect(x: 0, y: 0, width: 320, height: 480))
    view.backgroundColor = .white
  }
  
  private func setupButton() {
    button = UIButton()
    view.addSubview(button)
    button.translatesAutoresizingMaskIntoConstraints = false
    button.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    button.centerYAnchor.constraint(equalTo: view.centerYAnchor).isActive = true
    button.heightAnchor.constraint(greaterThanOrEqualToConstant: 100).isActive = true
    button.widthAnchor.constraint(equalTo: view.widthAnchor).isActive = true
    
    button.setTitle("How to Code", for: .normal)
    button.backgroundColor = .black
    button.addTarget(self, action: #selector(buttonPressed(_:)), for: .touchUpInside)
  }
  
  @objc private func buttonPressed(_ sender: AnyObject) {
    onButtonPressed?()
  }
}

// MARK: - Constructors
extension HomeViewController {
  
  public class func instantiate() -> HomeViewController {
    let viewController = HomeViewController()
    viewController.title = "Code"
    return viewController
  }
}
```

```swift
// MARK: - StartOverViewControllerDelegate
public protocol StartOverViewControllerDelegate: class {
  func startOverViewControllerDidPressStartOver(_ controller: StartOverViewController)
}

// MARK: - StartOverViewController
public class StartOverViewController: UIViewController {
  
  // MARK: - Instance Properties
  public var delegate: StartOverViewControllerDelegate?
  
  // MARK: - Views
  private var button: UIButton!
  private var label: UILabel!
  
  // MARK: - View Lifecycle
  public override func loadView() {
    setupView()
    setupButton()
  }
  
  private func setupView() {
    view = UIView(frame: CGRect(x: 0, y: 0, width: 320, height: 480))
    view.backgroundColor = .white
  }
  
  private func setupButton() {
    button = UIButton()
    view.addSubview(button)
    button.translatesAutoresizingMaskIntoConstraints = false
    button.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    button.centerYAnchor.constraint(equalTo: view.centerYAnchor).isActive = true
    button.heightAnchor.constraint(equalToConstant: 100).isActive = true
    button.widthAnchor.constraint(equalTo: view.widthAnchor).isActive = true
    
    button.setTitle("START OVER", for: .normal)
    button.backgroundColor = .black
    button.addTarget(self, action: #selector(startButtonPressed(_:)), for: .touchUpInside)
  }
  
  @objc private func startButtonPressed(_ sender: AnyObject) {
    delegate?.startOverViewControllerDidPressStartOver(self)
  }
}

// MARK: - Constructors
extension StartOverViewController {
  
  public class func instantiate(delegate: StartOverViewControllerDelegate?) -> StartOverViewController {
    let viewController = StartOverViewController()
    viewController.delegate = delegate
    viewController.title = "Start Over?"
    return viewController
  }
}
```

```swift
// MARK: - StepViewControllerDelegate
public protocol StepViewControllerDelegate: class {
  func stepViewControllerDidPressNext(_ controller: StepViewController)
}

// MARK: - StepViewController
public class StepViewController: UIViewController {
  
  // MARK: - Instance Properties
  public var delegate: StepViewControllerDelegate?
  
  private var buttonColor: UIColor!
  private var text: String!
  
  // MARK: - Views
  private var button: UIButton!
  private var label: UILabel!
  
  // MARK: - View Lifecycle
  public override func loadView() {
    setupView()
    setupButton()
  }
  
  private func setupView() {
    view = UIView(frame: CGRect(x: 0, y: 0, width: 320, height: 480))
    view.backgroundColor = .white
  }
  
  private func setupButton() {
    button = UIButton()
    view.addSubview(button)
    button.translatesAutoresizingMaskIntoConstraints = false
    button.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    button.centerYAnchor.constraint(equalTo: view.centerYAnchor).isActive = true
    button.heightAnchor.constraint(greaterThanOrEqualToConstant: 100).isActive = true
    button.widthAnchor.constraint(equalTo: view.widthAnchor).isActive = true
    
    button.setTitle(text, for: .normal)
    button.titleLabel?.numberOfLines = 0
    button.titleLabel?.lineBreakMode = .byWordWrapping
    button.backgroundColor = buttonColor
    button.addTarget(self, action: #selector(nextButtonPressed(_:)), for: .touchUpInside)
  }
  
  @objc private func nextButtonPressed(_ sender: AnyObject) {
    delegate?.stepViewControllerDidPressNext(self)
  }
}

// MARK: - Constructors
extension StepViewController {
  
  public class func instantiate(delegate: StepViewControllerDelegate?,
                                buttonColor: UIColor,
                                text: String,
                                title: String) -> StepViewController {
    let viewController = StepViewController()
    viewController.delegate = delegate
    viewController.buttonColor = buttonColor
    viewController.text = text
    viewController.title = title
    return viewController
  }
}
```

* PlayGround

```swift
import PlaygroundSupport
import UIKit

let homeViewController = HomeViewController.instantiate()
let navigationController = UINavigationController(rootViewController: homeViewController)

let router = NavigationRouter(navigationController: navigationController)
let coordinator = HowToCodeCoordinator(router: router)

homeViewController.onButtonPressed = { [weak coordinator] in
    coordinator?.present(animated: true, onDismissed: nil)
}

PlaygroundPage.current.liveView = navigationController
```

> 흐름을 파악 해보자!
> Coordinator -> Router -> Present View Controller -> Pass Delegate To Coordinator -> Router -> Present View Controller ...

## 책의 내용과는 차이가 있는 예제코드의 내용

coordinator pattern in swift로 구글링 하니 예제로 나와있는 코드의 내용은 대부분 
  
Coordinator 프로토콜에 navigation controller 프로퍼티와 start()라는 메서드를 정의하여 start() 메서드 내부에서 Concrete View Controller를 생성하고 navigation 처리 내용을 작성하는 방식으로 진행하였다.
  
이부분에서 도서의 예제코드 내용에 차이가 있어서 학습할 때에 혼동이 있었지만, 결국 start() 메서드의 역할이 Router와 동일하다는 것을 알게 되었다.
  
책에서 제공하는 내용이 좀더 정리된(?) 내용으로 예정된 레거시 프로젝트에 반영할때에 Router 프로토콜을 활용할것이다.

## 핵심요약

* Coordinator 패턴은 View Controller 간의 흐름을 구성한다.
  (Coordinator Protocol, Concrete Coorinator, Router Protocol, Concrete Router)

* Coordinator는 모든 Concrete Coordinator가 반드시 구현해야 하는 프로퍼티와 메서드를 정의하는 프로토콜이다.

* Concrete Coordinator는 View Controller의 순서와 어떻게 만들어지는지 알고 있다.

* Router는 모든 Concrete Router가 반드시 구현해야 하는 프로퍼티와 메서드를 정의하는 프로토콜이다.

* Concrete Router는 View Controller가 어떻게 보여지는지 알고 있다.

* Concrete View Controller는 서로에 대해서 어떤 View Controller인지 모른다.

* Coordinator 패턴은 앱의 일부 혹은 전체에 걸쳐서 사용 될 수 있다.

***
##### Artwork/images/designs: from Design Patterns by Tutorials, available at www.raywenderlich.com