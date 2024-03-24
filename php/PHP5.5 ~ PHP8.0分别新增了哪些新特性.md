PHP5.6 ~ PHP7.2 新特性
---

> 官方文档：http://php.net/manual/zh/appendices.php

### 一、PHP 5.6.x

##### **使用表达式定义常量**

在之前的 PHP 版本中， 必须使用静态值来定义常量，声明属性以及指定函数参数默认值。 现在你可以使用包括数值、字符串字面量以及其他常量在内的数值表达式来 定义常量、声明属性以及设置函数参数默认值。

```php
<?php
const ONE = 1;
const TWO = ONE * 2;

class C {
	const THREE = TWO + 1;
	const ONE_THIRD = ONE / self::THREE;
	const SENTENCE = 'The value of THREE is '.self::THREE;
}
```

现在可以通过 const 关键字来定义类型为 array 的常量。
```php
<?php
const ARR = ['a', 'b'];

echo ARR[0];
```
##### **使用 `...` 运算符定义变长参数函数**

现在可以不依赖 func_get_args()， 使用 ... 运算符 来实现 变长参数函数。
```php
<?php
function f($req, $opt = null, ...$params) {
    // $params 是一个包含了剩余参数的数组
    printf('$req: %d; $opt: %d; number of params: %d'."\n",
           $req, $opt, count($params));
}

f(1);
f(1, 2);
f(1, 2, 3);
f(1, 2, 3, 4);
?>
```
以上例程会输出：

	$req: 1; $opt: 0; number of params: 0
	$req: 1; $opt: 2; number of params: 0
	$req: 1; $opt: 2; number of params: 1
	$req: 1; $opt: 2; number of params: 2

##### **使用 `...` 运算符进行参数展开**

在调用函数的时候，使用 ... 运算符， 将 数组 和 可遍历 对象展开为函数参数。 在其他编程语言，比如 Ruby中，这被称为连接运算符。
```php
<?php
function add($a, $b, $c) {
    return $a + $b + $c;
}

$operators = [2, 3];
echo add(1, ...$operators);
?>
```
以上例程会输出：

	6

##### **use function 以及 use const**

use 运算符 被进行了扩展以支持在类中导入外部的函数和常量。 对应的结构为 use function 和 use const。
```php
<?php
namespace Name\Space {
    const FOO = 42;
    function f() { echo __FUNCTION__."\n"; }
}

namespace {
    use const Name\Space\FOO;
    use function Name\Space\f;

    echo FOO."\n";
    f();
}
?>
```
以上例程会输出：

	42
	Name\Space\f
	
##### **使用 hash_equals() 比较字符串避免时序攻击**

---

### 二、PHP 7.0.x

##### **标量类型声明**

标量类型声明 有两种模式: 强制 (默认) 和 严格模式。 现在可以使用下列类型参数（无论用强制模式还是严格模式）： 字符串(string), 整数 (int), 浮点数 (float), 以及布尔值 (bool)。
```php
<?php
// Coercive mode
function sumOfInts(int ...$ints)
{
    return array_sum($ints);
}

var_dump(sumOfInts(2, '3', 4.1));
```
以上例程会输出：

	int(9)

##### **返回值类型声明**

PHP 7 增加了对返回类型声明的支持。 类似于参数类型声明，返回类型声明指明了函数返回值的类型。可用的类型与参数声明中可用的类型相同。
```php
<?php

function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}
```

##### **null合并运算符**

由于日常使用中存在大量同时使用三元表达式和 isset()的情况， 我们添加了null合并运算符 (`??`) 这个语法糖。如果变量存在且值不为NULL， 它就会返回自身的值，否则返回它的第二个操作数。
```php
<?php
// Fetches the value of $_GET['user'] and returns 'nobody' if it does not exist.
$username = $_GET['user'] ?? 'nobody';
// This is equivalent to:
$username = isset($_GET['user']) ? $_GET['user'] : 'nobody';

// Coalesces can be chained: this will return the first defined value out of $_GET['user'], $_POST['user'], and 'nobody'.
$username = $_GET['user'] ?? $_POST['user'] ?? 'nobody';
?>
```

##### **太空船操作符（组合比较符）**

太空船操作符用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1。 比较的原则是沿用 PHP 的常规比较规则进行的。
```php
<?php
// 整数
echo 1 <=> '1'; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1

// 浮点数
echo '1.50' <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
 
// 字符串
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
?>
```

##### **通过 define() 定义常量数组**

Array 类型的常量现在可以通过 define() 来定义。在 PHP5.6 中仅能通过 const 定义。
```php
define('ANIMALS', [
    'dog',
    'cat',
    'bird'
]);

echo ANIMALS[1]; // 输出 "cat"
```

##### **Closure::call()**

Closure::call() 现在有着更好的性能，简短干练的暂时绑定一个方法到对象上闭包并调用它。
```php
<?php
class A {private $x = 1;}

// PHP 7 之前版本的代码
$getXCB = function() {return $this->x;};
$getX = $getXCB->bindTo(new A, 'A'); // 中间层闭包
echo $getX();

// PHP 7+ 及更高版本的代码
$getX = function() {return $this->x;};
echo $getX->call(new A);
```

以上例程会输出：
	
	1


##### **分组 `use` 声明**

从同一 namespace 导入的类、函数和常量现在可以通过单个 use 语句 一次性导入了。
```php
<?php

// PHP 7 之前的代码
use some\namespace\ClassA;
use some\namespace\ClassB;
use some\namespace\ClassC as C;

use function some\namespace\fn_a;
use function some\namespace\fn_b;
use function some\namespace\fn_c;

use const some\namespace\ConstA;
use const some\namespace\ConstB;
use const some\namespace\ConstC;

// PHP 7+ 及更高版本的代码
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
?>
```

##### **生成器可以返回表达式**

此特性基于 PHP 5.5 版本中引入的生成器特性构建的。 它允许在生成器函数中通过使用 return 语法来返回一个表达式 （但是不允许返回引用值）， 可以通过调用 Generator::getReturn() 方法来获取生成器的返回值， 但是这个方法只能在生成器完成产生工作以后调用一次。

##### **整数除法函数 intdiv()**

---

### PHP 7.1.x

##### **可为空（Nullable）类型**

参数以及返回值的类型现在可以通过在类型前加上一个问号使之允许为空。 当启用这个特性时，传入的参数或者函数返回的结果要么是给定的类型，要么是 null 。
```php
<?php

function testReturn(): ?string
{
    return 'elePHPant';
}

var_dump(testReturn());

function testReturn(): ?string
{
    return null;
}

var_dump(testReturn());

function test(?string $name)
{
    var_dump($name);
}

test('elePHPant');
test(null);
test();
```
以上例程会输出：

	string(10) "elePHPant"
	NULL
	string(10) "elePHPant"
	NULL
	Uncaught Error: Too few arguments to function test(), 0 passed in...

##### **Void 函数**

一个新的返回值类型void被引入。 返回值声明为 void 类型的方法要么干脆省去 return 语句，要么使用一个空的 return 语句。 对于 void 函数来说，NULL 不是一个合法的返回值。
```php
<?php
function swap(&$left, &$right) : void
{
    if ($left === $right) {
        return;
    }

    $tmp = $left;
    $left = $right;
    $right = $tmp;
}

$a = 1;
$b = 2;
var_dump(swap($a, $b), $a, $b);
```
以上例程会输出：

	null
	int(2)
	int(1)


##### **Symmetric array destructuring**

短数组语法（[]）现在作为list()语法的一个备选项，可以用于将数组的值赋给一些变量（包括在foreach中）。
```php
<?php
$data = [
    [1, 'Tom'],
    [2, 'Fred'],
];

// list() style
list($id1, $name1) = $data[0];

// [] style
[$id1, $name1] = $data[0];

// list() style
foreach ($data as list($id, $name)) {
    // logic here with $id and $name
}

// [] style
foreach ($data as [$id, $name]) {
    // logic here with $id and $name
}
```

##### **类常量可见性**

现在起支持设置类常量的可见性。
```php
<?php
class ConstDemo
{
    const PUBLIC_CONST_A = 1;
    public const PUBLIC_CONST_B = 2;
    protected const PROTECTED_CONST = 3;
    private const PRIVATE_CONST = 4;
}
```

##### **iterable 伪类**

现在引入了一个新的被称为iterable的伪类 (与callable类似)。 这可以被用在参数或者返回值类型中，它代表接受数组或者实现了Traversable接口的对象。 至于子类，当用作参数时，子类可以收紧父类的iterable类型到array 或一个实现了Traversable的对象。对于返回值，子类可以拓宽父类的 array或对象返回值类型到iterable。
```php
<?php
function iterator(iterable $iter) : iterable
{
    foreach ($iter as $val) {
        //
    }
}
```

##### **多异常捕获处理**

一个catch语句块现在可以通过管道字符(|)来实现多个异常的捕获。 这对于需要同时处理来自不同类的不同异常时很有用。
```php
<?php
try {
    // some code
} catch (FirstException | SecondException $e) {
    // handle first and second exceptions
}
```

##### **`list()`现在支持键名**

现在list()和它的新的[]语法支持在它内部去指定键名。这意味着它可以将任意类型的数组 都赋值给一些变量（与短数组语法类似）
```php
<?php
$data = [
    ["id" => 1, "name" => 'Tom'],
    ["id" => 2, "name" => 'Fred'],
];

// list() style
list("id" => $id1, "name" => $name1) = $data[0];

// [] style
["id" => $id1, "name" => $name1] = $data[0];

// list() style
foreach ($data as list("id" => $id, "name" => $name)) {
    // logic here with $id and $name
}

// [] style
foreach ($data as ["id" => $id, "name" => $name]) {
    // logic here with $id and $name
}
```

---

### 从PHP 7.1.x 移植到 PHP 7.2.x

##### **新的对象类型**

这种新的对象类型, object, 引进了可用于逆变（contravariant）参数输入和协变（covariant）返回任何对象类型。
```php
<?php

function test(object $obj) : object
{
    return new SplQueue();
}

test(new StdClass());
```

##### **允许重写抽象方法(Abstract method)**

当一个抽象类继承于另外一个抽象类的时候，继承后的抽象类可以重写被继承的抽象类的抽象方法。
```php
abstract class A
{
    abstract function test(string $s);
}
abstract class B extends A
{
    // overridden - still maintaining contravariance for parameters and covariance for return
    abstract function test($s) : int;
}
```

##### **扩展了参数类型**

重写方法和接口实现的参数类型现在可以省略了。不过这仍然是符合LSP，因为现在这种参数类型是逆变的。
```php
interface A
{
    public function Test(array $input);
}

class B implements A
{
    public function Test($input){} // type omitted for $input
}
```

##### **允许分组命名空间的尾部逗号**

命名空间可以在PHP 7中使用尾随逗号进行分组引入。
```php
use Foo\Bar\{
    Foo,
    Bar,
    Baz,
};
```

PHP5.5 ~ PHP7.2 新特性整理
---

> 官方文档：http://php.net/manual/zh/appendices.php

### 一、从PHP 5.5.x 移植到 PHP 5.6.x

##### **使用表达式定义常量**

在之前的 PHP 版本中， 必须使用静态值来定义常量，声明属性以及指定函数参数默认值。 现在你可以使用包括数值、字符串字面量以及其他常量在内的数值表达式来 定义常量、声明属性以及设置函数参数默认值。

```php
<?php
const ONE = 1;
const TWO = ONE * 2;

class C {
	const THREE = TWO + 1;
	const ONE_THIRD = ONE / self::THREE;
	const SENTENCE = 'The value of THREE is '.self::THREE;
}
```

现在可以通过 const 关键字来定义类型为 array 的常量。
```php
<?php
const ARR = ['a', 'b'];

echo ARR[0];
```
##### **使用 `...` 运算符定义变长参数函数**

现在可以不依赖 func_get_args()， 使用 ... 运算符 来实现 变长参数函数。
```php
<?php
function f($req, $opt = null, ...$params) {
    // $params 是一个包含了剩余参数的数组
    printf('$req: %d; $opt: %d; number of params: %d'."\n",
           $req, $opt, count($params));
}

f(1);
f(1, 2);
f(1, 2, 3);
f(1, 2, 3, 4);
?>
```
以上例程会输出：

	$req: 1; $opt: 0; number of params: 0
	$req: 1; $opt: 2; number of params: 0
	$req: 1; $opt: 2; number of params: 1
	$req: 1; $opt: 2; number of params: 2

##### **使用 `...` 运算符进行参数展开**

在调用函数的时候，使用 ... 运算符， 将 数组 和 可遍历 对象展开为函数参数。 在其他编程语言，比如 Ruby中，这被称为连接运算符。
```php
<?php
function add($a, $b, $c) {
    return $a + $b + $c;
}

$operators = [2, 3];
echo add(1, ...$operators);
?>
```
以上例程会输出：

	6

##### **use function 以及 use const**

use 运算符 被进行了扩展以支持在类中导入外部的函数和常量。 对应的结构为 use function 和 use const。
```php
<?php
namespace Name\Space {
    const FOO = 42;
    function f() { echo __FUNCTION__."\n"; }
}

namespace {
    use const Name\Space\FOO;
    use function Name\Space\f;

    echo FOO."\n";
    f();
}
?>
```
以上例程会输出：

	42
	Name\Space\f
	
##### **使用 hash_equals() 比较字符串避免时序攻击**

---

### 二、从PHP 5.6.x 移植到 PHP 7.0.x

##### **标量类型声明**

标量类型声明 有两种模式: 强制 (默认) 和 严格模式。 现在可以使用下列类型参数（无论用强制模式还是严格模式）： 字符串(string), 整数 (int), 浮点数 (float), 以及布尔值 (bool)。
```php
<?php
// Coercive mode
function sumOfInts(int ...$ints)
{
    return array_sum($ints);
}

var_dump(sumOfInts(2, '3', 4.1));
```
以上例程会输出：

	int(9)

##### **返回值类型声明**

PHP 7 增加了对返回类型声明的支持。 类似于参数类型声明，返回类型声明指明了函数返回值的类型。可用的类型与参数声明中可用的类型相同。
```php
<?php

function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}
```

##### **null合并运算符**

由于日常使用中存在大量同时使用三元表达式和 isset()的情况， 我们添加了null合并运算符 (`??`) 这个语法糖。如果变量存在且值不为NULL， 它就会返回自身的值，否则返回它的第二个操作数。
```php
<?php
// Fetches the value of $_GET['user'] and returns 'nobody' if it does not exist.
$username = $_GET['user'] ?? 'nobody';
// This is equivalent to:
$username = isset($_GET['user']) ? $_GET['user'] : 'nobody';

// Coalesces can be chained: this will return the first defined value out of $_GET['user'], $_POST['user'], and 'nobody'.
$username = $_GET['user'] ?? $_POST['user'] ?? 'nobody';
?>
```

##### **太空船操作符（组合比较符）**

太空船操作符用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1。 比较的原则是沿用 PHP 的常规比较规则进行的。
```php
<?php
// 整数
echo 1 <=> '1'; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1

// 浮点数
echo '1.50' <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
 
// 字符串
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
?>
```

##### **通过 define() 定义常量数组**

Array 类型的常量现在可以通过 define() 来定义。在 PHP5.6 中仅能通过 const 定义。
```php
define('ANIMALS', [
    'dog',
    'cat',
    'bird'
]);

echo ANIMALS[1]; // 输出 "cat"
```

##### **Closure::call()**

Closure::call() 现在有着更好的性能，简短干练的暂时绑定一个方法到对象上闭包并调用它。
```php
<?php
class A {private $x = 1;}

// PHP 7 之前版本的代码
$getXCB = function() {return $this->x;};
$getX = $getXCB->bindTo(new A, 'A'); // 中间层闭包
echo $getX();

// PHP 7+ 及更高版本的代码
$getX = function() {return $this->x;};
echo $getX->call(new A);
```

以上例程会输出：
	
	1


##### **分组 `use` 声明**

从同一 namespace 导入的类、函数和常量现在可以通过单个 use 语句 一次性导入了。
```php
<?php

// PHP 7 之前的代码
use some\namespace\ClassA;
use some\namespace\ClassB;
use some\namespace\ClassC as C;

use function some\namespace\fn_a;
use function some\namespace\fn_b;
use function some\namespace\fn_c;

use const some\namespace\ConstA;
use const some\namespace\ConstB;
use const some\namespace\ConstC;

// PHP 7+ 及更高版本的代码
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
?>
```

##### **生成器可以返回表达式**

此特性基于 PHP 5.5 版本中引入的生成器特性构建的。 它允许在生成器函数中通过使用 return 语法来返回一个表达式 （但是不允许返回引用值）， 可以通过调用 Generator::getReturn() 方法来获取生成器的返回值， 但是这个方法只能在生成器完成产生工作以后调用一次。

##### **整数除法函数 intdiv()**

---

### PHP 7.1.x

##### **可为空（Nullable）类型**

参数以及返回值的类型现在可以通过在类型前加上一个问号使之允许为空。 当启用这个特性时，传入的参数或者函数返回的结果要么是给定的类型，要么是 null 。
```php
<?php

function testReturn(): ?string
{
    return 'elePHPant';
}

var_dump(testReturn());

function testReturn(): ?string
{
    return null;
}

var_dump(testReturn());

function test(?string $name)
{
    var_dump($name);
}

test('elePHPant');
test(null);
test();
```
以上例程会输出：

	string(10) "elePHPant"
	NULL
	string(10) "elePHPant"
	NULL
	Uncaught Error: Too few arguments to function test(), 0 passed in...

##### **Void 函数**

一个新的返回值类型void被引入。 返回值声明为 void 类型的方法要么干脆省去 return 语句，要么使用一个空的 return 语句。 对于 void 函数来说，NULL 不是一个合法的返回值。
```php
<?php
function swap(&$left, &$right) : void
{
    if ($left === $right) {
        return;
    }

    $tmp = $left;
    $left = $right;
    $right = $tmp;
}

$a = 1;
$b = 2;
var_dump(swap($a, $b), $a, $b);
```
以上例程会输出：

	null
	int(2)
	int(1)


##### **Symmetric array destructuring**

短数组语法（[]）现在作为list()语法的一个备选项，可以用于将数组的值赋给一些变量（包括在foreach中）。
```php
<?php
$data = [
    [1, 'Tom'],
    [2, 'Fred'],
];

// list() style
list($id1, $name1) = $data[0];

// [] style
[$id1, $name1] = $data[0];

// list() style
foreach ($data as list($id, $name)) {
    // logic here with $id and $name
}

// [] style
foreach ($data as [$id, $name]) {
    // logic here with $id and $name
}
```

##### **类常量可见性**

现在起支持设置类常量的可见性。
```php
<?php
class ConstDemo
{
    const PUBLIC_CONST_A = 1;
    public const PUBLIC_CONST_B = 2;
    protected const PROTECTED_CONST = 3;
    private const PRIVATE_CONST = 4;
}
```

##### **iterable 伪类**

现在引入了一个新的被称为iterable的伪类 (与callable类似)。 这可以被用在参数或者返回值类型中，它代表接受数组或者实现了Traversable接口的对象。 至于子类，当用作参数时，子类可以收紧父类的iterable类型到array 或一个实现了Traversable的对象。对于返回值，子类可以拓宽父类的 array或对象返回值类型到iterable。
```php
<?php
function iterator(iterable $iter) : iterable
{
    foreach ($iter as $val) {
        //
    }
}
```

##### **多异常捕获处理**

一个catch语句块现在可以通过管道字符(|)来实现多个异常的捕获。 这对于需要同时处理来自不同类的不同异常时很有用。
```php
<?php
try {
    // some code
} catch (FirstException | SecondException $e) {
    // handle first and second exceptions
}
```

##### **`list()`现在支持键名**

现在list()和它的新的[]语法支持在它内部去指定键名。这意味着它可以将任意类型的数组 都赋值给一些变量（与短数组语法类似）
```php
<?php
$data = [
    ["id" => 1, "name" => 'Tom'],
    ["id" => 2, "name" => 'Fred'],
];

// list() style
list("id" => $id1, "name" => $name1) = $data[0];

// [] style
["id" => $id1, "name" => $name1] = $data[0];

// list() style
foreach ($data as list("id" => $id, "name" => $name)) {
    // logic here with $id and $name
}

// [] style
foreach ($data as ["id" => $id, "name" => $name]) {
    // logic here with $id and $name
}
```

---
### PHP 7.2.x

##### **新的对象类型**

这种新的对象类型, object, 引进了可用于逆变（contravariant）参数输入和协变（covariant）返回任何对象类型。
```php
<?php

function test(object $obj) : object
{
    return new SplQueue();
}

test(new StdClass());
```

##### **允许重写抽象方法(Abstract method)**

当一个抽象类继承于另外一个抽象类的时候，继承后的抽象类可以重写被继承的抽象类的抽象方法。
```php
abstract class A
{
    abstract function test(string $s);
}
abstract class B extends A
{
    // overridden - still maintaining contravariance for parameters and covariance for return
    abstract function test($s) : int;
}
```

##### **扩展了参数类型**

重写方法和接口实现的参数类型现在可以省略了。不过这仍然是符合LSP，因为现在这种参数类型是逆变的。
```php
interface A
{
    public function Test(array $input);
}

class B implements A
{
    public function Test($input){} // type omitted for $input
}
```

##### **允许分组命名空间的尾部逗号**

命名空间可以在PHP 7中使用尾随逗号进行分组引入。
```php
use Foo\Bar\{
    Foo,
    Bar,
    Baz,
};
```


### PHP 7.3.x

> https://www.php.net/manual/zh/migration73.new-features.php

#### Heredoc 和 Nowdoc 语法更加灵活

Heredoc 和 Nowdoc 语法支持闭合标记符的缩进，并且不再强制闭合标记符的换行。

在 PHP <= 7.2.x，Heredoc 和 Nowdoc 只能这样写，闭合标记符必须单独占一行，且前不允许出现包括空格在内的任何字符。

```
function a () {
    $a = json_decode(<<<JSON
{
    "id": 1, 
    "name": "张三"
}
JSON
, true);
    
    return $a;
}
```

在 PHP 7.3.x，允许这样写：

```
function a () {
    $a = json_decode(<<<JSON
        {
            "id": 1, 
            "name": "张三"
        }
        JSON, true);
    
    return $a;
}
```
闭合标记符可以跟内容一起缩进，且不必单独占用一行。

#### 数组解构支持引用赋值

```
[&$a, [$b, &$c]] = $d
```

### PHP 7.4.x

> https://www.php.net/manual/zh/migration74.php


### PHP 8.0.x

> https://www.php.net/manual/zh/migration74.php