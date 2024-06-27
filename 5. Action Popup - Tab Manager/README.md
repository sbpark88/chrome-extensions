# 크롬 확장 프로그램 튜토리얼 5

### [탭 관리]

`action`의 `default_popup`을 통해 툴바 아이콘을 클릭했을 때 보이는 팝업 페이지를 열 수 있다. 팝업은 웹페이지와 비슷하지만
인라인 JavaScript 를 실행할 수 없다는 한 가지 예외가 있다.

이 예제에서는 [Tabs API][JavaScript APIs - tabs] 를 사용해 브라우저의 탭 정보를 읽고 Popup 페이지에 링크를 생성하는 것과
[Chrome Tab Groups API][Chrome Extensions - tabGroups] 를 사용해 탭을 그룹화 하는 것을 구현한다.

[탭 관리]: https://developer.chrome.com/docs/extensions/get-started/tutorial/popup-tabs-manager?hl=ko
[JavaScript APIs - tabs]: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/tabs
[Chrome Extensions - tabGroups]: https://developer.chrome.com/docs/extensions/reference/api/tabGroups?hl=ko
