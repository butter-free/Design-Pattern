# :mortar_board: MVC 패턴

> 오랫동안 널리 사용된 MVC 패턴의 장단점은 무엇일까?
> 특히 애플식(?) MVC패턴이 Massive View Controller로도 불리게 되었는지 알아보자!

## MVC 구조

![MVC](/2.Fundamental%20Design%20Patterns/mvc.png)

MVC 패턴은 Model - View - Controller로 구분된다.

* **Model**: 저장된 데이터 및 연관된 로직이며, 보통 구조체나 간단한 클래스로 구성 한다.
* **View**: 눈에 보여지는 화면의 구성 요소를 말한다.
* **Controller**: Model과 View의 상태에 따라서 각각을 제어하는 역할을 한다.

MVC의 특징으로는 컨트롤러는 모델과 뷰를 가지고 있기 때문에 직접적으로 접근 가능하다. 또한 컨트롤러는 하나 이상의 모델과 화면을 가지고 있을 수 있다.

반대로 **순환 참조**가 발생 하기 때문에 모델과 뷰는 속해있는 컨트롤러의 **강한 참조**를 가져서는 안된다.

모델은 프로퍼티 옵저빙을 통해, 뷰는 IBAction을 통해 컨트롤러와 커뮤니케이션 한다.

> 
> [순환 참조?? 참조가 뭐지?](/2.Fundamental%20Design%20Patterns/RetainCycle.md)
