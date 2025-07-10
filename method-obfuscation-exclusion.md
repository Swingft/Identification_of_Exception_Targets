## 1. Selector
SEL 타입(메서드를 식별하는 고유한 키)을 이용해 런타임에 메서드를 동적으로 호출하는 방식

### `perform(_:with:)`
NSSelectorFromString에 문자열로 들어가기 때문에 난독화로 이름이 바뀌면 responds에서 메서드를 찾지 못함
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
⇒ #selector(method)의 method도 같이 바꾸면 문제 없음(난독화 가능) **but** 메서드 참조 관계를 정확히 파악해서 조심히 바꿔야 함

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
⇒ #selector(method)의 method도 같이 바꾸면 문제 없음(난독화 가능) **but** 메서드 참조 관계를 정확히 파악해서 조심히 바꿔야 함

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
⇒ #selector(method)의 method도 같이 바꾸면 문제 없음(난독화 가능) **but** 메서드 참조 관계를 정확히 파악해서 조심히 바꿔야 함

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

### ⇒ @objc 메서드를 제외하는 방식이 가장 안전하고, selector와 메서드 이름을 매핑해서 같이 난독화하면 난독화 제외 대상을 줄여 난독화 정도를 높일 수 있지만 구현 난이도가 높음

## 2. `@IBAction`
스토리보드에서 발생한 터치 등의 사용자 이벤트를 매서드와 연결
```swift
@IBAction func buttonTapped(_ sender: UIButton) {     // 난독화 X
    print("버튼이 눌렸어요!")
}
→ 스토리보드 내부에 문자열로 메서드 이름이 저장되어 있기 때문에 이름을 변경하면 스토리보드 내부에서 찾을 수 없어 크래시 발생
```
⇒ .storyboard, .xib 파일에 저장된 이름을 같이 수정하면 문제 없음(난독화 가능)

## 3. 시스템 호출
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

## 4. Protocol
프로토콜이 요구하는 정확한 시그니처와 이름을 지켜야 메서드 호출이 가능함
