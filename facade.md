# 外观
 
> 译者：[韩忠康](http://hellokang.net)。校对：南宁宁

---

[TOC]

## # 引言
外观提供对了对应用[服务容器](service-container)中可用的类的的“静态”访问接口。Laravel内置了很多用于访问Laravel功能的外观。Laravel的外观可看作服务容器中类的“静态代理”，与传统的静态方法相比，在保持了易测试性和扩展性的前提下提供了更简洁优雅的语法。

全部的Laravel外观定义在`Illuminate\Support\Facades`命名空间中。因此，可方便的通过下面的语法访问到外观：

	use Illuminate\Support\Facades\Cache;

	Route::get('/cache', function () {
	    return Cache::get('key');
	});

整篇文档，很多例子都使用外观来展示框架的各种功能。

## # 何时使用外观
外观有很多优势。提供了简洁明了的语法使得在不用记住冗长的类名就可以使用Laravel的各种功能。此外，对于PHP动态方法的独特使用，使得也很易于测试。

但，在使用外观时也必须要注意一些地方。最主要的风险就是类的体积在蔓延。由于外观易于使用并且不需要注入，这会导致在一个类中大量的使用外观进而会导致类体积持续增长。使用依赖注入，该问题会缓解，因为包含大量注入的构造方法会让我们注意到类在不断的变得臃肿。因此，使用外观时赢注意控制类的体积，使其职责有限可控。

> 在构建Laravel的第三方扩展包时，最好是通过注入[Laravel契约]()来代替使用外观。因为构建与Laravel之外的扩展包将不能使用Laravel的外观测试助手函数。

### ## 外观 Vs. 依赖注入
依赖注入的最大优点是可以更换注入类的实现。这在测试过程中很有用，因为可以注入一个mock模拟或stub存根，并断言在stub存根上调用了各种方法。

通常是不能模拟mock或存根stub真正的静态方法的。但由于外观使用动态方法来代理服务容器中解析对象的方法调用，因此就可以像测试注入类实例一样去测试外观。例如，下面的路由：

	use Illuminate\Support\Facades\Cache;

	Route::get('/cache', function () {
	    return Cache::get('key');
	});

我们可以编写下面的测试来验证`Cache::get`方法是否以我们期望的方式被调用：

	use Illuminate\Support\Facades\Cache;

	/**
	 * A basic functional test example.
	 *
	 * @return void
	 */
	public function testBasicExample()
	{
	    Cache::shouldReceive('get')
	         ->with('key')
	         ->andReturn('value');

	    $this->visit('/cache')
	         ->see('value');
	}

### ## 外观 Vs. 助手函数
除了外观，Laravel还提供了各种“助手”函数来实现常用的功能，就像生成视图，触发事件，分发任务，或发送HTTP响应等。很多助手函数与相关外观都执行相同的功能。例如下面对外观和助手函数的调用是相同的：

	return View::make('profile');

	return view('profile');

外观和助手函数之间没有实质差别。使用助手函数时，完全可以像测试外观一样测试他们。例如，看下面的路由：

	Route::get('/cache', function () {
	    return cache('key');
	});

底层调用时，`cache`助手函数一样会去调用`Cache`外观上的`get`方法。