- Codable, Decodable
    
    CodingKeys를 사용하지 않고 선언된 변수 
    
    ```jsx
    struct User: Codable {
        var userId: String  //  난독화하면 JSON 키 매핑 실패
        var name: String    // 
    }
    ```
    
    → CodingKeys 없을 경우 변수 명을 그대로 JSON키로 사용해서 변수명이 바뀌면 decode(forKey:)가 key를 찾지 못해 디코딩 실패
    
- NSCoding / NSSecureCoding
    
    encode(forKey:) 또는 decode(forKey:)에서 사용하는 변수
    
    ```jsx
    class Person: NSObject, NSCoding {
        var name: String
    
        func encode(with coder: NSCoder) {
            coder.encode(name, forKey: "name")  // 변수명이 바뀌면 직렬화 안 됨
        }
    
        required init?(coder: NSCoder) {
            name = coder.decodeObject(forKey: "name") as! String
        }
    }
    ```
    
- @objc, dynamic
    - Objective-C 런타임에 노출되는 변수
    - KVC (`setValue:forKey:`) 등에서 문자열로 참조될 수 있음
    
    ```idris
    @objc class MyModel: NSObject {
        @objc var userName: String = ""  // X
        @objc dynamic var age: Int = 0   // X
    }
    ```
    
    ```jsx
    @objc class MyModel: NSObject {
        @objc var userName: String = ""  // X 난독화 제외
    }
    ```
    
    ```jsx
    object.setValue("John", forKey: "userName")  // 런타임에서 이 key를 사용
    ```
    
    "userName"이라는 key가 없어져서 KVC/KVO 실패 → 크래시
    
- @IBOutle
    
    ```jsx
    @IBOutlet weak var submitButton: UIButton!  // X
    ```
    
- Swift UI 관련 (View, DynamicProperty) -`@State`, `@ObservedObject`, `@Binding`, `@EnvironmentObject`
    
    SwiftUI View 내부에서 사용되는 상태 변수
    
    ```jsx
    @ObservedObject var viewModel: MyViewModel  //X
    @State private var isLoading = false        // X
    @Binding var inputText: String              // X
    @EnvironmentObject var settings: AppSettings // X
    ```
    
    SwiftUI는 내부적으로 프로퍼티 이름 기반으로 상태를 추적하거나 연결
    
- ObservableObject + @Published
    
    `@Published` 속성이 붙은 변수
    
    ```jsx
    class UserViewModel: ObservableObject {
        @Published var name: String = ""  // X
    }
    ```
    
    `@Published`는 Combine 퍼블리셔로 SwiftUI와 연결됨
    
    변수명 기반으로 내부 연결 추적 → 이름 바뀌면 View가 상태 감지 못함
    
- `Mirror(reflecting:)`, `dump()` 등 리플렉션 대상 구조체 내 변수
내부적으로 디버깅, 로깅, 테스트용으로 `.label`이 출력되는 변수
    
    → 만약에 사용자가 디버깅 코드 제거를 원하지 않으면 이건 난독화 제외 항목에 들어가긴 해야 함 
    
    ```idris
    struct User {
        var name: String
        var age: Int
    }
    
    let user = User(name: "Jin", age: 30)
    dump(user)
    -> dump라는 디버깅 도구도 활용을 못하게 할 것인지 ~ 
    ```
    
    ```jsx
    struct DebugInfo {
        var filePath: String     // X
        var lineNumber: Int      // X
    }
    ```
    
- `.id(\.variable)` 또는 `\.key` 형태의 KeyPath에 사용되는 변수
    
    SwiftUI, Combine 등에서 KeyPath를 통해 상태나 식별자 추적하는 경우
    
    ```idris
    struct User {
        var username: String
        var age: Int
    }
    
    struct UserListView: View {
        let users: [User]
    
        var body: some View {
            ForEach(users, id: \.username) { user in
                Text(user.username)
            }
        }
    }
    ```
    
    → username을 난독화 하는 경우에  \.username 이나 .username도 같은 형식으로 난독화 하면 되는거 아님?
    
    ⇒ 이론적으로 가능하다만…  SwiftUI의 View identity 및 상태 재사용 시스템을 깨뜨릴 위험이 너무 크기 때문에, KeyPath로 쓰이는 변수는 난독화 대상에서 제외
    
- Identifiable 프로토콜의 id
    
    ```idris
    struct User: Identifiable {
        let id: String  // 이건 이름 바뀌면 안 됨
    }
    -> List(users)에서 SwiftUI는 내부적으로 user.id를 호출하기 때문에
    이 이름이 바뀌면 런타임 에러나 바인딩 오류 발생 가능성 있음.
    ```
    
- RawRepresentable
    
    RawRepresentable을 채택한 어떤 타입이든, rawValue 변수명은 고정이며 난독화 불
    
    ```idris
    class MyType: RawRepresentable {
        typealias RawValue = String
    
        let value: String  // 이름 바꾸면 컴파일 에러
    
        required init?(rawValue: String) {
            self.value = rawValue
        }
    } 
    ```
    
- CustomStringConvertible
    
    ```idris
    struct User: CustomStringConvertible {
        var description: String {  // 이름 바꾸면 컴파일 에러
            "Wrong"
        }
    }
    ```
    
    →`CustomStringConvertible` 프로토콜이 요구하는 이름이 사라지기 때문에 난독화 불가 변수