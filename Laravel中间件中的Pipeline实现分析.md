# Laravel中间件中的Pipeline实现分析

## `Pipeline`核心代码：

```php
class Pipeline
{
    private $passable;
    private $pipes;
	
	//设置管道要处理的数据
    public function send($passable)
    {
        $this->passable = $passable;

        return $this;
    }
	
	//设置数据需要经过的各层管道
    public function through($pipes)
    {
        $this->pipes = is_array($pipes) ? $pipes : func_get_args();

        return $this;
    }
	
	//运行管道，并将管道结果交个一个闭包回调进行处理
    public function then(Closure $destination)
    {
        $pipeline = array_reduce(
            array_reverse($this->pipes),
            $this->carry(),
            $this->prepareDestination($destination)
        );

        return $pipeline($this->passable);
    }
	
	//将每一层中间件进行闭包处理，得到洋葱闭包
    protected function carry()
    {
        return function ($stack, $pipe) {
            return function ($passable) use ($stack, $pipe) {
                return $pipe($passable, $stack);
            };
        };
    }
	
	//获取最后洋葱闭包最后一层
    protected function prepareDestination(Closure $destination)
    {
        return function ($passable) use ($destination) {
            return $destination($passable);
        };
    }
}
```

## 测试代码：

```php
$pipe1 = function ($poster, Closure $next) {
    $poster += 1;
    echo "pipe1: $poster\n";
    return $next($poster);
};

$pipe2 = function ($poster, Closure $next) {
    if ($poster > 7) {
        return $poster;
    }

    $poster += 3;
    echo "pipe2: $poster\n";
    return $next($poster);
};

$pipe3 = function ($poster, Closure $next) {
    $result = $next($poster);
    echo "pipe3: $result\n";
    return $result * 2;
};

$pipe4 = function ($poster, Closure $next) {
    $poster += 2;
    echo "pipe4 : $poster\n";
    return $next($poster);
};

$pipes = [$pipe1, $pipe2, $pipe3, $pipe4];

function dispatcher($poster, $pipes)
{
    echo "result: " . (new Pipeline)->send($poster)->through($pipes)->then(function ($poster) {
            echo "received: $poster\n";
            return 3;
        }) . "\n";
}

echo "==> action 1:\n";
dispatcher(5, $pipes);
echo "==> action 2:\n";
dispatcher(7, $pipes);
```

## 运行结果：

```
==> action 1:
pipe1: 6
pipe2: 9
pipe4 : 11
received: 11
pipe3: 3
result: 6
==> action 2:
pipe1: 8
result: 8

```

## 代码运行分析

代码的核心片段是：

```php
$pipeline = array_reduce(
            array_reverse($this->pipes),
            $this->carry(),
            $this->prepareDestination($destination)
        );
```

理解的关键在于函数`array_reduce`，下面对此函数做点讲解。

```php
function sum(int $carry, int $i)  int
{
	return $carry + $i;
}
$summed = array_reduce([l, 2, 3, 4], 'sum', 0);

```
上面代码执行的过程：
```php
$sum($sum($sum($sum(0,1),2),3),4)；
//数学式子是：((((0+1)+2)+3)+4)
```
对代码稍加修改

```php
$summed = array_reduce(array_reverse([l, 2, 3, 4]), 'sum', 0);
```
上面代码执行的过程：
```php
$sum($sum($sum($sum(0,4),3),2),1);
//数学式子是：((((0+4)+3)+2)+1)
```
数学上等价于
```php
(1+(2+(3+(4+0))))
```
因此，`array_reduce`的通用叫法是`left fold`；加入`array_reverse`后，就是`right fold`了。

### 对测试代码的核心片段进行化简：

```php
(new Pipeline)
    ->send($poster)
    ->through($pipes)
    ->then(function ($poster) {
                        echo "received: $poster\n";
                        return 3;
                    });
```
简化为

```php
array_reduce(
	array_reverse([$pipe1, $pipe2, $pipe3, $pipe4]),
	function ($stack, $pipe){
		return function ($passable) use ($stack, $pipe) {
			return $pipe($passable, $stack);
		};
    },
	function ($passable) use ($destination) {
		return $destination($passable);
	}
);
```
其中
```php
$destination = function ($poster) {
    echo "received: $poster\n";
    return 3;
};
```

令
```php
$carry = function ($stack, $pipe){
    return function ($passable) use ($stack, $pipe) {
    	return $pipe($passable, $stack);
    };
};
```
继续化简：

```php
$carry(
	$carry(
		$carry(
			$carry(
				$destination, 
                $pipe4
            ),
			$pie3
		),
		$pipe2
	),
	$pipe1
);
```

代入变量，继续
```php
$carry(
	$carry(
		$carry(
            $result1,
			$pie3
		),
		$pipe2
	),
	$pipe1
);

$result1 = function ($passable) use ($destination, $pipe4) {
	return $pipe4($passable, $destination);
};
//继续
$carry(
	$carry(
		$result2,
		$pipe2
	),
	$pipe1
);

$result2 = function ($passable) use($result1, $pipe3) {
	return $pipe3($passable, $result1);
};
//继续
$carry(
	$result3,
	$pipe1
);

$result3 = function ($passable) use ($result2, $pipe2) {
	return $pipe2($passable, $result2);
};
//继续
($result4);

$result4 = function ($passable) use ($result3, $pipe1) {
	return $pipe1($passable, $result3);
};

dispatcher(5, $pipes);
//结果应该等同于
echo "result: ". $result4(5) . "\n";
```

