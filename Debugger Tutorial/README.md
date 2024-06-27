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

[디버그 확장 프로그램]: https://developer.chrome.com/docs/extensions/get-started/tutorial/debug?hl=ko

---

## Context

This is a simple sample extension used in the [Debugging extensions][1] tutorial. The purpose of this extension is to demonstrate how to debug different extension components. This extension changes the background color of active tab.

## Running this extension

1. Clone this repository.
2. Load this directory in Chrome as an [unpacked extension][2].
3. Go to https://developer.chrome.com/docs/extensions.
4. Open the extension popup and click on the color.
5. The background color will change.

[1]: https://developer.chrome.com/docs/extensions/mv3/tut_debugging/
[2]: https://developer.chrome.com/docs/extensions/mv3/getstarted/development-basics/#load-unpacked
