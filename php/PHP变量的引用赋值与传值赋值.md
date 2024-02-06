PHP变量的引用赋值与传值赋值
---

## 一、使用 [memory_get_usage()](http://php.net/manual/zh/function.memory-get-usage.php) 查看PHP内存使用量

### 1. 传值赋值

```php
// 定义一个变量
$a = range(0, 10000);
var_dump(memory_get_usage());

// 定义变量b，将a变量的值赋值给b
$b = $a;
var_dump(memory_get_usage());

// 对a进行修改
// COW: Copy-On-Write
$a = range(0, 10000);
var_dump(memory_get_usage());
```
输出结果：
```
int(989768)
int(989856)
int(1855608)
```

#### **定义一个变量 `$a = range(0, 10000);`**

![定义一个变量 $a = range(0, 10000)](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_190024.jpg)

#### **`$b = $a;`**

![$b = $a](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_185929.jpg)

#### **对a进行修改 `$a = range(0, 10000);`**

![对a进行修改 $a = range(0, 10000)](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_190230.jpg)

> PHP写时复制机制（Copy-on-Write，也缩写为COW）

顾名思义，就是在写入时才真正复制一份内存进行修改。
COW最早应用在Unix系统中对线程与内存使用的优化，后面广泛的被使用在各种编程语言中，如C++的STL等。 
在PHP内核中，COW也是主要的内存优化手段。
在通过变量赋值的方式赋值给变量时，不会申请新内存来存放新变量的值，而是简单的通过一个计数器来共用内存。只有在其中的一个引用指向变量的值发生变化时，才申请新空间来保存值内容，以减少对内存的占用。
在很多场景下PHP都使用COW进行内存的优化。比如：变量的多次赋值、函数参数传递，并在函数体内修改实参等。

### 2. 引用赋值

```php
// 定义一个变量
$a = range(0, 10000);
var_dump(memory_get_usage());

// 定义变量b，将a变量的引用赋给b
$b = &$a;
var_dump(memory_get_usage());

// 对a进行修改
$a = range(0, 10000);
var_dump(memory_get_usage());
```
输出结果：
```
int(989760)
int(989848)
int(989840)
```


#### **定义一个变量 `$a = range(0, 10000);`**

![定义一个变量 $a = range(0, 10000)](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_190857.jpg)

#### **定义变量b，将a变量的引用赋给b `$b = &$a;`**

![定义变量b，将a变量的引用赋给b](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_190949.jpg)

#### **对a进行修改 `$a = range(0, 10000);`**

![对a进行修改](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_191022.jpg)


## 二、使用 `xdebug_debug_zval()` 查看变量的引用情况

> `xdebug_debug_zval()` 用于显示变量的信息。需要安装xdebug扩展。

### 1. 传值赋值

```php
$a = 1;
xdebug_debug_zval('a');

// 定义变量b，把a的值赋值给b
$b = $a;
xdebug_debug_zval('a');
xdebug_debug_zval('b');

// a进行写操作
$a = 2;
xdebug_debug_zval('a');
xdebug_debug_zval('b');
```
输出结果：
```
a: (refcount=1, is_ref=0)=1
a: (refcount=2, is_ref=0)=1
b: (refcount=2, is_ref=0)=1
a: (refcount=1, is_ref=0)=2
b: (refcount=1, is_ref=0)=1
```

#### **定义变量 `$a = 1;`**
```php
$a = 1;
xdebug_debug_zval('a');
```
输出
```
a: (refcount=1, is_ref=0)=1
```
`refcount=1` 表示该变量指向的内存地址的引用个数变为1
`is_ref=0` 表示该变量不是引用

![引用个数](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_192206.jpg)

#### **定义变量 `$b` ，把 `$a` 的值赋给 `$b`， `$b = $a;`**
```
$b = $a;
xdebug_debug_zval('a');
xdebug_debug_zval('b');
```
输出
```
a: (refcount=2, is_ref=0)=1
b: (refcount=2, is_ref=0)=1
```
`refcount=2` 表示该变量指向的内存地址的引用个数变为2
`is_ref=0` 表示该变量不是引用

![refcount=2](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_192805.jpg)

#### **对变量 `$a` 进行写操作 `$a = 2;`**
```
$a = 2;
xdebug_debug_zval('a');
xdebug_debug_zval('b');
```
输出
```
a: (refcount=1, is_ref=0)=2
b: (refcount=1, is_ref=0)=1
```
因为COW机制，对变量 `$a` 进行写操作时，会为变量 `$a` 新分配一块内存空间，用于存储变量 `$a` 的值。
此时 `$a` 和  `$b` 指向的内存地址的引用个数都变为1。

![对变量 $a 进行写操作](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_192916.jpg)


### 2. 引用赋值

```php
$a = 1;
xdebug_debug_zval('a');

// 定义变量b，把a的引用赋给b
$b = &$a;
xdebug_debug_zval('a');
xdebug_debug_zval('b');

// a进行写操作
$a = 2;
xdebug_debug_zval('a');
xdebug_debug_zval('b');
```

```
a: (refcount=1, is_ref=0)=1
a: (refcount=2, is_ref=1)=1
b: (refcount=2, is_ref=1)=1
a: (refcount=2, is_ref=1)=2
b: (refcount=2, is_ref=1)=2
```

#### **定义变量 `$a = 1;`**
```php
$a = 1;
xdebug_debug_zval('a');
```
输出
```
a: (refcount=1, is_ref=0)=1
```
`refcount=1` 表示该变量指向的内存地址的引用个数变为1
`is_ref=0` 表示该变量不是引用

![$a = 1](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_193406.jpg)

#### **定义变量 `$b` ，把 `$a` 的引用赋给 `$b`， `$b = &$a;`**
```
$b = &$a;
xdebug_debug_zval('a');
xdebug_debug_zval('b');
```
输出
```
a: (refcount=2, is_ref=1)=1
b: (refcount=2, is_ref=1)=1
```
`refcount=2` 表示该变量指向的内存地址的引用个数变为2
`is_ref=1` 表示该变量是引用

![$b = &$a](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_193434.jpg)


#### **对变量 `$a` 进行写操作 `$a = 2;`**
```
$a = 2;
xdebug_debug_zval('a');
xdebug_debug_zval('b');
```
输出
```
a: (refcount=2, is_ref=1)=2
b: (refcount=2, is_ref=1)=2
```
因为变量 `$a` 和变量 `$b` 指向相同的内存地址，其实引用。
对变量 `$a` 进行写操作时，会直接修改指向的内存空间的值，因此变量 `$b` 的值会跟着一起改变。

![对变量 $a 进行写操作](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_193538.jpg)


## 三、当变量时引用时，[unset()](http://www.php.net/manual/zh/function.unset.php)只会取消引用，不会销毁内存空间

```php
$a = 1;
$b = &$a;

// unset 只会取消引用，不会销毁内存空间
unset($b);

echo $a;
```
输出
```
1
```

#### **定义变量 `$a` ，并将 `$a` 的引用赋给变量  `$b`**
```
$a = 1;
$b = &$a;
```
![定义变量 $a，并将 $a 的引用赋给变量 $b](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_193902.jpg)

#### **销毁 `$b`**
```
unset($b);
```
![销毁 $b](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_193938.jpg)

#### **输出 `$a`**

虽然销毁的 `$b`，但是 `$a` 的引用和内存空间依旧存在。
```
echo $a;
```
输出
```
1
```

## 四、php中对象本身就是引用赋值


```php
class Person
{
    public $age = 1;
}

$p1 = new Person;
xdebug_debug_zval('p1');

$p2 = $p1;
xdebug_debug_zval('p1');
xdebug_debug_zval('p2');

$p2->age = 2;
xdebug_debug_zval('p1');
xdebug_debug_zval('p2');
```

```
p1: (refcount=1, is_ref=0)=class Person { public $age = (refcount=2, is_ref=0)=1 }
p1: (refcount=2, is_ref=0)=class Person { public $age = (refcount=2, is_ref=0)=1 }
p2: (refcount=2, is_ref=0)=class Person { public $age = (refcount=2, is_ref=0)=1 }
p1: (refcount=2, is_ref=0)=class Person { public $age = (refcount=1, is_ref=0)=2 }
p2: (refcount=2, is_ref=0)=class Person { public $age = (refcount=1, is_ref=0)=2 }
```

#### **实例化对象 `$p1 = new Person;`**
```php
$p1 = new Person;
xdebug_debug_zval('p1');
```
输出
```
p1: (refcount=1, is_ref=0)=class Person { public $age = (refcount=2, is_ref=0)=1 }
```
`refcount=1` 表示该变量指向的内存地址的引用个数变为1
`is_ref=0` 表示该变量不是引用

#### **把 `$p1` 赋给 `$p2`**
```
$p2 = $p1;
xdebug_debug_zval('p1');
xdebug_debug_zval('p2');
```
输出
```
p1: (refcount=2, is_ref=0)=class Person { public $age = (refcount=2, is_ref=0)=1 }
p2: (refcount=2, is_ref=0)=class Person { public $age = (refcount=2, is_ref=0)=1 }
```
`refcount=2` 表示该变量指向的内存地址的引用个数变为2

![$p2 = $p1](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_194507.jpg)


#### **对 `$p2` 中的属性 `age` 进行写操作**
```
$p2->age = 2;
xdebug_debug_zval('p1');
xdebug_debug_zval('p2');
```
输出
```
p1: (refcount=2, is_ref=0)=class Person { public $age = (refcount=1, is_ref=0)=2 }
p2: (refcount=2, is_ref=0)=class Person { public $age = (refcount=1, is_ref=0)=2 }
```
因为php中对象本身就是引用赋值。对 `$p2` 中的属性 `age` 进行写操作时，会直接修改指向的内存空间的值，因此变量 `$p1` 的 `age` 属性的值会跟着一起改变。


## 五、实战例题分析

```
/**
 * 写出如下程序的输出结果
 *
 * $d = ['a', 'b', 'c'];
 *
 * foreach($d as $k => $v)
 * {
 *    $v = &$d[$k];
 * }
 * 
 * 程序运行时，每一次循环结束后变量 $d 的值是什么？请解释。
 * 程序执行完成后，变量 $d 的值是什么？请解释。
 */
```

### 1. 第一次循环

#### **推算出进入 `foreach` 时 `$v`、`$d[$k]` 的值**
```
$k = 0
$v = 'a'
$d[$k] = $d[0] = 'a'
```
此时，`$v` 和 `$d[0]` 在内存中分别开辟了一块空间

![$v 和 $d[0] 在内存中分别开辟了一块空间](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_195622.jpg)

#### **`$v = &$d[0]` 改变了 $v 指向的内存地址**
```
$v = &$d[0]
```

![$v = &$d[0] 改变了 $val 指向的内存地址](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_195415.jpg)

#### **第一次循环后 $d 的值：**

	['a', 'b', 'c']

### 2. 第二次循环

#### **进入 `foreach` 时 `$v` 被赋值为 'b'，此时`$v`指向的内存地址与 `$d[0]` 相同，且为引用，因此 `$d[0]` 的值被修改为 'b'**

`$v = 'b'`  => `$d[0] = 'b'`

![$v = ‘b’  => $d[0] = ‘b’](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_195842.jpg)

#### **推算出进入 `foreach` 时 `$d[$k]` 的值**
```
$k = 1
$d[$k] = $d[1] = 'b'
```
![$d[2] = ‘b’](http://md.ws1031.cn/xsj/2018_8_28_2018-08-28_170603.jpg)

#### **`$v = &$d[1]` 改变了 $v 指向的内存地址**

```
$v = &$d[1]
```

![$v = &$d[1]](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_200018.jpg)

#### **第二次循环后 `$d` 的值**

	['b', 'b', 'c']

### 3. 第三次循环

#### **进入 `foreach` 时 `$v` 被赋值为 'c'，此时`$v`指向的内存地址与 `$d[1]` 相同，且为引用，因此 `$d[1]` 的值被修改为 'c'**

`$v = 'c'`  => `$d[1] = 'c'`

![$v = ‘c’  => $d[1] = ‘c’](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_200244.jpg)

#### **推算出进入 `foreach` 时 `$d[$k]` 的值**
```
$k = 2
$d[2] = 'c'
```
![$d[2] = ‘c’](http://md.ws1031.cn/xsj/2018_8_28_2018-08-28_170745.jpg)

#### **`$v = &$d[2]` 改变了 $v 指向的内存地址**
```
$v = &$d[2]
```

![$v = &$d[2]](http://md.ws1031.cn/xsj/2018_8_24_2018-08-24_200158.jpg)

#### **第三次循环后 `$d` 的值**

	['b', 'c', 'c']

### 4. 实测

```php
$d = ['a', 'b', 'c'];

foreach ($d as $k=>$v)
{
    $v = &$d[$k];
    print_r($d);
}

print_r($d);
```

#### **输出：**
```
Array
(
    [0] => a
    [1] => b
    [2] => c
)
Array
(
    [0] => b
    [1] => b
    [2] => c
)
Array
(
    [0] => b
    [1] => c
    [2] => c
)
Array
(
    [0] => b
    [1] => c
    [2] => c
)
```