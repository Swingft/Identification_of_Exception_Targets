## 1. Selector
SEL 타입(메서드를 식별하는 고유한 키)을 이용해 런타임에 메서드를 동적으로 호출하는 방식  
⇒ @objc 메서드를 제외하는 방식이 가장 안전하고, selector와 메서드 이름을 매핑해서 같이 난독화하면 난독화 제외 대상을 줄여 난독화 정도를 높일 수 있지만 구현 난이도가 높음  
⇒ #selector인지 Selector/NSSelectorFromString인지에 따라 처리 방식이 달라짐

### `perform(_:with:)`
```swift
class Greeter: NSObject {
    @objc func sayHello() {     // 난독화 X
        print("Hello!")
    }
}

let greeter = Greeter()
let selector = NSSelectorFromString("sayHello")  
//let selector = Selector("sayHello")

if greeter.responds(to: selector) { 
    greeter.perform(selector) 
}
```

### `addTarget(_:action:for:)`
```swift
class ViewController: UIViewController {
    let button = UIButton()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)
    }
    
    @objc func buttonTapped() {     // 난독화 X
        print("Button tapped!")
    }
}
```

### `Timer.scheduledTimer`
```swift
class TimerHandler: NSObject {
    @objc func timerDidFire() {
        print("Timer fired!")
    }
}

let handler = TimerHandler()

Timer.scheduledTimer(timeInterval: 1.0, target: handler, selector: #selector(handler.timerDidFire), userInfo: nil, repeats: false)
```

### `NotificationCenter.addObserver`
```swift
class Listener: NSObject {
    @objc func handleNotification(_ notification: Notification) {     // 난독화 X
        print("Notification received: \(notification.name)")
    }
}

let listener = Listener()
let name = Notification.Name("MyNotification")

NotificationCenter.default.addObserver(listener, selector: #selector(listener.handleNotification(_:)), name: name, object: nil)

NotificationCenter.default.post(name: name, object: nil)
```

### Method Swizzling 
`method_exchangeImplementations(original, swizzled)`
```swift
extension UIViewController {
    static let swizzleViewDidAppear: Void = {
        let original = #selector(viewDidAppear(_:))
        let swizzled = #selector(my_viewDidAppear(_:))

        let originalMethod = class_getInstanceMethod(UIViewController.self, original)
        let swizzledMethod = class_getInstanceMethod(UIViewController.self, swizzled)

        method_exchangeImplementations(originalMethod!, swizzledMethod!)
    }()

    @objc func my_viewDidAppear(_ animated: Bool) {     // 난독화 X
        print("뷰가 나타났음!")
        self.my_viewDidAppear(animated) 
    }
}
→ original을 호출하면 swizzled가 실행되고, swizzled를 호출하면 original이 실행됨
```

## 2. `override`
부모 클래스의 메서드를 자식 클래스에서 재정의하는 경우  
→ 부모와 자식 간의 연결을 유지해야 하기 때문에, 난독화를 한다면 부모와 자식을 같은 이름으로 난독화해야 함  
⇒ SwiftSyntax에서 `DeclModifierSyntax.name.text == override` 조건으로 override 여부 확인 가능

## 3. `overloading`
동일한 이름에 매개변수 타입이나 개수가 다른 메서드를 정의하는 경우  
→ 모두 다 같은 이름으로 난독화해야 함  
→ 다른 이름으로 변경하면 호출부에서도 매개변수에 맞춰서 각각 변경해야 하기 때문에 변경이 복잡해짐  
⇒ `FunctionDeclSyntax` 노드에서 함수 이름은 같고 파라미터 리스트가 다르면 오버로드 함수로 판단  
(`@objc`는 오버로드를 지원하지 않기 때문에 오버로딩된 메서드에 `@objc`를 붙이는 경우, 컴파일 에러가 발생할 수 있음)

## 4. `@IBAction`
스토리보드에서 발생한 터치 등의 사용자 이벤트를 매서드와 연결
```swift
@IBAction func buttonTapped(_ sender: UIButton) {     // 난독화 X
    print("버튼이 눌렸어요!")
}
→ 스토리보드 내부에 문자열로 메서드 이름이 저장되어 있기 때문에 이름을 변경하면 스토리보드 내부에서 찾을 수 없어 크래시 발생
```
⇒ .storyboard, .xib 파일에 저장된 이름을 같이 수정하면 문제 없음(난독화 가능)

## 5. 시스템 호출
시스템이 자동으로 호출하는 메서드이기 때문에 시그니처나 이름이 변경되면 호출되지 않음

### `AppDelegate` 
- `application(_:didFinishLaunchingWithOptions:)`
- `applicationDidBecomeActive(_:)`
- `applicationWillResignActive(_:)`
- `applicationWillEnterForeground(_:)`
- `applicationDidEnterBackground(_:)`
- `applicationWillTerminate(_:)`

### `SceneDelegate` 
- `scene(_:willConnectTo:options:)`
- `sceneDidBecomeActive(_:)`
- `sceneWillResignActive(_:)`
- `sceneWillEnterForground(_:)`
- `sceneDidEnterBackground(_:)`

### `UIViewController` 
- `viewDidLoad()`
- `viewWillAppear(_:)`
- `viewDidAppear(_:)`
- `viewWillDisappear(_:)`
- `viewDidDisappear(_:)`
- `viewWillLayoutSubviews()`
- `viewDidLayoutSubviews()`
- `loadView()`

### 시스템 생성 및 해제 메서드
- `init()`
- `deinit()`

## 6. Protocol
프로토콜이 요구하는 정확한 시그니처와 이름을 지켜야 메서드 호출이 가능함  
→ 프로토콜의 required 메서드 이름을 난독화하면 프로토콜 준수가 깨지면서 에러 발생  
→ struct/class에서 채택한 표준 프로토콜과 required 메서드를 추출해 난독화에서 제외

### 표준 프로토콜 vs 사용자 정의 프로토콜
→ 사용자 정의 프로토콜의 경우 `protocol SomeProtocol {}` 선언이 소스코드에 존재하고, 이는 AST 상에 `ProtocolDeclSyntax` 노드로 나타남  
→ SwiftSyntax는 사용자가 작성한 파일만 분석하기 때문에, 표준 라이브러리나 SDK 내부 프로토콜은 정의부가 AST에 포함되지 않고, `ProtocolDeclSyntax`로 보이지 않음 (표준 프로토콜과 외부 라이브러리 구분이 안 됨 but 둘 다 제외 대상이기 때문에 구분할 필요 없음)

### required 메서드
→ SourceKit-LSP로 분석해 `key.related_entities`에 `role.implementation`가 있는지 확인
- required 메서드
```json
{
    "key.related_entities": [
        {
            "key.name": "UITableViewDelegate",
            "key.roles": [
                "role.implementation",
                "role.reference"
            ],
            "key.kind": "source.lang.swift.ref.protocol",
            "key.usr": "s:UIKit22UITableViewDelegateP"
        }
    ] 
}
{
    "_comment": "role.implementation : 특정 프로토콜의 요구사항 구현부",
    "_comment": "key.kind : 심볼의 종류로, swift.ref.protocol인 경우 Swift 프로토콜을 참조하는 코드 요소임"
}
```
- 사용자 추가 메서드
```json
{
    "key.related_entities": [
        {
            "key.name": "UIView",
            "key.roles": ["role.reference"]
        }
    ]
}
``` 
⇒ ProtocolDeclSyntax가 없고, key.roles에 role.implementation이 있으면 난독화 제외 대상 
