# 0. iOS 앱 실행 구조 개요

앱 실행 → iOS 런타임 → 진입 클래스 → 앱 생명 주기 관리

## **✅private란?**

- **정의한 스코프(예: struct, class) 내부에서만 접근 가능**
- 외부에서는 **직접 접근 불가능**
- 즉, private var로 선언된 변수는 **해당 View 구조체 안에서만 사용할 수 있음**

![image.png](attachment:15640498-1286-4b14-be06-30de2aef2bff:image.png)

이렇게 해놓으면 이 View 내에서만 사용 가능

| **키워드** | **접근 가능 범위** |
| --- | --- |
| open | 모듈 외부 + 서브클래싱 허용 |
| public | 모듈 외부에서 접근 가능 |
| internal (기본값) | 같은 모듈 내 |
| fileprivate | 같은 파일 내 |
| private | 선언한 타입 내부에서만 |

### 이해용 c++과 비교

| **개념** | **Swift (**protocol**)** | **C++ (abstract class with pure virtual functions)** |
| --- | --- | --- |
| 목적 | 어떤 **행동의 계약(약속)**을 정의 | 특정 행동을 **강제적으로 구현하도록** 요구 |
| 기본 문법 | protocol SomeProtocol { func doSomething() } | class Base { virtual void doSomething() = 0; }; |
| 인스턴스화 | 직접 인스턴스 불가 | 직접 인스턴스 불가 |
| 사용 방법 | class MyClass: SomeProtocol { ... } | class Derived : public Base { ... } |
| 다중 채택 | 가능 (: A, B, C) | 가능 (public A, public B) (단, 다이아몬드 문제 주의) |
| 접근 제한자 | 없음 (protocol은 모두 public으로 열림) | public, protected, private 등 가능 |
| 구현 제공 | protocol에 기본 구현 불가 (단, **extension**으로 제공 가능) | 추상 클래스는 일부 메서드 구현 가능 |

```swift
protocol Animal {
    func speak()
}

class Dog: Animal {
    func speak() {
        print("멍멍")
    }
}
```

```cpp
class Animal {
public:
    virtual void speak() = 0; // 순수 가상 함수
};

class Dog : public Animal {
public:
    void speak() override {
        std::cout << "멍멍" << std::endl;
    }
};
```

## Delegate vs Protocol

| **구분** | **delegate** | **protocol** |
| --- | --- | --- |
| 개념 | 어떤 일을 **대신 처리해주는 객체(대리인)** | “이런 기능을 구현해라”는 **구체적 약속(청사진)** |
| 실체 | 자체 문법 없음 (용어, 설계 패턴) | Swift 문법으로 존재 |
| 명시 가능? | delegate 자체는 키워드가 아님 | protocol SomeDelegate { ... } |
| 예시 | tableView.delegate = self | UITableViewDelegate 프로토콜 구현 |

delegate는 문법은 아니고 설계 개념이고, 이를 구현하는게 protocol

## **다형성(Polymorphism)이란?**

> **같은 타입(기본 클래스 포인터/참조)** 을 통해 서로 다른 하위 클래스의 동작을 호출할 수 있는 것 즉, **“하나의 이름으로 다양한 동작을 수행”** 하는 성질을 의미.
> 

```cpp
class Animal {
public:
    virtual void speak() { std::cout << "..." << std::endl; }
};

class Dog : public Animal {
public:
    void speak() override { std::cout << "멍멍" << std::endl; }
};

void makeSound(Animal* a) {
    a->speak();  // a가 Dog이면 멍멍, Cat이면 야옹
}
```

## **순수 가상 함수(Pure Virtual Function)**

> 기본 클래스가 어떤 **동작을 “반드시” 하위 클래스가 구현해야 한다고 강제**
> 

```cpp
class Animal {
public:
    virtual void speak() = 0;  // 순수 가상 함수 → 구현이 없음
};
```

## 리플렉션 방식

**클래스 이름을 문자열(String)로 가지고 있다가, 런타임에 그 이름을 기반으로 클래스를 찾고 사용하는 방식**

```swift
if let clazz = NSClassFromString("GADAppDelegate") as? NSObject.Type {
    let instance = clazz.init()
    // SDK가 내부적으로 delegate로 사용하거나 메서드 호출
}
```

주로 저런 형식으로 가져옴

## **“문자열”은 어디에 존재하는가**

크게 2가지 경우로 나뉨:

**① 소스코드 내부에 문자열로 하드코딩**

- 예: SDK 내부에 NSClassFromString("GADAppDelegate")처럼 문자열로 적혀 있음
- 이 문자열은 빌드 시 바이너리 내에 그대로 저장됨 → 런타임에 사용됨

 strings [binary] | grep ClassName 같은 도구로 확인 가능

---

### **② Info.plist나 설정 파일 내**

- 예: NSExtensionPrincipalClass, NSPrincipalClass 등
- 여기에 **“MyNotificationService”**, **“MyAppDelegate”** 같은 클래스 이름이 저장됨
- 시스템은 이 값을 읽고 문자열로서 클래스를 찾음

따라서 **이름이 난독화되거나 변경되면**, 문자열과 매칭되지 않아 **크래시 발생** 가능

---

## **✅ 왜 중요한가?**

리플렉션 기반으로 클래스 이름을 찾는 구조는,

- **앱 자체나 시스템**,
- **외부 SDK (예: Firebase, Google Ads)**
    
    등이 클래스명을 문자열로 **정확히 참조**해야 하기 때문에,
    
    **클래스명을 바꾸거나 난독화하면 작동이 완전히 깨집니다.**
    

---

| 1 | **OS 엔트리 클래스** AppDelegate, SceneDelegate, @main struct MyApp | ‣ *UIKit* : UIApplicationMain 또는 @UIApplicationMain 속성이 **Info.plist** 에 기록된 Principal Class 이름을 그대로 사용해 앱을 부트스트랩. ‣ *SwiftUI* : @main 속성이 붙은 타입을 LLVM이 **맹글링 이름 그대로** main 함수로 변환. | 런타임이 문자열로 클래스를 찾은 뒤 alloc / init. 이름이 달라지면 **앱이 런치되지 않고 즉시 크래시** (UIApplicationMain failed to instantiate delegate class) | · grep -R "UIApplicationMain" .*· SwiftSyntax로 @UIApplicationMain / @main 속성 스캔 |
| --- | --- | --- | --- | --- |
| **2** | **시스템·3rd-party 후킹**푸시, 인앱결제, 광고 등 | 예)    • UNNotificationServiceExtension**의 서브클래스**는 **Info.plist**의 NSExtensionPrincipalClass 키에 그대로 기록됨.   • Ad SDK(예: Google Mobile Ads)가 GADAppDelegate 이름을 리플렉션으로 찾음. | iOS / SDK 가 **문자열 → 클래스 인스턴스**로 매핑. 이름이 변하면 서비스(푸시, 광고 로드 등) 자체가 동작 불가. | · 각 타깃 번들의 Info.plist에서 *PrincipalClass 값 추출· CocoaPods / SPM 모듈의 README에서 “Subclass XXX” 문구 grep |
| **3** | **Interface Builder 연결**Storyboard·XIB의 customClass | .storyboard, .xib, .nib 안에 <customClass="ProfileViewController">가 하드코딩. 런타임에 UINib이 **NSClassFromString(“ProfileViewController”)** 호출. | 이름이 어긋나면 불러올 때 throw: -[UIClassSwapper setClassName:]: class ViewController ... could not be loaded. | · 빌드 스크립트에서 grep -o 'customClass="[^"]*"' *.storyboard· SwiftSyntax로 class .*ViewController 패턴 태그 |
| **4** | **@objc / dynamic 노출 클래스** | @objc class AnalyticsManager: NSObject {}dynamic class HotPatch: NSObject {} | Obj-C 런타임 심볼 테이블(**SEL · Class_⊕ 이름**)이 그대로 공개되어야 **KVC/KVO, Method Swizzling, NSCoding**이 정상 작동. | · AST에 attributes.contains("objc") or "dynamic"· nm -gU로 바이너리에 _OBJC_CLASS_$_ 접두사 찾기 |
| **5** | **@IBInspectable / @IBDesignable** | Interface Builder(IB)가 Design-time 렌더링 때 클래스를 로드하여 **속성을 미리보기**. | 이름이 바뀌면 IB 미리보기 실패→ Xcode 경고 & 런타임 nib load 크래시. | · AST attribute "IBDesignable"·"IBInspectable" 스캔 |
| **6** | **JSBridge / WebKit 메시지 핸들러** | swift<br>userContentController.add(self, name:"logClick")<br>JS → window.webkit.messageHandlers.logClick.postMessage(...) | WebKit이 **name 문자열만**으로 핸들러 객체를 찾음. 클래스 이름이 변하면 매핑 테이블에 없어서 JS와 Native 통신 단절. | · 모든 add(_:name:) 호출의 두 번째 파라미터 값 수집 |
| **7** | **SDK 전용 필수 클래스** | Facebook SDK: ApplicationDelegate,Firebase Auth UI: FUIAuthDelegate,GameKit: GKGameCenterControllerDelegate | SDK 내부(Obj-C)에서 NSClassFromString("ApplicationDelegate")로 특정 이름을 검사하거나, **Method Swizzling** 대상 판단. | · Pods/**/*.m 에서 NSClassFromString("...") grep· SDK 문서 “Do not rename this class” 절 자동 스크랩 |
| **8** | **수동 Reflection 등록 클래스** | swift<br>if let cls = NSClassFromString(className) as? Patch.Type { ... }<br>또는 objc_getClass("Patch") | 개발자가 **문자열로 직접 로드**; 이름 바꾸면 패치·플러그인·Hotfix 로직이 전부 무력화. | · 데이터-플로 분석으로 NSClassFromString 인자 소스 추적· 문자열 리터럴 ↔ 심볼 교집합 확대 매칭 |

# 1. OS 엔트리 클래스

## AppDelegate, SceneDelegate, @main struct MyApp

### 언제 어떻게 동작하는가

*UIKit* : UIApplicationMain 또는 @UIApplicationMain 속성이 **Info.plist** 에 기록된 Principal Class 이름을 그대로 사용해 앱을 부트스트랩. ‣ *SwiftUI* : @main 속성이 붙은 타입을 LLVM이 **맹글링 이름 그대로** main 함수로 변환.

런타임이 문자열로 클래스를 찾은 뒤 alloc / init. 이름이 달라지면 **앱이 런치되지 않고 즉시 크래시** (UIApplicationMain failed to instantiate delegate class)

· grep -R "UIApplicationMain" .*· SwiftSyntax로 @UIApplicationMain / @main 속성 스캔

# **2. 시스템·3rd-party 후킹**푸시, 인앱결제, 광고 등

## 언제 어떻게 동작?

1. UNNotificationServiceExtension**의 서브클래스**는 **Info.plist**의 NSExtensionPrincipalClass 키에 그대로 기록됨.   

```xml
<key>NSExtension</key>
<dict>
  <key>NSExtensionPrincipalClass</key>
  <string>MyNotificationService</string>
</dict>
```

1. Ad SDK(예: Google Mobile Ads)가 GADAppDelegate 이름을 리플렉션으로 찾음.

```swift
let className = "GADAppDelegate"
if let clazz = NSClassFromString(className) as? NSObject.Type {
    let instance = clazz.init()
    // delegate로 등록하거나, 메서드 호출
}
```

UNNotificationServiceExtension은 시스템 프레임워크에 존재하는 클래스로 수정 절대안됨.

ASPresentationAnchorProvider

1. **ASPresentationAnchorProvider**
- 이는 SwiftUI에서 인증 관련 기능을 구현할 때 사용하는 클래스
- 마찬가지로, 특정 이름을 가진 클래스나 메서드가 **정해진 인터페이스(프로토콜)**를 충족하도록 구현돼야 하며,
- 일부는 **시스템이 직접 클래스명을 찾아서 사용**.

# 3. **Interface Builder 연결** Storyboard, XIB의 customClass

## customClass란

**Storyboard나 XIB 내부에서 특정 UI 요소가 연결될 Swift 클래스 이름을 명시하는 속성으로**

xml파일 보고 빼면됨

충돌나는 이유는 2번이랑 유사함

# 4. @objc / dynamic 노출 클래스

- @objc = **이걸 Obj-C에서 쓸 수 있게 해주는 역할**
- dynamic = **Swift 컴파일러 최적화 말고 Obj-C처럼 동적으로 호출하게 해주는 역할**

동적 디스패치

```swift
class Animal {
    func speak() {
        print("...")
    }
}

class Dog: Animal {
    override func speak() {
        print("멍멍")
    }
}

let a: Animal = Dog()
a.speak() 
```

Animal 타입의 Dog 인스턴스 생성

컴파일 타임에서 a.speak()하면 Animal.speak()

런타임에서는 Dog.speak()

“메서드 호출 시점을 **런타임에 결정**해서, 실제 인스턴스의 구현을 찾아 호출하는 메커니즘”

상속과는 조금 다른 개념으로

상속은 동적 디스패치를 사용할 수 있게 해주는 전제조건

### **상속(Inheritance)은 구조(구조 관계)**

- 클래스 간 계층 구조를 정의
- Dog: Animal처럼 부모 클래스의 기능을 자식 클래스에 전달

### **동적 디스패치(Dynamic Dispatch)는 동작(실행 방식)**

- 런타임에 **실제 객체의 타입**을 보고 어떤 메서드를 호출할지 결정
- 보통 **오버라이딩된 메서드가 존재**할 때 동적 디스패치가 필요함

복잡하게 왜 저렇게 애매하게 동적 디스패치를 하냐

```swift
class Animal {
    func speak() {
        print("...")
    }
}

class Dog: Animal {
    override func speak() {
        print("멍멍")
    }
}

class Cat: Animal {
    override func speak() {
        print("야옹")
    }
}

func makeItSpeak(_ animal: Animal) {
    animal.speak()
}

let dog = Dog()
let cat = Cat()

makeItSpeak(dog) // 멍멍
makeItSpeak(cat) // 야옹
```

이렇게 작성하면 makeItSpeak는 Animal 타입만 알면된다.

Dog이든, Cat이든 상관 없이 speak() 호출 가능하다.

1. **상속 관계가 유지될 것**
    - Dog는 여전히 Animal을 상속
2. **오버라이딩된 메서드명이 일관될 것**
    - speak() → _x4로 바뀌었다면,
        
        Animal, Dog, Cat 모두에서 speak()를 _x4로 동일하게 바꿔야 함
        
3. **런타임에 해당 메서드를 직접 문자열로 찾지 않을 것**
    - 즉, NSSelectorFromString("speak") 같은 **리플렉션 기반 호출이 없을 경우**

이런 조건이면 가능

# **5. @IBInspectable / @IBDesignable**

**@IBInspectable / @IBDesignable 관련 클래스는 내부 파일들(예: XIB, Storyboard, Nib 등)을 정확하게 수정만 해준다면 난독화해도 괜찮긴 할듯**

외부 SDK에서 호출되는 경우가 없어서 상관 없을 것 같음

얘도 customClass에 써지기는 하지만, @태그로 알아볼 수 있기 때문에, 난독화 적용은 가능함. 대신 UI 쪽이라 좀 고민해봐야 할듯

# 6. **JSBridge / WebKit 메시지 핸들러**

는 뒤에 딸린 문자열만 아니면 난독화 해도 될듯

| 등록된 name | userContentController.add(self, name: "logClick") ← 이 "logClick"은 하드코딩된 문자열 |
| --- | --- |
| 리플렉션 아님 | 이 경우는 클래스명을 NSClassFromString으로 찾는 게 아니라, **name 문자열 자체로 매핑** |
| 안전 조치 | Swift 코드에서 add(_:name:)의 두 번째 인자만 정확히 스캔하면 안전하게 제외 처리 가능 |

# 7. **SDK 전용 필수 클래스**

이거는 외부에서 리플렉션으로 참조하는지 plist로 참조하는지 알 방법이 없음(소스코드가 오픈소스가 아닌이상, 그리고 소스코드가 언제 바뀔지도 모르고)

| **SDK** | **우리가 구현해야 하는 클래스 이름** | **설명** |
| --- | --- | --- |
| **Facebook SDK** | ApplicationDelegate | Facebook 로그인/앱 초기화 관련 delegate |
| **Firebase Auth UI** | FUIAuthDelegate | 인증 완료 시 콜백 제공 |
| **GameKit** | GKGameCenterControllerDelegate | Game Center 관련 이벤트 처리 |

예시로 저런 이름의 클래스를 구현해야 SDK를 사용 가능함

이게 확실한 애들은 넣을 생각 하지 말고, 그냥 제외시킬 방법이나 고려해야 할듯

| **구분** | **방법** | **설명** |
| --- | --- | --- |
| **1. SDK 문서 기반 수작업 식별** | "Do not rename" 문구 자동 검색 | Facebook, Firebase, GameKit 등에서 **명시적으로 클래스명을 요구하는 경우가 많음** → README, 공식 문서, GitHub 내 md/html 파일 grep |
| **2. 바이너리 문자열 분석** | `strings libXYZ.a | grep ‘Delegate’<br>strings libXYZ.framework/libXYZ |
| **3. .m / .swift 코드 내 검색** | grep NSClassFromString | CocoaPods로 받은 SDK의 .m 소스파일이 있는 경우 (드물지만 있음) 직접 리플렉션 호출 여부 확인 가능 |
| **4. 런타임 로깅** (고급) | dyld hooking, objc_getClass 후킹 | 실제 런타임 중 어떤 클래스 이름이 동적으로 불리는지를 기록하는 방법. iOS에서는 어렵지만 **macOS 시뮬레이터**에서는 가능 |
| **5. 클래스가 반드시 존재해야 앱이 동작하는 경우** | 앱을 실행해보고 특정 클래스 이름을 변경한 상태로 크래시 여부 확인 | 가장 원시적이지만 확실한 방법. 크래시 메시지에 Could not instantiate class ...가 뜨면 외부 의존성 존재 |

## 만약 자동으로 제외시킨다면

1. **Podfile.lock / Package.swift**에서 사용 중인 SDK 리스트를 파악
2. SDK별로 [사전 정리된 위험 클래스 리스트](예: Firebase → FUIAuthDelegate, Facebook → ApplicationDelegate)를 보유
3. 빌드시 이 리스트에 포함된 클래스명은 무조건 제외 처리
4. strings-based 추출은 보조 수단으로 사용 (SDK 업데이트 시 바뀔 수 있으므로 신뢰도 낮음)

# 8. 수동 Reflection 등록 클래스

```swift
let className = "MyHotfixClass" // ← 문자열 직접 지정
if let cls = NSClassFromString(className) as? Patch.Type {
    let instance = cls.init()
    instance.applyPatch()
}
```

예시코드로

저런 이름의 클래스가 구현되어 있다고 가정하고 쓰는 부분이긴 한데, 클래스 이름 바꿀 때 변수에 들어가는 문자열과 동시에 잘 바꾸기만 하면 문제는 없는 것으로 보임

---

# 구조체

| **항목** | **설명** | **제외 필요성** |
| --- | --- | --- |
| 1. @main struct MyApp | SwiftUI 앱의 진입점(엔트리포인트). 런타임에 클래스/함수 진입점으로 변환됨. | **필수 제외**이름 바뀌면 앱 실행 실패 |
| 2. ASPresentationAnchorProvider 구현 구조체 | 인증(예: Sign in with Apple) 시 시스템이 이 프로토콜을 만족하는 인스턴스를 요구 | 구현 구조체가 시스템에 의해 요구될 경우, 이름 또는 메서드 유지 필요 |
| 3. @IBInspectable 속성이 있는 구조체 | IB에서 디자인 타임에 렌더링하기 위해 인식됨 | 대부분 클래스에 적용되지만 구조체도 지원 가능→ 사용 여부 확인 후 제외 여부 결정 |
| 4. SwiftUI View 구조체 (struct ContentView: View) | 일반적인 SwiftUI 뷰 구조체 | View 이름이 외부에서 참조되지 않으면 안전 단, Preview나 자동 테스트에서 이름 기반 매칭할 경우 주의 필요 |
| 5. ObservableObject + @Published로 KVO 흉내내는 구조체 | Swift에서는 NSObject 기반 KVO 대신 Combine 사용 | 외부 런타임 접근 없으면 안전 단, @objc와 같이 사용되면 주의 |
| 6. @objc struct (비정상적 사용) | Swift 구조체에 @objc는 거의 쓰지 않음. 컴파일 오류 가능 | 보통 불필요 / 구조체에는 잘 안 씀 |

![image.png](attachment:53eafbd4-3974-426b-a1c0-5c34424e5dbe:image.png)

SPrincipalClass, UIMainStoryboardFile, UISceneDelegateClassName 등등 Info.plist에 적혀있는 시스템 클래스 키

예시)

| **Key 이름** | **용도** | **설명 / 동작 시점** |
| --- | --- | --- |
| **NSPrincipalClass** | 앱 진입점 | 앱의 시작 클래스 (@UIApplicationMain이 붙은 클래스) → OS가 이 클래스 이름으로 런타임에서 NSClassFromString 후 alloc/init |
| **UIApplicationSceneManifest > UISceneDelegateClassName** | SceneDelegate 클래스 지정 | iOS 13+부터 멀티 윈도우를 위한 SceneDelegate 클래스 |
| **UIApplicationSceneManifest > UIApplicationSupportsMultipleScenes** | Scene 시스템 활성화 | (부가 설정 – 클래스명은 아님) |
| **NSExtension > NSExtensionPrincipalClass** | Extension 진입점 | 오늘 위젯, 공유 확장, Siri 등 Extension의 시작 클래스 → OS가 이 클래스 이름으로 시작 |
| **NSApplicationClass** (macOS) | macOS 앱 진입점 | macOS 앱에서 기본 Application 클래스 지정 (보통 NSApplication) |
| **NSServices > NSMessage > NSClass** | macOS 서비스 클래스 | 서비스 호출 처리 클래스 지정 (macOS) |
| **NSAttributedStringDocumentType** 등 일부 문서 형식 클래스 지정 | 문서 기반 앱 | UIDocument 서브클래스를 명시하는 경우 있음 (iOS/macOS) |
| **XPCService > NSPrincipalClass** | XPC 서비스의 진입점 클래스 | macOS/iOS의 XPC 서비스 번들 내부에서 지정 |
| **NIB/XIB/Storyboard 내부의 class=”” 속성 (간접)** | ViewController, View 등 클래스 | 런타임이 XML 속성 값을 기반으로 NSClassFromString 호출해 객체 생성 |

| **① NSPrincipalClass** | 주 앱 Info.plist | ✅ |
| --- | --- | --- |
| **② UISceneDelegateClassName**(UIApplicationSceneManifest › UISceneDelegateClassName) | 주 앱 Info.plist | ✅ |
| **③ NSExtensionPrincipalClass** | 각 Extension 번들 Info.plist | ✅ |
| **④ UIApplicationShortcutItems › UIApplicationShortcutItemTargetContentIdentifier** | 주 앱 Info.plist | ✅ |
- 각 번들별 plist파일도 체크하기

| **② 외부 SDK 하드코딩** | ⑥ | 3rd-party SDK 바이너리 내부 NSClassFromString("YourClass") | SDK-전용 Delegate·Handler | 문자열이 .a/.framework 안 상수 → grep 불가 | SDK 문서·strings 스캔 후 화이트리스트 |
| --- | --- | --- | --- | --- | --- |

| kind | class |
| --- | --- |
| attributes | sdk_hardcoded |
| modifiers | - |
| protocolContext | FirebaseMessagingDelegate, KakaoLoginHandlerProtocol…
+a |
| declContext | sdk_binary |
| ui_related | - |

이거는 외부 sdk 직접 조사(공식문서 읽어서 바뀌면 안되는 클래스 이름 조사)해서 화이트리스트를 작성해야 할듯

JS_Bridge

| kind | class |
| --- | --- |
| attributes | js_bridge |
| modifiers | - |
| protocolContext | WKScriptMessageHandler
 |
| declContext | webkit_bridge |
| ui_related | web |

외부 플러그인 가져다 쓰는 경우

“번들( .bundle / .plug-in / .framework ) **Info.plist → NSPrincipalClass** 로 로드되는 **플러그인 엔트리 클래스**” 라는 카테고리 태그

| kind | class |
| --- | --- |
| attributes | bundle_plugin |
| modifiers | - |
| protocolContext | NSObject |
| declContext | plugin_bundle |
| ui_related | - |

---

여기까지가 무조건 제외해야 할 대상 + UI쪽은 무조건 제외, **Handoff activityType** 에 **클래스명 하드코딩** 된 경우도 제외

HandOff 부분은

NSUserActivity.activityType 을 클래스명으로 하드코딩한 경우를 말하는 것으로 저런 문자열이 들어가있으면 제외 시키면 됨.

---

| **구분** | **대표 키워드 / 특징** | **‘외부 호출될 수 있는’ 전형적 경로** | **점검 실마리(내가 확인할 수 있는 것)** | **보수적 권장** | **LLM 에게 맡길 수 있는 판단 포인트** |
| --- | --- | --- | --- | --- | --- |
| **A. @objc 클래스/메서드** | @objc, @objcMembers | *SDK·UIKit·플러그인* 이 objc_getClass, objc_msgSend 로 호출 | • 소스/AST에 @objc 어노테이션 위치• 외부 SDK 문서에 “이름 고정” 여부 | 이름 고정 또는 @objc("…") 별칭 고정 | “해당 클래스가 어떤 프로토콜·SDK delegate 를 채택하는가?” |
| **B. Selector API 직접 사용** | #selector(…), perform(Selector(…)), responds(to:) | Obj-C 런타임이 문자열 Selector 로 호출 | • grep/AST로 Selector 인수 문자열 확인 | Selector 이름 보존 | “Selector 가 동적 조합·Reflect 사용인지?” |
| **C. Interface Builder 연결** | @IBAction, @IBOutlet, XML selector="…", class="…" | Storyboard/XIB 가 런타임 XML 문자열로 매핑 | • 스토리보드/XIB grep (class=", selector=") | 클래스·메서드명 고정 | “XIB 내에서만 쓰이는가, 코드에서도 쓰이는가?” |
| **D. KVC/KVO / dynamic** | value(forKey:), setValue(_:forKey:), @objc dynamic | 런타임이 프로퍼티명을 문자열로 찾음 | • grep for "forKey:", Mirror( | 프로퍼티명 고정 or @objc("…") | “Key 문자열이 리터럴? 외부서버·JSON 값?” |
| **E. NSClassFromString in app** | NSClassFromString("SomeClass") | 앱 내부 플러그인·전략 패턴 로드 | • 정적 문자열 vs 변수 조합• 해당 클래스 프로토콜 확인 | 클래스명 고정 | “문자열이 하드코딩·UserDefaults·서버 전달?” |
| **F. WebKit JS 브리지 (앱이 직접 등록)** | userContentController.add(_: name:"channel") | JS → Swift 채널명 문자열 | • 채널명 문자열 찾기• JS 번들·HTML에 일치 채널 존재? | 채널명 고정, Handler 이름 @objc 고정 | “채널명이 서버 HTML에서 주입될 가능성?” |
| **G. Runtime Swizzling / Method Exchange** | class_replaceMethod, method_exchangeImplementations | 다른 모듈이 Selector 문자열로 교환 | • swizzle 코드 위치·대상 확인 | 대상 Selector 고정 | “swizzle 대상이 UIKit? 자체 클래스?” |
| **H. 3rd-SDK Delegate 프로토콜 (문서만 존재)** | *ex.* SomeSDKLoginDelegate | SDK 내부가 Selector 문자열 호출 | • 문서/헤더에서 optional func onLogin() 등 확인 | 고정 | “SDK가 @objc optional method를 요구?” |

| **kind** | **attributes**  | **modifiers** | **protocolContext** | **declContext** | **ui_related** | **LLM 판단 포인트**  |
| --- | --- | --- | --- | --- | --- | --- |
| class | @objc
 | ― | — | source_code | — | “@objc 붙었지만 Selector API·SDK 문서 참조가 있는가?” |
| class | **selector_dynamic** | ― | — | source_code | - | “#selector / performSelector에 하드 코딩된 문자열이 들어가는가? 아니면 변수로 들어가는가? |
| class | @IBOutlet, @IBAction
 | ― | UIViewController / UIView | storyboard_xml | ui | ui라 그냥 다 빼면 될듯 |
| class | **kvc_dynamic** | dynamic | — | source_code | — | “value(forKey:) · KVO 동적 접근이 외부 문자열(서버 JSON 등)인가?” |
| class | **nsclass_app** | ― | SomeProtocol | source_code | — | “앱 내부 NSClassFromString("X") → X 가 하드코딩? 서버 주입? |
| class | **js_bridge_local** | ― | WKScriptMessageHandler | webkit_local | web | “userContentController.add(_:name:) 채널명이 **내부 상수**? CDN/HTML 외부 자원?” |
| class | **swizzle_target** | ― | NSObject | source_code | — | “method_exchangeImplementations 대상 Selector 이름이 고정돼야 하나?” |

위 표에 있는 부분은 저런 내용이 있으면 난독화시 충돌 가능성이 있지만, 반드시 그런 것은 아니라서 llm의 도움이 필요한 부분