# 请求生命周期-注

> 作者：[韩忠康](http://hellokang.net)。校对：南宁宁

> 原文：[https://laravel.com/docs/5.5/lifecycle](https://laravel.com/docs/5.5/lifecycle)
> 中文（laravel-china翻译）：[https://d.laravel-china.org/docs/5.5/lifecycle](https://d.laravel-china.org/docs/5.5/lifecycle)
> Laravel版本：5.5
> 本文是Laravel文档中请求生命周期章节的解析，建议与文档对照阅读。
---

[TOC]

## # 概述
本章主要说明 Laravel 在接收到请求后的程序执行流程。讲解的核心内容是请求由入口文件接受请求，然后初始化app也就是服务容器，再由web内核处理请求，生成响应。最后将响应交给浏览器。其中主要的工作就是启动应用，处理请求和生成响应。在编码设计实现上，使用了服务容器，服务提供者，内核，路由等概念。
下面我们就对请求生命周期做一个注解。

## # 整体流程
先看流程图：

![总体流程](http://www.hellokang.net/asset/image/request-lifecycle-mainflow.png)
注：彩色的是相对重要的环节，请特别关注。

以上图片，主要说明了一个请求的整体流程，参考 `public/index.php` 入口文件即可看到该流程。
提取后的文件内容如下：

    public/index.php

    # 载入自动加载
    require __DIR__.'/../bootstrap/autoload.php';

    # 初始化app
    $app = require_once __DIR__.'/../bootstrap/app.php';

    # 解析内核对象
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

    # 内核处理请求，生成响应
    $response = $kernel->handle(
        $request = Illuminate\Http\Request::capture()
    );

    # 发送响应
    $response->send();

    # 内核终止
    $kernel->terminate($request, $response);


整体流程中比较中复杂和重要的环境有 启动应用 和 内核处理请求生成响应 这两个环节。我们对这两个环境做更深入的说明。


## # 启动应用


## # 内核处理请求生成响应

