# :mortar_board: MVC 패턴

> 오랫동안 널리 사용된 MVC 패턴의 장단점은 무엇일까?
> 특히 애플식(?) MVC패턴이 Massive View Controller로도 불리게 되었는지 알아보자!

## MVC 구조

MVC 패턴은 Model - View - Controller로 구분된다.

![MVC](/2.Fundamental%20Design%20Patterns/mvc.png)

* **Model**: 저장된 데이터 및 비즈니스 로직의 집합이며, 보통 구조체나 간단한 클래스로 구성 한다.
* **View**: 눈에 보여지는 화면의 구성 요소를 말한다.
* **Controller**: Model과 View의 상태에 따라서 각각을 제어하는 역할을 한다.

MVC의 특징으로는 컨트롤러는 모델과 뷰를 가지고 있기 때문에 직접적으로 접근 가능하다. 또한 컨트롤러는 하나 이상의 모델과 화면을 가지고 있을 수 있다.

반대로 **순환 참조**가 발생 하기 때문에 모델과 뷰는 속해있는 컨트롤러의 **강한 참조**를 가져서는 안된다.

모델은 프로퍼티 옵저빙을 통해, 뷰는 IBAction을 통해 컨트롤러와 커뮤니케이션 한다.

> [순환 참조? 참조?](/2.Fundamental%20Design%20Patterns/RetainCycle.md)

## 코드 예제

```swift
import UIKit

public struct Address {
    public var street: String
    public var city: String
    public var state: String
    public var zipCode: String
}

public final class AddressView: UIView {
    @IBOutlet public var streetTextField: UITextField!
    @IBOutlet public var cityTextField: UITextField!
    @IBOutlet public var stateTextField: UITextField!
    @IBOutlet public var zipCodeTextField: UITextField!
}

public final class AddressViewController: UIViewController {

    // MARK: - Properties
    // 2. updateAddressFromView내에 address 초기화로 address값이 저장된 직후에
    // Property Observer didSet으로 인해 updateViewFromAddress 실행.
    public var address: Address? {
        didSet {
            updateViewFromAddress()
        }
    }

    public var addressView: AddressView! {
        guard isViewLoaded else { return nil }
        return (view as! AddressView)
    }

    // MARK: - View Lifecycle
    public override func viewDidLoad() {
        super.viewDidLoad()
        updateViewFromAddress()
    }

    // 3.뷰 업데이트
    private func updateViewFromAddress() {
        guard let addressView = addressView,
            let address = address else { return }
        addressView.streetTextField.text = address.street
        addressView.cityTextField.text = address.city
        addressView.stateTextField.text = address.state
        addressView.zipCodeTextField.text = address.zipCode
    }

    // MARK: - Actions
    // 1. 주소를 입력받는 뷰에 주소를 입력하고 업데이트 버튼을 클릭!.
    @IBAction public func updateAddressFromView(_ sender: AnyObject) {
        guard let street = addressView.streetTextField.text,
            street.count > 0,
            let city = addressView.cityTextField.text,
            city.count > 0,
            let state = addressView.stateTextField.text,
            state.count > 0,
            let zipCode = addressView.zipCodeTextField.text,
            zipCode.count > 0 else {
            return
        }
        address = Address(street: street, city: city, state: state, zipCode: zipCode)
    }
}
```

> 코드 예제를 보면 뷰나 모델의 코드에 비해서 뷰 컨트롤러의 코드의 양이 많은 것을 볼 수 있다. 뷰 컨트롤러의 비즈니스 로직 코드를 뷰나 모델에서 나눠서 처리하도록 하면 코드 양을 줄일 수 있다.

## 코드 예제 수정

```swift
import UIKit

public struct Address {
    public var street: String
    public var city: String
    public var state: String
    public var zipCode: String
}

public final class AddressView: UIView {
    @IBOutlet public var streetTextField: UITextField!
    @IBOutlet public var cityTextField: UITextField!
    @IBOutlet public var stateTextField: UITextField!
    @IBOutlet public var zipCodeTextField: UITextField!

    func updateViewFromAddress(_ address: Address?) {
        guard let address = address else { return }
        addressView.streetTextField.text = address.street
        addressView.cityTextField.text = address.city
        addressView.stateTextField.text = address.state
        addressView.zipCodeTextField.text = address.zipCode
    }

    func getAddress() -> Address? {
        guard let street = addressView.streetTextField.text,
            street.count > 0,
            let city = addressView.cityTextField.text,
            city.count > 0,
            let state = addressView.stateTextField.text,
            state.count > 0,
            let zipCode = addressView.zipCodeTextField.text,
            zipCode.count > 0 else {
            return nil
        }

        return Address(street: street, city: city, state: state, zipCode: 
    }
}

public final class AddressViewController: UIViewController {

    // MARK: - Properties
    // 2. updateAddressFromView내에 address 초기화로 address값이 저장된 직후에
    // Property Observer didSet으로 인해 updateViewFromAddress 실행.
    public lazy var address: Address? {
        didSet {
            // 3.뷰에 모델을 직접 전달하여 업데이트
            addressView.updateViewFromAddress(oldValue)
        }
    }

    public lazy var addressView: AddressView! {
        guard isViewLoaded else { return nil }
        return (view as! AddressView)
    }

    // MARK: - View Lifecycle
    public override func viewDidLoad() {
        super.viewDidLoad()
        address = Address(street: "", city: "", state: "", zipCode: "") // 초깃값 설정.
    }

    // MARK: - Actions
    // 1. 주소를 입력받는 뷰에 주소를 입력하고 업데이트 버튼을 클릭!.
    @IBAction public func updateAddressFromView(_ sender: AnyObject) {
        guard let address = addressView.getAddress() else { return }
        address = address
    }
}
```

> 뷰 컨트롤러에 있던 코드를 뷰 내부로 이동시켜 코드를 줄여 보았다.

## MVC 패턴 사용시 주의할 점!

예제 코드를 보면 모델, 뷰의 내용과 비즈니스 로직 등이 컨트롤러 안에 같이 존재하여 컨트롤러의 코드가 방대해지기 쉽다.

> MVC 패턴이 Massive View Controller라고 불리게 된 이유이다. 이러한 문제점을 방지하기 위해서는 어떤 방법이 있을까?
> MVC의 장점은 무엇일까?
