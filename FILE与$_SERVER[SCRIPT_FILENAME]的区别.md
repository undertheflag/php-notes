## __FILE__与$\_SERVER["SCRIPT_FILENAME"]的区别
`__FILE__`：返回`"__FILE__"`所在文件的绝对路径；
`$_SERVER["SCRIPT_FILENAME"]`：返回顶层执行文件的绝对路径

```php
// D:/www/test.php 
require 'common/inc.php'; 
```

```php
// D:/www/common/inc.php 
echo 'SCRIPT_FILENAME 为：' . $_SERVER['SCRIPT_FILENAME']; 
echo '<br />'; 
echo '__FILE__为：' . __FILE__; 
```

执行test.php，显示结果为：

SCRIPT_FILENAME 为：D:/www/test.php
\_\_FILE\_\_为：D:/www/common/inc.php

获取当前执行脚本的目录:

```php
$dir = dirname($_SERVER['SCRIPT_FILENAME']);
```

