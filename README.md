# 확장 프로그램에서 주로 사용하는 것은 브라우저에 Add-on 으로 추가된 API 다.

크롬에서만 사용 가능한 API 도 일부 있지만 대부분의 API 는 브라우저가 공통으로 지원하는 `Add-ons`다. 따라서

[Browser Extensions - JavaScript APIs] 와 [Chrome Extensions - API Reference(Manifest V3)] 를
참고하면 대부분의 브라우저 기능을 찾을 수 있으니 참고하는 것이 좋다.

프로젝트 구성은 여러 방법이 있으나 Safari Extensions 까지 고려한다면 작업을 두 번 하지 않기 위해
아래와 같이 프로젝트를 구성하는 것이 좋다.

### Manifest

웹 확장 프로그램은 웹 콘텐츠 문서가 아닌 브라우저에서 작동하며, 브라우저가 제공하는 `JavaScript API`를 사용하는
것이기 때문에 어떤 API 를 사용할지를 정의하고, 권한을 요청하는 등 웹 확장 프로그램의 구조를 정의하는 파일이다.

### Action

툴바 아이콘을 제어하며, `popup.html`, `popup.js`, `popup.css` 파일 3개를 만들어 관리한다. 액션의
팝업은 웹페이지와 비슷하지만 인라인 JavaScript 를 실행할 수 없다는 한 가지 예외가 있다.

툴바 아이콘을 우클릭 후 `Inspect(검사)` 버튼을 클릭하면 팝업의 개발자 도구를 열 수 있다.

### Content Scripts

DOM 조작을 제어하며, `content.js` 파일을 만들어 관리한다. `content_scripts`에 개별 스크립트 파일을
불러와 적용시킨다. 이때 주의해야 할 것이, HTML 문서가 로딩된 후 한 번 실행되기 때문에 React 처럼 일부만
새롭게 랜더링 하는 경우 재실행이 안 된다. 만약 이런 상황에서 매번 새롭게 실행되길 원하면 로직을 구현해야한다.

툴바 아이콘의 Inspect 로 나오는 개발자 도구는 `popup.html`에서 발생하는 에러를 보여준다.
반면 Content Scripts 는 메인 웹 콘텐츠 페이지에서 실행되기 때문에 메인 페이지의 개발자 도구에서 오류를 확인해야한다.

### Background - Service Worker

DOM 조작이 아닌 모든 백그라운드 작업 및 브라우저 API 를 제어하며, `background.js` 파일을 만들어 관리한다.
`background`의 `Service Worker`에 단일 스크립트 파일을 불러와 적용시키며, 여러 스크립트를 적용시킬 경우,
`import`로 파일 자체를 불러올 수 있다. 웹 페이지 메인 콘텐츠 스크립트의 스레드를 사용하지 않기 때문에 블로킹 당하지
않으므로 독립적이다. 알람, 브라우저 이벤트, 비동기 작업과 같은 것에 사용된다.

`Window` 객체에 접근할 수 없으며 모든 Chrome API 이벤트 리스너 및 메서드는 Service Worker 의 30초 종료
타이머를 다시 시작한다.

Service Worker 는 `chrome-extension://YOUR_EXTENSION_ID/manifest.json` 주소에서 Service Worker 에
접근해 `start`, `stop`을 컨트롤 할 수 있을 뿐 아니라 서비스 워커 코드를 변경한 경우 `Update` 또는 `skipWaiting`을
클릭해 변경사항을 즉시 적용할 수 있다.

### Content Scripts 와 Background Service Worker 의 차이

| 구분      | `content_scripts`                                   | `background`                                       |
| --------- | --------------------------------------------------- | -------------------------------------------------- |
| 실행 위치 | 웹 페이지 콘텐츠                                    | 브라우저 백그라운드                                |
| 기능      | DOM 조작, CSS 변경, 비동기 요청, 사용자 인터랙션    | 백그라운드, 브라우저 이벤트, 확장 프로그램 로직    |
| 사용 예   | 광고 차단, 가격 비교, 콘텐츠 편집, 로그인 정보 저장 | 백그라운드 동기화, 알림 기능, 통계 추적, 자동 작업 |

### Content Scripts 와 Background Service Worker 의 공통점

Contexnt Script 와 Background Service Worker 둘 다 `export/import`를 사용한 값이나 함수의 사용이
불가능하다. 만약, 이렇게 분리개발을 하려면 Vanilla JS 자체로는 불가능하고, Webpack 이나 Parcel 과 같은 번들러로
빌드한 결과물을 사용해야한다. 따라서 권장하는 방법은 다음과 같다.

1. 기본 구조에 맞춰 각 기능은 하나의 스크립트에서 모두 구현되도록 Vanilla JS 로 구현한다.
2. NPM 을 이용해 [chrome-types](https://www.npmjs.com/package/chrome-types) 를 설치하고
   TypeScript 를 사용해 자동완성의 도움을 받아 1번과 같은 방법으로 하나의 스크립트에서 모두 구현되도록 한다.
3. 각 기능을 단일 파일에 구현하기에 코드가 너무 클 경우 단순히 번들러를 사용하기 보다는 처음부터 React 로 구현하는
   편이 더 낫다. [Chrome Extensions with React TypeScript] 를 참고하도록 한다.

번들러를 사용하지 않더라도 파일을 분할할 수는 있다. 2개, 3개의 기능이 포함될 경우, 각 기능별로 스크립트 파일을 나누되,
하나의 기능은 각각의 단일 스크립트 파일 안에서 완벽히 작동해야한다. 여러 개로 나눠 작성한 스크립트 파일을 적용하기 위해
Content Scripts 는 `manifest.json`의 `content_scripts`에 여러 스크립트를 등록하고, Service Worker 는
`import`를 사용해 파일 자체를 불러온다(값이나 함수를 export/import 할 수 있는 것은 아니다).

# Module Scripts

Content Scripts, Service Worker, Action 의 `default_popup` HTML 페이지에서 불러오는 스크립트 모두
`export/import`를 사용한 값이나 함수를 불러오는 로직은 번들러 없이는 사용할 수 없다. 하지만 Content Scripts 와
달리 Service Worker 와 Action 은 `Module Scripts`를 사용할 수 있다.

`export/import`도 사용할 수 없는데 이게 무슨 의미가 있을가? 싶지만, `Module Scripts`로 정의가 가능하다는 것은
`@main`이 표시된 것과 같은 의미를 갖는다. 즉, `async`로 감싼 context 가 아니더라도 `await`을 사용할 수 있다!!

# 사용자 데이터 보호

브라우저의 JavaScript API 를 사용하기 위해 권한을 요청하는 것과 마찬가지로, 사용자 개인 정보를 보호하기 위해 코드가
작동하는 범위를 제한시킬 수 있다.

- 활성 탭으로 제한하기

```json
{
  "permissions": ["activeTab"]
}
```

- 특정 호스트로 제한하기

```json
{
  "host_permissions": ["https://developer.chrome.com/*"]
}
```

- Content Scripts 제한하기

```json
{
  "content_scripts": [
    {
      "js": ["scripts/content.js"],
      "matches": [
        "https://developer.chrome.com/docs/extensions/*",
        "https://developer.chrome.com/docs/webstore/*"
      ]
    }
  ]
}
```

# 선택적 권한

선택적 권한은 사용자의 개인 정보를 보호할 뿐 아니라, `권한 경고`를 트리거하지 않으므로 사용자에게 불안감을 주지 않는 장점이 있다.

- 활성 탭으로 제한하기

```json
{
  "permissions": ["activeTab"]
}
```

- Optional Permissions, Optional Host Permissions 구현하기

```json
{
  "optional_permissions": ["tabs"],
  "optional_host_permissions": ["https://www.google.com/"]
}
```

[선택적 권한 구현](https://developer.chrome.com/docs/extensions/reference/api/permissions?hl=ko#implement_optional_permissions)
를 참고한다.

---

# 크롬 확장 프로그램 튜토리얼 1

### [Hello World]

아무런 고급 기능을 사용하지 않은 최소한의 hello world 샘플로, `manifest.json`의 필수 요소를 작성하는 방법과,
`HTML`과 `JavaScript`가 어떻게 확장 프로그램의 페이지를 만들어내는지를 알아본다.

---

# 크롬 확장 프로그램 튜토리얼 2

### [모든 페이지에서 스크립트 실행]

`https://developer.chrome.com/docs/extensions/*`, `https://developer.chrome.com/docs/webstore/*`
하위 페이지를 새로고침 할 경우, 문서를 읽는데 소요되는 시간을 DOM 에 표시하는 예제를 통해 확장 프로그램이 `DOM 을 읽고 조작`하는
방법을 알아본다. 이것은 `manifest.json`에서 `content_scripts` 항목을 통해 DOM 접근을 다룬다.

참고로, Service Worker 를 사용하지 않기 때문에 다른 글을 읽을 때 발생하는 리플로우에 대해서는 작동하지 않는다.
따라서 새로고침을 해야 결과를 확인할 수 있다.

---

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

---

# 크롬 확장 프로그램 튜토리얼 4

### [서비스 워커로 이벤트 처리]

이 예제에는 `Service Worker`를 사용하는 데, `sw-omnibox`는 주소 표시줄을 이용해 특정 키워드를 입력 후 해당 엔진 안에서
검색할 수 있는 기능이고, `sw-tips`는 구글 확장에서 팁을 받아와 그 중 하나를 골라 알람을 보내고, `content_scripts`가
`sw-tips`가 받아온 팁을 팝업시키는 버튼을 본문에 추가한다.

여기서 알아야 할 특징들은 다음과 같다.

- 모든 Chrome API 이벤트 리스너 및 메서드는 Service Worker 의 30초 종료 타이머를 다시 시작한다. Service Worker 는
  `lifecycle` 동안만 유효하고 비활성화 상태에 들어가므로 Service Worker 세션 간에 상태를 유지하기 위해서는 데이터를 저장해야한다.
  하지만 Service Worker 는 `Window` 객체에 접근할 수 없으므로 `window.localStorage`를 사용할 수 없다.  
  대신 [Storage API][JavaScript APIs - storage]를 사용해 `chrome.storage.local.set()`,
  `chrome.storage.local.get()` 메서드로 데이터를 저장하고 상태 관리를 할 수 있다(이것을 사용하기 위해 `permissions`에
  "storage" 권한을 추가한다).
- `window.onload`를 사용하듯 Service Worker 는 `chrome.runtime.onInstalled.addListener`를 사용해 초기 설정을
  할 수 있다.
- Omnibox 는 [Omnibox API][JavaScript APIs - omnibox]를 사용해 `chrome.omnibox.onInputChanged.addListener`,
  `chrome.omnibox.onInputEntered.addListener`, 이벤트와 `chrome.omnibox.setDefaultSuggestion` 메서드로 입력을
  감지하고 제안을 하도록 구현했다.
- Service Worker 에서 `setTimeout`, `setInterval` API 는 Service Worker 가 종료될 때 스케줄러 타이머를 취소하기
  때문에 실패할 수 있다. 대신 [Alarm API][JavaScript APIs - alarms]를 사용해 처리한다.  
  `chrome.alarms.get`, `chrome.alarms.create` 메서드와 `chrome.alarms.onAlarm.addListener` 이벤트를
  사용했다(이것을 사용하기 위해 `permissions`에 "alarm" 권한을 추가한다).
- 확장 프로그램은 Content Script 로 DOM 에 접근하고, Service Worker 로 브라우저의 백그라운드 "알람, 메시지, 스토어,
  활성 탭 관리 및 스크립트 삽입/제거"와 같은 DOM 이외의 것들을 처리한다. Content Script 가 모듈을 사용하지 않기 때문에
  데이터 교환을 위해 [Runtime API][JavaScript APIs - runtime] 의 메시지 기능을 사용한다. 이를 위해 여기서 사용한 API
  `chrome.runtime.onMessage.addListener` 이벤트와 `chrome.runtime.sendMessage` 메서드를 사용했다.

---

# 크롬 확장 프로그램 튜토리얼 5

### [탭 관리]

`action`의 `default_popup`을 통해 툴바 아이콘을 클릭했을 때 보이는 팝업 페이지를 열 수 있다. 팝업은 웹페이지와 비슷하지만
인라인 JavaScript 를 실행할 수 없다는 한 가지 예외가 있다.

이 예제에서는 [Tabs API][JavaScript APIs - tabs] 를 사용해 브라우저의 탭 정보를 읽고 Popup 페이지에 링크를 생성하는 것과
[Chrome Tab Groups API][Chrome Extensions - tabGroups] 를 사용해 탭을 그룹화 하는 것을 구현한다.

---

# 크롬 확장 프로그램 튜토리얼 6

### [디버그 확장 프로그램]

각 기능의 디버깅 방법을 설명하는 튜토리얼이다.

- Manifest.json 및 프로젝트 구조: 확장 프로그램 대시보드 `chrome://extensions/`에서 각 확장 프로그램
  카드 안에 `Errors`버튼을 통해 문제를 확인할 수 있다.
- Action: 툴바 아이콘을 우클릭 후 `Inspect(검사)` 버튼을 클릭하면 팝업의 개발자 도구를 열 수 있다.
- Content Scripts: 툴바 아이콘의 Inspect 로 나오는 개발자 도구는 `popup.html`에서 발생하는 에러를 보여준다.
  반면 Content Scripts 는 메인 웹 콘텐츠 페이지에서 실행되기 때문에 메인 페이지의 개발자 도구에서 오류를 확인해야한다.
- Background - Service Worker: 확장 프로그램 대시보드에서 `ID`를 구한 후
  `chrome-extension://YOUR_EXTENSION_ID/manifest.json` 주소로 접근해 개별 확장 프로그램 페이지에 접근할 수
  있다. 가볍게 툴바 아이콘 우클릭의 `Inspect` 클릭을 통한 접근도 가능하지만, 이렇게 직접 주소로 접근해서 열면 페이지가
  닫히지 않고 계속 열린 채로 디버깅 할 수 있다는 장점이 있다. 이 상태로 Service Worker 에 접근해 `start`, `stop`을
  컨트롤 할 수 있을 뿐 아니라 서비스 워커 코드를 변경한 경우 `Update` 또는 `skipWaiting`을 클릭해 변경사항을 즉시
  적용할 수 있다.

[Hello World]: https://developer.chrome.com/docs/extensions/get-started/tutorial/hello-world?hl=ko
[모든 페이지에서 스크립트 실행]: https://developer.chrome.com/docs/extensions/get-started/tutorial/scripts-on-every-tab?hl=ko
[활성 탭에 스크립트 삽입]: https://developer.chrome.com/docs/extensions/get-started/tutorial/scripts-activetab?hl=ko
[서비스 워커로 이벤트 처리]: https://developer.chrome.com/docs/extensions/get-started/tutorial/service-worker-events?hl=ko
[탭 관리]: https://developer.chrome.com/docs/extensions/get-started/tutorial/popup-tabs-manager?hl=ko
[디버그 확장 프로그램]: https://developer.chrome.com/docs/extensions/get-started/tutorial/debug?hl=ko
[JavaScript APIs - action]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/action
[JavaScript APIs - alarms]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/alarms
[JavaScript APIs - clipboard]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/clipboard
[JavaScript APIs - omnibox]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/omnibox
[JavaScript APIs - permissions]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/permissions
[JavaScript APIs - runtime]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/runtime
[JavaScript APIs - scripting]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/scripting
[JavaScript APIs - storage]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/storage
[JavaScript APIs - tabs]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/tabs
[Chrome Extensions - tabGroups]: https://developer.chrome.com/docs/extensions/reference/api/tabGroups?hl=ko
[Browser Extensions - JavaScript APIs]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Browser_support_for_JavaScript_APIs
[Chrome Extensions - API Reference(Manifest V3)]: https://developer.chrome.com/docs/extensions/reference/api?hl=ko
[Chrome Extensions with React TypeScript]: https://medium.com/@ahsan-ali-mansoor/creating-a-chrome-extension-with-react-and-typescript-1378cc9f29ef
