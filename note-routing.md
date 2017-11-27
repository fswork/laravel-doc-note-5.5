# 路由-注

> 作者：[韩忠康](http://hellokang.net)
> 原文：[https://laravel.com/docs/5.5/routing](https://laravel.com/docs/5.5/routing)
> 中文（laravel-china翻译）：[https://d.laravel-china.org/docs/5.5/routing](https://d.laravel-china.org/docs/5.5/routing)
> Laravel版本：5.5
> 本文是Laravel文档中routing路由章节的解析，建议与文档对照阅读。
---

[TOC]

[# 概述](#概述)
[# 缓存服务实现与Cache契约结构说明](#缓存服务实现与Cache契约结构说明)
[# 契约在编码时的使用](#契约在编码时的使用)
[# 官方文档概要](#官方文档概要)
[# 结语](#结语)

## # 概述
路由routing，指的就是URI与处理程序（通常就是控制器动作）之间的映射关系。说白了就是哪种格式的URI应该匹配到哪个控制器动作。就是框架中的路由系统。
路由的核心目的，就是规范，美化URL，实现伪静态。
同时在一些功能强大的路由系统中，还可以处理一些通用的任务，例如在Laravel的路由系统中，支持设置中间件，检测CSRF令牌等功能。
在Laravel中，请求处理的起始点，就是路由。我们在实现因为逻辑时也是从路由开始的。

## # 基本原理
路由处理的基本过程，由：

- 配置路由映射。
- 解析URI，分发请求

两个步骤，就是配置和解析 完成。
配置，就是我们指定哪种格式的URI，应该由哪个处理动作来处理。
解析，就是当请求来到时，确定当前的URI，匹配之后，分发到预先设置好的处理动作。

Laravel中 `routes/` 目录用来保存全部的路由配置，我们就是在这些文件中完成路由配置的，示例配置如下：

    routes/web.php 路由配置
    Route::get('/cache', function() {
        return cache('key');
    });

以上配置就表示，如果请求的URI为：/cache，则由对应的匿名函数进行处理。
