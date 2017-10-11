# 服务容器-注

> 作者：[韩忠康](http://hellokang.net)
> 原文：[https://laravel.com/docs/5.5/container](https://laravel.com/docs/5.5/container)
> 中文（laravel-china翻译）：[https://d.laravel-china.org/docs/5.5/container](https://d.laravel-china.org/docs/5.5/container)
> Laravel版本：5.5
> 本文是Laravel文档中服务容器章节的解析，建议与文档对照阅读。
---

[TOC]

## # 概述
官方文档中关于服务容器的介绍，主要是针对于这个服务容器提供的绑定，解析的语法上。是建立在读者对服务容器，依赖注入，控制反转有一定的认知的基础上进行说明的。本篇注解的主要目的就是在官方文档的基础上，补充上服务容器设计上的的内容，便于理解Laravel提供的语法。

Laravel的应用Application的实现就是一个服务容器，目的是用来管理Laravel框架中各种对某些对象的依赖关系。体现的设计思想是控制反转。目的是尽可能降低类对象间的耦合度。我们一步步的看。

## # 一个基本的服务容器
先搞清楚服务的概念，不要想多了，任何一个功能，任务都可以叫做服务service。所以说功能类对象，就是服务。
再看容器，把这些服务装在一起，装在哪？就是一个容器。其实就是一个可以找到这些服务的一个对象。

通常一个容器要具有绑定和解析两个操作。

- 绑定，指的是将获取服务对象的方法在容器总进行注册。相当于将服务装入到了容器中。
- 解析，指的是将绑定到容器中的服务从容器中提取出来，注意通常我们绑定的不是对象本身，而是生成对象的代码，因此解析时通常是执行代码来得到对象。

看代码演示，本段代码与Laravel无关：可以在github上下载得到示例代码：

    # 1, 服务容器定义
    /**
     * Class Application
     * 服务容器类，类名参考Laravel
     */
    class Application
    {
    //    已绑定（注册）的服务
        private $services = [];

        /**
         * 绑定（注册）
         * @param $class string
         * @param $generator Closure
         */
        public function bind($class, $generator)
        {
            $this->services[$class] = $generator;
        }

        /**
         * 解析
         * @param $class string
         */
        public function make($class)
        {
            return call_user_func($this->services[$class]);
        }
    }

    # 2, 服务类示例
    /**
     * Class Kernel
     * 内核服务类
     */
    class Kernel
    {
    }
    /**
     * Class Request
     * 请求服务类
     */
    class Request
    {
    }

    # 3, 绑定服务到容器，通常在程序初始化阶段完成
    $app = new Application();
    $app->bind('Kernel', function() {
        return new Kernel();
    });
    $app->bind('Request', function() {
        return new Request();
    });

    # 4, 需要时从容器中解析
    $kernel = $app->make('Kernel');
    var_dump($kernel);
    $request = $app->make('Request');
    var_dump($request);

观察上面的代码，Application类就是服务容器类，实例化的$app就是服务容器对象。服务都绑定在这个容器上。
Kernel 和 Request 就是具体的某个服务。别忘了任何功能都可以是服务。
$app->bind()，就是将服务绑定到容器中，注意，绑定的是对象生成代码，而不是对象本身。因此绑定时，并没有去实例化对象。
$app->make()，就是从服务容器中解析服务，其实就是调用对应类的生成代码，得到对应的服务对象。

以上就是一个基本的服务容器。提供服务容器这种架构的目的，就是将项目中各种各样复杂多样，需要复用的功能整理到一起来管理。

## # Laravel的服务容器
有了基本的服务容器概念，Laravel文档中本章的大部分就可以看懂了。服务容器这篇文档中，大部分内容都在说，laravel的服务容器提供了哪些绑定和解析的相关方法。我们先看看Laravel服务容器的实现代码。

### ## 服务容器实现
Laravel中的服务容器主要由 `Illuminate\Foundation\Application` 和其父类 `Illuminate\Container\Container` 来实现。其中 `Illuminate\Container\Container` 类实现了一个容器应该有的绑定，解析等功能，而 `Illuminate\Foundation\Application` 类继承Container，完成了基础服务的绑定，和一些基础的初始化操作。容器类图如下，展示了基本的成员：

[laravel容器实现类图](http://www.hellokang.net/asset/image/container.png)

类图中可以看出，容器的和新方法都是Container实现。Application利用该容器完成了初始绑定的一些列工作。还可以参考代码，`Illuminate\Foundation\Application::__construct()` 来看看初始化绑定的内容：

    # 构造方法
    public function __construct($basePath = null)
    {
        if ($basePath) {
            $this->setBasePath($basePath);
        }
        // 注册基础绑定
        $this->registerBaseBindings();
        // 注册基础服务容器
        $this->registerBaseServiceProviders();
        // 注册核心容器别名
        $this->registerCoreContainerAliases();
    }

再看看基础绑定都有些啥，参考代码 `Illuminate\Foundation\Application::registerBaseBindings()`：

    protected function registerBaseBindings()
    {
        static::setInstance($this);
        // 将$app容器对象绑定到容器，标识为app
        $this->instance('app', $this);
        // 将$app容器对象绑定到容器，标识为Container
        $this->instance(Container::class, $this);
    }

看一下这个方法的原因，是你要留意，Laravel的服务容器，将服务容器本身也作为服务，绑定在容器中。这个比较狠..

还有一个方法：registerBaseServiceProviders() 注册基本的服务提供者。这个服务提供者，其实就是用来完成服务绑定的独立功能类，我们在《服务提供者-注》中再详细讨论。

回头看看，Laravel提供的服务容器，功能更多，适应场景也就更多。但本质不要迷失了！

### ## 绑定语法索引
Laravel实现的服务容器，提供了多种绑定方法，核心思路都是一样的绑定，提供了不同的效果支持，下面做一个索引。具体请参考官方文档。

|方法|作用|说明|
|----|----|----|
|$app->bind(类或接口, 具体实现)|基础绑定|与我们的示例相同，提供类或者接口实现的绑定|
|$app->singleton(类或接口, 具体实现)|绑定单例|多次解析得到的是单例对象|
|$app->instance(类或接口, 对象)|绑定对象|直接绑定已经存在的对象，每次解析获取得到是对象本身|
|$app->when('App\Http\Controllers\UserController')->needs('$variableName')->give($value);|原始值绑定|逻辑是某个类需要某个变量时解析某个值|
|$this->app->when(PhotoController::class) ->needs(Filesystem::class) ->give(function () {return Storage::disk('local'); });|上下文绑定|逻辑是某个类需要某个类时解析某个对象|
|

### ## 解析语法索引
也是一样，整理了从容器中解析的方法，做一个索引：

|方法|作用|说明|
|----|----|----|
|$app->make(类或接口)|解析||
|$app->makeWith(类或接口, 参数)|携参解析||
|resolve(类或接口)|解析|使用辅助函数解析，在没有$app容器对象时可用|

### ## 批量绑定解析
Laravel还提供了批量解析的方法。思路是，为多个绑定的服务，打上相同的标签，然后通过标签标识进行批量解析，用到的方法：

    // 绑定服务
    $this->app->bind('SpeedReport', function () {});
    $this->app->bind('MemoryReport', function () {});

    // 打标签
    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

    // 批量解析
    $app->tagged('reports');

以上就是Laravel服务容器的实现！再进一步：

## # 自动依赖注入
注意laravel中代码，经常会出现下面的语法来使用一个对象，（摘自官方文档例子）：

    <?php
    namespace App\Http\Controllers;
    use App\Users\Repository as UserRepository;
    class UserController extends Controller
    {
        /**
         * 用户存储库实例。
         */
        protected $users;

        /**
         * 创建一个新的控制器实例。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users) # 注意参数
        {
            $this->users = $users;
        }
    }

上面代码中，__construct构造方法中，可以直接使用UserRepository类的对象了，不用再去实例化等操作。也就是在构造方法执行时，UserRepository类的对象已经实例化好，并传递（注入）到了构造方法中。这种语法，就称之为自动依赖注入。
我们分两步来理解：1，依赖注入。2，自动注入。

### ## 依赖注入 
依赖注入，DI, Dependency Injection，指的是将依赖的资源由外部注入到方法内部。是相对于在方法内部实例化来说的。说白了就是在调用方法前，将对象（或其他资源）准备好了，再传递给方法。就叫依赖注入。

这么做的目的是什么呢？
解耦。如果在方法内部去实例化一个对象，那么该方法就和这个对象完全紧耦合在一起了。而如果在方法外去实例化对象，这个对应就和方法没有直接关系了，也就是降低了对象和方法间的耦合度。举个例子，如果一个方法需要一个日志处理对象，如果在方法内部实例化了一个LogFile对象，这样当我们需要其他方式处理日志例如LogDB处理日志时，这个方法就好重写。而如果采用注入依赖的方式，方法本身不会发生改变。仅仅是在调用前注入时选择不同的Log对象即可。这就是DI的目的，降低耦合度。

在Laravel中，控制器、事件监听器、队列任务、中间件都支持依赖注入。

### ## 自动注入
自动注入，就是自动依赖注入，是laravel中最常用的依赖对象注入方式。语法上，就是通过参数的类型约束来确定参数的类型，实现自动化注入。

实现自动依赖注入通常需要反射机制来确定参数类型，然后从服务容器中解析需要的服务，最后作为实参传递到方法中。

还是使用我们的代码进行演示：

    # 5, 需要注入依赖的控制器动作
    class Controller
    {
        // 依赖于Request对象
        public function show(Request $request)
        {
            var_dump($request);
        }
    }

    # 6, 自动依赖注入
    $class = new ReflectionClass('Controller');
    $method = $class->getMethod('show');
    // 全部参数类型
    $arguments = [];
    foreach($method->getParameters() as $parameter) {
        $type = $parameter->getClass()->getName();
        $arguments[] = $app->make($type);
    }
    // 调用方法，同时注入依赖对象
    $method->invokeArgs($class->newInstance(), $arguments);

代码是继续上面的服务容器的测试代码。
一个控制器动作，依赖Request对象。通过反射调用该方法，之前先判断参数类型，从服务容器中解析对象，然后注入到方法中完成调用。
以上就是一个基本的自动注入依赖的演示。

在Laravel中，建议使用这种方式来完成，可以简洁化我们的代码，优雅！

## # 控制反转

最后再说一个概念，控制反转，IoC, Inversion of Control。IoC控制反转是一种设计模式, 用来解决对象间的过度依赖问题. 解决思路是, 设法不在依赖对象中去获取(new)被依赖对象. 最典型的实现方式就是DI依赖注入了.
我们说的服务容器，一般也叫做：IoC容器！

就到这里啦！
相关阅读：请求周期-注，服务提供者-注。
