---
title: JS 关闭当前标签页
date: 2016-10-31 13:36:32
tags: FE
---

```js
function closeWindow() {
    var browserName = navigator.appName;
    var browserVer = parseInt(navigator.appVersion);

    if (browserName == "Microsoft Internet Explorer") {
        var ie7 = (document.all && !window.opera && window.XMLHttpRequest) ? true : false;
        if (ie7) {
            // This method is required to close a window without any prompt for IE7 & greater versions.
            window.open('', '_parent', '');
            window.close();
        } else {
            // This method is required to close a window without any prompt for IE6
            this.focus();
            self.opener = this;
            self.close();
        }
    } else {
        // For NON-IE Browsers except Firefox which doesnt support Auto Close
        try {
            this.focus();
            self.opener = this;
            self.close();
        } catch (e) {

        }

        // For Firefox
        try {
            window.location.replace("about:blank");
        } catch (e) {

        }
    }
}
```
