+++
date = "2015-07-02T14:25:17+08:00"
menu = "main"
tags = ["php"]
title = "PHP 知识点"

+++

本文来自《Learning PHP, MySQL, JavaScript和CSS》读书笔记，总结了一些有些深入的知识

PHP作为一种脚本语言，使用它的时候必然会HTML有所牵扯，有些人会将整个文档用<?php ?>标记，然后再内部输出HTML文档，有些人只是在需要动态脚本的时候将最少的PHP代码段放入标记之间，剩下的文档保留在标准的HTML中


关于PHP
=====

---

HTML对文本进行解析的时候换行符会被解析成空格，而空格则是会被忽略的（所有的空格都会被忽略）

常量
------

关于常量，需要记住的是：首先它们之前不能有$(常规变量必须有)，其次只能用define函数定义它们

	define("ROOT_LOCATION", "/usr/local/www")


多行命令
------

很多时候PHP界面需要输出很多文本，多次使用echo可能会比较麻烦，PHP提供了利用<<<运算符构建多行字符序列的方法

	<?php
	$author = "lucifer";
	echo <<<_END
	This is a HeadLine
	This is the first line
	This is the second
	 - Written by $auther
	_END

开发者可以将一整段HTML代码放在PHP代码中，然后用PHP的变量代替具体动态部分，使用这个结构时，不必在结尾加入 \n 符号另起一行，因为这个结构会自动给回车并另起一行。

不要在第一个 _END 后直接加分号。

\_END 可以换成 \_SECTION1,  \_OUTPUT



预定义常量
--------

直接在代码里使用即可

* \_\_LINE\_\_ 文件的当前行号
* \_\_FILE\_\_ 文件的完整路径和文件名，这个话类似 **F:\wamp\wamp\www\learningPHP\index.php**
* \_\_DIR\_\_ 文件路径，类似 **F:\wamp\wamp\www\learningPHP**
* \_\_FUNCTION\_\_ 函数名
* \_\_CLASS\_\_ 类名
* \_\_METHORD\_\_ 类方法名
* \_\_NAMESPACE\_\_ 当前空间的名字


print和echo的区别
------

echo不是一个函数，不能作为复杂表达式的一部分，而print可以。

	$b ? print "TRUE" : print "FALSE";


变量
----

* 局部变量
* 全局变量

		global $is_logged_in;
* 静态变量
	
		static $count = 0;
* 超级全局变量
	* $GLOBALS 当前定义在脚本全局范围内的全部变量，变量名是数组的键
	* $_SERVER 标题，路径和脚本位置之类的信息
	* $_GET 由HTTP的GET方法传递给当前脚本的变量
	* $_POST 由HTTP的POST方法传递给当前脚本的变量
	* $_FILES 由HTTP的POST方法传递给当前脚本的项目
	* $_COOKIE 由HTTP的cookies传递给当前脚本的变量
	* $_SESSION 当前脚本可用的会话变量
	* $\_REQUEST 由默认的$\_GET, $_POST, $_COOKIE传给浏览器的信息内容
	* $_NEW 由环境方法传给当前脚本的变量

---

PHP中表达式和控制流
===

在PHP中，TRUE代表的是1，FALSE代表的是NULL。NULL可以认为是空且其输出结果是一个空字符串

相等运算符和一致性运算符
---------------------

	<?php
	$a = "1000";
	$b = "+1000";
	if ($a == $b) echo "1";
	if ($a === $b) echo "2";

上面的结果输出的结果是数字1。因为这两个字符串都首先被转化成数字，1000和+1000在数值上是相等的。

相反，第二个if语句使用了一致性运算符，阻止php进行自动的类型转换。因此$a和$b作为字符串进行比较且被发现是不同的。对应的，不一致是 !==

逻辑运算符
--------

1. && 与
2. AND 低优先级与，and 也可以
3. || 或
4. OR 低优先级或，or 也可以
4. XOR 异或
5. NOT 非
6. ! 非

类型转换
------

1. (int)(integer) 整型
2. (bool)(boolean) 布尔
3. (float)(double)(real) 浮点数
4. (string) 字符串
5. (array) 数组
6. (object) 对象                                                                                                                                                                                                                                                                                                                                   


PHP函数与对象
===========

函数名是大小写不敏感的也就是说function test(){}和function Test(){}放在一起会报错。而变量名和类名是大小写敏感的。

在一个变量前面加上 **&** 表示引用的意思，这个引用和Java中对于对象的引用是一样的

	$a = "gone with wind";
	$b = $a;
	$b = "gone with rain";
	echo $a."<br>".$b."<br>";

这段代码输出的结果是
	
	gone with wind
	gone with rain

而如下
	
	$a = "gone with wind";
	$b = &$a;
	$b = "gone with rain";
	echo $a."<br>".$b."<br>";

的输出结果是

	gone with rain
	gone with rain
	
PHP中创建了对象后，在作为参数传递时通过引用来传递

包含文件和请求文件
---------------

使用include和include_once，当文件没有被找到程序依然会执行，然后使用require和require_once时，如果文件没有被找到就会报错

类
---------

可以使用 clone 操作符，它创建类的新实例，并将原实例的属性赋值给新实例。

	<?php
	$object1 = new User();
	$object1->name = "Alice";
	$object2 = clone $object1;
	$object2->name = "Bob";
	?>

PHP中的静态方法使用 **::** 进行访问，静态的方法不能访问非静态的成员变量

	User::pwd_string();
	class User{
		static function pwd_string(){
		}
	}

PHP在类中不必直接声明属性，因为初次使用它们时可以被间接定义，下面的程序是可以正常输出的

	$object1 = new User();
	$object1->name = "Alice";
	echo object1->name;
	class User{}

在类中声明属性时，可以给属性一个默认值。但是这个值必须是常量，不能是函数或者表达式的计算结果

	class Test{
		public $name = "Paul Smith"; // 有效
		public $age = 42; // 有效
		public $time = time(); // 无效
		public $score = $level * 2; // 无效
	}

类中的常量使用 const 进行声明，使用类进行访问，同时PHP5.3.0之后可以使用对象进行访问，常量在进行声明时不需要是用 $ 符号

	Translate::lookup;
	echo Translate::ENGLISH;
	class Translate{
		const ENGLISH = 0;
		const SPANISH = 1;
		const FRENCH = 2;
		const GERMAN = 3;

		// ..
		static function lookup(){
			echo self::SPANISH;
		}
	}

当派生一个子类并声明自己的构造方法时，PHP不会自动调用父类的构造方法（当子类没有声明构造方法时，默认自动调用父类的构造方法），可以使用 `parent::__construct()` 这样的结构来调用父类的构造方法。而确保你的代码调用当前类中的方法，可以使用关键字 self，比如 `self::methord();`

final 关键字可以防止子类方法，覆盖父类方法


数组
===

* 带数值下标的数组

		$paper[] = "Copier";
		$pager[] = "Inkjet";
		$pager[] = "Laser";
		$pager[] = "Photo";

		// 也可以这样

		$paper[0] = "Copier";
		$paper[1] = "Inkjet";

* 关联数组

		$paper['copier'] = "Copier & Mutipurpose";
		$paper['inkjet'] = "Inkjet Printer";
		foreach($paper as $item=> $description){
			echo "$item: $description";
		}

使用数组关键字进行赋值

	$p1 = array("Copier", "Inkjet");
	$p2 = array('copier' => "Copier & Mutipurpose",
				'inkjet' => "Inkjet Printer");


实用PHP技术
=========

printf: 通过在字符串中键入特殊的格式化字符控制文本的输出格式。可以选择输出的类型，设定精确度，进行字符串填充等操作

* % 显示一个%符号
* b 将参数显示为二进制整数
* c 将参数显示为ASCII符
* d 将参数显示为有符号十进制整数
* e 用科学制显示
* f 将参数显示为浮点数
* o 将参数显示为八进制整数
* s 将参数显示为字符串
* u 将参数显示为无符号十进制数
* x 将参数显示为小写十六进制数
* X 将参数显示为大写十六进制数

sprintf: 将printf的结果传递给一个变量

文件操作
------

有些函数

* fgets()：读取一行
* fread(): 从文件中获取多行数据或者多行中的部分数据
* copy("testfile.txt", "testfile2.txt") 复制文件
* rename("testfile2.txt", "testfile2.new") 重新命名
* unlink("testfile2.new") 删除文件，当文件不存在时会报错

更新文件

	$fh = fopen("testfile.txt", "r+") or die("Failed to open file");
	$text = fgets($fh);
	fseek($fh, 0, SEEK_END); // SEEK_END 告诉当前函数将文件指针移动至文件末尾，0告诉当前函数当文件指针移动至文件末尾后需要从文件末尾向回移动多少位
	fwrite($fh, "text") or die("Could not write to file");
	fclose();
	echo "File 'testfile.txt' successfully updated";

还有两个可选参数能够传递给 fseek() 函数： SEEK_SET 和 SEEK_CUR

* SEEK_SET 告诉当前函数将文件指针设定在前面参数指向的位置。 `fseek($fh, 18, SEEK_SET)`  将文件指针移动到标识为18的位置
* SEEK_CUR 将文件指针指向当前位置并在此基础上偏移给定的偏移量。 `fseek($fh, 5, SEEK_CUR)` 如果文件指针指向的当前位置是18，接下来的代码将其移动到23的位置

文件资讯锁：该加锁函数仅仅对其他调用它的进程加锁。如果代码在没有实现flock文件加锁的情况下进入文件并对其进行修改，此时代码所做的修改同样会覆盖处于锁定状态的进程所做的修改。

	$fh = fopen("testfile.txt", "r+") or die("Failed to open");
	$text = fgets($fh);
	if(flock($fh, LOCK_EX)){
		fseek($fh, 0, SEEK_END);
		fwrite($fh, "text") or die("Could not to write to file");
		flock($fh, LOCK_UN);
	}
	fclose($fh);

读取整个文件 file_get_contents()

HTML5文档需要加一个文档类型的声明 `<!DOCTYPE html>`


SQLite操作
=========

防止SQL注入，第一件事就是不要依赖于PHP内置魔幻引用，因为这会导致单一好或双引号之类的任何字符在前面加入了 \ 自动转义。

	/*
	 * 净化用户输入，如果魔幻引用被激活，函数get_magic_quotes_gpc()就会返回true
	 * 在这种情况下，被夹在到字符串中的所有斜线都要被删除，或者函数mysql_real_escape_string可以终止某些创建损坏字符串的双转义字符。
	 */
	function mysql_fix_string($string){
		if(get_magic_quotes_gpc()) $string = stripslashes($string);
		return mysql_real_escape_string($string);
	}

防止HTML注入，也就是XSS，需要使用htmlentities给它进行杀毒


表单提交
=======

textarea 的wrap属性

* Off 文本不换行，当用户输入时行完全出现
* Soft 文本换行，当一串长字符串没有回车换行时发送给服务器
* hard 文本换行，当一串长字符串有软回车换行时以换行格式发送给服务器

如果不对 checkbox 的value进行赋值，默认提交的是 "on"


Cookies，会话和身份验证
====================

Cookies
-------

	// 设置cookie的函数
	setcookie(name, value, expire, path, domain, secure, httponly); // expire：过期，设置为0或者省略时会话结束，cookie会自动删除，secure和httponly均是bool型的
	setcookie('username', 'Hannah', time()+60*60*24*7, '/');
	
	// 使用cookie
	if(isset($_COOKIE['username'))
		$username = $_COOKIE['username'];

	// 删除cookie
	setcookie('username', 'Hannah', time()+2592000, '/'); // 设置2592000即一个月作为过期时间

Session
-------

开始会话，需要加一个 `session_start()` 函数

设置会话 `$_SESSION['username'] = 'nick'`

结束会话 session_destroy() 函数

设置超时

	ini_set('session.gc_maxlifetime', 60*60*24) // 设置超时日期，一天
	ini_get('session.gc_maxlifetime') // 获取超时周期

防止会话被截获

	// 获取用户的ip地址，使用md5进行加密，然后与浏览器用户代理字符串连接起来，作为session里check的值，在 different_user() 里通常需要做的事删掉原有会话，并让用户重新登陆
	if($_SESSION['check'] != md5($_SEVER['REMOTE_ADDR']).$_SEVER['HTTP_USER_AGENT'])
		different_user();

防止会话被固定：当有恶意用户试图为服务器提交一个会话ID而不是让服务器创建ID时，将会发生会话固定，可以使用 `session_regenerate_id()` 函数改变会话ID


JS
==

JavaScript每条语句结束后可以不用加 **；** ，在JavaScript中换行符起到了分号的作用，但是一行中要是加入多条语句的话需要使用分号隔开

JavaScript变量名区分大小写，Count，count和COUNT是不同的变量

同样，JavaScript的类型也很松散

	<script type="text/script">
	n = '838102050'; // 将 'n' 设置为字符串
	document.write('n = ' + n + ', and is a ' + typeof n + "<br />");
	n = 12345*67890; // 将 'n' 设置为数字
	document.write('n = ' + n + ', and is a ' + typeof n + "<br />");
	n += ' plus some text';
	document.write('n = ' + n + ', and is a ' + typeof n + "<br />");
	</script>

全局变量：在函数外定义的变量（ 或者在函数内，但没有关键字var ），无论是否用关键字var，只要在函数外定义变量，它就属于全局变量。

数组是用过引用传递给函数的，人应该以在数组参数中修改任一个元素，那么最初数组的元素也会被修改

局部变量

	<script>
	function test(){
		a = 123; // Global
		var b = 456; // Local
		if (a == 123) var c = 789; // Local
	}
	</script>

JavaScript中的with语句: with可以简化JavaScript中语句的一些类型，即把对一个对象的多个引用将为一个引用

	<script>
	string = "The quick browm fox jumps over the lazy dog";
	with(string){
		document.wirte("The string is " + length + "characters<br />);
		document.write("In upper case it's " + toUpperCase());
	}
	</script>

JavaScript中的类型转换

* int, integer: pasreInt()
* Bool, Boolean: Boolean()
* Float, double, real: pasreFloat()
* String: String()
* Array: Array()

参数数组，当在一个函数中需要向一个函数传递多余5个参数时，可以使用参数数组

	displayItems("Dog", "Cat", "Pony", "Hamster", "Tortoise")
	
	function displayItems(v1, v2, v3, v4, v5){
		document.write(v1 + "<br />");
		document.write(v2 + "<br />");
		document.write(v3 + "<br />");
		document.write(v4 + "<br />");
		document.write(v5 + "<br />");
	}
	// 也可以这样写
	
	function dispalyItems(){
	for (j=0; j < displayItems.arguments.length ; ++j){
		for( j = 0; j < displayItems.arguments.length ; ++j){
			document.write(displayItems.arguments[j] + "<br />")
		}
	}

JavaScript中的对象可以在创建了之后添加新的属性，新的方法

	// 声明User类和它的构造方法
	function User(forename, username, passwd){
		this.forename = forename;
		this.username = username;
		this.passwd = passwd;
		
		this.showUser = function(){
			document.write("Forename:" + this.forename + "<br />");
		}
	}
	// 程序通过运行这个函数创建一个User实例时，this引用被创建的实例。同一个方法可以被不同的不同的参数多次调用，每次都创建一个带有不同forename以其它属性值得新User

上面的是一个包含方法的类的构造方法，也可以引用构造方法定义之外的函数

	function User(forename, username, passwd){
		this.forename = forename
		this.username = username
		this.passwd = passwd
		this.showUser = showUser
	}
	
	function showUser(){
		document.write("Forename:" + this.forename + "<br />")
	}

在任何时候都可以添加 prototype 属性或者方法，所有对象都可以继承这个属性

	User.prototype.greeting = "Hello"
	document.write(details.greteing)

	// 也可以添加方法

	User.prototype.showUser = function(){
		document.write("Name" + this.forename + "<br />")
	}
	details.showUser();

JS中的静态属性和方法：JS也支持static属性和方法，以便于从类的prototype属性中存储和检索数据。

	User.prototype.greeting = "Hello"
	document.write(User.prototype.greeting)

JavaScript数组

	numbers = Array("One", "Two", "Three")

关联数组

	balls = {
		"golf": "Golf balls, 6", 
		"tennis": "Tennis balls, 3",
		"soccer": "Soccer ball, 1"}
	for (ball in balls)
		document.write(ball + "=" + balls[ball] + "<br />")

使用数组方法

* concat 方法可以将两个数组或者一个数组中的一些列值连接起来。

		fruit = ['Banana', 'Grape']
		veg = ['Carrot', 'Cabbage']
		document.write(fruit.concat(veg))
		
		// other operations
		pet = ["Cat", "Dog", "Fish"]
		more_pet = pets.concat("Rabbit", "Hamster");
		document.write(more_pet)

* forEach

		pets = ["Cat", "Dog", "Rabbit", "Hamster"]
		pets.forEach(output)
		
		function output(element, index. array){
			documen.write("Element at index " + index + " has the value " + element + "<br/>")
		}

* push And pop：push可以不指定元素增量简单对数组进行赋值向数组中添加新元素，pop的话是弹出最近添加的一个元素

		sports = ["Football", "Tennis", "Baseball"]
		sports.push("Hockey")
		removed = sports.pop()

* reverse：将一个数组中的元素颠倒顺序
* join：将组内所有值转为字符串1
* sort：排序

		sports = ["Football", "Tennis", "Baseball", "Hockey"]
		sports.sort() // 按字母表进行排序
		sports.sort().reverse() // 字母表逆向
		sports.sort(function(a, b){ return a-b; })
		sports.sort(function(a, b){ return b-a; })

