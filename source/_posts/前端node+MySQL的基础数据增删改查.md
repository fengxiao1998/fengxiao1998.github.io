---
title: 前端node + MySQL的基础数据增删改查
date: 2022-06-21 15:03:18
tags: [node,MySQL]
---

## 前言

对与用node进行后端服务，操作数据库是最基本的要求，本文默认安装了读者安装了**MySQL**和**workbench**,在此基础上进行操作数据库。

## SQL基本操作指令

#### 查询

```sql
-- select A from B 
--表示从表B查询Key值A，当A为*时表示查询全部

-- select * from users
-- 示例，表示查询users表中的所有数据

-- select username,password from users
-- 要查询多个Key值时，用逗号连接

-- select * from users where id = 3
-- 示例，表示查询users表中id=3的所有数据

-- select * from users order by status desc
-- 按照某个要求升序(asc)(默认)、降序(desc)，其中order by为关键字，上述语句表示查询users表所有数据并按照status的大小降序排列。

-- select * from users order by status desc,username asc
-- 多重排序，先按照status升序，再按照username降序

-- select count(*) from users where status = 0
-- 查询users中的status为0的总数据条数，count(*)为计数关键字

-- select username as uname,password as upwd from users
-- 如果希望给查询出来的列名称设置别名，可以使用AS关键字，示例表示将users表中的username用uname代替，password用 upwd替代展示

-- 其余写法可以将 where、order by、count(*)、as等关键字灵活组合，交给读者自行举一反三。
```

#### 插入(新增)

```sql
-- insert into users (username,password) values ('name','0999users')
-- 示例，表示向users表插入key为username，values为name，key为password，values为0999users的数据。
-- 其余写法可根据查询语句举一反三
```

#### 更新(更改)

```sql
-- update users set password = '888888',status = 1  where id = 3
-- 示例，表示更改users表中id为3的password和status
-- 其余写法可根据查询语句举一反三
```

#### 删除

```sql
-- delete from users where id = 3
-- 示例，删除users表中的id为3的数据
-- 其余写法可根据查询语句举一反三
```

## node+mysql

#### 新建MySQL数据库与表单

笔者预先新建了一个名为**my_db_01**的数据库，并新建了一个users表单，表单中现有数据如下：
![image-20220621142546564](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220621142546564.png)

#### node环境安装、连接mysql

node环境下，需要安装第三方插件，终端中输入`npm i mysql`。新建index.js文件，引入mysql模块以及建立连接，代码如下：

```javascript
//导入mysql模块
const mysql = require('mysql')
// 建立与MYSQL数据库的链接
const db = mysql.createPool({
  host: '127.0.0.1', //数据库ip地址，链接本地数据的话即本地ip地址，不确定的话可以在cmd中输入ipconfig查询。
  user: 'root', //数据库账号，安装数据库时默认用户名为root
  password: 'yourpassword', //数据库密码，安装数据库时你自行设置的密码
  database: 'yourdb' //指定操作数据库，为你新建数据库的名字，笔者的为my_db_01
})
//至此，数据库链接工作完成
```

#### 查询数据

```javascript
// 查询users中所有的数据
const sqlStr = 'select * from users'
db.query(sqlStr,(err,result)=>{
  if (err) return console.log(err.message)
  //查询语句返回的是数组
  console.log(result)
})
// 通过db.query方法执行sql语句，err为执行失败时的信息，result为成功时返回的数据。
// 执行node index.js，执行后结果如下,可以看到，查询语句返回的是数组格式，读者可根据前文中提及的多种查询变化方式进行实验，查看返回结果不同
/*
[
  RowDataPacket {        
    id: 1,
    username: 'zs',      
    password: 'zs123456',
    status: 0
  },
  RowDataPacket {
    id: 2,
    username: 'ls',
    password: 'ls123456',
    status: 0
  },
  RowDataPacket {
    id: 6,
    username: 'sadasda',
    password: 'aaaaa',
    status: 0
  },
  RowDataPacket {
    id: 7,
    username: 'saasda',
    password: 'aaaaa',
    status: 0
  }
]
*/
```

#### 新增数据

##### 普通写法

```javascript
const user = { username: 'sql-man', password: 'asdadasd' }
//待执行的sql语句，英文的 ? 表示占位符
const sqlStr = 'insert into users (username,password) values (?,?)'
//使用数组的形式，以此为 ? 占位符指定具体的值
db.query(sqlStr, [user.username, user.password], (err, result) => {
  if (err) return console.log(err.message)
  if (result.affectedRows >= 1) { console.log('插入成功', result) }
})
/*
可以看到，上文中sqlStr中的values后有两个英文?(注意是英文的?),该?起到占位符的作用，也即便于动态传值
db.query的第二个参数即为数组形式的value值，注意，数组中参数的顺序应与sqlStr中的key值顺序保持一致
成功后结果如下，可以看到是对象的形式，接下来请记住，只有查询语句返回的是数组，新增更改删除返回值都为对象形式。
插入成功 OkPacket {okpacket，字面意思ok包，事实也即如此，当mysql执行了一次语句后，会给你返回一个这种名字的包表示收到了执行命令并进行了执行
  fieldCount: 0, //文件数量
  affectedRows: 1, //影响行数，因为我们新增了一条数据，因此影响了一行,新增删除和更改时affectedRows都会大于或等于1，因此可以用result.affectedRows来判断是否数据新增、更改、删除成功，上文中也是这么判断的
  insertId: 8,//插入的id值
  serverStatus: 2, //服务器状态，不用过多了解
  warningCount: 0, //警告数量
  message: '', //返回消息
  protocol41: true,//服务协议，不用过多了解
  changedRows: 0 //改变行数，与影响行数不同，只有在更新数据时，这里才会有数值
}
*/
```

##### 便捷写法

```javascript
// 新增数据时，如果数据对象每个属性和数据表中字段一一对应，则可以通过如下方式快速插入数据
//要插入表的数据对象,只要key值对应，顺序不影响
const user = { password: 'aaaaa', username: 'saasda' }
//待执行的SQL语句，其中英文名的 ? 表示占位符
const sqlStr = 'insert into users set ?'
//直接将数据对象当作占位符的值
db.query(sqlStr, user, (err, result) => {
  if (err) return console.log(err.message)
  if (result.affectedRows === 1) { console.log('插入成功', result) }
})
//以上写法的好处是，在构造sqlStr语句时有所简化，并且user对象中key值的顺序不需要按照规定顺序排列，但要求对象中的key值应与表中的key值【完全一致】
```

#### 更新数据

##### 普通写法

```javascript
// 要更新的对象，要携带id，同时id要在表中存在
const user = { id: 1, username: 'aaabc', password: '000' }
// 要执行的sql语句
const sqlStr = 'update users set username=?,password=? where id=?'
db.query(sqlStr, [user.username, user.password, user.id], (err, result)=>{
  if (err) return console.log(err.message)
  if (result.affectedRows === 1) { console.log('更新成功', result) }
  else{
    console.log('更新失败',result)
  }
})
```

##### 便捷写法

```javascript
//要更新的数据对象
const user = { id: 5, username: 'something', password: 'ccccc' }
//要执行的SQL语句
const sqlStr = 'update users set ? where id=?'
//执行语句时,使用数组以此为占位符指定具体的值
db.query(sqlStr, [user, user.id], (err, result) => {
    if (err) return console.log(err.message)
    if (result.affectedRows === 1) { console.log('更新成功', result) }
})
// 与新增快捷写法相似，但需要额外传入要更改的id
```

#### 删除数据

```javascript
//删除数据，推荐根据唯一标识删除数据
const sqlStr = 'delete from users where id= ?'
//当SQL语句中只有一个占位符时，不需要数组，如果有多个占位符，需要用数组承接
db.query(sqlStr, 5, (err, result) => {
    if (err) return console.log(err.message)
    if (result.affectedRows === 1) { console.log('删除成功', result) }
})
//附，在现实开发中，不推荐用delete,这个会物理的将数据删除，删库跑路就是这种操作，这种是不可逆的操作，在真实开发中，推荐使用逻辑删除，比如在每个数据行新增一个del字段，为布尔值，通过【update方法】对数据标注0或者1来模拟数据的删除，这种方式的好处是不会物理删除数据，也可以达到“删除”的目的。
```

## 综述

以上只是最基础的增删改查，如果是复杂要求，比如批量修改数据，笔者想到的解决方案可能是for循环加sql语句的搭配使用，实际开发中肯定不会这么简单，可能还会涉及到多张表的联动查询修改，这些将交给笔者慢慢探索。