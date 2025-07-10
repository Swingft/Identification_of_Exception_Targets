- `RawRepresentable` (`String` or `Int` 기반 enum)
    
    ```idris
    	enum Page: String {
        case home
        case profile
    }
    -> String이 아니라 Int의 경우도 제외 항목
    ```
    
- `CaseIterable` 사용 시
    
    ```idris
    enum Action: String, CaseIterable {
        case create
        case edit
        case delete
    }
    
    ```
    
    ```idris
    사용처
    for action in Action.allCases {
        print(action.rawValue)
    }
    -> allcase는 case 이름 순서와 명칭을 그대로 보존
    -> UI, 로그, 테스트 등에 문자열 출력용으로 자주 사용됨
    ```
    
- `Codable` enum
    
    ```idris
    enum Status: String, Codable {
        case success
        case error
    }
    -> case 이름이 JSON에 직렬화 됨 --> 서버와 호환
    ```
    
- @objc enum (`@objc enum` or `@objc public enum`)
Objective-C와 호환을 위해 사용
    
    ```idris
    @objc enum Direction: Int {
        case left = 0
        case right = 1
    }
    
    ```
    
- View/Logic에서 `.id(\.enumCase)`와 연결되는 경우
    
    ```
    enum Tab: String, CaseIterable, Identifiable {
        case home
        case settings
    
        var id: String { rawValue }
    }
    
    ```
    
    ```idris
    사용처
    ForEach(tabs, id: \.id) { tab in
        Text(tab.rawValue)
    }
    
    ```
    
    case 이름이 그대로 .id에 사용되며 UI 뷰 식별자로 작동
    
    이름이 바뀌면 SwiftUI View 상태 추적이 깨짐