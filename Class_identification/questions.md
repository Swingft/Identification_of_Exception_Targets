2025/07/09 20:18

현재 클래스쪽 난독화 제외 대상을 식별하는 과정에서 확신이 서지 않는 부분이 있어 의견을 여쭤보고자 합니다.

예를 들어 UNNotificationServiceExtension, AppDelegate 등 일부 클래스는 시스템이나 SDK가 Info.plist 내 문자열로 지정된 클래스명을 기반으로 런타임에 로딩하는 구조로 이해하고 있습니다.

따라서 해당 클래스들에 난독화를 적용하려면, Info.plist 내 클래스명도 함께 난독화된 이름으로 정확히 수정되어야 정상적으로 동작할 것으로 이해했습니다.


고민하는 지점

Info.plist까지 함께 수정하면,
외부에서 참조되는 클래스도 난독화가 가능해져 적용 범위가 훨씬 넓어지는 장점이 있는 반면,
잘못 변경하거나 SDK가 내부적으로 클래스명을 문자열로 직접 참조(NSClassFromString 등)하는 경우
plist만 수정해도 런타임 충돌이 발생할 수 있다는 점이 걱정입니다.

참조할만한 소스코드 난독화 솔루션이 없어서 SwiftShield를 확인해봐도 Info.plist는 건드리지 않는 것으로 알고 있으며,
문제는 Facebook Audience Network 등 일부 SDK 문서를 보면, Info.plist를 확인하지 않고, SDK내부에서 특정 클래스 이름을 명시적으로 정의해둘 것을 가정하는 사례도 있습니다.
https://developers.facebook.com/docs/audience-network/setting-up/ad-setup/ios/banner

이런 이유로 Info.plist를 수정하는 접근이 오히려 예기치 않은 충돌 가능성을 높이는 것은 아닌지 고민되고 있습니다.

1.	Info.plist까지 포함해 난독화 이름을 정확히 반영하는 방식이 현실적으로 안전할지
2.	혹은 Info.plist는 그대로 두고 일부 클래스만 제외하는 방식이 더 현실적인지

이 두 가지를 여쭤보고 싶습니다.
