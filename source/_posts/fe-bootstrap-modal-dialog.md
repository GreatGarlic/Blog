---
title: Bootstrap 模态对话框
date: 2016-05-15 21:28:33
tags: FE
---

Bootstrap 的对话框用起来比较繁复，例如

```html
<div class="modal fade">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal" aria-hidden="true">×</button>
        <h4 class="modal-title">Modal title</h4>
      </div>
      <div class="modal-body">
        <p>One fine body…</p>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
        <button type="button" class="btn btn-primary">Save changes</button>
      </div>
    </div><!-- /.modal-content -->
  </div><!-- /.modal-dialog -->
</div><!-- /.modal -->
```

使用 [`bootstrap-dialog`](https://nakupanda.github.io/bootstrap3-dialog) 达到相同的效果只需要 `BootstrapDialog.alert('I want banana!')`，不用写 HTML 代码，插件里已经写了。

<!--more-->

## 常用对话框

```js
BootstrapDialog.alert('Hi Apple!');
BootstrapDialog.confirm('Hi Apple, are you sure?');
```

```js
BootstrapDialog.confirm({
    title: 'WARNING',
    message: 'Warning! Drop your banana?',
    callback: function(result) {
        // result will be true if button was click, while it will be false if users close the dialog directly.
        if(result) {
            alert('Yup.');
        } else {
            alert('Nope.');
        }
    }
});
```

```js
BootstrapDialog.confirm({
    title: '注意',
    message: '确定要删除吗？',
    type: BootstrapDialog.TYPE_DANGER,
    btnCancelLabel: '取消',
    btnOKLabel: '确定',
    callback: function(result) {
        if(result) {
        }
    }
});
```

```js
BootstrapDialog.show({
    title: 'Gut',
    message: 'Hello world!',
    type: BootstrapDialog.TYPE_WARNING,
    onshow: function(dialogRef) {
        alert('Dialog is popping up, its message is ' + dialogRef.getMessage());
    },
    onshown: function(dialogRef) {
        alert('Dialog is popped up.');
    },
    onhide: function(dialogRef) {
        alert('Dialog is popping down, its message is ' + dialogRef.getMessage());
    },
    onhidden: function(dialogRef) {
        alert('Dialog is popped down.');
    }
});
```

```js
BootstrapDialog.show({
    title: '选择文件夹',
    message: $('#directories').show(),
    buttons: [{
        label: '取消',
        action: function(dialogRef) {
            dialogRef.close();
        }
    }, {
        label: '确定',
        cssClass: 'btn-primary',
        action: function(dialogRef) {
            dialogRef.close();
        }
    }]
});
```

更多例子和文档请参考 [bootstrap-dialog](https://nakupanda.github.io/bootstrap3-dialog)

