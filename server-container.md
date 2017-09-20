# 服务容器

> 译者：[韩忠康](http://hellokang.net)。校对：南宁宁

---

[TOC]

## #引言
Laravel服务器容器是用来管理类依赖关系和执行依赖注入的用力工具。依赖注入这个高大上的术语其本质的是：类的依赖是通过构造方法或一些类似于setter方法注入到类中的。（作责添加：Laravel的应用就是一个服务容器）

下面看一个简单的例子：

	<?php

	namespace App\Http\Controllers;

	use App\User;
	use App\Repositories\UserRepository;
	use App\Http\Controllers\Controller;

	class UserController extends Controller
	{
	    /**
	     * The user repository implementation.
	     *
	     * @var UserRepository
	     */
	    protected $users;

	    /**
	     * Create a new controller instance.
	     *
	     * @param  UserRepository  $users
	     * @return void
	     */
	    public function __construct(UserRepository $users)
	    {
	        $this->users = $users;
	    }

	    /**
	     * Show the profile for the given user.
	     *
	     * @param  int  $id
	     * @return Response
	     */
	    public function show($id)
	    {
	        $user = $this->users->find($id);

	        return view('user.profile', ['user' => $user]);
	    }
	}

本例中，`UserController` 需要从数据源得到用户。因此，我们注入了一个可以获取用户服务。当前的代码环境，我们的 `UserRepository` 最有可能使用 [Eloquent]() 来从数据库获取用户信息。然而，由于该存储机制是注入而来的，可见我们可以很容易的由其他的机制来代替实现。 我们也可轻松地模拟，或者使用伪造的 `UserRepository` 来测试我们的应用。

深入理解Laravel的服务容器对于构建一个大型复杂的应用是很有必要的，以及贡献构建Laravel核心代码也是一样。

## #绑定

### 绑定基础
几乎所有服务容器的绑定都在 [服务提供者]() 中注册完成，因此下面的例子都使用服务提供者来完成服务容器使用的演示。

> 如果类没有实现任何接口则没有必要绑定到服务容器上。不必告知容器如何构建这些绑定的对象，因为容器可以利用反射自动解析这些对象。

#### 简单绑定
在服务提供者中，你可以通过 `$this->app` 属性来访问到容器。可以使用 `bind` 方法来注册一个绑定，传递想要绑定的类或接口名，以及可以返回类实例的 `闭包函数` 作为参数：

	$this->app->bind('HelpSpot\API', function ($app) {
	    return new HelpSpot\API($app->make('HttpClient'));
	});

注意，这个解析闭包函数会得到当前容器作为参数，可以用来解决嵌套的依赖关系，就是当前依赖的子依赖。

#### 绑定单例
利用`singleton` 方法绑定到容器的类或接口仅会被解析一次。一个单例一旦被解析，那么容器的后续的调用都会得到同一个对象：

	$this->app->singleton('HelpSpot\API', function ($app) {
	    return new HelpSpot\API($app->make('HttpClient'));
	});

#### 绑定实例
也可以绑定已存在的对象实例到容器，需要使用 `instance` 方法。这个给定的实例会在容器的后续的调用中被返回：

	$api = new HelpSpot\API(new HttpClient);

	$this->app->instance('HelpSpot\API', $api);

#### 绑定原生数据
有时类需要接收一些注入的类，有的时候类也需要注入一些原生数据，例如整型。可以轻松地使用上下文绑定来注入类可能需要的任何值：

	$this->app->when('App\Http\Controllers\UserController')
	          ->needs('$variableName')
	          ->give($value);

### 绑定接口的实现
绑定接口到某个给定的实现是服务容器一个非常强大的功能。例如，假定我们有 `EventPusher` 接口和 `RedisEventPusher` 实现。一旦我们完成了这个接口的 `RedisEventPusher`实现，我们通过下面的方法可以将其注册到服务容器中：

	$this->app->bind(
	    'App\Contracts\EventPusher',
	    'App\Services\RedisEventPusher'
	);

该语句告知服务容器当类需要 `EventPusher` 的实现时 `RedisEventPusher` 会被注入。现在可以在构造方法中使用 `EventPusher` 作为类型提示，或其他任何使用服务容器实现依赖注入的位置：

	use App\Contracts\EventPusher;

	/**
	 * Create a new class instance.
	 *
	 * @param  EventPusher  $pusher
	 * @return void
	 */
	public function __construct(EventPusher $pusher)
	{
	    $this->pusher = $pusher;
	}


### 上下文绑定
有时会有2个类同时使用一个接口，但是希望得到的是不同的实现。例如，两个控制器依赖于不同的 `Illuminate\Contracts\Filesystem\Filesystem` [契约](contracts) 的实现。Laravel为此提供了一种简单，平滑的操作接口：

	use Illuminate\Support\Facades\Storage;
	use App\Http\Controllers\PhotoController;
	use App\Http\Controllers\VideoController;
	use Illuminate\Contracts\Filesystem\Filesystem;

	$this->app->when(PhotoController::class)
	          ->needs(Filesystem::class)
	          ->give(function () {
	              return Storage::disk('local');
	          });

	$this->app->when(VideoController::class)
	          ->needs(Filesystem::class)
	          ->give(function () {
	              return Storage::disk('s3');
	          });

### 标签
有时，可能会需要解析某个“分类”下的全部绑定。例如，你准备实现一个可以从一组不同的 `Report` 报告实现中接收消息的报告聚合器。在注册了 `Report` 的实现后，可以使用 `tag` 方法为他们分配一个标签：

	$this->app->bind('SpeedReport', function () {
	    //
	});

	$this->app->bind('MemoryReport', function () {
	    //
	});

	$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports'); 

一旦服务器被打上标签，通过 `tagged` 方法可以将他们全部解析：

$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});


## #解析

### `make` 方法
使用 `make` 方法可以从服务容器中解析类的实例。`make` 方法以需要解析的类或接口名作为参数：

	$api = $this->app->make('HelpSpot\API');

如果你所在的代码不能访问 `$app` 变量，则可以使用全局助手函数 `resolve`：

	$api = resolve('HelpSpot\API');

如果类的一些依赖不能通过服务容器解析，可以通过向 `makeWith` 方法传递一个关联数组来完成注入解析：
	
	$api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);

### 自动注入
另外，也是最重要的，通过类的构造方法的类型提示来注入依赖，包括[控制器](controller)，[事件监听](event)，[队列](queue)，[中间件][middleware]都支持这种方式。实操中，这是最常用的解析方式。

例如，可以在控制器的构造方法中使用类型提示的方式注入一个仓库，该仓库会被自动解析，并注入到类中：

	<?php

	namespace App\Http\Controllers;

	use App\Users\Repository as UserRepository;

	class UserController extends Controller
	{
	    /**
	     * The user repository instance.
	     */
	    protected $users;

	    /**
	     * Create a new controller instance.
	     *
	     * @param  UserRepository  $users
	     * @return void
	     */
	    public function __construct(UserRepository $users)
	    {
	        $this->users = $users;
	    }

	    /**
	     * Show the user with the given ID.
	     *
	     * @param  int  $id
	     * @return Response
	     */
	    public function show($id)
	    {
	        //
	    }
	}

## #容器事件
服务容器在解析对象时会触发一个事件。可以监听该事件通过 `resolving` 方法：

	$this->app->resolving(function ($object, $app) {
	    // Called when container resolves object of any type...
	});

	$this->app->resolving(HelpSpot\API::class, function ($api, $app) {
	    // Called when container resolves objects of type "HelpSpot\API"...
	});

如你所见，被解析的对象会传递到这个回调函数中，允许在对象被消费之前设置一些附加的属性。

## PSR-11
Laravel的服务容器实现了PSR-11接口。因此，可以提示PSR-11容器接口来获取Laravel服务器容器的实例：

	use Psr\Container\ContainerInterface;

	Route::get('/', function (ContainerInterface $container) {
	    $service = $container->get('Service');

	    //
	});

> 如果在标识没有绑定到容器时就调用了get方法，会抛出异常。