---
title: 使用 HttpClient 下载图片
date: 2016-04-13 23:10:33
tags: Util
---

使用 JDK 自带的 `UrlConnection` 也能访问网络资源，但是要处理 SSL，Cookie，Proxy 等时编码就会很麻烦，`Apache HttpClient` 的功能非常强大，封装了很多操作，使用简单，能很方便的访问网络资源，其示例代码中有很多有用的例子，下载地址为 <http://hc.apache.org/downloads.cgi>

<!--more-->

```
├── ClientAbortMethod.java
├── ClientAuthentication.java
├── ClientChunkEncodedPost.java
├── ClientConfiguration.java
├── ClientConnectionRelease.java
├── ClientCustomContext.java
├── ClientCustomPublicSuffixList.java
├── ClientCustomSSL.java
├── ClientEvictExpiredConnections.java
├── ClientExecuteProxy.java
├── ClientExecuteSOCKS.java
├── ClientFormLogin.java
├── ClientMultiThreadedExecution.java
├── ClientPreemptiveBasicAuthentication.java
├── ClientPreemptiveDigestAuthentication.java
├── ClientProxyAuthentication.java
├── ClientWithRequestFuture.java
├── ClientWithResponseHandler.java
├── ProxyTunnelDemo.java
├── QuickStart.java
```

还有 FluentApi 的例子

```
├── FluentAsync.java
├── FluentExecutor.java
├── FluentQuickStart.java
├── FluentRequests.java
└── FluentResponseHandling.java
```

下面简单的介绍一下使用 HttpClient 下载一幅图片，FluentApi 使用更加简洁，但是功能相对较少。

## Gradle 依赖
```
dependencies {
    compile 'org.apache.httpcomponents:httpclient:4.5.2'
    compile 'org.apache.httpcomponents:fluent-hc:4.5.2'
}
```

## 使用 FluentApi 下载文件
> The fluent API relieves the user from having to deal with manual deallocation of system resources at the cost of having to buffer response content in memory in some cases.
> 
> 使用 FluentApi 不需要手动的缓存 response 数据到内存和释放相应的资源，FluentApi 已经帮我们处理好了。

```java
import org.apache.http.client.fluent.Request;

import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;

public class DownloadUtil {
    /**
     * 使用 HttpClient 的 FluentApi 下载文件
     * @param url 文件的 url
     * @param localPath 本地存储路径
     * @throws IOException 如果 url 的文件找不到，超时等会抛出异常
     */
    public static void downloadFile(String url, String localPath) throws IOException {
        byte[] content = Request.Get(url).connectTimeout(5000).socketTimeout(5000).execute().returnContent().asBytes();
        BufferedOutputStream out = new BufferedOutputStream(new FileOutputStream(new File(localPath)));
        out.write(content);
        out.flush();
        out.close();
    }

    public static void main(String[] args) throws IOException {
        downloadFile("http://xtuer.github.io/img/dog.png", "/Users/Biao/Desktop/a.png"); // 下载图片
    }
}
```

## 使用普通的 Api 下载文件
同样是下载文件，相比之下，代码复杂了不少

```java
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;

import java.io.*;

public class CommonDownloadUtil {
    public static void downloadFile(String url, String localPath) throws IOException {
        CloseableHttpClient httpclient = HttpClients.createDefault();
        try {
            HttpGet httpget = new HttpGet(url);

            System.out.println("Executing request " + httpget.getRequestLine());
            CloseableHttpResponse response = httpclient.execute(httpget);
            try {
                System.out.println("----------------------------------------");
                System.out.println(response.getStatusLine());

                // Get hold of the response entity
                HttpEntity entity = response.getEntity();

                // If the response does not enclose an entity, there is no need
                // to bother about connection release
                if (entity != null) {
                    InputStream in = entity.getContent();
                    try {
                        // do something useful with the response
                        byte[] buffer = new byte[1024];
                        BufferedInputStream bufferedIn = new BufferedInputStream(in);
                        int len = 0;

                        FileOutputStream fileOutStream = new FileOutputStream(new File(localPath));
                        BufferedOutputStream bufferedOut = new BufferedOutputStream(fileOutStream);

                        while ((len = bufferedIn.read(buffer, 0, 1024)) != -1) {
                            bufferedOut.write(buffer, 0, len);
                        }
                        bufferedOut.flush();
                        bufferedOut.close();
                    } catch (IOException ex) {
                        // In case of an IOException the connection will be released
                        // back to the connection manager automatically
                        throw ex;
                    } finally {
                        // Closing the input stream will trigger connection release
                        in.close();
                    }
                }
            } finally {
                response.close();
            }
        } finally {
            httpclient.close();
        }
    }

    public final static void main(String[] args) throws Exception {
        downloadFile("http://xtuer.github.io/img/dog.png", "/Users/Biao/Desktop/a.png"); // 下载图片
    }
}
```

## 参考
* [HttpClient Tutorial](https://hc.apache.org/httpcomponents-client-ga/tutorial/html/index.html)
* [HttpClient Examples](https://hc.apache.org/httpcomponents-client-ga/examples.html)
