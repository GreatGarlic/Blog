---
title: CentOS7 安装 LibreOffice
date: 2018-07-22 14:23:27
tags: Mac
---

使用 CentOS7 无界面版本作为服务器的操作系统，在里面安装 LibreOffice 用于转换各种文档到 PDF，LibreOffice 的安装步骤如下:

1. 下载:

   1. 访问 <https://www.libreoffice.org/download/download/>
   2. 选择 **Linux x86_64(rpm)** 的版本
   3. 下载得到 LibreOffice_6.0.5_Linux_x86-64_rpm.tar.gz (目前最新版为 6.0.5)

2. 安装:

   1. 删除: 在安装之前，先删除已经安装的 LibreOffice: `yum remove libreoffice*`
   2. 解压: `tar -xvf LibreOffice_6.0.5_Linux_x86-64_rpm.tar.gz`
   3. 安装:
      1. `cd LibreOffice_6.0.5.2_Linux_x86-64_rpm/RPMS`
      2. `yum localinstall *.rpm`
   4. 查看:
      1. `which libreoffice6.0` 看到路径为 **/usr/bin/libreoffice6.0**
      2. `ll /usr/bin/libreoffice6.0` 得到 **/opt/libreoffice6.0/program/soffice**，说明安装到了 **/opt/libreoffice6.0**

3. 依赖:

   执行 `libreoffice6.0` 可能会提示库文件找不到，如 libcairo.so.2，libcups.so.2，libSM.so.6 等，执行下面几条命令安装需要的库:

   * `yum install cairo -y`
   * `yum install cups-libs -y`
   * `yum install libSM -y` 

安装 LibreOffice 可参考 <https://www.tecmint.com/install-libreoffice-on-rhel-centos-fedora-debian-ubuntu-linux-mint/>。

Java 中可使用 [JODConverter](https://github.com/sbraconnier/jodconverter) 调用 LibreOffice 进行文件格式转换，可参考 [Office 文档转为 PDF 和 HTML](https://qtdebug.com/office-to-pdf-html/)。