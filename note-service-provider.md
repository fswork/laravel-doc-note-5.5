# 服务提供者-注

> 作者：[韩忠康](http://hellokang.net)
> 原文：[https://laravel.com/docs/5.5/providers](https://laravel.com/docs/5.5/providers)
> 中文（laravel-china翻译）：[https://d.laravel-china.org/docs/5.5/providers](https://d.laravel-china.org/docs/5.5/providers)
> Laravel版本：5.5
> 本文是Laravel文档中服务提供者章节的解析，建议与文档对照阅读。
---

[TOC]


## # 概述
官方文档中关于服务提供者的这篇文档，主要是说明如何编写自定义的服务提供者。本篇注解，就来说说啥是服务提供者，然后再总结一下如何编写服务提供者。

服务提供者想要理解，首先需要理解什么是服务容器，可以移步 [服务容器-注](https://laravel-china.org/articles/6333/service-containers-notes) 进行了解。在有了服务容器的概念后，就可以很容易的理解什么是服务提供者了。

服务容器是盛放服务的容器，有绑定和解析两个主要操作。绑定就是将服务注册到容器中。而服务提供者，就是完成绑定服务到容器任务的单元。看名字就知道，是用来提供服务的单位。简言之，服务容器中绑定的服务，就是由服务提供者绑定进去的。就是这么个功能。 详细看看。

## # Laravel中服务提供者的结构

服务提供者类，都要继承自 `Illuminate\Support\ServiceProvider` 服务提供者基类。

服务提供者一定要有一个 `register` 方法，这个方法功能就是完成服务绑定到容器的。参考日志服务提供者 `Illuminate\Log\LogServiceProvider` register的实现：

    class LogServiceProvider extends ServiceProvider
    {
        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            // 绑定一个单例对象，用于提供日志服务
            $this->app->singleton('log', function () {
                return $this->createLogger();
            });
        }
    }

可以看到register()方法，确实是完成服务容器的绑定。这就是服务提供者的主要任务。
除了这个register()方法外，还有boot()也是比较常用的方法，后边会提到。

## # 绑定的时机
再看就是这个register()方法是在何时被调用的，就是何时完成注册的。

一部分，就是在应用服务容器初始化时，完成基础的服务绑定，参考代码：

文件 `Illuminate\Foundation\Application` :
构造方法：

    public function __construct($basePath = null)
        {
            if ($basePath) {
                $this->setBasePath($basePath);
            }

            $this->registerBaseBindings();
            // 注册基础服务提供者，其他代码忽略
            $this->registerBaseServiceProviders();

            $this->registerCoreContainerAliases();
        }

调用的 `Illuminate\Foundation\Application::registerBaseServiceProviders()`:

    protected function registerBaseServiceProviders()
    {
        $this->register(new EventServiceProvider($this));

        $this->register(new LogServiceProvider($this));

        $this->register(new RoutingServiceProvider($this));
    }


这个 `$this->register()` 就可以触发对应的服务提供者的register()方法。


另一部分是在内核Kernel处理请求前，会绑定一些基础功能服务提供者。

可以参考代码 `Illuminate\Foundation\Application::registerConfiguredProviders()` :

    public function registerConfiguredProviders()
    {
        (new ProviderRepository($this, new Filesystem, $this->getCachedServicesPath()))
                    ->load($this->config['app.providers']);
    }

注意观察 `$this->config['app.providers']` 这行代码就是从配置文件中读取配置好的服务提供者列表，进行注册绑定，`app.providers' 这个配置可以在 `config/app.php` 中看到， 列举一部分：

    'providers' => [
        /*
         * Laravel Framework Service Providers...
         */
        Illuminate\Auth\AuthServiceProvider::class,
        ... 好多，好多

        /*
         * Package Service Providers...
         */
        Laravel\Tinker\TinkerServiceProvider::class,

        /*
         * Application Service Providers...
         */
        ... 好多，好多
        App\Providers\RouteServiceProvider::class,

    ],

而这个 registerConfiguredProviders() 方法，是在 $kernel->handle()的内部进行调用的。详细的调用过程，大家可以利用ide进行追踪下。需要注意的就是，这个方法是在内核处理请求阶段执行的。

以上就是两处服务提供者，绑定服务的时机。1，应用服务容器初始化过程；2，内核处理请求过程中。

## # 服务容器的boot()方法
一个基本的服务器容器，除了register()方法外，还会存在一个boot方法。对比register()方法，他们的主要差异是执行时机不同，进而导致了所完成的任务特征也有所不同。
register(), 是服务提供者的主要任务，就完成某些服务的注册。使用服务提供者，核心目的就是为了注册服务。
boot()，是为了初始化某些服务特殊工作。任何服务提供者的boot()方法都会在全部服务提供者注册之后运行。那也就可以保证在boot()方法中，可以使用全部的注册服务。而对应的register()是在每个服务提供者注册时执行，其中不能使用其他服务提供者绑定的服务，因为我们不能完全确定其先后顺序。除非你非常熟悉。

执行时机的差异，可以参考代码：`Illuminate\Foundation\Http\Kernel::$bootstrappers` 属性，即时在内核处理请求时，需要启动的功能数组定义：

    protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
        // 先启动注册服务提供者，触发服务提供者的register()方法
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
        // 注册完毕后，在启动boot服务提供者，触发全部服务提供者的boot()方法
        \Illuminate\Foundation\Bootstrap\BootProviders::class,
    ];

注意此数组的最后两个元素，显示注册服务提供者，再时启动服务提供者。这就确定了服务提供者方法的执行时机。


## # 延迟绑定的优化
在 `app.providers` 配置项中，这么多的服务提供者，是不是都需要在初始化公共阶段完成绑定呢？如果某些服务器在整个周期可能会用不到呢，那岂不是白白绑定了？Laravel通过 将某些服务提供者设置为延迟状态，来解决这个性能问题。

就是说，如果某个服务提供者被设置为延迟处理，那么不会在初始化时就绑定，而是在真正使用时才绑定，使用属性 $defer = true 就可做到。那么启动阶段在处理到该服务提供者时，如果发现是延迟的，那么不立即绑定，等到需要时在绑定。

参考某个访问（Laravel文档提供）：

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * 是否延时加载提供器。
         *
         * @var bool
         */
        protected $defer = true;
    ｝

定义超级简单，一个属性搞定。这是个内部优化措施。与整体结构关系不大！

## # Laravel文档概要
注解完毕后，最官网文档做个概要：
主要说如何编写服务提供者，过程如下：

- 通过 artisan make:provider 来创建基础服务提供者结构
- 实现注册 register() 方法
- 实现注册 boot() 方法，没有可以留空
- 在配置文件中，配置定义好的服务提供者

经过以上步骤，就完成了服务提供者的定义。当我们需要在Laravel中增加自己的服务时，就可以使用服务提供者来绑定我们的服务到服务容器，便于后期依赖注入到需要的位置！


## # 结语
总结一下 服务，服务容器，服务提供者的关系，看个图总结一下：

![服务-容器-提供者的关系](http://www.hellokang.net/asset/image/service-container-provider.png)

就这么多！

