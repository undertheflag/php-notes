###  array_merge 和 + 的区别

---------------------

**示例 1：**

```php
$first = [
    'a',
    'b',
];

$second = [
    'c',
];
```

```php
array_merge($first, $second);

[
    'a',
    'b',
    'c',
]

$first + $second;

[
    'a',
    'b',
]
```

----------------------------------
**示例 2：**

```php
$first = ['a'];
$second = ['b'];
$third = ['c'];
```

```php
array_merge($first, $second, $third)

[
    'a',
    'b',
    'c',
]

$first + $second + $third

[
    'a',
]
```

-----------------------------

**示例 3：**

```php
$first = [
  '3' => 'a',
  '5' => 'b'
];

$second = [
  'C' => 'c',
  '4' => 'd'
];
```

```php
array_merge($first, $second);

[
    0 => 'a',
    1 => 'b',
    'C' => 'c',
    2 => 'd'
]

$first + $second;

[
    3 => 'a',
    5 => 'b',
    'C' => 'c',
    4 => 'd'
]
```

------------------------------
**示例 4：**

```php
$first = [
    'A' => [
        'B' => true,
        'C' => true,
    ],
];

$second = [
    'A' => [
        'B' => false,
        'C' => false,
    ],
];
```

```php
array_merge($first, $second);

[
    'A' => [
        'B' => false,
        'C' => false,
    ],
]

$first + $second;

[
    'A' => [
        'B' => true,
        'C' => true,
    ],
]

array_merge_recursive($first, $second);

[
    'A' => [
        'B' => [
            true,
            false,
        ],
        'C' => [
            true,
            false,
        ],
    ],
]
```

--------------------------
**结论：**

* array_merge 键名相同时，对数值键不覆盖，merge后会对数值重新排序，且从零开始；对字符键进行覆盖，后面的覆盖前面的
* \+ 键名相同时，只保留先出现的，后出现的全部丢弃；+ 后`不会`对数值重新排序
* array_merge_recursive 不会进行键名覆盖，而是将多个相同键名的值递归组成一个数组

