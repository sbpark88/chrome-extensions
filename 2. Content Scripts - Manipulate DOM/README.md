# 크롬 확장 프로그램 튜토리얼 2

### [모든 페이지에서 스크립트 실행]

`https://developer.chrome.com/docs/extensions/*`, `https://developer.chrome.com/docs/webstore/*`
하위 페이지를 새로고침 할 경우, 문서를 읽는데 소요되는 시간을 DOM 에 표시하는 예제를 통해 확장 프로그램이 `DOM 을 읽고 조작`하는
방법을 알아본다. 이것은 `manifest.json`에서 `content_scripts` 항목을 통해 DOM 접근을 다룬다.

참고로, Service Worker 를 사용하지 않기 때문에 다른 글을 읽을 때 발생하는 리플로우에 대해서는 작동하지 않는다.
따라서 새로고침을 해야 결과를 확인할 수 있다.

[모든 페이지에서 스크립트 실행]: https://developer.chrome.com/docs/extensions/get-started/tutorial/scripts-on-every-tab?hl=ko

---

# Tutorial: Run scripts on every page

This sample demonstrates how to run scripts on any Chrome extension and Chrome Web Store documentation page using an extension called _Reading Time_.

## Overview

This sample demonstrates how developers can use content scripts which employ Document Object Models to read and change the content of a page. In this instance, the extension checks to find an article element, counts all the words inside of it, and then creates a paragraph that estimates the total reading time for that article.

## Running this extension

1. Clone this repository.
2. Load this directory in Chrome as an [unpacked extension](https://developer.chrome.com/docs/extensions/mv3/getstarted/development-basics/#load-unpacked).
3. Navigate to a Chrome extension or Chrome Webstore documentation page with an article element. [Here](https://developer.chrome.com/docs/webstore/publish) is an example of a webpage with an article element.
4. The extension will provide an estimated reading time for the text in that article element. Here is the [link](https://developer.chrome.com/docs/extensions/get-started/tutorial/scripts-on-every-tab) for the full instructions.
