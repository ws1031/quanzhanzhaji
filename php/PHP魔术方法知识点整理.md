PHP魔术方法知识点整理
---

> 代码使用PHP7.2语法编写

## 一、[构造函数和析构函数](https://www.php.net/manual/zh/language.oop5.decon.php)

### __construct()

#### **构造函数**

	__construct ([ mixed $args [, $... ]] ) : void

具有构造函数的类会在每次创建新对象时先调用此方法，所以非常适合在使用对象之前做一些初始化工作。

>  如果子类中定义了构造函数则不会隐式调用其父类的构造函数。要执行父类的构造函数，需要在子类的构造函数中调用 `parent::__construct()`。如果子类没有定义构造函数则会从父类继承。
> 当子类的 `__construct()` 与父类 `__construct()` 具有不同参数不同时，PHP 不会产生错误信息。这一点与其他的类方法不同。

#### **Code**

* 书籍类
```php
/**
 * 书籍类
 * Class Book
 */
class Book {
    /**
     * 书籍名称
     * @var string $name
     */
    public $name;

    /**
     * 书籍作者
     * @var string $author
     */
    public $author;

    /**
     * 构造函数
     * @param $name
     * @param $author
     */
    public function __construct(string $name, string $author)
    {
        $this->name = $name;
        $this->author = $author;
    }
}
```

* 计算机书籍类继承自Book类，且增加了一个属性 $category
```php
/**
 * 计算机书籍类
 * Class ComputerBook
 */
class ComputerBook extends Book {

    /**
     * 计算机书籍多了分类属性
     * @var string $category
     */
    public $category;

    /**
     * 构造函数
     * 这里的构造函数币父类的构造函数多了一个参数，但是PHP并不会报错。
     * 而其他的类方法在这种情况下会报错。
     * @param $name
     * @param $author
     * @param $category
     */
    public function __construct(string $name, string $author, string $category)
    {
        parent::__construct($name, $author);
        $this->category = $category;
    }
}
```

```php
$book = new Book('茶馆', '老舍');

echo <<<TXT
书籍名称： {$book->name}
书籍作者： {$book->author}
---\n
TXT
;

$computerBook = new ComputerBook('高性能MySQL','Baron', '数据库');

echo <<<TXT
书籍名称： {$computerBook->name}
书籍作者： {$computerBook->author}
书籍分类： {$computerBook->category}
---\n
TXT
;
```

	书籍名称： 茶馆
	书籍作者： 老舍
	---
	书籍名称： 高性能MySQL
	书籍作者： Baron
	书籍分类： 数据库
	---


### __destruct()

#### **析构函数**

	__destruct ( void ) : void

析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。

> 如果子类中定义了析构函数则不会隐式调用其父类的析构函数。要执行父类的析构函数，必须在子类的析构函数体中显式调用 `parent::__destruct()`。如果子类没有定义析构函数则会从父类继承。
> 析构函数即使在使用 `exit()` 终止脚本运行时也会被调用。
> 在析构函数中调用 `exit()` 将会中止其余关闭操作的运行。
> 析构函数中抛异常会导致致命错误。

#### **Code**

* 析构函数即使在使用 `exit()` 终止脚本运行时也会被调用。

```php
/**
 * 房子类
 * Class House
 */
class House {

    /**
     * 析构函数
     */
    public function __destruct()
    {
        echo "要开始拆房子了\n";
        $this->dismantleRoof();
        $this->demolishWall();
    }

    /**
     * 拆除屋顶
     */
    private function dismantleRoof()
    {
        echo "屋顶被拆除\n";
    }

    /**
     * 推掉墙
     */
    private function demolishWall()
    {
        echo "墙被推到\n";
    }
}

// 新建一座房子
$house = new House();
// 脚本停止运行
exit();
```

	要开始拆房子了
	屋顶被拆除
	墙被推到

* 在析构函数中调用 `exit()` 将会中止其余关闭操作的运行。

```php
/**
 * 别墅类
 * Class House
 */
class Villa extends House {

    /**
     * 析构函数
     */
    public function __destruct()
    {
        $this->demolishSwimmingPool();
        // 拆掉游泳池后后悔了，不想继续拆了
        exit();
        parent::__destruct();
    }

    /**
     * 拆掉游泳池
     */
    private function demolishSwimmingPool()
    {
        echo "拆掉游泳池\n";
    }
}

// 新建一栋别墅
$villa = new Villa();
// 脚本停止运行
exit();
```

	拆掉游泳池

* 析构函数中抛异常会导致致命错误。

```
/**
 * 大厦类
 * Class House
 */
class Mansion extends House {

    /**
     * 析构函数
     */
    public function __destruct()
    {
        $this->evacuateCrowd();
        parent::__destruct();
    }

    /**
     * 疏散大厦里的人
     */
    private function evacuateCrowd()
    {
        throw new Exception('大厦里还有人，不能拆！');
    }
}

// 新建一座大厦
$mansion = new Mansion();
// 脚本停止运行
exit();
```

	PHP Fatal error:  Uncaught Exception: 大厦里还有人，不能拆！

## 二、[对象复制](https://www.php.net/manual/zh/language.oop5.cloning.php#object.clone)

### __clone()

	__clone ( void ) : void

通过 clone 关键字克隆对象，当复制完成时，如果对象定义了 `__clone()` 方法，则新创建的对象（复制生成的对象）中的 `__clone()` 方法会被调用。

> 对象中的 __clone() 方法不能被直接调用。

#### Code

当对象被复制后，PHP 5 会对对象的所有属性执行一个浅复制（shallow copy）。所有的引用属性仍然会是一个指向原来的变量的引用。此时我们需要在 `__clone` 中强制复制一份对象中的引用属性。

```php
/**
 * Class MyCloneable
 */
class MyCloneable
{
    public $objectA;
    public $objectB;

    function __clone()
    {
        echo "MyCloneable::__clone\n";
        // 强制复制一份this->objectA， 否则仍然指向同一个对象
        $this->objectA = clone $this->objectA;
    }
}

/**
 * Class SubObject
 */
class SubObject
{
    /**
     * 静态计数器，对象每被实例化或clone一次，则+1
     * @var int
     */
    public static $counter = 0;

    public $instance;

    public function __construct()
    {
        echo "SubObject::__construct\n\n";
        $this->instance = ++ self::$counter;
    }

    public function __clone()
    {
        echo "SubObject::__clone\n\n";
        $this->instance = ++ self::$counter;
    }
}
```
```php
$myCloneable = new MyCloneable();

echo "实例化对象SubObject，并复制给 \$myCloneable->objectA ：\n";
$myCloneable->objectA = new SubObject();

echo "实例化对象SubObject，并复制给 \$myCloneable->objectB ：\n";
$myCloneable->objectB = new SubObject();

echo "复制对象 \$myCloneable \n";
$myCloneable2 = clone $myCloneable;

echo "-------------------\n";
echo "原始对象：\n";
print_r($myCloneable);

echo "复制的对象：\n";
print_r($myCloneable2);
```

	实例化对象SubObject，并复制给 $myCloneable->objectA ：
	SubObject::__construct

	实例化对象SubObject，并复制给 $myCloneable->objectB ：
	SubObject::__construct

	复制对象 $myCloneable
	MyCloneable::__clone
	SubObject::__clone

	-------------------
	原始对象：
	MyCloneable Object
	(
		[objectA] => SubObject Object
			(
				[instance] => 1
			)

		[objectB] => SubObject Object
			(
				[instance] => 2
			)

	)
	复制的对象：
	MyCloneable Object
	(
		[objectA] => SubObject Object
			(
				[instance] => 3
			)

		[objectB] => SubObject Object
			(
				[instance] => 2
			)

	)


## 三、[重载魔术方法](https://www.php.net/manual/zh/language.oop5.overloading.php)

PHP中的重载与其它绝大多数面向对象语言不同。传统的重载是用于提供多个同名的类方法，但各方法的参数类型和个数不同。  
PHP的重载（overloading）是指动态地创建类属性和方法。是通过魔术方法（magic methods）来实现的。

当调用当前环境下不可访问（**未定义或不可见**）的类属性或方法时，重载方法会被调用。

所有的重载方法都必须被声明为 public，且参数都不能通过引用传递。

属性重载的魔术方法有：`__set()`, `__get()`, `__isset()`, `__unset()`。
方法重载的魔术方法有：`__call()`, `__callStatic()`

### **属性重载**

### __set()

在给不可访问属性赋值时，`__set()` 会被调用。

	public __set ( string $name , mixed $value ) : void

`$name` 为要赋值的属性名，`$value` 为属性值。

### __get()

读取不可访问属性的值时，`__get()` 会被调用。

	public __get ( string $name ) : mixed

`$name` 为要访问的属性名。

### __isset()

当对不可访问属性调用 `isset()` 或 `empty()` 时，`__isset()` 会被调用。

	public __isset ( string $name ) : bool

`$name` 为属性名。

在除 isset() 外的其它语言结构中无法使用重载的属性，这意味着当对一个重载的属性使用 empty() 时，重载魔术方法将不会被调用。为避开此限制，必须将重载属性赋值到变量再使用 empty()。

### __unset()

当对不可访问属性调用 `unset()` 时，`__unset()` 会被调用。

	public __unset ( string $name ) : void

`$name` 为属性名。

#### **Code**

```php
class Property {
    /**
     * 被重载的数据保存在该属性
     * @var array
     */
    private $data = [];

    /**
     * 已经定义的属性不会调用重载魔术方法
     * @var int
     */
    public $declared = 1;

    /**
     * 只有从类外部访问属性时，才会调用重载魔术方法
     * @var int
     */
    private $hidden = 2;

    /**
     * @param $name
     * @param $value
     */
    public function __set($name, $value)
    {
        echo "Setting '{$name}' to '{$value}'\n";
        $this->data[$name] = $value;
    }

    /**
     * @param $name
     * @return mixed|null
     */
    public function __get($name)
    {
        echo "Getting '$name' ";
        if (array_key_exists($name, $this->data)) {
            return $this->data[$name];
        }
    }

    /**
     * @param $name
     * @return bool
     */
    public function __isset($name)
    {
        echo "Is '$name' set?\n";
        return isset($this->data[$name]);
    }

    /**
     * @param $name
     */
    public function __unset($name)
    {
        echo "Unsetting '$name'\n";
        unset($this->data[$name]);
    }

    /**
     * 非魔术方法
     * @return int
     */
    public function getHidden()
    {
        return $this->hidden;
    }
}
```
```php
$obj = new Property;

echo '为不存在的属性a赋值:';
$obj->a = 1;
echo '获取不存在的属性a的值：', $obj->a . "\n";

echo "----------------------------\n";

echo "使用empty判断属性a是否为空\n";
var_dump(empty($obj->a));

echo "使用isset判断属性a是否存在\n";
var_dump(isset($obj->a));

echo "----------------------------\n";

echo "使用unset注销掉属性a的值\n";
unset($obj->a);

echo "使用empty判断属性a是否为空\n";
var_dump(empty($obj->a));

echo "使用isset判断属性a是否存在\n";
var_dump(isset($obj->a));

echo "----------------------------\n";

echo '获取可访问的属性declared的值：', $obj->declared . "\n";

echo "为不可访问的属性hidden赋值0，此时调用了魔术方法__set()\n";
$obj->hidden = 0;

echo "通过public方法getHidden()从类内部访问私有属性hidden的值，不会调用魔术方法：\n", $obj->getHidden() . "\n";

echo "获取不可访问的属性hidden的值，此时调用了魔术方法__get()：\n" , $obj->hidden . "\n";
```

	为不存在的属性a赋值:Setting 'a' to '1'
	获取不存在的属性a的值：Getting 'a' 1
	----------------------------
	使用empty判断属性a是否为空
	Is 'a' set?
	Getting 'a' /vagrant/magic/overloading.php:119:
	bool(false)
	使用isset判断属性a是否存在
	Is 'a' set?
	/vagrant/magic/overloading.php:122:
	bool(true)
	----------------------------
	使用unset注销掉属性a的值
	Unsetting 'a'
	使用empty判断属性a是否为空
	Is 'a' set?
	/vagrant/magic/overloading.php:130:
	bool(true)
	使用isset判断属性a是否存在
	Is 'a' set?
	/vagrant/magic/overloading.php:133:
	bool(false)
	----------------------------
	获取可访问的属性declared的值：1
	为不可访问的属性hidden赋值0，此时调用了魔术方法__set()
	Setting 'hidden' to '0'
	通过public方法getHidden()从类内部访问私有属性hidden的值，不会调用魔术方法：
	2
	获取不可访问的属性hidden的值，此时调用了魔术方法__get()：
	Getting 'hidden' 0

### **方法重载**

### __call()

在对象中调用一个不可访问的方法时，__call() 会被调用。

	public __call ( string $name , array $arguments ) : mixed

`$name` 参数是要调用的方法名称。`$arguments` 参数是一个枚举数组，包含着要传递给方法 `$name` 的参数。

### __callStatic()

调用一个不可访问的类静态方法时，__callStatic() 会被调用。

	public static __callStatic ( string $name , array $arguments ) : mixed

`$name` 参数是要调用的方法名称。`$arguments` 参数是一个枚举数组，包含着要传递给方法 `$name` 的参数。

#### **Code**

```php
class Method
{
    public function __call($name, $arguments)
    {
        // 注意: $name 的值区分大小写
        echo "调用对象不可访问的方法 '$name' ", implode(', ', $arguments). "\n";
    }

    public static function __callStatic($name, $arguments)
    {
        // 注意: $name 的值区分大小写
        echo "调用不可访问的类静态方法 '$name' ", implode(', ', $arguments). "\n";
    }
}
```
```php
$obj = new Method;
$obj->run('参数1', '参数2', '参数3');

Method::run('参数1', '参数2', '参数3');
```

	调用对象不可访问的方法 'run' 参数1, 参数2, 参数3
	调用不可访问的类静态方法 'run' 参数1, 参数2, 参数3

## 四、[序列化魔术方法](https://www.php.net/manual/zh/language.oop5.magic.php)


### __sleep()

	public __sleep ( void ) : array

使用 `serialize()` 函数对对象进行序列化时，若类中存在魔术方法 `__sleep()` ，则会先被调用 `__sleep()`，然后才执行序列化操作。
此功能可以用于清理对象，并返回一个包含对象中所有应被序列化的变量名称的数组。如果该方法未返回任何内容，则 NULL 被序列化，并产生一个 E_NOTICE 级别的错误。
`__sleep()` 方法常用于提交未提交的数据，或类似的清理操作。同时，如果有一些很大的对象，但不需要全部保存，这个功能就很好用。

> __sleep() 不能返回父类的私有成员的名字。这样做会产生一个 E_NOTICE 级别的错误。可以用 Serializable 接口来替代。

### __wakeup()

	__wakeup ( void ) : void

使用 `unserialize()` 函数对对象进行反序列化时，若类中存在魔术方法 `__wakeup() ` ，则会先被调用 `__wakeup() `，然后才执行反序列化操作。
在反序列化操作中，`__wakeup()` 方法常用于重新建立数据库连接，或执行其它初始化操作。

#### Code

```php
class Connection
{
    /**
     * 数据库连接资源
     * @var
     */
    protected $link;

    /**
     * 连接数据库所需要的属性
     * @var
     */
    private $server, $username, $password, $db;

    /**
     * 构造函数，建立数据库连接
     * unserialize 执行反序列化时不会调用构造函数
     * @param $server
     * @param $username
     * @param $password
     * @param $db
     */
    public function __construct($server, $username, $password, $db)
    {
        $this->server = $server;
        $this->username = $username;
        $this->password = $password;
        $this->db = $db;
        $this->connect();
    }

    /**
     * 连接数据库
     */
    private function connect()
    {
        $this->link = mysqli_connect($this->server, $this->username, $this->password);
        mysqli_select_db($this->db, $this->link);
    }

    /**
     * 使用serialize序列化对象时，只保存以下四个对象属性
     * @return array
     */
    public function __sleep()
    {
        return ['server', 'username', 'password', 'db'];
    }

    /**
     * unserialize 执行反序列化时不会调用构造函数
     * 因此使用 __wakeup 函数重新进行数据库连接
     */
    public function __wakeup()
    {
        $this->connect();
    }
}
```

## 五、[其他魔术方法](https://www.php.net/manual/zh/language.oop5.magic.php)

### __toString()

	public __toString ( void ) : string

当对象被当做字符串使用时，调用 `__toString()` 方法，该方法必须返回一个字符串。
若对象未定义 `__toString()` 方法，或  `__toString()` 方法返回值不是字符串，则会产生 E_RECOVERABLE_ERROR 级别的致命错误。

> 不能在 __toString() 方法中抛出异常。这么做会导致致命错误。

#### Code

```php
class ToString
{
    public $foo;

    public function __construct($foo)
    {
        $this->foo = $foo;
    }

    public function __toString()
    {
        return $this->foo;
    }
}

$class = new ToString('Hello World!');
echo $class;
```

	Hello World!


### __invoke()

	__invoke ([ $... ] ) : mixed

当尝试以调用函数的方式调用一个对象时，`__invoke()` 方法会被自动调用。

#### Code

```php
class CallableClass
{
    function __invoke($foo)
    {
        return $foo;
    }
}

$callable = new CallableClass();
echo $callable("Hello World!\n");
var_dump(is_callable($callable));
```

	Hello World!
	/vagrant/magic/__invoke.php:19:
	bool(true)

```php
class UncallableClass
{
}
$uncallable = new UncallableClass();
var_dump(is_callable($uncallable));
```

	/vagrant/magic/__invoke.php:27:
	bool(false)