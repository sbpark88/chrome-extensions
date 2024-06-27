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

[서비스 워커로 이벤트 처리]: https://developer.chrome.com/docs/extensions/get-started/tutorial/service-worker-events?hl=ko
[JavaScript APIs - alarms]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/alarms
[JavaScript APIs - omnibox]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/omnibox
[JavaScript APIs - runtime]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/runtime
[JavaScript APIs - storage]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/storage

---

# Tutorial: Handling events

This sample demonstrates how to handle events with service workers.

## Overview

The extension built allows users to quickly navigate to Chrome API reference pages using the omnibox.

The events from the following APIs have been handled: `chrome.runtime`, `chrome.omnibox` and `chrome.alarms`.

The complete tutorial is available [here](https://developer.chrome.com/docs/extensions/get-started/tutorial/service-worker-events).

## Running this extension

1. Clone this repository.
2. Load this directory in Chrome as an [unpacked extension](https://developer.chrome.com/docs/extensions/mv3/getstarted/development-basics/#load-unpacked).
3. Type "api" in the omnibox followed by tab or space and select a suggestion.
4. Click on "Tip" button in the navigation bar.
