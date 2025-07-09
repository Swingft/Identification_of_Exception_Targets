# Swift 변수명 난독화 예외 가이드

Swift에서 난독화를 적용하면 안 되는 주요 변수 패턴과 그 이유를 사례와 함께 설명합니다.

---

## 1. `@IBOutlet`

```swift
class MyViewController: UIViewController {
    @IBOutlet weak var titleLabel: UILabel!  // 난독화 금지

    override func viewDidLoad() {
        super.viewDidLoad()
        titleLabel.text = "Welcome"
    }
}
=> titleLabel은 Storyboard 또는 XIB에서 연결됨 → 이름 변경 시 연결 끊김 (앱 크래시)
```

---

## 2. `@Published`

```swift
class ViewModel: ObservableObject {
    @Published var username: String = ""  //  난독화 금지
}

struct ContentView: View {
    @StateObject var viewModel = ViewModel()

    var body: some View {
        TextField("Username", text: $viewModel.username)
    }
}
=> username → $username 자동 생성됨 → SwiftUI DSL에서 변수명 직접 참조됨
```

---

## 3. `@Binding`

→ SwiftUI의 UI 상태 연결(DSL)의 일부

```swift
struct ChildView: View {
    @Binding var a1b2c3: Bool  // 난독화된 이름으로 바꿈

    var body: some View {
        Toggle("Enable", isOn: $a1b2c3)
    }
}

struct ParentView: View {
    @State private var z9y8x7 = false

    var body: some View {
        ChildView(a1b2c3: $z9y8x7)  // 호출부도 같이 바꿔야 동작
    }
}
```

```swift
struct ChildView: View {
    @Binding var isOn: Bool  // 난독화 금지

    var body: some View {
        Toggle("Enable", isOn: $isOn)
    }
}

struct ParentView: View {
    @State private var toggleValue = false

    var body: some View {
        ChildView(isOn: $toggleValue)
    }
}
=> isOn은 상위 뷰에서 변수명을 기준으로 전달됨 → 여러 뷰에서 호출부 추적 어려움
```

---

## 4. `@State`

```swift
struct CounterView: View {
    @State private var count = 0  //  난독화 금지

    var body: some View {
        Button("Count: \(count)") {
            count += 1
        }
    }
}
=> count는 SwiftUI 뷰 상태에 직접 연결됨 → DSL 내부 동작에 영향 → 이름 변경 시 이상 동작
```

---

## 5. Codable

→ `Codable`에서 자동 키 매핑을 사용하는 변수만 난독화 금지 대상

```swift
struct User: Codable {
    var email: String  // 난독화 금지
}
=> JSON { "email": "aaa@bbb.com" }과 자동 매핑됨 → email 이름 변경 시 디코딩 실패
```

→ 하지만 CodingKeys로 키를 명시적으로 지정했다면 변수명은 자유롭게 바꿔도 OK (→ 난독화 가능)

```swift
struct User: Codable {
    var e1x9z2: String  // 난독화 가능 (email → e1x9z2)

    enum CodingKeys: String, CodingKey {
        case e1x9z2 = "email"
    }
}
```

---

## 6. NSCoding

→ forKey: "..." 문자열 키로 연결되는 변수만 정확히 골라서 제외

```swift
class User: NSObject, NSCoding {
    var email: String = ""  // 난독화 금지

    func encode(with coder: NSCoder) {
        coder.encode(email, forKey: "email")
    }

    required init?(coder: NSCoder) {
        email = coder.decodeObject(forKey: "email") as? String ?? ""
    }
}
=> email이 forKey 문자열로 저장됨 → 변수명 변경 시 직렬화/역직렬화 실패
```

---

## 7. `@objc dynamic`

→ KVO/KVC 등 문자열 기반 런타임 참조에 사용됨

```swift
class Person: NSObject {
    @objc dynamic var age: Int = 0  // 난독화 금지
}

// 다른 클래스에서
person.observe(\.age, options: [.new]) { obj, change in ... }

=> KVO에서 "age" 문자열 기반으로 추적 → 이름 변경 시 observer 무력화
```

---

## 8. Realm

```swift
class User: Object {
    @objc dynamic var id: String = ""  // 난독화 금지 -> 이 변수명이 DB 칼럼명으로 사용됨

    override static func primaryKey() -> String? {
        return "id" // 문자열로 명시됨 -> 런타임에 이 key로 칼럼 접근 
    }
}
=> Realm DB에서 id가 컬럼명으로 사용됨 → 이름 바꾸면 쿼리 및 마이그레이션 오류 발생
```

→ return "id" 도 난독화 하면 되는거 아닌가? →  
변수명을 바꾸면 Realm은 새 컬럼이 생긴 것으로 판단 → 마이그레이션 블록 추가 필요

```swift
let config = Realm.Configuration(
    schemaVersion: 2,  // ← 버전 증가
    migrationBlock: { migration, oldSchemaVersion in
        if oldSchemaVersion < 2 {
            // User 클래스의 "id" → "a1b2c3"로 컬럼 이름 변경
            migration.renameProperty(onType: "User", from: "id", to: "a1b2c3")
        }
    }
)

Realm.Configuration.defaultConfiguration = config
```

⇒ 하지만 Realm 모델은 보통 `@objc dynamic` 이 필수이며, 이 구조 자체가 난독화 제외의 핵심 원인입니다.