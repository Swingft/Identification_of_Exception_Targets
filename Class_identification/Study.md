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

# 1. OS 엔트리 클래스

## AppDelegate, SceneDelegate, @main struct MyApp

### 언제 어떻게 동작하는가

*UIKit* : UIApplicationMain 또는 @UIApplicationMain 속성이 **Info.plist** 에 기록된 Principal Class 이름을 그대로 사용해 앱을 부트스트랩. ‣ *SwiftUI* : @main 속성이 붙은 타입을 LLVM이 **맹글링 이름 그대로** main 함수로 변환.

런타임이 문자열로 클래스를 찾은 뒤 alloc / init. 이름이 달라지면 **앱이 런치되지 않고 즉시 크래시** (UIApplicationMain failed to instantiate delegate class)

· grep -R "UIApplicationMain" .*· SwiftSyntax로 @UIApplicationMain / @main 속성 스캔

# **2. 시스템·3rd-party 후킹**푸시, 인앱결제, 광고 등

## 언제 어떻게 동작?

UNNotificationServiceExtension**의 서브클래스**는 **Info.plist**의 NSExtensionPrincipalClass 키에 그대로 기록됨.   • Ad SDK(예: Google Mobile Ads)가 GADAppDelegate 이름을 리플렉션으로 찾음.

UNNotificationServiceExtension은 시스템 프레임워크에 존재하는 클래스로 수정 절대안됨.

ASPresentationAnchorProvider