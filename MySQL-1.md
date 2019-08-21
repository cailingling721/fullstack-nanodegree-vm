---
title: MySQL
date: 2019-08-21 13:31:30
tags: MySQL
---


## MySQL支持的数据类型
* 文本和字符串类型
	text — 任何长度的字符串，例如 Python str 或 unicode 类型。
	char(n) — 长度为 n 个字符的字符串。
	varchar(n) — 长度上限为 n 个字符的字符串。

* 数值类型
	integer — 整型值，例如 Python int。
	real — 浮点型值，例如 Python float。精确到小数点后 6 位。
	double precision — 精度更高的浮点型值。精确到小数点后 15 位。
	decimal — 精确的十进制值。

* 日期和时间类型
	date — 日历日期，包括年月日。
	time — 一天中的时间。
	timestamp — 日期和时间相结合。

注意：在 SQL 中，字符串和日期值必须用单引号括起来。

## select语句
具有 where 条件的 select 语句的语法：
select columns from tables where condition ;
每列用逗号分隔；使用 * 可选择所有列。
condition 是列值的布尔表达式。SQL 支持布尔运算 and、or 和 not，和 Python 中的运算规则一样。
表达式 (not X) and (not Y) 和 not (X or Y) 可以互换，这是由德摩根定律决定的。

SQL 中的比较运算符和 Python 中的几乎一样：< 表示小于，> 表示大于，!= 表示不等于，<= 表示小于或等于，等等。
其中一个差别是 SQL 使用 =（而不是 ==）表示等于。你可以将所有的基本比较运算符应用于字符串、数字、日期以及其他值。以下是用于创建预览表格的 SQL 指令：

animals 表格中的列包括 name（文本字符串）、species（同样是文本字符串）和 birthdate（日期）。 注意：我们的数据库中的日期始终为国际标准格式，例如 '1999-12-31'。确保日期要用单引号括起来。

```mysql
QUERY = '''select name, birthdate from animals where species = 'gorilla';'''
QUERY = '''select name from animals where species != 'gorilla' and name != "Max";'''
QUERY = '''select name from animals where (not species = 'gorilla') and (not name = "Max");'''
QUERY = '''select name from animals where not (species = 'gorilla' and name = "Max");'''
QUERY = '''select name from animals where species = "llama" and birthdate >= '1995-01-01' and birthdate <= '1998-12-31''''
```

### 一些新的select语句

... limit count 只返回结果表格的前 count 行。
... limit count offset skip 返回前 skip 行之后的 count 行。
... order by columns ... order by columns desc 使用 columns（一列或多列，用逗号分隔）作为排序键对列排序。数字列将按数字顺序排序；字符串列将按字母顺序排序。desc 表示顺序为逆序（desc-结尾的排序）。
... group by columns 更改集合的行为，例如 max、count 和 sum。对于 group by，集合将为 columns 中的每个唯一值返回一行。

```
QUERY = "select max(name) from animals;"
QUERY = "select * from animals limit 10;"
QUERY = "select * from animals where species = 'orangutan' order by birthdate;"
QUERY = "select name from animals where species = 'orangutan' order by birthdate desc;"
QUERY = "select name, birthdate from animals order by name limit 10 offset 20;"
QUERY = "select species, min(birthdate) from animals group by species;"
QUERY = '''
select name, count(*) as num from animals
group by name
order by num desc
limit 5;
'''
```

这些是我们到目前为止见到的所有 select 子句：
* where 子句表示限制条件 — 从表格中过滤出符合特定规则的行。where 支持等于、不等于和布尔运算符等：
	where species = 'gorilla' — 仅返回物种列的值为“gorilla”的行。
	where name >= 'George' — 仅返回名称列在“George”之后（按字母顺序）的行。
	where species != 'gorilla' and name != 'George' — 仅返回物种不是“gorilla”并且名称不是“George”的行。
* limit / offset
	limit 子句对结果表格可以返回的行数做出限制。可选 offset 子句表示要在结果中跳过多少行。所以 limit 10 offset 100 将返回 10 条结果，从第 101 行开始。
* order by
	order by 子句告诉数据库如何对结果排序 — 通常根据一个或多个列。所以 order by species, name 表示首先按照物种列排序，然后在每个物种里按照名称排序。
排序发生在 limit/offset 之前，所以你可以使用它们来提取出按字母顺序排列的页面结果（想想字典的页面）。

* 可选 desc 修饰符告诉数据库按照降序对结果排序，例如从大到小或从 Z 到 A。

* group by
group by 子句只能用于汇总，例如 max 或 sum。没有 group by 子句的话，对集合执行选择语句将对整个选定表格进行汇总，只返回一行。对于 group by 子句，它将对 group by 子句中的列或表达式的每个唯一值返回一行。


### 数据库的优势
像是limit,offset以及order by这些SQL功能，其实可以在python中执行，如下所示：

```
count(\*) --> len(results)
limit 100 offset 10 --> results[10, 110]
order by column --> sorted(results, key = lambda x: x[column])
```
为什么要在数据库中执行这些操作呢？那就是因为数据库中执行这些操作，不仅速度快而且占用较少的内存。

```
QUERY = "select species,count(species) from animals group by species order by count(species) desc"
```

### insert语句

* insert 语句的基本语法：
insert into table ( column1, column2, ... ) values ( val1, val2, ... );

* 如果值和表格的列顺序一样（从第一列开始），则不需要在 insert 语句中指定列：
insert into table values ( val1, val2, ... );
例如，如果表格有三列 (a, b, c)，你想要向 a 和 b 中插入值，你可以在 insert 语句中省略列名称。但是如果你想向 b 和 c 或 a 和 c 中插入值，则需要指定列。
单个 insert 语句只能插入一个表格中（而 select 语句可以使用 join 从多个表格中获取数据）。

### join两个表格

要连接（join）两个表格，首先选择连接条件，即数据库将表格一中的行与表格二中的行相匹配时采用的规则。然后编写连接语句，包含每个表格中的列。
例如，如果你想连接表格 T 和 S，其中 T.color 和 S.paint 要相同，则需要使用 T join S on T.color = S.paint 编写一个 select 语句。

```
QUERY = '''
select name from animals join diet on animals.species = diet.species where food = 'fish' group by name
'''
```

### having子句

having 子句和 where 子句工作原理差不多，但是它应用于 group by 汇总发生之后。语法如下所示：

```
select columns from tables group by column having condition ;
```

通常，至少有一列将是汇总函数，例如对表格的某列执行 count、max 或 sum 操作。要对汇总列应用 having，你需要使用 as 为其设定名称。例如，如果你有一个商店所售商品的表格，并且想要找出售出数量超过 5 件的所有商品，则可以使用：

```
select name, count(\) as num* from sales having num > 5;
```

你可以在 select 语句中仅使用 where、仅使用 group by 、或使用 group by 和 having 、或使用 where 和 group by 或三个都用到！
但是如果没有 group by 的情况下，使用 having 通常是不合理的。
如果你同时使用了 where 和 having， where 条件将过滤即将被汇总的行，having 条件将过滤汇总后的行。
关于 having 的更多详情请参阅以下[网址](http://www.postgresql.org/docs/9.4/static/sql-select.html#SQL-HAVING) 。

你可以采用几种不同的方式来解决这一问题，下面是其中一个示例：
```
select food, count(animals.name) as num
	   from diet join animals
	   on diet.species = animals.species
	   group by food
	   having num = 1
```

下面是另一个示例：
```
select food, count(animals.name) as num
	   from diet, animals
	   where diet.species = animals.species
	   group by food
	   having num = 1
```

我的解法实际上与上面的第一个示例等同：
```
QUERY = '''
select food,count(food) from animals join diet on animals.species = diet.species group by food having count(food) =1
'''
```


### 更多连接练习

```
QUERY = '''
select ordernames.name, count(*) as num
  from animals, taxonomy, ordernames
  where animals.species = taxonomy.name
	and taxonomy.t_order = ordernames.t_order
  group by ordernames.name
  order by num desc
'''
```

显式连接格式的另一种解决方案：
```
select ordernames.name, count(*) as num
  from (animals join taxonomy
				on animals.species = taxonomy.name)
				as ani_tax
		join ordernames
			 on ani_tax.t_order = ordernames.t_order
  group by ordernames.name
  order by num desc
```

在显式连接格式中，你需要明确告诉数据库按照什么顺序连接表格 — ((a join b) join c) — 而不是让数据库自己去判断。

如果你使用的是更加框架性的数据库（例如 SQLite），那么显式连接格式可能会存在性能优势。但是对于我们将在下节课中用到的面向服务器的数据库系统 PostgreSQL，query planner 应该消除任何差异。


### 使用DB-API

```
import sqlite3

# Fetch some student records from the database.
db = sqlite3.connect("students")
c = db.cursor()
query = "select name, id from students order by name;"
c.execute(query)
rows = c.fetchall()

# First, what data structure did we get?
print "Row data:"
print rows

# And let's loop over it too:
print
print "Student names:"
for row in rows:
  print "  ", row[0]

db.close()
```

### DP-API中的插入操作

```
import sqlite3

db = sqlite3.connect("testdb")
c = db.cursor()
c.execute("insert into balloons values ('blue', 'water') ")
db.commit()
db.close()
```

### 运行forum

使用vagrant ssh登入虚拟机。如果重启计算机，你需要运行 vagrant up 来重启虚拟机。

``` sh
cd /vagrant/forum
python forum.py
# 通过如下网址访问
http://localhost:8000/
```

这时候可以访问论坛，但是无论在论坛中输入什么刷新一下都会不存在了。然后可以使用psql允许登入数据库并且交互的 进行查询操作。使用psql forum可以登入forum数据库。实际上虚拟机里一开始就是一张空表，需要把它跟论坛应用连起来。

```sh
select 2+2 as a, 4+4 as b
select * from posts;
```

此时的forumdb.py内容如下：它是用来存放数据库代码的。DB是使用一个列表存储的。就是这个原因导致刷新时帖子都消失。

``` python
# -*- coding: utf-8 -*-
# Database access functions for the web forum.
# 

import time

## Database connection
DB = []

## Get posts from database.
def GetAllPosts():
	'''Get all the posts from the database, sorted with the newest first.

	Returns:
	  A list of dictionaries, where each dictionary has a 'content' key
	  pointing to the post content, and 'time' key pointing to the time
	  it was posted.
	'''
	posts = [{'content': str(row[1]), 'time': str(row[0])} for row in DB]
	posts.sort(key=lambda row: row['time'], reverse=True)
	return posts

## Add a post to the database.
def AddPost(content):
	'''Add a new post to the database.

	Args:
	  content: The text content of the new post.
	'''
	t = time.strftime('%c', time.localtime())
	DB.append((t, content))
```

forum.py

``` python
# -*- coding: utf-8 -*-
# DB Forum - a buggy web forum server backed by a good database
#

# The forumdb module is where the database interface code goes.
import forumdb

# Other modules used to run a web server.
import cgi
from wsgiref.simple_server import make_server
from wsgiref import util

# HTML template for the forum page
HTML_WRAP = '''\
<!DOCTYPE html>
<html>
  <head>
	<title>DB Forum</title>
	<style>
	  h1, form { text-align: center; }
	  textarea { width: 400px; height: 100px; }
	  div.post { border: 1px solid #999;
				 padding: 10px 10px;
		 margin: 10px 20%%; }
	  hr.postbound { width: 50%%; }
	  em.date { color: #999 }
	</style>
  </head>
  <body>
	<h1>DB Forum</h1>
	<form method=post action="/post">
	  <div><textarea id="content" name="content"></textarea></div>
	  <div><button id="go" type="submit">Post message</button></div>
	</form>
	<!-- post content will go here -->
%s
  </body>
</html>
'''

# HTML template for an individual comment
POST = '''\
	<div class=post><em class=date>%(time)s</em><br>%(content)s</div>
'''

## Request handler for main page
def View(env, resp):
	'''View is the 'main page' of the forum.

	It displays the submission form and the previously posted messages.
	'''
	# get posts from database
	posts = forumdb.GetAllPosts()
	# send results
	headers = [('Content-type', 'text/html')]
	resp('200 OK', headers)
	return [HTML_WRAP % ''.join(POST % p for p in posts)]

## Request handler for posting - inserts to database
def Post(env, resp):
	'''Post handles a submission of the forum's form.
  
	The message the user posted is saved in the database, then it sends a 302
	Redirect back to the main page so the user can see their new post.
	'''
	# Get post content
	input = env['wsgi.input']
	length = int(env.get('CONTENT_LENGTH', 0))
	# If length is zero, post is empty - don't save it.
	if length > 0:
		postdata = input.read(length)
		fields = cgi.parse_qs(postdata)
		content = fields['content'][0]
		# If the post is just whitespace, don't save it.
		content = content.strip()
		if content:
			# Save it in the database
			forumdb.AddPost(content)
	# 302 redirect back to the main page
	headers = [('Location', '/'),
			   ('Content-type', 'text/plain')]
	resp('302 REDIRECT', headers) 
	return ['Redirecting']

## Dispatch table - maps URL prefixes to request handlers
DISPATCH = {'': View,
			'post': Post,
		}
## Dispatcher forwards requests according to the DISPATCH table.
def Dispatcher(env, resp):
	'''Send requests to handlers based on the first path component.'''
	page = util.shift_path_inTo(env)
	if page in DISPATCH:
		return DISPATCH[page](env, resp)
	else:
		status = '404 Not Found'
		headers = [('Content-type', 'text/plain')]
		resp(status, headers)	
		return ['Not Found: ' + page]

# Run this bad server only on localhost!
httpd = make_server('', 8000, Dispatcher)
print("Serving HTTP on port 8000...")
httpd.serve_forever()
```

### 为应用添加后端数据库

我们已经为你创建好了 forum 数据库。你需要使用 psycopg2.connect("dbname=forum") 将你的代码与该数据库相连，然后对 posts 表格执行 select 和 insert 操作。

现有的 GetAllPosts 函数会返回列表中的所有条目。所以它的数据库版本应该返回 posts 表格的所有条目。

同样，现有的 AddPost 函数会向列表中插入条目。插入帖子时，你不需要提供 time 列。表格已经设置成可以提供时间戳。

现有的 GetAllPosts 函数使用 Python sort 函数对帖子进行排序。当你使用数据库实现此函数时，可以通过使用 SQL 排序而避免使用 Python 排序吗？

为了改变forumdb.py这部分代码就可以真正的使用数据库了。

``` python
# -*- coding: utf-8 -*-
# Database access functions for the web forum.
# 

import time
import psycopg2

## Database connection
DB = []

## Get posts from database.
def GetAllPosts():
	'''Get all the posts from the database, sorted with the newest first.

	Returns:
	  A list of dictionaries, where each dictionary has a 'content' key
	  pointing to the post content, and 'time' key pointing to the time
	  it was posted.
	'''
	DB = psycopg2.connect("dbname=forum") # 连接数据库
	c = DB.cursor() # 创建一个光标
	c.execute("SELECT time,content FROM posts ORDER BY time DESC") # 执行
	posts = [{'content': str(row[1]), 'time': str(row[0])} for row in c.fetchall()] # 获得了结果，并且将它们转换成与代码匹配的字典形式
	DB.close() # 关闭连接
	return posts

## Add a post to the database.
def AddPost(content):
	'''Add a new post to the database.

	Args:
	  content: The text content of the new post.
	'''
	DB = psycopg2.connect("dbname=forum") # 连接数据库
	c = DB.cursor() # 创建一个光标
	c.execute("INSERT INTO posts (content) VALUES ('%s')" % content) # 执行
	DB.commit() #  提交改动
	DB.close()
```

但是这里的代码有很严重的bug, 例如输入的帖子名字含有“‘”,就会导致错误。另外，如果提交这个句子：“'); delete from posts; --”，就会从posts表单中删除所有数据。这其实是一种叫做SQL注入攻击的安全漏洞，帖子里的内容被当成了数据库命令,而不是单纯的文本值。我们还有另外一种方法可以让我们做出安全的请求语句。当我们执行请求语句时，我们在语句文本中用%s通配符。之后使用一个元组参数来执行这个调用，数据库会将其替换进请求语句，它会使用一种安全的方法，来避免这种问题再次发生。

``` python
# -*- coding: utf-8 -*-
## Add a post to the database.
def AddPost(content):
	'''Add a new post to the database.

	Args:
	  content: The text content of the new post.
	'''
	DB = psycopg2.connect("dbname=forum") # 连接数据库
	c = DB.cursor() # 创建一个光标
	c.execute("INSERT INTO posts (content) VALUES ('%s')", (content, )) # 执行
	DB.commit() #  提交改动
	DB.close()
```

### SQL注入攻击

SQL注入攻击是一种常见的安全漏洞。论坛程序确实是把每个帖子当成了一段文字，但是浏览器会把它们当作代码，这就是web应用的另一个安全漏洞，即脚本注入攻击。这就是为什么真正的网页论坛不允许用户在评论中输入任意的JavaScript代码。

我们需要确保在content列的数据永远都被当成文本，而不是JavaScript代码。实际上有一个很好用的python库，即bleach，可以用来终结不安全的HTML。避免攻击者通过数据库给用户造成干扰。


### 更新垃圾内容

如何清除垃圾数据呢？有很多种方法，使用的无害的东西替换，需要使用update命令。update 语句的语法如下所示：

```
update table set column = value where restriction;
update table set column = value where content='the while spam post';  # 此种方法太过于痛苦了，需要一个个打出来垃圾内容
```
会把符合限制条件的行记录都覆盖上新的值。update语句里的where限制条件分句与select 语句里的运行机制相同。如果不声明它，update操作就会应用到表单的所有行。

```
update table set column = value where content like '%awful%';
update posts set content = 'cheese' where content like '%awful%';
```
like 运算符支持简单的文本模式匹配。运算符左侧的内容（通常是文本列的名称）将与右侧的模式相匹配，该模式是一种 SQL 文本字符串（所以是单引号），并且可以使用 % 符号与任何子字符串相匹配，包括空字符串。如果你熟悉正则表达式的话，可以将 like 模式中的 % 看作正则表达式 .\*（点星号）。如果你更加熟悉 Unix shell 或 Windows 命令提示符中的文件名模式，那么 % 就像这些系统里的 \*（星号）。

例如，对于某个表格行，其中列 fish 的值为 'salmon’，下面的所有限制条件都为真：
fish like 'salmon'
fish like 'salmon%'
fish like 'sal%'
fish like '%n'
fish like 's%n'
fish like '%al%'
fish like '%'
fish like '%%%'

下面的都为假：
fish like 'carp'
fish like 'salmonella'
fish like '%b%'
fish like 'b%'
fish like ''

### 替换垃圾内容

delete 命令的语法为： 
delete from table where restriction ; 

restriction 和 select 语句中的一样，允许使用相同的运算符。


### 数据库的规范化设计

为了避免重复，我们将一个对象或者一条记录中的信息拆分到两个单独的表格中，在数据库表格中，这被称为规范化形式，其他为非规范化形式。规范化涉及在数据库的表格间建立关系。对有关系的但是存在于不同表的数据进行匹配。这个理论背后有一堆数据库理论。

规范化表格的规则

第一条规则就是：每行拥有的列数相同。一列的值可以为空或者为零，但不能某些行有两列，而另一行有三列。如果有两个值，就像是动物喜欢吃的食物不只是一种，它们的主键相同，则需要将它们分成不同的行。

第二条规则：表格的一列或者多列组成键，它确定每一行表示的是什么。在一行中，键对该行其余列的描述提供了主题。

第三条规则：在一个规范化的表格中非键列对键进行描述，他们不描述其他非键列。例如items表格的主键为item，有两列location和address，实际上可以增加一个表格叫locations,这个表格里面存储locations以及对应的address。就像是人民公园和人民公园的地址。

第四条规则：在一个规范化表格中，行不会对不存在关系的数据暗示其关系。这是要小心的地方，尤其是相同的实体有多行记录而且你想记录它的多个事实的时候。例如记录每个人技能的表格，第一列是人名字，第二列是他掌握的计算机技术技能，第三列代表语言技能。这样的表格好像第二列和第三列之间有关系。这样就可以将此表格拆分成两个表格，表格1代表计算机技能，表格2代表其语言技能。













