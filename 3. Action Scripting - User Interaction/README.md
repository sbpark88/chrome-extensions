# 크롬 확장 프로그램 튜토리얼 3

### [활성 탭에 스크립트 삽입]

CSS 를 사용해 '읽기 도구(focus-mode)' 효과를 내는 기능을 구현하고 사용자의 클릭에 의해 작동하도록 하는 `action`과
이 `action`을 단축키로 활성화 시키는 방법에 대해 다룬다.

이 예제는 `https://developer.chrome.com/docs/extensions/*`, `https://developer.chrome.com/docs/webstore/*`
페이지에 있을 경우 사용자의 아이콘 클릭에 의해 focus-mode 가 활성화/비활성화 되며 CSS 가 적용된다.

우선, `action`을 사용하기 위해 `manifest.json`에 `action`을 추가해야하며, 이것은 `default_icon`,
`default_title`(툴팁에 보임), `default_popup`을 포함할 수 있으며, 모두 optional 이다. 하지만 `action`을
사용하지 않더라도 확장 프로그램은 툴바에 계속 표시되기 때문에 `action`와 `default_icon`은 포함하는 것이 권장된다.

그리고 DOM 을 조작하는 것이 아니므로 `manifest.json`에 `content_scripts`가 아닌 `background`의
`service_worker`를 통해 코드를 작성할 것이다.

그리고 사용자의 개인 정보를 보호하고, **권한 경고**를 트리거하지 않도록 `manifest.json`에 `permissions`를 추가하고,
배열에 "activeTab"을 추가한다. 또한 `content_scripts`가 아니므로 DOM 을 직접 다루지 못하는 대신
[Scripting API][JavaScript APIs - scripting]를 통해 CSS 스타일 시트를 삽입하고 삭제할 것이다.
따라서 `permissions`에 "scripting" 권한을 추가한다.

이제 `Service Worker`에 등록할 스크립트 작성을 해야하는 데, `chrome.runtime.onInstalled`은 `window.onload`와
유사하게 프로그램이 설치된 직후의 행동을 정의하고, `chrome.action.onClicked.addListener`는 사용자의 클릭 또는
단축키 입력에 의해 `action`이 작동했을 때의 동작을 정의한다.

[활성 탭에 스크립트 삽입]: https://developer.chrome.com/docs/extensions/get-started/tutorial/scripts-activetab?hl=ko
[JavaScript APIs - scripting]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/scripting
