---
layout: post
title:  "Android dialog 'show' method"
date:   2017-08-16 09:09:54 +0800
categories: jekyll update
---

# 关于Android dialog 'show' 方法的一次记录  #

事情是这样发生的···

在一个月黑风高的夜晚，我正在封装一个BaseAcvitity，其中有一个功能是弹出加载提示框，就像下图这样：

![Loading Dialog](../res/2017-08-16/loading-dialog.png)

>那当然，还有代码入如下：

```Java

ProgressDialog mLoadingDialog;

protected void showLoadingDialog(String title, String content) {
    if (mLoadingDialog == null) {
        mLoadingDialog = new ProgressDialog(this);
    }

    mLoadingDialog.show(this, title, content);
}

protected void hideLoadingDialog() {
    if (mLoadingDialog != null) {
        mLoadingDialog.dismiss();
    }
}

```

>然后还有使用的代码：

```Java

showLoadingDialog("Requesting...", "Get result from Server, please wait...")

HttpClient.get(url, new HttpRequestCallback() {
   
        Override
        onSuccess(JSONObject result) {
            hideLoadingDialog();
        }

        Override
        onFailure(int failureCode, String failureMsg) {
            hideLoadingDialog();
        }

});

```

>然而，这个Loading Dialog却从来不会消失...

# 为什么呢？ #

为什么调用dismiss()方法一点效果都没有呢？？？

走进源码看看···

```Java

public static ProgressDialog show(Context context, CharSequence title, CharSequence message) {
    return show(context, title, message, false);
}

public void show() {
    // Do show dialog···
}

```

好了现在事情就清楚了

```Java
show(context, title, message) 
```
>是一个静态的函数，会返回一个ProgressDialog的实例。

```Java
show()
```

>是一个成员函数，返回值是void

调用静态方法show(context, title, message)会返回一个ProgressDialog的实例，后续要直接在返回的的实例上操作：

```Java

mLoadingDialog = mLoadingDialog.show(context, title, message);

```

但是这样写呢，从源码上看会每次都__new__一个ProgressDialog，你知道__new__操作总是要谨慎使用的...

# 正确的使用 #

所以正确的封装方法应该像下面这样：

```Java

ProgressDialog mLoadingDialog;

protected void showLoadingDialog(String title, String content) {
    if (mLoadingDialog == null) {
        mLoadingDialog = new ProgressDialog(this);
    }

    mLoadingDialog.setTitle(title);
    mLoadingDialog.setMessage(content);
    mLoadingDialog.show();
}

protected void hideLoadingDialog() {
    if (mLoadingDialog != null) {
        mLoadingDialog.dismiss();
    }
}

```

# 后记 #

>踩了个小坑，记录一下，知识要慢慢积累