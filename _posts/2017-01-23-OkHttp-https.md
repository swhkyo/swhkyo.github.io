---
layout: post
title:  "OkHttp Https"
date:   2017-01-23 11:06 +0800
categories: jekyll update
---

# OkHttp #

OkHttp是一个著名第三方网络库...

用法也很简单:

```Java

new OkHttpClient().newCall(request).enqueue(okHttpCallback);

```

当然，以上是一段很纯粹没有经过任何重构和思考就写下来的代码，只是举个例子，危险动作切勿模仿。

- 官方推荐是封装成一个单例...
- 回调的Callback最好封装一层...
- ......

不好意思，扯远了。

关于OkHttp的用法大家可以参考[**官方教程**](http://square.github.io/okhttp/)，很简单也很好用。

# OkHttp 的 Https #

如果要请求的URL是带证书验证的Https链接，在初始化OkHttpClient的时候，需要把我们用到的证书导入到一个TrustManager里面:

```Java
/**
* @return trustManager for SSL handshake.
* @throws GeneralSecurityException
*/
private X509TrustManager trustManagerForCertificates(KeyStore keyStore) throws GeneralSecurityException {
	// Use it to build an X509 trust manager.
	TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
	trustManagerFactory.init(keyStore);
	TrustManager[] trustManagers = trustManagerFactory.getTrustManagers();
	if (trustManagers.length != 1 || !(trustManagers[0] instanceof X509TrustManager)) {
		throw new IllegalStateException("Unexpected default trust managers:" + Arrays.toString(trustManagers));
	}
		
	return (X509TrustManager) trustManagers[0];
}
```

然后用这个自己证书的TrustManager去构建OkHttpClient:

```Java
sslSocketFactory = new WalSslSocketFactory(new TrustManager[] { trustManager });
mOkHttpClient = new OkHttpClient.Builder().sslSocketFactory(sslSocketFactory, trustManager).build();
```

**当然，打包的证书是要受信任的**

# Https的连接过程 #

Https连接的过程可能会抛出两种Exception，分别是：

- SSLHandShakeException
- SSLPeerUnverifiedException

>那么这两个Exception是从哪里抛出的呢？

看了两天源码，最后定位到：

```
okhttp3.internal.connection.RealConnection.java

connectTls(int readTimeout, int writeTimeout, ConnectionSpecSelector connectionSpecSelector)
```

connectTls方法中，一共做了三步的验证:

1. SSL握手
2. 验证Host Name是否正确
3. 验证证书是否受信于证书锁

**其中1会抛出SSLHandShakeException，2 or 3或抛出SSLPeerUnverifiedException**

## SSLHandShakeException ##

由源码中的
```Java
sslSocket.startHandshake();
```
抛出

SSLHandShakeException发生在https的握手阶段，证书错误或者过期等原因会抛出该异常。

## SSLPeerUnverifiedException ##

### 第一种情况是域名验证不通过 ###

源码中的表现为：

```Java
if (!address.hostnameVerifier().verify(address.url().host(), sslSocket.getSession())) {
    X509Certificate cert = (X509Certificate) unverifiedHandshake.peerCertificates().get(0);
    throw new SSLPeerUnverifiedException("Hostname " + address.url().host() + " not verified:"
        + "\n    certificate: " + CertificatePinner.pin(cert)
        + "\n    DN: " + cert.getSubjectDN().getName()
        + "\n    subjectAltNames: " + OkHostnameVerifier.allSubjectAltNames(cert));
}
```

>域名校验是什么？

当SSL握手成功之后，Client会获取到Server返回来的证书，通过对比Server返回的域名证书和我们请求的域名作对比，能判断服务器是否返回来正确的证书给我们。

具体怎么校验我们可以看到:

```Java
okhttp3.internal.tls.OkHostnameVerifier.java
```
首先我们从session中获取证书:

	Certificate[] certificates = session.getPeerCertificates();
	X509Certificate cer = (X509Certificate) certificates[0];

然后从cer中获取颁发给的IP列表和域名列表，每一个和我们请求的host name做对比，如果有一样的代表当前请求的host name是可信的。

### 第二种情况是证书锁校验不通过 ###

源码中的表现为：

	address.certificatePinner().check(address.url().host(), unverifiedHandshake.peerCertificates());

如果我们队OkHttpClient设置的证书锁为空，则是默认的不校验，每次校验都通过

```Java
okhttp3.CertificatePinner.java
if (pins.isEmpty()) return; // return 代表校验通过，否则会抛出Exception
```

>一般情况下，不建议我们设置证书锁


# 总结 #

嗯，没有总结。

因为还没有完全搞清楚，等我下次更新再说。