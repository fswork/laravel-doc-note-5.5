# Contract（契约）-注

> 作者：[韩忠康](http://hellokang.net)
> 原文：[https://laravel.com/docs/5.5/contracts](https://laravel.com/docs/5.5/contracts)
> 中文（laravel-china翻译）：[https://d.laravel-china.org/docs/5.5/contracts](https://d.laravel-china.org/docs/5.5/contracts)
> Laravel版本：5.5
> 本文是Laravel文档中Facade章节的解析，建议与文档对照阅读。
---

[TOC]

[# 概述](#概述)
[# 缓存服务实现与Cache契约结构说明](#缓存服务实现与Cache契约结构说明)
[# 契约在编码时的使用](#契约在编码时的使用)
[# 官方文档概要](#官方文档概要)
[# 结语](#结语)

## # 概述
Contract，翻译过来叫契约、协议等。在Laravel-china的翻译中，是一个不翻词，这里也使用Contract来代替。
Contract就是接口Interface，用来规范某些服务的功能结构的，在Laravel中称之为契约。

以缓存操作为例，我们直接使用 `Cache::get()` 和 `Cache::put()` 即可完成缓存的获取和设置，语法很简单。此时问题来了：缓存的实现有很多种，例如文件缓存，Memcache缓存，Redis缓存等，要保证任何一种缓存的操作都具备get和put方法，如何保证？就是需要每种缓存服务的实现，都实现Cache契约，契约就是接口，就可以保证每种缓存的实现都具有一致的结构了。

## # 缓存服务实现与Cache契约结构说明
我们以Cache契约和一系列Cache服务实现为例，说一下这个结构：
缓存的相关实现，都位于：Illuminate\Cache\下：

Laravel5.5预设了：Apc, Array, File, Database, Memcached, Null, Redis, Taggable 多种缓存的具体实现方案，以常用的Redis为例，`Illuminate\Cache\RedisStore` 就是redis缓存服务实现，查看其源代码：

    namespace Illuminate\Cache;
    
    // 导入契约Store
    use Illuminate\Contracts\Cache\Store;
    use Illuminate\Contracts\Redis\Factory as Redis;

    // RedisStore 需要实现 Store，这个Store就是一个契约接口
    class RedisStore extends TaggableStore implements Store
    {
    }

可见，该Redis缓存功能类，需要实现一个Store的接口，这个接口就是 `Illuminate\Contracts\Cache\Store` ，在Laravel中称之为契约，看该Cache契约的实现：

    namespace Illuminate\Contracts\Cache;

    interface Store
    {
        // 获取缓存项
        public function get($key);
        // 设置缓存项
        public function put($key, $value, $minutes);
    }

接口契约中定义了关于缓存应该具备的方法，这样在缓存操作时，无论文件，Redis或者Apc缓存，都具有了统一的接口，不用担心使用上的语法差异了。

说了这么多，其实就是操作上的抽象层，将需要的操作提取，保证所有驱动实现具有统一的结构。这就是接口的常规目的，Laravel中叫成了契约而已，没有特殊功能。

## # 契约在编码时的使用

定义好的契约，在编程时的主要使用方式，是从服务容器中解析绑定到该契约上的实现。
还是以Cache缓存为例，我们使用时可以使用Facade来使用：`Cache::get('key')` 又或者使用助手函数来使用： `cache('key')` 。但是无论如何使用，语法都是 `app('cache')->get('key')` 这个语法来实现的（这个可以参考 Facade-注 这个章节）。
其中 `app('cache')` 就是从服务容器中，解析cache对应的服务实现。这个服务，实在Cache服务提供者中绑定到容器中的，参考代码如下：

文件 `Illuminate\Cache\CacheServiceProvider`:

    class CacheServiceProvider extends ServiceProvider
    {
        // 延迟到使用时才绑定
        protected $defer = true;
        public function register()
        {
            // 绑定cache到服务容器。
            $this->app->singleton('cache', function ($app) {
                return new CacheManager($app);
            });
        }

那也就意味着，如果需要在程序中使用缓存对象对象，仅仅需要从容器中解析出来即可。

另一个需要使用契约的场景，就是需要编写扩展时，为了保证扩展是通用的，扩展通常都需要实现某个契约，这样你的扩展就是一个通用的扩展。


## # 官方文档概要
在契约这篇官方文档中，主要说明契约的优势。内容如下：

* 使用契约原因，可以使得代码低耦合，保证代码的简洁性。
* 内置契约参考列表

文档中，有一个章节是比较 Facade 和 Contract。放在一起比较的原因，应该是在Laraval中，通常每个Contract都有对应的Facade。就像我们Cache，就有Cache的Facace和Cache的Contract。当使用Cache时，会导致我们确定不了当前时契约Contract还是门面Facade。这就是放在一起比较的原因吧，语法类似。
但除此之外，Contract与Facade功能完全不同，作用也不同，其实没有什么可比性。

* Facade，简化服务的调用语法的功能。
* Contract，定义一组服务的通用操作接口。



一组相关的服务，既需要通用的接口，也需要简化调用的操作。就是需要Contract也需要Facade，两者作用完全不同，其实不用混淆，不用放在一起比较的！

## # 结语

总结一下：
Laraval中，定义一组相关服务通用操作接口，称之为契约！就这么简单。

![Laravel-helloKang](http://www.hellokang.net/asset/image/kang-laravel.png)