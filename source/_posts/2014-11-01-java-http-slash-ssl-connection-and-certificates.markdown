---
layout: post
title: "Java Http/SSL Connection and Certificates"
date: 2014-11-01 23:50
comments: true
categories: [Java]
---

在墙内想用Twitter API开发点东西真是痛苦。一般就两种手段，免费的Http代理和一些付费的VPN。VPN这里就不说了。如果要用代理的话，比如GoAgent，在Java中只需要在建立连接之前加入下面一段代码：

``` java java_https_goagent
        String host = "127.0.0.1";
        String port = "8087";
        System.setProperty("proxySet", "true");
        System.setProperty("https.proxyHost", host);
        System.setProperty("https.proxyPort", port);
```

但是如果这是你第一次这么干的话，一般会报以下这个错误：<!--more-->


``` java
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException:
PKIX path building failed
```
这是因为在建立SSL连接的时候因为安全认证的问题，需要在Client端导入证书`%JAVA_HOME%\lib\security\cacerts`
。进入到security目录下，直接用GoAgent里面的CA.crt导入就可以了。

``` bash
$ keytool -import -alias cacerts -keystore cacerts -file /path/to/CA.crt
```

导入的时候需要输入密码，直接输入"changeit"就行了，默认是这个。

这样一般能应付大部分的API requests了。但是在用到twitter的Streaming API的时候，访问stream.twitter.com总是一直返回443，并出现错误：

``` java
javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated
```

上网一查，这是一个典型的[没有服务器端证书的问题](http://stackoverflow.com/questions/11750413/ssl-peer-not-authenticated-error-with-httpclient-4-1)。一般有两种解决方案，一种就是导入证书，我安twitter doc的说明试了无数种证书，依然出现这种错误。纠结了很久，还是向[第二种方案](http://stackoverflow.com/questions/12095758/apache-httpclient-sslpeerunverifiedexception)妥协了，就是给原来HttpClient对象重新包装一下，让其强制接受任何证书。对于HttpClient 4.x以下的写法就是调用以下函数重新包装一下：

``` java
private static DefaultHttpClient getSecuredHttpClient(HttpClient httpClient) {
    final X509Certificate[] _AcceptedIssuers = new X509Certificate[]{};
    try {
        SSLContext ctx = SSLContext.getInstance("TLS");
        X509TrustManager tm = new X509TrustManager() {
            @Override
            public X509Certificate[] getAcceptedIssuers() {
                return _AcceptedIssuers;
            }

            @Override
            public void checkServerTrusted(X509Certificate[] chain,
                    String authType) throws CertificateException {
            }

            @Override
            public void checkClientTrusted(X509Certificate[] chain,
                    String authType) throws CertificateException {
            }
        };
        X509HostnameVerifier verifier = new X509HostnameVerifier() {
            @Override
            public void verify(String string, SSLSocket ssls) throws IOException {
            }

            @Override
            public void verify(String string, X509Certificate xc) throws SSLException {
            }

            @Override
            public void verify(String string, String[] strings, String[] strings1) throws SSLException {
            }

            @Override
            public boolean verify(String string, SSLSession ssls) {
                return true;
            }
        };
        ctx.init(null, new TrustManager[]{tm}, new SecureRandom());
        SSLSocketFactory ssf = new SSLSocketFactory(ctx,
                SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);
        ssf.setHostnameVerifier(verifier);
        ClientConnectionManager ccm = httpClient.getConnectionManager();
        SchemeRegistry sr = ccm.getSchemeRegistry();
        sr.register(new Scheme("https", 443, ssf));
        return new DefaultHttpClient(ccm, httpClient.getParams());
    } catch (Exception e) {
        System.out.println("=====:=====");
        e.printStackTrace();
    }
    return null;
}
```

而对于HttpClient 4.2以上的在建立HttpClient对象之前写成这样就行了：


``` java
SchemeRegistry schemeRegistry = null;
try {
    SSLContext sslContext = SSLContext.getInstance("SSL");

    // set up a TrustManager that trusts everything
    sslContext.init(null, new TrustManager[]{new X509TrustManager() {
            public X509Certificate[] getAcceptedIssuers() {
                System.out.println("getAcceptedIssuers =============");
                return null;
            }

            public void checkClientTrusted(X509Certificate[] certs,
                    String authType) {
                System.out.println("checkClientTrusted =============");
            }

            public void checkServerTrusted(X509Certificate[] certs,
                    String authType) {
                System.out.println("checkServerTrusted =============");
            }
        }}, new SecureRandom());
    SSLSocketFactory sf = new SSLSocketFactory(sslContext);
    Scheme httpsScheme = new Scheme("https", 443, sf);
    schemeRegistry = new SchemeRegistry();
    schemeRegistry.register(httpsScheme);
} catch (Exception e) {
    System.out.println("=====:=====");
    e.printStackTrace();
}

// apache HttpClient version >4.2 should use BasicClientConnectionManager
ClientConnectionManager cm = new BasicClientConnectionManager(schemeRegistry);
HttpClient httpClient = new DefaultHttpClient(cm);
```

这样就算是SSL依然返回443，还是能够建立起连接。


