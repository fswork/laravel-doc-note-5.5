# 请求生命周期-注

> 作者：[韩忠康](http://hellokang.net)
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

应用，也就是服务容器（Service Container，`$app` ），是 `Illuminate\Foundation\Application` 的实例。之所以称应用是服务容器，顾名思义，就是用来盛放服务（就是某些特定功能，例如请求处理，响应处理等都算是服务）的一个容器，一个盒子。初始化容器时现将需要的服务装入容器，使用时从容器中获取，可以做到服务的集中，自动化管理等，关于服务容器的详细的注解，会有对应于官方文档的独立的章节。我们先看启动应用时Laravel都做了什么。

启动应用时，任务如下：

- 首先创建Laravel应用实例，实例化时会做很多服务容器初始化的工作，例如基础服务的绑定，这个暂且不表。目前仅考虑此处得到一个可以盛放服务的容器应用。
- 其次在应用容器中装入（专业的称呼是绑定，`bind` ）Http内核服务（用于处理http请求），Console内核服务（用于处理控制台程序），ExceptionHandler服务（用于处理异常）。
- 最后返回这个绑定了基础功能的应用服务容器对象。

参考流程图如下：

![启动应用流程](http://hellokang.net/asset/image/app-start.png)

在入口文件中 `public/index.php` ，载入app.php:

    // 载入应用启动代码
    $app = require_once __DIR__.'/../bootstrap/app.php';

对应的代码主要是 `bootstrap/app.php` ，整理过的代码如下：

    bootstrap/app.php
    <?php
    // 创建应用服务容器对象
    $app = new Illuminate\Foundation\Application(
        realpath(__DIR__.'/../')
    );

    // 绑定Http内核服务
    $app->singleton(
        Illuminate\Contracts\Http\Kernel::class,
        App\Http\Kernel::class
    );
    // 绑定Console内核服务
    $app->singleton(
        Illuminate\Contracts\Console\Kernel::class,
        App\Console\Kernel::class
    );
    // 绑定ExceptionHandler服务
    $app->singleton(
        Illuminate\Contracts\Debug\ExceptionHandler::class,
        App\Exceptions\Handler::class
    );

    // 返回服务对象
    return $app;

关于实例化时的任务在服务容器中讲解。方法 `$app->singleton()` 就是用来在服务容器应用中绑定服务的，该方法绑定的是单例对象。


## # 内核处理请求和生成响应

内核Kernel，是处理的最核心部分。几乎完成了功能上的全部任务。（app服务容器是完成了结构上的任务）。在启动应用时，绑定了内核服务，在启动应用容器后，从应用服务容器中解析出来内核服务，来处理请求并，生成响应，可以从入口文件了解该流程， `public/index.php` :

    # 解析内核对象
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

    # 内核处理请求，生成响应
    $response = $kernel->handle(
        $request = Illuminate\Http\Request::capture()
    );

`$app->make()` 就是从应用服务容器中解析对象的方法，此处解析出内核对象，就是 `App\Http\Kernel` 类对象（参考上面的绑定，可确定该类）。其次调用内核对象的 `handle()` 方法处理捕获的请求。

先看一下请求对象类的结构：
主要由两个类实现，`App\Http\Kernel` 继承自 `Illuminate\Foundation\Http\Kernel`。主要的成员如下类图：


在 `$app->make(Illuminate\Contracts\Http\Kernel::class)` 时会调用对象的构造方法，也就是 `App\Http\Kernel` 继承的构造方法，该方法中主要完成了路由中间件的处理，参考代码如下，位于 `Illuminate\Foundation\Http\Kernel::__construct()`， 整理过的代码如下：

    /**
     * Create a new HTTP kernel instance.
     *
     * @param  \Illuminate\Contracts\Foundation\Application  $app
     * @param  \Illuminate\Routing\Router  $router
     * @return void
     */
    public function __construct(Application $app, Router $router)
    {
        $this->app = $app;
        $this->router = $router;

        $router->middlewarePriority = $this->middlewarePriority;
        
        // 处理路由组，有意义的属性值在App\Http\Kernel中完成定义
        foreach ($this->middlewareGroups as $key => $middleware) {
            $router->middlewareGroup($key, $middleware);
        }
        
        // 处理路由，有意义的属性值在App\Http\Kernel中完成定义
        foreach ($this->routeMiddleware as $key => $middleware) {
            $router->aliasMiddleware($key, $middleware);
        }
    }

在完成内核对象的创建后，调用了内核对象的 `$kernel->handle()` 方法来处理请求。 参数是请求对象，请求对象通过 `Illuminate\Http\Request::capture()` 捕获创建的，其中包含了请求参数等请求相关数据。创建的过程可以参考 `Illuminate\Http\Request::createFromBase()` 方法，此处不进行深入，可以参考请求-注章节。

我们主要看内核处理请求的流程，也就是handle()的方法执行过程。操作有，按照执行流程列出如下：

- 内核启动
    - 加载环境变量
    - 加载配置项
    - 异常处理
    - 注册Facades（门面，外观）
    - 注册Providers（服务提供者）
    - 启动Providers（服务提供者）
- 中间件处理请求
- 路由分发请求，分发到控制器动作或者匿名函数

可以参考属性 `$bootstrappers` 定义，其中包含了需要在内核启动时完成的操作定义，位于 `Illuminate\Foundation\Http\Kernel::$bootstrappers` ：

    文件：Illuminate\Foundation\Http\Kernel::$bootstrappers
    protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
        \Illuminate\Foundation\Bootstrap\BootProviders::class,
    ];

以上数组的每个元素，就是需要启动的每个任务。

再看方法 `Illuminate\Foundation\Http\Kernel::sendRequestThroughRouter()` ，完成了中间件和分发请求的工作：

    protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);

        Facade::clearResolvedInstance('request');
        
        // 启动内核
        $this->bootstrap();

        // 利用管道，将请求通过中间件再分发到路由处理器
        return (new Pipeline($this->app)) # 管道
                    ->send($request) # 发送请求
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware) # 中间件
                    ->then($this->dispatchToRouter()); # 分发请求
    }

完成以上工作，请求就被分发到具体的处理器进行处理，就是控制器动作或者匿名函数。控制器处理完毕后，需要返回return一个响应Response对象。返回的响应对象，会被Http内核接收到，并最终由 `$kernel->handle()` 方法返回，最后响应在发送到请求代理端。此处可以参考，控制器动作的return, `$kernel->handle()` 的return。

这样，内核的处理工作就结束了。


## # 总结
简言之，3个步骤，其一，Laravel使用服务容器（应用）来管理各个服务的绑定解析。其二，具体的功能流程由内核完成调度。其三，由控制器动作完成业务逻辑处理。

参考阅读：服务容器-注，服务提供者-注，Facade-注，Service Provider-注。