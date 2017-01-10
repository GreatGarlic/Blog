---
title: https-pkix
date: 2017-01-10 17:30:04
tags: [Mac, Java]
---

使用 Gradle 下载 jar 包时，如果遇到 **PKIX path building failed and unable to find valid certification path to requested target** 错误，则说明是 ssl 证书的问题，把证书加入到 JVM 的 **cacerts** 文件即可(使用 Firefox 来下载网站的证书)

1. 使用 Firefox 打开 https 的链接

2. 点击地址栏中的小锁图标

3. 点击 Security Connection 右边的向右箭头

4. More Information > Security > View Certificate > Details > Export

5. 例如上面 cert 文件保存为 **repo1.maven.org.crt**

6. 打开终端，导入 cert 文件

   ```
   sudo keytool -import -alias maven -keystore /Library/Java/JavaVirtualMachines/jdk1.8.0_77.jdk/Contents/Home/jre/lib/security/cacerts  -file repo1.maven.org.crt
   ```

   > cacerts 文件的路径和 JRE/JDK 安装的路径有关

7. 输入 cert 文件的密码，默认的都是 `changeit`

8. 重启系统