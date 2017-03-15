## Https证书

​	经验证不论使用Android原生的HttpsURLConection进行Https通信，还是利用Okhttp（retrofit依赖于Okhhttp），如果用的是CA机构颁发的证书，则不需要进行相关配置，反之，如果服务端利用的是自定义证书，则需要进行证书本地相关配置 自定义信任链 实现TrustManager和HostnameVerifier中的方法，预埋证书 

- CA颁发证书

 ​不需要进行配置 利用Android本身的HttpsURLConnection进行通信或者okhttp都可以

- 自前证书

  ​	需要进行证书本地相关配置 自定义信任链 实现TrustManager和HostnameVerifier中的方法，预埋证书等操作

  - 弊端及建议

    阿里聚安全的漏洞扫描器发现，很多APP都存在HTTPS使用不当的风险。正确使用HTTPS能有效抵御在用户设备上安装证书进行中间人攻击和SSL Strip攻击。

    但是上述方法都需要在客户端中预埋证书文件，或者将证书硬编码写在代码中，如果服务器端证书到期或者因为泄露等其他原因需要更换证书，也就必须强制用户进行客户端升级，体验效果不好。阿里聚安全推出了一个能完美解决这个问题的安全组件。APP开发者只需要将公钥放在安全组件中，安全组件的动态密钥功能可以实现公钥的动态升级。

    另外正确使用HTTPS并非完全能够防住客户端的Hook分析修改，要想保证通信安全，也需要依靠其他方法，比如重要信息在交给HTTPS传输之前进行加密，另外实现客户端请求的签名处理，保证客户端与服务端通信请求不被伪造。目前阿里聚安全的安全组件已经具备以上所有功能，此外还有安全存储、模拟器检测，人机识别等功能。安全组件还具有实时更新客户端模块的功能，保证攻防对抗强度。


- [AndroidHttps详解](http://blog.csdn.net/wzy_1988/article/details/51143499)
- [Android安全开发](http://www.cnblogs.com/alisecurity/p/5939336.html)


## Sample

```
  /**
         *以下例子是进行了httpsURLConnection的测试 针对HttpsURLConnection 访问https请求是可以的 而访问
         *http请求则会报错 HttpsURLConnection是不能转换为HttpURLConnection的 而访问不信任的证书同样会出现错误
         *javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found 
         * 综上所述 也证明了上看的结论
         */

        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
//                    URL url = new URL("https://www.alipay.com/");
                    URL url = new URL("https://kyfw.12306.cn/otn/");
//                    URL url = new URL("ha/");

                    HttpsURLConnection urlConnection = (HttpsURLConnection) url.openConnection();
                    InputStream input = urlConnection.getInputStream();

                    BufferedReader reader = new BufferedReader(new InputStreamReader(input, "UTF-8"));
                    StringBuffer result = new StringBuffer();
                    String line = "";
                    while ((line = reader.readLine()) != null) {
                        result.append(line);
                    }
                    System.out.println(result);
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        });
        thread.start();
```

### WebView的https安全

- WebView 漏洞

  ​	Android WebView组件加载网页发生证书认证错误时，会调用WebViewClient类的onReceivedSslError方法，如果该方法实现调用了handler.proceed()来忽略该证书错误，则会受到中间人攻击的威胁，可能导致隐私泄露。

  ​	目前很多应用都用webview加载H5页面，如果服务端采用的是可信CA颁发的证书，在 webView.setWebViewClient(webviewClient) 时重载 WebViewClient的onReceivedSslError() ，如果出现证书错误，直接调用handler.proceed()会忽略错误继续加载证书有问题的页面，如果调用handler.cancel()可以终止加载证书有问题的页面，证书出现问题了，可以提示用户风险，让用户选择加载与否，如果是需要安全级别比较高，可以直接终止页面加载，提示用户网络环境有风险：

![提示代码](http://ww4.sinaimg.cn/large/a15b4afegw1f8kuitdbcgj20lh0osq6a?_=5939336)

如果webview加载https需要强校验服务端证书，可以在 onPageStarted() 中用 HttpsURLConnection 强校验证书的方式来校验服务端证书，如果校验不通过停止加载网页。当然这样会拖慢网页的加载速度，需要进一步优化，具体优化的办法不在本次讨论范围，这里也不详细讲解了。

**WebView同样的如果是CA证书，则不需要进行校验，但是需要重载 WebViewClient的onReceivedSslError() **

- [WebView校验SSL证书](http://www.cnblogs.com/sslwork/p/6193258.html)
- [用WebView访问证书有问题的SSL网页](http://www.tuicool.com/articles/MNN3Qn)