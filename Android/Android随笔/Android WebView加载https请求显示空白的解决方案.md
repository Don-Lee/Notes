当WebView加载的https页面时，如果这个网站的安全证书在Android无法得到认证，WebView就会变成一个空白页，解决方案如下：  
```java
webview.setWebViewClient(new WebViewClient(){
 
@override
public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error){
 
//handler.cancel(); //默认的处理方式，WebView变成空白页
 handler.process();//接受证书

}
 
// 这行代码一定加上否则没效果 
 webView.getSettings().setJavaScriptEnabled(true); 
```

另外，当WebView加载https和http混合的请求(例：http的网页中包含https的图片)时需要做如下设置：

```java
//允许加载混合网络协议
webview.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
```
