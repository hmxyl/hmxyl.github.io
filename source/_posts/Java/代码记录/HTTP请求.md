# HTTP请求

## # 模拟POST请求

```java
public static String getPostDataJsoup(String requestUrl, String encode, Map<String, String> paramMap) {
        HttpResponse response = null;
        DefaultHttpClient client = null;
        try {
             // 创建httpclient对象
             client = new DefaultHttpClient();
             // 创建post方式请求对象
             HttpPost httpPost = new HttpPost(requestUrl);
             // 装填参数
             List<NameValuePair> nvps = new ArrayList<NameValuePair>();
             if (paramMap != null) {
                  for (Entry<String, String> entry : paramMap.entrySet()) {
                        nvps.add(new BasicNameValuePair(entry.getKey(), entry.getValue()));
                  }
             }
             // 设置参数到请求对象中
             httpPost.setEntity(new UrlEncodedFormEntity(nvps, encode));
             // 设置header信息
             // 指定报文头【Content-type】、【User-Agent】
             httpPost.setHeader("Content-type", "application/x-www-form-urlencoded");
             httpPost.setHeader("User-Agent", "Mozilla/4.0 (compatible; MSIE 5.0; Windows NT; DigExt)");
             response = client.execute(httpPost);
             HttpEntity entity = null;
             if (response != null && response.getStatusLine().getStatusCode() >= 200 && response.getStatusLine().getStatusCode() < 300) {
                  entity = response.getEntity();
             }
             if (entity != null) {
                  return Jsoup.parse(entity.getContent(), encode, requestUrl).select("body").text().replaceAll("\r", "").replaceAll("\n", "").toString();
             } else {
                  return null;
             }
        } catch (Exception e) {
             e.printStackTrace();
        } finally {
             // 释放链接
        }
        return null;
  }

```

## httpclient4.5获取和设置cookie

```java
public static void main(String[] args) {
      CookieStore cookieStore = new BasicCookieStore();
      CloseableHttpClient httpClient = HttpClients.custom().setDefaultCookieStore(cookieStore).build();
      try {
          HttpPost post = new HttpPost("http://www.baiduc.com");
          BasicClientCookie cookie = new BasicClientCookie("name", "zhaoke");
          cookie.setVersion(0);
          cookie.setDomain("/pms/"); //设置范围
          cookie.setPath("/");
          cookieStore.addCookie(cookie);
          httpClient.execute(post);//
          List<Cookie> cookies = cookieStore.getCookies();
          for (int i = 0; i < cookies.size(); i++) {
           System.out.println("Local cookie: " + cookies.get(i));
          }
      } catch (Exception e) {
          e.printStackTrace();
      }finally{
      }
  }

```

## HttpClient模拟登陆

​			HTTP协议本来是无状态的，但为了保持会话的状态，使用Cookie保存Session信息，当向服务器发送请求时会附加一些会话信息，从而能区分不同会话的状态。用户登陆过程，其实简单而言，就是首先验证用户名与密码，然后服务器生成会话信息保存到本地，最后用户凭借会话信息能够访问类似用户信息等需登陆的网页。

​			HttpClient4.5通过CookieStore保存用户的会话信息，还提供HttpClientContext保存用户连接的信息。下面是一个使用HttpClient模拟知乎登陆的简单案例。

```java
package com.httpclient.demo;
 
import java.io.IOException;
import java.util.LinkedList;
import java.util.List;
 
import org.apache.http.Consts;
import org.apache.http.NameValuePair;
import org.apache.http.client.CookieStore;
import org.apache.http.client.config.CookieSpecs;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.protocol.HttpClientContext;
import org.apache.http.cookie.Cookie;
import org.apache.http.impl.client.BasicCookieStore;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;
 
/**
 * 模拟登陆知乎
 */
public class ZhiHuTest {
 
  public static void main(String[] args) throws java.text.ParseException {
    String name = "username";
    String password = "password"
     
    // 全局请求设置
    RequestConfig globalConfig = RequestConfig.custom().setCookieSpec(CookieSpecs.STANDARD).build();
    // 创建cookie store的本地实例
    CookieStore cookieStore = new BasicCookieStore();
    // 创建HttpClient上下文
    HttpClientContext context = HttpClientContext.create();
    context.setCookieStore(cookieStore);
 
    // 创建一个HttpClient
    CloseableHttpClient httpClient = HttpClients.custom().setDefaultRequestConfig(globalConfig)
        .setDefaultCookieStore(cookieStore).build();
 
    CloseableHttpResponse res = null;
 
    // 创建本地的HTTP内容
    try {
      try {
        // 创建一个get请求用来获取必要的Cookie，如_xsrf信息
        HttpGet get = new HttpGet("http://www.zhihu.com/");
 
        res = httpClient.execute(get, context);
        // 获取常用Cookie,包括_xsrf信息
        System.out.println("访问知乎首页后的获取的常规Cookie:===============");
        for (Cookie c : cookieStore.getCookies()) {
          System.out.println(c.getName() + ": " + c.getValue());
        }
        res.close();
 
        // 构造post数据
        List<NameValuePair> valuePairs = new LinkedList<NameValuePair>();
        valuePairs.add(new BasicNameValuePair("email", name));
        valuePairs.add(new BasicNameValuePair("password", password));
        valuePairs.add(new BasicNameValuePair("remember_me", "true"));
        UrlEncodedFormEntity entity = new UrlEncodedFormEntity(valuePairs, Consts.UTF_8);
        entity.setContentType("application/x-www-form-urlencoded");
 
        // 创建一个post请求
        HttpPost post = new HttpPost("https://www.zhihu.com/login/email");
        // 注入post数据
        post.setEntity(entity);
        res = httpClient.execute(post, context);
 
        // 打印响应信息，查看是否登陆是否成功
        System.out.println("打印响应信息===========");
        HttpClientUtils.printResponse(res);
        res.close();
 
        System.out.println("登陆成功后,新的Cookie:===============");
        for (Cookie c : context.getCookieStore().getCookies()) {
          System.out.println(c.getName() + ": " + c.getValue());
        }
 
        // 构造一个新的get请求，用来测试登录是否成功
        HttpGet newGet = new HttpGet("http://www.zhihu.com/question/following");
        res = httpClient.execute(newGet, context);
        String content = EntityUtils.toString(res.getEntity());
        System.out.println("登陆成功后访问的页面===============");
        System.out.println(content);
        res.close();
 
      } finally {
        httpClient.close();
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

