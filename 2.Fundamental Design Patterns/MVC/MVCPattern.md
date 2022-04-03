# :mortar_board: MVC 패턴

> 오랫동안 널리 사용된 MVC 패턴에 대해 알아보자.

## MVC 구조

MVC 패턴은 Model - View - Controller로 구분된다.

![MVC](/2.Fundamental%20Design%20Patterns/MVC/Resources/mvc.png)

:heavy_check_mark:  **Model**: 저장된 **데이터 및 비즈니스 로직의 집합**이며, 보통 구조체나 간단한 클래스로 구성 한다.
:heavy_check_mark:  **View**: 보여지는 **화면의 구성 요소**를 말한다.
:heavy_check_mark:  **Controller**: Model과 View의 **상태**에 따라서 각각을 **제어하는 역할**을 한다.

MVC의 특징으로는 **컨트롤러는 모델과 뷰를 소유하고 있기 때문에 직접적으로 접근 가능**하다. 또한 컨트롤러는 하나 이상의 모델과 화면을 가지고 있을 수 있다.

반대로 **순환 참조**가 발생 하기 때문에 모델과 뷰는 속해있는 컨트롤러의 **강한 참조**를 가져서는 안된다.

:link: [참조](/2.Fundamental%20Design%20Patterns/RetainCycle/RetainCycle.md)

### :heavy_exclamation_mark:**apple's MVC**
> 애플이 제공하는 프로그래밍 환경에서는 기본적으로 ViewController가 존재한다.
> 명칭과 같이 View와 Controller의 역할을 하고, 일반적인 MVC패턴과 비교해 봐도 차이가 있다.

:arrow_forward: MVC
![MVC](/2.Fundamental%20Design%20Patterns/MVC/Resources/traditional_mvc.png)

> Controller가 View로부터 이벤트(User aciton)를 전달받아 Model을 업데이트 하고 Model의 상태변화 또는 Controller에 의해 View가 업데이트 된다.

> **Controller, View, Model 서로 관계가 얽혀있고, 분리가 쉽지않아 재사용이 쉽지않다.**

:arrow_forward: Cocoa MVC
![MVC](/2.Fundamental%20Design%20Patterns/MVC/Resources/apple_mvc.png)

> Controller가 View로부터 이벤트(User aciton)를 전달받아 Model을 업데이트 하고 Model의 상태변화를 Controller가 감지하여 컨트롤러에 의해 View가 업데이트 된다.

> 일반적인 MVC와는 다르게 Model의 상태변화를 Controller가 전달 받게되어 **View와 Model간의 관계 분리가 이루어 졌다. (서로 모르는 상태)**

서로 비교하게 되면, 일반적인 MVC보다 Cocoa MVC에서는 Controller의 역할이 늘어났다.
View와 Model이 분리되었기 때문에 서로에 대해서 신경쓰지않고 개발이 가능하다.

:arrow_forward: 현실적인 Cocoa MVC

![MVC](/2.Fundamental%20Design%20Patterns/MVC/Resources/cocoa_mvc.png)

ViewController가 View의 라이프 사이클에 과도하게 관여하고 있고, View에 대한 Delegate(대리자) 및 DataSource(데이터) 구현의 역할(TableView, CollectionView 등)도 담당하고 있기 때문에 **ViewController는 Massive(거대한)한 형태가 되기 쉬워진 구조가 되어버린다.**


***
_출처: https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html#//apple_ref/doc/uid/TP40010810-CH14_

https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52

***
##### Artwork/images/designs: from Design Patterns by Tutorials, available at www.raywenderlich.com