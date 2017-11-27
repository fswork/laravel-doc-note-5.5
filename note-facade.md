# Facade（门面，外观）-注

> 作者：[韩忠康](http://hellokang.net)
> 原文：[https://laravel.com/docs/5.5/facades](https://laravel.com/docs/5.5/facades)
> 中文（laravel-china翻译）：[https://d.laravel-china.org/docs/5.5/facades](https://d.laravel-china.org/docs/5.5/facades)
> Laravel版本：5.5
> 本文是Laravel文档中Facade章节的解析，建议与文档对照阅读。
---

[TOC]

[# 概述](#概述)
[# Facade简化调用的例子](#Facade简化调用的例子)
[# Facade的执行过程](#Facade的执行过程)
[# Facade的基本结构](#Facade的基本结构)
[# Facade与助手函数](#Facade与助手函数)
[# 官方文档概要](#官方文档概要)
[# 结语](#结语)

## # 概述
Facade，一般翻译成外观或者门面。在Laravel的翻译文档中，是不翻词，这里我也直接使用Facade来表述。
Facade的主要作用，是 **简化类调用的快捷语法** 。因为在结构复杂，功能完善的框架中，往往类的结构，层次也比较复杂，Laravel也是如此复杂的框架，因此为了简化使用，我们就定义了类的快捷访问方式，在Laravel中，就是Facade！
常规设计模式中的外观模式（Facade Pattern），就是解决快捷访问问题的，因此Laravel的Facade就是外观模式的实现。可以参考关于外观模式的说明，来进一步了解。


## # Facade简化调用的例子
我们使用一个Laravel中的例子，来说明一下Facade是如何简化调用的。我们需要调用设置数据缓存的方法，使用Facade的语法如下：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function() {

        // 事先保证执行下面的put方法，将缓存存入
        // Cache::put('key', 'HelloKang', 10);
        
        // 获取缓存项key的内容
        return Cache::get('key');
    }

如果不使用Facade来调用，那么调用的语法如下：

    Route::get('/cache', function() {   
        // 获取缓存项key的内容
        return app('cache')->get('key');
    }

语法过程就是先从服务容器中解析出来缓存对象，再利用缓存对象将缓存项提取。

对比这两种使用方式，第一种显而易见，更加直观，简单一些。这就是Facade的主要目的。

## # Facade的执行过程
其实上面例子中第二个，利用服务容器解析对象的过程，就是Facade的执行过程。详细来说如下：

- 调用Facade的静态方法（也就是对应服务的对象方法）来完成功能
- Facade利用静态方法重载魔术方法 `static __callStatic()` 来处理该静态方法调用。因为Facade是不会存在服务的对象方法，例如上面的Cache::get(), Cache这个Facade是没有get静态方法的，因此会由Cache类的__callStatic()来处理。
- __callStatic()方法过程，就是先从服务容器解析对应的服务，再调用服务对象的方法来实现功能。

核心任务就是获取Facade对应的服务对象，这样才可以解析出来对应的服务，才可以完成功能。那么在编码实现上，是如何确定Facade对应的服务的呢？ 我们再看看 Facade 的基本结构：

## # Facade的基本结构

** Facade类都需要继承自 `Illuminate\Support\Facades\Facade` 类**。
** 每个Facade类例如Cache，都需要有一个getFacadeAccessor()方法，来确定其对应的服务名称**
** Facade基类Facade中定义了__callStatic()这个魔术方法，用来完成从服务容器解析服务对象，调用服务对应方法的任务** 

参考代码如下：
`Illuminate\Support\Facades\Cache` 类

    namespace Illuminate\Support\Facades;

    /**
     * @see \Illuminate\Cache\CacheManager
     * @see \Illuminate\Cache\Repository
     */
    // 继承 Facade
    class Cache extends Facade
    {
        // 用来确定该Facade对应那个服务的方法，其返回值就是在服务容器中注册的服务标识。
        protected static function getFacadeAccessor()
        {
            return 'cache';
        }
    }

`Illuminate\Support\Facades\Facade` 类

    abstract class Facade {
        // 静态方法重载
        public static function __callStatic($method, $args)
        {
            // 确定对应的服务对象
            $instance = static::getFacadeRoot();

            if (! $instance) {
                throw new RuntimeException('A facade root has not been set.');
            }
            // 调用服务对象的方法
            return $instance->$method(...$args);
        }
        
        // 从服务容器中解析对象
         public static function getFacadeRoot()
        {
            // 利用某个Facade的getFacadeAccess()方法获取服务名称，解析服务对象
            return static::resolveFacadeInstance(static::getFacadeAccessor());
        }
    }

以上代码可以清晰看到Facade的结构以及方法。了解到这里就应该可以知道Facade的作用以及其如何实现的了。

## # Facade与助手函数
类似于简化语法的操作，除了使用Facade来实现外，Laravel还提供了函数语法，称之为助手函数，helper。其目的都是一致的，就是简化调用语法。
`Cache::get('key')` 这个语法可以更简单的由 `cache('key')` 来实现。是不是语法更简单了呢。

其实这两个语法的本质是一样的，我们可以观察 `cache()` 函数的实现：

定义与：`/vendor/laravel/framework/src/Illuminate/Foundation/helpers.php`
    function cache()
    {
        $arguments = func_get_args();

        // 如果只有一个参数，则解析出cache服务，完成get操作
        if (is_string($arguments[0])) {
            return app('cache')->get($arguments[0], isset($arguments[1]) ? $arguments[1] : null);
        }

        // 其他代码略...

        // 如果只有两个参数，则解析出cache服务，完成set操作
        return app('cache')->put(key($arguments[0]), reset($arguments[0]), $arguments[1]);
    }
} 

可以看出来，cache这个助手函数，也是完成解析服务，调用方法操作的工作。与Facade是一致的！

** 那么疑问来了：是不是功能重复了？我们该选择使用Facade还是Helper？**
是重复了，但是对于Cache来说，仅仅重复了put和get这两个操作。
助手函数，通常仅仅实现了最最常用的服务功能。而Facede就是直接解析出来服务，调用启发方法，换句话说，就是提供完善的服务功能。
这样就可以选择了，如果需要获取最简单的代码来实现最常用的功能，那么有些助手函数即可实现，例如缓存的设置和获取。但是如果需要完成复杂完成的服务功能，就需要使用Facade了，例如缓冲带清空，删除，拉取等工作。

到此，就完成了Facade的说明。下面看看文档中都说了什么。

## # 官方文档概要

文档中，提供了下面的内容：

* 如何使用Facade。
* Facade与助手函数功能类似。
* Facade结构和语法实现原理。
* 核心的Facade列表

## # 结语

总结一下：

* Facade，就是访问服务方法的快捷语法，不同我们从服务容器中手动解析，直接调用封装好的Facade即可完成任务。
* 其语法实现是利用PHP类的静态方法重载来实现的。
* 更加快捷的语法是助手函数，但是助手函数不能提供完善的服务功能，仅仅是最常用的功能，很多时候还是要使用Facade。

![Laravel-helloKang](http://www.hellokang.net/asset/image/kang-laravel.png)