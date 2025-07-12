| kind | attributes | modifiers | typeContext | declContext |
| --- | --- | --- | --- | --- |
| EnumDecl |   |  | `String` | `enum` |
| EnumDecl |  |  | `int` | `enum`  |
| EnumDecl |  |  | `Double` | `enum`  |
| EnumDecl |  |  | `UInt` | `enum`  |
| EnumDecl |  |  | `Float` | `enum`  |
| EnumDecl |  |  | `Identifiable` |  |
| EnumDecl | `@objc` |  |  |  |
| EnumDecl | `@objcMembers` |  |  |  |
| EnumDecl |  |  | `CaseIterable` |  |
| EnumDecl |  |  | `Encodable` |  |
| EnumDecl |  |  | `Decodable` |  |
| EnumDecl |  |  | `Codable` |  |
- enum의 경우 enum 이름 난독화, case 난독화로 경우가 두가지인데
    - 위의 경우는 이름과 case모두 난독화 하면 안되는 경우이다
    - Rawvalue의 경우는 Character, UInt, Double, Float 도 있음
    - 결국 내부적으로 (private, internal) switch, if-else, 비교용으로 사용할 때는 Case, 이름 다 난독화

```
{
  "exclude_enum_decls": [
    {
      "kind": "EnumDecl",
      "inheritedTypes": ["String", "Int", "UInt", "Double", "Float", "Character"]
    },
    {
      "kind": "EnumDecl",
      "inheritedTypes": ["Identifiable"]
    },
    {
      "kind": "EnumDecl",
      "attributes": ["@objc"]
    },
    {
      "kind": "EnumDecl",
      "attributes": ["@objcMembers"]
    },
    {
      "kind": "EnumDecl",
      "inheritedTypes": ["CaseIterable"]
    },
    {
      "kind": "EnumDecl",
      "inheritedTypes": ["Encodable"]
    },
    {
      "kind": "EnumDecl",
      "inheritedTypes": ["Decodable"]
    },
    {
      "kind": "EnumDecl",
      "inheritedTypes": ["Codable"]
    }
  ]
}

```
