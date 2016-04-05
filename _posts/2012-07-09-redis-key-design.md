---
layout: post
title: 浅谈REDIS数据库的键值设计
date: 2012-07-09 19:59
author: onecoder
comments: true
categories: [key, Redis, Redis, 设计]
---
还是一篇老文，不过对于新人的我，颇有帮助，拿来分享。<br />
<br />
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	丰富的数据结构使得redis的设计非常的有趣。不像关系型数据库那样，DEV和DBA需要深度沟通，review每行sql语句，也不像memcached那样，不需要DBA的参与。redis的DBA需要熟悉数据结构，并能了解使用场景。</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	下面举一些常见适合kv数据库的例子来谈谈键值的设计，并与关系型数据库做一个对比，发现关系型的不足之处。</p>
<h2 style="font-family: 'Trebuchet MS', 'Lucida Grande', Arial, sans-serif; text-transform: uppercase; font-size: 2em; margin: 0px; padding: 0px; line-height: 19.600000381469727px; ">
	用户登录系统</h2>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	记录用户登录信息的一个系统， 我们简化业务后只留下一张表。</p>
<h3 style="font-family: 'Trebuchet MS', 'Lucida Grande', Arial, sans-serif; text-transform: uppercase; font-size: 1.2em; line-height: 19.600000381469727px; ">
	关系型数据库的设计</h3>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
mysql&gt; select * from login;
+---------+----------------+-------------+---------------------+
| user_id | name&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | login_times | last_login_time&nbsp;&nbsp;&nbsp;&nbsp; |
+---------+----------------+-------------+---------------------+
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 1 | ken thompson&nbsp;&nbsp; |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 5 | 2011-01-01 00:00:00 |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 2 | dennis ritchie |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 1 | 2011-02-01 00:00:00 |
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 3 | Joe Armstrong&nbsp; |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 2 | 2011-03-01 00:00:00 |
+---------+----------------+-------------+---------------------+</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	user_id表的主键，name表示用户名，login_times表示该用户的登录次数，每次用户登录后，login_times会自增，而last_login_time更新为当前时间。</p>
<h3 style="font-family: 'Trebuchet MS', 'Lucida Grande', Arial, sans-serif; text-transform: uppercase; font-size: 1.2em; line-height: 19.600000381469727px; ">
	REDIS的设计</h3>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	关系型数据转化为KV数据库，我的方法如下：</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	key 表名：主键值：列名</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	value 列值</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	一般使用冒号做分割符，这是不成文的规矩。比如在php-admin for redis系统里，就是默认以冒号分割，于是user:1 user:2等key会分成一组。于是以上的关系数据转化成kv数据后记录如下：</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
Set login:1:login_times 5
Set login:2:login_times 1
Set login:3:login_times 2

Set login:1:last_login_time 2011-1-1
Set login:2:last_login_time 2011-2-1
Set login:3:last_login_time 2011-3-1

set login:1:name &rdquo;ken thompson&ldquo;
set login:2:name &ldquo;dennis ritchie&rdquo;
set login:3:name &rdquo;Joe Armstrong&ldquo;</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	这样在已知主键的情况下，通过get、set就可以获得或者修改用户的登录次数和最后登录时间和姓名。</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	一般用户是无法知道自己的id的，只知道自己的用户名，所以还必须有一个从name到id的映射关系，这里的设计与上面的有所不同。</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
set &quot;login:ken thompson:id&quot; &nbsp;    1
set &quot;login:dennis ritchie:id&quot;&nbsp;&nbsp;&nbsp; 2
set &quot;login:&nbsp;Joe Armstrong:id&quot;&nbsp;   3</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	这样每次用户登录的时候业务逻辑如下（python版），r是redis对象，name是已经获知的用户名。</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
#获得用户的id
uid = r.get(&quot;login:%s:id&quot; % name)
#自增用户的登录次数
ret = r.incr(&quot;login:%s:login_times&quot; % uid)
#更新该用户的最后登录时间
ret = r.set(&quot;login:%s:last_login_time&quot; % uid, datetime.datetime.now())</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	如果需求仅仅是已知id，更新或者获取某个用户的最后登录时间，登录次数，关系型和kv数据库无啥区别。一个通过btree pk，一个通过hash，效果都很好。</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	假设有如下需求，查找最近登录的N个用户。开发人员看看，还是比较简单的，一个sql搞定。</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
select * from login order by last_login_time desc limit N</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	DBA了解需求后，考虑到以后表如果比较大，所以在last_login_time上建个索引。执行计划从索引leafblock 的最右边开始访问N条记录，再回表N次，效果很好。</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	过了两天，又来一个需求，需要知道登录次数最多的人是谁。同样的关系型如何处理？DEV说简单</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
select * from login order by login_times desc limit N</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	DBA一看，又要在login_time上建立一个索引。有没有觉得有点问题呢，表上每个字段上都有素引。</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	关系型数据库的数据存储的的不灵活是问题的源头，数据仅有一种储存方法，那就是按行排列的堆表。统一的数据结构意味着你必须使用索引来改变sql的访问路径来快速访问某个列的，而访问路径的增加又意味着你必须使用统计信息来辅助，于是一大堆的问题就出现了。</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	没有索引，没有统计计划，没有执行计划，这就是kv数据库。</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	redis里如何满足以上的需求呢？ 对于求最新的N条数据的需求，链表的后进后出的特点非常适合。我们在上面的登录代码之后添加一段代码，维护一个登录的链表，控制他的长度，使得里面永远保存的是最近的N个登录用户。</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
#把当前登录人添加到链表里
ret = r.lpush(&quot;login:last_login_times&quot;, uid)
#保持链表只有N位
ret = redis.ltrim(&quot;login:last_login_times&quot;, 0, N-1)</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	这样需要获得最新登录人的id，如下的代码即可</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
last_login_list = r.lrange(&quot;login:last_login_times&quot;, 0, N-1)</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	另外，求登录次数最多的人，对于排序，积分榜这类需求，sorted set非常的适合，我们把用户和登录次数统一存储在一个sorted set里。</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
zadd login:login_times 5 1
zadd login:login_times 1 2
zadd login:login_times 2 3</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	这样假如某个用户登录，额外维护一个sorted set，代码如此</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
#对该用户的登录次数自增1
ret = r.zincrby(&quot;login:login_times&quot;, 1, uid)</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	那么如何获得登录次数最多的用户呢，逆序排列取的排名第N的用户即可</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
ret = r.zrevrange(&quot;login:login_times&quot;, 0, N-1)</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	可以看出，DEV需要添加2行代码，而DBA不需要考虑索引什么的。</p>
<h2 style="font-family: 'Trebuchet MS', 'Lucida Grande', Arial, sans-serif; text-transform: uppercase; font-size: 2em; margin: 0px; padding: 0px; line-height: 19.600000381469727px; ">
	TAG系统</h2>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	tag在互联网应用里尤其多见，如果以传统的关系型数据库来设计有点不伦不类。我们以查找书的例子来看看redis在这方面的优势。</p>
<h3 style="font-family: 'Trebuchet MS', 'Lucida Grande', Arial, sans-serif; text-transform: uppercase; font-size: 1.2em; line-height: 19.600000381469727px; ">
	关系型数据库的设计</h3>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	两张表，一张book的明细，一张tag表，表示每本的tag，一本书存在多个tag。</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
mysql&gt; select * from book;
+------+-------------------------------+----------------+
| id   | name                          | author         |
+------+-------------------------------+----------------+
|    1 | The Ruby Programming Language | Mark Pilgrim   |
|    1 | Ruby on rail                  | David Flanagan |
|    1 | Programming Erlang            | Joe Armstrong  |
+------+-------------------------------+----------------+

mysql&gt; select * from tag;
+---------+---------+
| tagname | book_id |
+---------+---------+
| ruby    |       1 |
| ruby    |       2 |
| web     |       2 |
| erlang  |       3 |
+---------+---------+

假如有如此需求，查找即是ruby又是web方面的书籍，如果以关系型数据库会怎么处理？</pre>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
select b.name, b.author&nbsp; from tag t1, tag t2, book b
where t1.tagname = &#39;web&#39; and t2.tagname = &#39;ruby&#39; and t1.book_id = t2.book_id and b.id = t1.book_id</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	tag表自关联2次再与book关联，这个sql还是比较复杂的，如果要求即ruby，但不是web方面的书籍呢？</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	关系型数据其实并不太适合这些集合操作。</p>
<h3 style="font-family: 'Trebuchet MS', 'Lucida Grande', Arial, sans-serif; text-transform: uppercase; font-size: 1.2em; line-height: 19.600000381469727px; ">
	REDIS的设计</h3>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	首先book的数据肯定要存储的，和上面一样。</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
set book:1:name&nbsp;&nbsp; &nbsp;&rdquo;The Ruby Programming Language&rdquo;
Set book:2:name&nbsp;&nbsp; &nbsp; &rdquo;Ruby on rail&rdquo;
Set book:3:name&nbsp;&nbsp; &nbsp; &rdquo;Programming Erlang&rdquo;

set book:1:author&nbsp;&nbsp; &nbsp;&rdquo;Mark Pilgrim&rdquo;
Set book:2:author&nbsp;&nbsp; &nbsp; &rdquo;David Flanagan&rdquo;
Set book:3:author&nbsp;&nbsp; &nbsp; &rdquo;Joe Armstrong&rdquo;</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	tag表我们使用集合来存储数据，因为集合擅长求交集、并集</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
sadd tag:ruby 1
sadd tag:ruby 2
sadd tag:web 2
sadd tag:erlang 3</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	那么，即属于ruby又属于web的书？</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
inter_list = redis.sinter(&quot;tag.web&quot;, &quot;tag:ruby&quot;)</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	即属于ruby，但不属于web的书？</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
inter_list = redis.sdiff(&quot;tag.ruby&quot;, &quot;tag:web&quot;)</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	属于ruby和属于web的书的合集？</p>
<pre style="font-family: 'Courier New', Courier, mono; background-color: rgb(239, 239, 239); margin-top: 5pt; padding-top: 5pt; padding-bottom: 5pt; padding-left: 5pt; white-space: pre-wrap; word-wrap: break-word; line-height: 19.600000381469727px; text-align: justify; ">
inter_list = redis.sunion(&quot;tag.ruby&quot;, &quot;tag:web&quot;)</pre>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	简单到不行阿。</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	从以上2个例子可以看出在某些场景里，关系型数据库是不太适合的，你可能能够设计出满足需求的系统，但总是感觉的怪怪的，有种生搬硬套的感觉。</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	尤其登录系统这个例子，频繁的为业务建立索引。放在一个复杂的系统里，ddl（创建索引）有可能改变执行计划。导致其它的sql采用不同的执行计划，业务复杂的老系统，这个问题是很难预估的，sql千奇百怪。要求DBA对这个系统里所有的sql都了解，这点太难了。这个问题在oracle里尤其严重，每个DBA估计都碰到过。对于MySQL这类系统，ddl又不方便（虽然现在有online ddl的方法）。碰到大表，DBA凌晨爬起来在业务低峰期操作，这事我没少干过。而这种需求放到redis里就很好处理，DBA仅仅对容量进行预估即可。</p>
<p style="font-family: Arial, sans-serif; font-size: 14px; line-height: 19.600000381469727px; text-align: justify; ">
	未来的OLTP系统应该是kv和关系型的紧密结合。<br />
	<br />
	转自：<a href="http://www.hoterran.info/redis_kv_design">http://www.hoterran.info/redis_kv_design</a></p>

