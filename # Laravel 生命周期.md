# Laravel 生命周期

## index.php

```php
<?php
define('LARAVEL_START', microtime(true)); //应用启动时刻，可用来计算应用启动总时间；LARAVEL_START从未在框架中使用

require __DIR__.'/../vendor/autoload.php'; //加载Composer生成的自动加载文件；Composer 自动加载和框架没有任何关系，只是起到方便Class的使用和管理

$app = require_once __DIR__.'/../bootstrap/app.php'; //应用实例化,app.php用来做Application工厂

$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);

$response->send();

$kernel->terminate($request, $response);
```

### /bootstrap/app.php

```php 
<?php
$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
); // Application 类实例化；Application 类是 Container 类的子类

$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);

return $app;
```

#### Application实例化详解

实例化过程中调用的方法如下：

```php
$this->setBasePath($basePath);
$this->registerBaseBindings();
$this->registerBaseServiceProviders();
$this->registerCoreContainerAliases();
```

#####  setBasePath($rootDirectory)

注册相关路径到应用容器里。可以直接在app.php中继承Application，自定义路径：

```php
class MyCoolApplication extends Illuminate\Foundation\Application
{
    public function langPath()
    {
        // /languages/*
        return $this->basePath.DIRECTORY_SEPARATOR.'language';
    }

    public function configPath()
    {
        // resources/configuration/*.php
        return $this->resourcesPath().DIRECTORY_SEPARATOR.'configuration';
    }
}

// then new-up this class instead of Laravel's one
$app = new MyCoolApplication(
    realpath(__DIR__.'/../')
);
```
注册完成后，容器中的路径可以用`$app->make('path.{what-you-want}')`获取

#####　registerBaseBindings()

registerBaseBindings() 注册容器本身，得到一个全局单例容器；于是可以调用app(), app(Container::class) or app('Illuminate\Container\Container')。还会绑定package-loader，Illuminate\Foundation\PackageManifest，Note that this class's constructor does nothing but set up some base paths for itself and store the Filesystem class within itself. No package has been loaded yet. 

