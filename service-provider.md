# 服务提供者
 
> 译者：[韩忠康](http://hellokang.net)。校对：南宁宁

---

[TOC]

## # 引言
服务提供者是Laravel应用引导启动的核心。你的应用，又或者Laravel的核心服务都是通过服务提供者启动的。

我们所谓的“启动”是什么意思呢？通常来讲，我们指的是 *注册* 一些事项，包括注册服务容器的绑定，事件监听，中间件，也包括路由。服务提供者也是应用配置的核心。

你打开`config/app.php`文件，会看到`providers`数组项。这是应用要加载的全部的服务提供者类。当然，其中很多是延迟加载的，也就是不是在每个请求都需要加载，而是当所提供的服务被实际使用时才会加载。

本篇文档中将会告诉你如何编写自己的服务提供者以及如何将其注册到Laravel应用中。

## # 编写服务提供者
服务提供者都继承自`Illuminate\Support\ServiceProvider`类。大多数服务提供者都包含`register`和`boot`方法。在`register`方法中仅需要绑定事项到[服务容器](service-container)。不应该试图在`register`方法中注册任何事件监听，路由，或其他任何额外的功能。

Artisan命令行可以生成新的服务提供者，需要使用`make:provider`命令：

    php artisan make:provider RiakServiceProvider

### ## register方法
如上所述，在`register`方法中，仅应该绑定事项到 [服务容器](service-container)。 不应该试图在`register`方法中注册任何事件监听，路由，或其他任何额外的功能。否则，你可能会无意地使用到还没有加载的服务提供者提供的服务。

让我们来看一看一个基础的服务提供者的代码。任何服务提供者的方法中，经常会使用`$app`属性来访问服务容器。

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

以上这个服务提供者仅仅定义了`register`方法，在其中为服务容器定了一个`Riak\Connection`的实现。关于服务容器的工作方式，请参考[他的文档](service-container)。

### ## boot方法
那么，如果需要在服务提供者中注册一个视图编辑器（view composer）该如何处理呢？可以在`boot`方法中完成。*该方法会在全部的服务提供者被注册完毕后调用* ,也就是说该方法可以使用到框架注册的全部服务：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }
    }

#### ### boot方法的依赖注入
可以通过类型提示向`boot`方法注入依赖。[服务容器](service-container)会自动注入你需要的任何依赖：

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

## # 注册服务提供者
所有的服务提供者在`config/app.php`配置文件中注册。该文件内定了`providers`数组，其中列出了服务提供者的类名。默认情况下，Laravel的核心服务提供者列在该数组中。这些服务提供者将会启动Laravel的核心组件，例如mailer邮件, queue队列, cache缓存等。

注册自定义的服务提供者，仅需要将其加入到数组中：

    'providers' => [
        // Other Service Providers

        App\Providers\ComposerServiceProvider::class,
    ],


## # 延迟加载服务提供者
如果服务提供者仅仅向[服务容器](service-container)注册绑定事项，可以选择延迟到真正使用时再加载。延迟加载可以提升应用性能，因为不用每个请求都从文件系统载入。

Laravel会编译和存储一系列由延迟服务提供者提供的服务，以及他们对应的服务提供者的名字。然后，当试图去解析其中的服务时，Laravel才会去加载服务提供者。

要延迟加载服务提供者，将`defer`属性设置为`true`，同时定义`providers`方法。`providers`方法应该返回该服务提供者绑定到服务器容器的标识列表：
   
    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Indicates if loading of the provider is deferred.
         *
         * @var bool
         */
        protected $defer = true;

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }

    } 