---
layout: post
title:  "Android自动化构建"
date:   2017-02-20 09:00 +0800
categories: jekyll update
---

# Android自动化构建 #

## 1.安装jenkins ##

首先从官网下载jenkins，下载地址：[**https://jenkins.io/**](https://jenkins.io/)

安装完毕后，打开浏览器输入地址：[**localhost:8080/**](localhost:8080/)，即可打开jenkins

## 2.注册jenkins管理员 ##

打开只有点击右上角“注册”一个用户，这样第一个用户就是管理员。

## 3.项目构建 ##

1. 在jenkins首页，点击 “新建”
2. 输入项目名称，选择 “构建一个自由风格的软件项目”
3. 在 “源码管理” 项中选择 “Subversion Release”
4. “Repository URL” 中 填写项目SVN的repository
5. “增加构建步骤”，选择“invoke Ant”
6. “Ant Version” 选择 “Default”， “Targets” 填写要使用的脚本(在第4步说明)

## 4.自动化构建脚本 ##

### 关于build.xml ###

1. build文件在项目根目录下 (build.xml)
2. build.xml使用的是全局的Android Environment，所以打包的机器必须配置Android的环境变量

### 关于customer_rules.xml ###

build.xml文件会连接到"../ant_build_file/customer_rules.xml"

这个文件描述构建过程，For example：

1. 复制所有文件到临时文件夹(不影响原来项目)
2. 获取包名
3. 修改版本号
4. 复制打包好的APK文件到指定目录
5. ......

### 关于ant.properties and debug.keystore ###

customer_rules.xml会使用ant.properties中指向的keystore和alias去给打包好的APK文件签名

### 打包后的APK在哪里 ###

根据customer_rules.xml，最终打包后的APK会出现在“jenkins安装目录/workspace/jenkins项目名/Android/builds/Walmart-Android.apk"

## 5.还有些话 ##

使用ant打包Android APK已经是老技术了，现在应该使用Gradle，这篇文章就当是纪念我们用eclipse写Android的时光吧

切换Android studio后，学习下Gradle再总结一次好了