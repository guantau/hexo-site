---
title: 编译dronekit-tower的一点问题
date: 2016-08-21 23:47:52
categories: 写点程序
tags: 
  - dronekit
---

从Github上直接clone下Tower（ https://github.com/DroidPlanner/Tower ）进行编译总是报错

> Gradle sync failed: Authentication scheme 'all'(Authentication) is not support

其实解决方法很简单，删除Tower/build.gradle中带认证的Maven库地址（第47-53行），变为以下内容即可：

    allprojects {
        repositories {
            jcenter()
            mavenCentral()
    
            maven { url 'https://maven.fabric.io/public' }
    
            flatDir {
                dirs 'libs'
            }
        }
    }

