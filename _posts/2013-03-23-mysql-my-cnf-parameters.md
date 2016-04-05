---
layout: post
title: MySQL配置文件my.cnf详解
date: 2013-03-23 12:24
author: onecoder
comments: true
categories: [my.cnf, mysql, MySQL]
---
<p>
	网上搜到的比较好介绍MySQL配置的文章，留下作为参考用。转自：<a href="http://www.ha97.com/4031.html">http://www.ha97.com/4031.html</a></p>
<p>
	MySQL配置文件my.cnf 例子最详细解释，可以保存做笔记用。<br />
	#BEGIN CONFIG INFO<br />
	#DESCR: 4GB RAM, 只使用InnoDB, ACID, 少量的连接, 队列负载大<br />
	#TYPE: SYSTEM<br />
	#END CONFIG INFO<br />
	#<br />
	# 此mysql配置文件例子针对4G内存。<br />
	# 主要使用INNODB<br />
	#处理复杂队列并且连接数量较少的mysql服务器<br />
	#<br />
	# 将此文件复制到/etc/my.cnf 作为全局设置,<br />
	# mysql-data-dir/my.cnf 作为服务器指定设置<br />
	# (@localstatedir@ for this installation) 或者放入<br />
	# ~/.my.cnf 作为用户设置.<br />
	#<br />
	# 在此配置文件中, 你可以使用所有程序支持的长选项.<br />
	# 如果想获悉程序支持的所有选项<br />
	# 请在程序后加上&rdquo;&ndash;help&rdquo;参数运行程序.<br />
	#<br />
	# 关于独立选项更多的细节信息可以在手册内找到<br />
	#<br />
	#<br />
	# 以下选项会被MySQL客户端应用读取.<br />
	# 注意只有MySQL附带的客户端应用程序保证可以读取这段内容.<br />
	# 如果你想你自己的MySQL应用程序获取这些值<br />
	# 需要在MySQL客户端库初始化的时候指定这些选项<br />
	#<br />
	[client]<br />
	#password = [your_password]<br />
	port = @MYSQL_TCP_PORT@<br />
	socket = @MYSQL_UNIX_ADDR@<br />
	# *** 应用定制选项 ***<br />
	#<br />
	# MySQL 服务端<br />
	#<br />
	[mysqld]<br />
	# 一般配置选项<br />
	port = @MYSQL_TCP_PORT@<br />
	socket = @MYSQL_UNIX_ADDR@<br />
	<br />
	# back_log 是操作系统在监听队列中所能保持的连接数,<br />
	# 队列保存了在MySQL连接管理器线程处理之前的连接.<br />
	# 如果你有非常高的连接率并且出现&rdquo;connection refused&rdquo; 报错,<br />
	# 你就应该增加此处的值.<br />
	# 检查你的操作系统文档来获取这个变量的最大值.<br />
	# 如果将back_log设定到比你操作系统限制更高的值,将会没有效果<br />
	back_log = 50<br />
	# 不在TCP/IP端口上进行监听.<br />
	# 如果所有的进程都是在同一台服务器连接到本地的mysqld,<br />
	# 这样设置将是增强安全的方法<br />
	# 所有mysqld的连接都是通过Unix sockets 或者命名管道进行的.<br />
	# 注意在windows下如果没有打开命名管道选项而只是用此项<br />
	# (通过 &ldquo;enable-named-pipe&rdquo; 选项) 将会导致mysql服务没有任何作用!<br />
	#skip-networking<br />
	# MySQL 服务所允许的同时会话数的上限<br />
	# 其中一个连接将被SUPER权限保留作为管理员登录.<br />
	# 即便已经达到了连接数的上限.<br />
	max_connections = 100<br />
	# 每个客户端连接最大的错误允许数量,如果达到了此限制.<br />
	# 这个客户端将会被MySQL服务阻止直到执行了&rdquo;FLUSH HOSTS&rdquo; 或者服务重启<br />
	# 非法的密码以及其他在链接时的错误会增加此值.<br />
	# 查看 &ldquo;Aborted_connects&rdquo; 状态来获取全局计数器.<br />
	max_connect_errors = 10<br />
	# 所有线程所打开表的数量.<br />
	# 增加此值就增加了mysqld所需要的文件描述符的数量<br />
	# 这样你需要确认在[mysqld_safe]中 &ldquo;open-files-limit&rdquo; 变量设置打开文件数量允许至少4096<br />
	table_cache = 2048<br />
	# 允许外部文件级别的锁. 打开文件锁会对性能造成负面影响<br />
	# 所以只有在你在同样的文件上运行多个数据库实例时才使用此选项(注意仍会有其他约束!)<br />
	# 或者你在文件层面上使用了其他一些软件依赖来锁定MyISAM表<br />
	#external-locking<br />
	# 服务所能处理的请求包的最大大小以及服务所能处理的最大的请求大小(当与大的BLOB字段一起工作时相当必要)<br />
	# 每个连接独立的大小.大小动态增加<br />
	max_allowed_packet = 16M<br />
	# 在一个事务中binlog为了记录SQL状态所持有的cache大小<br />
	# 如果你经常使用大的,多声明的事务,你可以增加此值来获取更大的性能.<br />
	# 所有从事务来的状态都将被缓冲在binlog缓冲中然后在提交后一次性写入到binlog中<br />
	# 如果事务比此值大, 会使用磁盘上的临时文件来替代.<br />
	# 此缓冲在每个连接的事务第一次更新状态时被创建<br />
	binlog_cache_size = 1M<br />
	# 独立的内存表所允许的最大容量.<br />
	# 此选项为了防止意外创建一个超大的内存表导致永尽所有的内存资源.<br />
	max_heap_table_size = 64M<br />
	# 排序缓冲被用来处理类似ORDER BY以及GROUP BY队列所引起的排序<br />
	# 如果排序后的数据无法放入排序缓冲,<br />
	# 一个用来替代的基于磁盘的合并分类会被使用<br />
	# 查看 &ldquo;Sort_merge_passes&rdquo; 状态变量.<br />
	# 在排序发生时由每个线程分配<br />
	sort_buffer_size = 8M<br />
	# 此缓冲被使用来优化全联合(full JOINs 不带索引的联合).<br />
	# 类似的联合在极大多数情况下有非常糟糕的性能表现,<br />
	# 但是将此值设大能够减轻性能影响.<br />
	# 通过 &ldquo;Select_full_join&rdquo; 状态变量查看全联合的数量<br />
	# 当全联合发生时,在每个线程中分配<br />
	join_buffer_size = 8M<br />
	# 我们在cache中保留多少线程用于重用<br />
	# 当一个客户端断开连接后,如果cache中的线程还少于thread_cache_size,<br />
	# 则客户端线程被放入cache中.<br />
	# 这可以在你需要大量新连接的时候极大的减少线程创建的开销<br />
	# (一般来说如果你有好的线程模型的话,这不会有明显的性能提升.)<br />
	thread_cache_size = 8<br />
	# 此允许应用程序给予线程系统一个提示在同一时间给予渴望被运行的线程的数量.<br />
	# 此值只对于支持 thread_concurrency() 函数的系统有意义( 例如Sun Solaris).<br />
	# 你可可以尝试使用 [CPU数量]*(2..4) 来作为thread_concurrency的值<br />
	thread_concurrency = 8<br />
	# 查询缓冲常被用来缓冲 SELECT 的结果并且在下一次同样查询的时候不再执行直接返回结果.<br />
	# 打开查询缓冲可以极大的提高服务器速度, 如果你有大量的相同的查询并且很少修改表.<br />
	# 查看 &ldquo;Qcache_lowmem_prunes&rdquo; 状态变量来检查是否当前值对于你的负载来说是否足够高.<br />
	# 注意: 在你表经常变化的情况下或者如果你的查询原文每次都不同,<br />
	# 查询缓冲也许引起性能下降而不是性能提升.<br />
	query_cache_size = 64M<br />
	# 只有小于此设定值的结果才会被缓冲<br />
	# 此设置用来保护查询缓冲,防止一个极大的结果集将其他所有的查询结果都覆盖.<br />
	query_cache_limit = 2M<br />
	# 被全文检索索引的最小的字长.<br />
	# 你也许希望减少它,如果你需要搜索更短字的时候.<br />
	# 注意在你修改此值之后,<br />
	# 你需要重建你的 FULLTEXT 索引<br />
	ft_min_word_len = 4<br />
	# 如果你的系统支持 memlock() 函数,你也许希望打开此选项用以让运行中的mysql在在内存高度紧张的时候,数据在内存中保持锁定并且防止可能被swapping out<br />
	# 此选项对于性能有益<br />
	#memlock<br />
	# 当创建新表时作为默认使用的表类型,<br />
	# 如果在创建表示没有特别执行表类型,将会使用此值<br />
	default_table_type = MYISAM<br />
	# 线程使用的堆大小. 此容量的内存在每次连接时被预留.<br />
	# MySQL 本身常不会需要超过64K的内存<br />
	# 如果你使用你自己的需要大量堆的UDF函数<br />
	# 或者你的操作系统对于某些操作需要更多的堆,<br />
	# 你也许需要将其设置的更高一点.<br />
	thread_stack = 192K<br />
	# 设定默认的事务隔离级别.可用的级别如下:<br />
	# READ-UNCOMMITTED, READ-COMMITTED, REPEATABLE-READ, SERIALIZABLE<br />
	transaction_isolation = REPEATABLE-READ<br />
	# 内部(内存中)临时表的最大大小<br />
	# 如果一个表增长到比此值更大,将会自动转换为基于磁盘的表.<br />
	# 此限制是针对单个表的,而不是总和.<br />
	tmp_table_size = 64M<br />
	# 打开二进制日志功能.<br />
	# 在复制(replication)配置中,作为MASTER主服务器必须打开此项<br />
	# 如果你需要从你最后的备份中做基于时间点的恢复,你也同样需要二进制日志.<br />
	log-bin=mysql-bin<br />
	# 如果你在使用链式从服务器结构的复制模式 (A-&gt;B-&gt;C),<br />
	# 你需要在服务器B上打开此项.<br />
	# 此选项打开在从线程上重做过的更新的日志,<br />
	# 并将其写入从服务器的二进制日志.<br />
	#log_slave_updates<br />
	# 打开全查询日志. 所有的由服务器接收到的查询 (甚至对于一个错误语法的查询)<br />
	# 都会被记录下来. 这对于调试非常有用, 在生产环境中常常关闭此项.<br />
	#log<br />
	# 将警告打印输出到错误log文件. 如果你对于MySQL有任何问题<br />
	# 你应该打开警告log并且仔细审查错误日志,查出可能的原因.<br />
	#log_warnings<br />
	# 记录慢速查询. 慢速查询是指消耗了比 &ldquo;long_query_time&rdquo; 定义的更多时间的查询.<br />
	# 如果 log_long_format 被打开,那些没有使用索引的查询也会被记录.<br />
	# 如果你经常增加新查询到已有的系统内的话. 一般来说这是一个好主意,<br />
	log_slow_queries<br />
	# 所有的使用了比这个时间(以秒为单位)更多的查询会被认为是慢速查询.<br />
	# 不要在这里使用&rdquo;1&Prime;, 否则会导致所有的查询,甚至非常快的查询页被记录下来(由于MySQL 目前时间的精确度只能达到秒的级别).<br />
	long_query_time = 2<br />
	# 在慢速日志中记录更多的信息.<br />
	# 一般此项最好打开.<br />
	# 打开此项会记录使得那些没有使用索引的查询也被作为到慢速查询附加到慢速日志里<br />
	log_long_format<br />
	# 此目录被MySQL用来保存临时文件.例如,<br />
	# 它被用来处理基于磁盘的大型排序,和内部排序一样.<br />
	# 以及简单的临时表.<br />
	# 如果你不创建非常大的临时文件,将其放置到 swapfs/tmpfs 文件系统上也许比较好<br />
	# 另一种选择是你也可以将其放置在独立的磁盘上.<br />
	# 你可以使用&rdquo;;&rdquo;来放置多个路径<br />
	# 他们会按照roud-robin方法被轮询使用.<br />
	#tmpdir = /tmp<br />
	# *** 复制有关的设置<br />
	# 唯一的服务辨识号,数值位于 1 到 2^32-1之间.<br />
	# 此值在master和slave上都需要设置.<br />
	# 如果 &ldquo;master-host&rdquo; 没有被设置,则默认为1, 但是如果忽略此选项,MySQL不会作为master生效.<br />
	server-id = 1<br />
	# 复制的Slave (去掉master段的注释来使其生效)<br />
	#<br />
	# 为了配置此主机作为复制的slave服务器,你可以选择两种方法:<br />
	#<br />
	# 1) 使用 CHANGE MASTER TO 命令 (在我们的手册中有完整描述) -<br />
	# 语法如下:<br />
	#<br />
	# CHANGE MASTER TO MASTER_HOST=, MASTER_PORT=<br />
	,<br />
	# MASTER_USER=, MASTER_PASSWORD=<br />
	;<br />
	#<br />
	# 你需要替换掉 , ,<br />
	等被尖括号包围的字段以及使用master的端口号替换<br />
	(默认3306).<br />
	#<br />
	# 例子:<br />
	#<br />
	# CHANGE MASTER TO MASTER_HOST=&rsquo;125.564.12.1&prime;, MASTER_PORT=3306,<br />
	# MASTER_USER=&rsquo;joe&rsquo;, MASTER_PASSWORD=&rsquo;secret&rsquo;;<br />
	#<br />
	# 或者<br />
	#<br />
	# 2) 设置以下的变量. 不论如何, 在你选择这种方法的情况下, 然后第一次启动复制(甚至不成功的情况下,<br />
	# 例如如果你输入错密码在master-password字段并且slave无法连接),<br />
	# slave会创建一个 master.info 文件,并且之后任何对于包含在此文件内的参数的变化都会被忽略<br />
	# 并且由 master.info 文件内的内容覆盖, 除非你关闭slave服务, 删除 master.info 并且重启slave 服务.<br />
	# 由于这个原因,你也许不想碰一下的配置(注释掉的) 并且使用 CHANGE MASTER TO (查看上面) 来代替<br />
	#<br />
	# 所需要的唯一id号位于 2 和 2^32 &ndash; 1之间<br />
	# (并且和master不同)<br />
	# 如果master-host被设置了.则默认值是2<br />
	# 但是如果省略,则不会生效<br />
	#server-id = 2<br />
	#<br />
	# 复制结构中的master &ndash; 必须<br />
	#master-host =<br />
	#<br />
	# 当连接到master上时slave所用来认证的用户名 &ndash; 必须<br />
	#master-user =<br />
	#<br />
	# 当连接到master上时slave所用来认证的密码 &ndash; 必须<br />
	#master-password =<br />
	#<br />
	# master监听的端口.<br />
	# 可选 &ndash; 默认是3306<br />
	#master-port =<br />
	# 使得slave只读.只有用户拥有SUPER权限和在上面的slave线程能够修改数据.<br />
	# 你可以使用此项去保证没有应用程序会意外的修改slave而不是master上的数据<br />
	#read_only<br />
	#*** MyISAM 相关选项<br />
	# 关键词缓冲的大小, 一般用来缓冲MyISAM表的索引块.<br />
	# 不要将其设置大于你可用内存的30%,<br />
	# 因为一部分内存同样被OS用来缓冲行数据<br />
	# 甚至在你并不使用MyISAM 表的情况下, 你也需要仍旧设置起 8-64M 内存由于它同样会被内部临时磁盘表使用.<br />
	key_buffer_size = 32M<br />
	# 用来做MyISAM表全表扫描的缓冲大小.<br />
	# 当全表扫描需要时,在对应线程中分配.<br />
	read_buffer_size = 2M<br />
	# 当在排序之后,从一个已经排序好的序列中读取行时,行数据将从这个缓冲中读取来防止磁盘寻道.<br />
	# 如果你增高此值,可以提高很多ORDER BY的性能.<br />
	# 当需要时由每个线程分配<br />
	read_rnd_buffer_size = 16M<br />
	# MyISAM 使用特殊的类似树的cache来使得突发插入<br />
	# (这些插入是,INSERT SELECT, INSERT VALUES (), (), , 以及 LOAD DATA<br />
	# INFILE) 更快. 此变量限制每个进程中缓冲树的字节数.<br />
	# 设置为 0 会关闭此优化.<br />
	# 为了最优化不要将此值设置大于 &ldquo;key_buffer_size&rdquo;.<br />
	# 当突发插入被检测到时此缓冲将被分配.<br />
	bulk_insert_buffer_size = 64M<br />
	# 此缓冲当MySQL需要在 REPAIR, OPTIMIZE, ALTER 以及 LOAD DATA INFILE 到一个空表中引起重建索引时被分配.<br />
	# 这在每个线程中被分配.所以在设置大值时需要小心.<br />
	myisam_sort_buffer_size = 128M<br />
	# MySQL重建索引时所允许的最大临时文件的大小 (当 REPAIR, ALTER TABLE 或者 LOAD DATA INFILE).<br />
	# 如果文件大小比此值更大,索引会通过键值缓冲创建(更慢)<br />
	myisam_max_sort_file_size = 10G<br />
	# 如果被用来更快的索引创建索引所使用临时文件大于制定的值,那就使用键值缓冲方法.<br />
	# 这主要用来强制在大表中长字串键去使用慢速的键值缓冲方法来创建索引.<br />
	myisam_max_extra_sort_file_size = 10G<br />
	# 如果一个表拥有超过一个索引, MyISAM 可以通过并行排序使用超过一个线程去修复他们.<br />
	# 这对于拥有多个CPU以及大量内存情况的用户,是一个很好的选择.<br />
	myisam_repair_threads = 1<br />
	# 自动检查和修复没有适当关闭的 MyISAM 表.<br />
	myisam_recover<br />
	# 默认关闭 Federated<br />
	skip-federated<br />
	# *** BDB 相关选项 ***<br />
	# 如果你运行的MySQL服务有BDB支持但是你不准备使用的时候使用此选项. 这会节省内存并且可能加速一些事.<br />
	skip-bdb<br />
	# *** INNODB 相关选项 ***<br />
	# 如果你的MySQL服务包含InnoDB支持但是并不打算使用的话,<br />
	# 使用此选项会节省内存以及磁盘空间,并且加速某些部分<br />
	#skip-innodb<br />
	# 附加的内存池被InnoDB用来保存 metadata 信息<br />
	# 如果InnoDB为此目的需要更多的内存,它会开始从OS这里申请内存.<br />
	# 由于这个操作在大多数现代操作系统上已经足够快, 你一般不需要修改此值.<br />
	# SHOW INNODB STATUS 命令会显示当先使用的数量.<br />
	innodb_additional_mem_pool_size = 16M<br />
	# InnoDB使用一个缓冲池来保存索引和原始数据, 不像 MyISAM.<br />
	# 这里你设置越大,你在存取表里面数据时所需要的磁盘I/O越少.<br />
	# 在一个独立使用的数据库服务器上,你可以设置这个变量到服务器物理内存大小的80%<br />
	# 不要设置过大,否则,由于物理内存的竞争可能导致操作系统的换页颠簸.<br />
	# 注意在32位系统上你每个进程可能被限制在 2-3.5G 用户层面内存限制,<br />
	# 所以不要设置的太高.<br />
	innodb_buffer_pool_size = 2G<br />
	# InnoDB 将数据保存在一个或者多个数据文件中成为表空间.<br />
	# 如果你只有单个逻辑驱动保存你的数据,一个单个的自增文件就足够好了.<br />
	# 其他情况下.每个设备一个文件一般都是个好的选择.<br />
	# 你也可以配置InnoDB来使用裸盘分区 &ndash; 请参考手册来获取更多相关内容<br />
	innodb_data_file_path = ibdata1:10M:autoextend<br />
	# 设置此选项如果你希望InnoDB表空间文件被保存在其他分区.<br />
	# 默认保存在MySQL的datadir中.<br />
	#innodb_data_home_dir =<br />
	# 用来同步IO操作的IO线程的数量. This value is<br />
	# 此值在Unix下被硬编码为4,但是在Windows磁盘I/O可能在一个大数值下表现的更好.<br />
	innodb_file_io_threads = 4<br />
	# 如果你发现InnoDB表空间损坏, 设置此值为一个非零值可能帮助你导出你的表.<br />
	# 从1开始并且增加此值知道你能够成功的导出表.<br />
	#innodb_force_recovery=1<br />
	# 在InnoDb核心内的允许线程数量.<br />
	# 最优值依赖于应用程序,硬件以及操作系统的调度方式.<br />
	# 过高的值可能导致线程的互斥颠簸.<br />
	innodb_thread_concurrency = 16<br />
	# 如果设置为1 ,InnoDB会在每次提交后刷新(fsync)事务日志到磁盘上,<br />
	# 这提供了完整的ACID行为.<br />
	# 如果你愿意对事务安全折衷, 并且你正在运行一个小的食物, 你可以设置此值到0或者2来减少由事务日志引起的磁盘I/O<br />
	# 0代表日志只大约每秒写入日志文件并且日志文件刷新到磁盘.<br />
	# 2代表日志写入日志文件在每次提交后,但是日志文件只有大约每秒才会刷新到磁盘上.<br />
	innodb_flush_log_at_trx_commit = 1<br />
	# 加速InnoDB的关闭. 这会阻止InnoDB在关闭时做全清除以及插入缓冲合并.<br />
	# 这可能极大增加关机时间, 但是取而代之的是InnoDB可能在下次启动时做这些操作.<br />
	#innodb_fast_shutdown<br />
	# 用来缓冲日志数据的缓冲区的大小.<br />
	# 当此值快满时, InnoDB将必须刷新数据到磁盘上.<br />
	# 由于基本上每秒都会刷新一次,所以没有必要将此值设置的太大(甚至对于长事务而言)<br />
	innodb_log_buffer_size = 8M<br />
	# 在日志组中每个日志文件的大小.<br />
	# 你应该设置日志文件总合大小到你缓冲池大小的25%~100%<br />
	# 来避免在日志文件覆写上不必要的缓冲池刷新行为.<br />
	# 不论如何, 请注意一个大的日志文件大小会增加恢复进程所需要的时间.<br />
	innodb_log_file_size = 256M<br />
	# 在日志组中的文件总数.<br />
	# 通常来说2~3是比较好的.<br />
	innodb_log_files_in_group = 3<br />
	# InnoDB的日志文件所在位置. 默认是MySQL的datadir.<br />
	# 你可以将其指定到一个独立的硬盘上或者一个RAID1卷上来提高其性能<br />
	#innodb_log_group_home_dir<br />
	# 在InnoDB缓冲池中最大允许的脏页面的比例.<br />
	# 如果达到限额, InnoDB会开始刷新他们防止他们妨碍到干净数据页面.<br />
	# 这是一个软限制,不被保证绝对执行.<br />
	innodb_max_dirty_pages_pct = 90<br />
	# InnoDB用来刷新日志的方法.<br />
	# 表空间总是使用双重写入刷新方法<br />
	# 默认值是 &ldquo;fdatasync&rdquo;, 另一个是 &ldquo;O_DSYNC&rdquo;.<br />
	#innodb_flush_method=O_DSYNC<br />
	# 在被回滚前,一个InnoDB的事务应该等待一个锁被批准多久.<br />
	# InnoDB在其拥有的锁表中自动检测事务死锁并且回滚事务.<br />
	# 如果你使用 LOCK TABLES 指令, 或者在同样事务中使用除了InnoDB以外的其他事务安全的存储引擎<br />
	# 那么一个死锁可能发生而InnoDB无法注意到.<br />
	# 这种情况下这个timeout值对于解决这种问题就非常有帮助.<br />
	innodb_lock_wait_timeout = 120<br />
	[mysqldump]<br />
	# 不要在将内存中的整个结果写入磁盘之前缓存. 在导出非常巨大的表时需要此项<br />
	quick<br />
	max_allowed_packet = 16M<br />
	[mysql]<br />
	no-auto-rehash<br />
	# 仅仅允许使用键值的 UPDATEs 和 DELETEs .<br />
	#safe-updates<br />
	[isamchk]<br />
	key_buffer = 512M<br />
	sort_buffer_size = 512M<br />
	read_buffer = 8M<br />
	write_buffer = 8M<br />
	[myisamchk]<br />
	key_buffer = 512M<br />
	sort_buffer_size = 512M<br />
	read_buffer = 8M<br />
	write_buffer = 8M<br />
	[mysqlhotcopy]<br />
	interactive-timeout<br />
	[mysqld_safe]<br />
	# 增加每个进程的可打开文件数量.<br />
	# 警告: 确认你已经将全系统限制设定的足够高!<br />
	# 打开大量表需要将此值设b<br />
	open-files-limit = 8192<br />
	======================================================================<br />
	Mysql参数优化对于新手来讲，是比较难懂的东西，其实这个参数优化，是个很复杂的东西，对于不同的网站，及其在线量，访问量，帖子数量，网络情况，以及机器硬件配置都有关系，优化不可能一次性完成，需要不断的观察以及调试，才有可能得到最佳效果。<br />
	下面先说我的服务器的硬件以及论坛情况，<br />
	CPU: 2颗四核Intel Xeon 2.00GHz<br />
	内存: 4GB DDR<br />
	硬盘: SCSI 146GB<br />
	论坛：在线会员 一般在 5000 人左右 &ndash; 最高记录是 13264.<br />
	下面，我们根据以上硬件配置结合一份已经做过一次优化的my.cnf进行分析说明：有些参数可能还得根据论坛的变化情况以及程序员的程序进行再调整。<br />
	[mysqld]<br />
	port = 3306<br />
	serverid = 1<br />
	socket = /tmp/mysql.sock<br />
	skip-locking # 避免MySQL的外部锁定，减少出错几率增强稳定性。 skip-name-resolve<br />
	禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求!<br />
	back_log = 500<br />
	要求 MySQL 能有的连接数量。当主要MySQL线程在一个很短时间内得到非常多的连接请求，这就起作用，然后主线程花些时间(尽管很短)检查连接并且启动一个新线程。<br />
	back_log值指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。只有如果期望在一个短时间内有很多连接，你需要增加它，换句话说，这值对到来的TCP/IP连接的侦听队列的大小。你的操作系统在这个队列大小上有它自己的限制。试图设定back_log高于你的操作系统的限制将是无效的。当你观察你的主机进程列表，发现大量 264084 | unauthenticated user | xxx.xxx.xxx.xxx | NULL | Connect | NULL | login | NULL 的待连接进程时，就要加大 back_log 的值了。默认数值是50，我把它改为500。<br />
	key_buffer_size = 384M<br />
	# key_buffer_size指定用于索引的缓冲区大小，增加它可得到更好处理的索引(对所有读和多重写)，到你能负担得起那样多。如果你使它太大，系统将开始换页并且真的变慢了。对于内存在4GB左右的服务器该参数可设置为384M或512M。通过检查状态值Key_read_requests和Key_reads,可以知道key_buffer_size设置是否合理。比例key_reads / key_read_requests应该尽可能的低，至少是1:100，1:1000更好(上述状态值可以使用SHOW STATUS LIKE &lsquo;key_read%&rsquo;获得)。注意：该参数值设置的过大反而会是服务器整体效率降低!<br />
	max_allowed_packet = 32M<br />
	增加该变量的值十分安全，这是因为仅当需要时才会分配额外内存。例如，仅当你发出长查询或mysqld必须返回大的结果行时mysqld才会分配更多内存。该变量之所以取较小默认值是一种预防措施，以捕获客户端和服务器之间的错误信息包，并确保不会因偶然使用大的信息包而导致内存溢出。<br />
	table_cache = 512<br />
	table_cache指定表高速缓存的大小。每当MySQL访问一个表时，如果在表缓冲区中还有空间，该表就被打开并放入其中，这样可以更快地访问表内容。通过检查峰值时间的状态值Open_tables和Opened_tables，可以决定是否需要增加table_cache的值。如果你发现open_tables等于table_cache，并且opened_tables在不断增长，那么你就需要增加table_cache的值了(上述状态值可以使用SHOW STATUS LIKE &lsquo;Open%tables&rsquo;获得)。注意，不能盲目地把table_cache设置成很大的值。如果设置得太高，可能会造成文件描述符不足，从而造成性能不稳定或者连接失败。<br />
	sort_buffer_size = 4M<br />
	查询排序时所能使用的缓冲区大小。注意：该参数对应的分配内存是每连接独占!如果有100个连接，那么实际分配的总共排序缓冲区大小为100 &times; 4 = 400MB。所以，对于内存在4GB左右的服务器推荐设置为4-8M。<br />
	read_buffer_size = 4M<br />
	读查询操作所能使用的缓冲区大小。和sort_buffer_size一样，该参数对应的分配内存也是每连接独享!<br />
	join_buffer_size = 8M<br />
	联合查询操作所能使用的缓冲区大小，和sort_buffer_size一样，该参数对应的分配内存也是每连接独享!<br />
	myisam_sort_buffer_size = 64M<br />
	MyISAM表发生变化时重新排序所需的缓冲<br />
	query_cache_size = 64M<br />
	指定MySQL查询缓冲区的大小。可以通过在MySQL控制台执行以下命令观察：<br />
	# &gt; SHOW VARIABLES LIKE &lsquo;%query_cache%&rsquo;; # &gt; SHOW STATUS LIKE &lsquo;Qcache%&rsquo;; # 如果Qcache_lowmem_prunes的值非常大，则表明经常出现缓冲不够的情况;<br />
	如果Qcache_hits的值非常大，则表明查询缓冲使用非常频繁，如果该值较小反而会影响效率，那么可以考虑不用查询缓冲;Qcache_free_blocks，如果该值非常大，则表明缓冲区中碎片很多。<br />
	thread_cache_size = 64<br />
	可以复用的保存在中的线程的数量。如果有，新的线程从缓存中取得，当断开连接的时候如果有空间，客户的线置在缓存中。如果有很多新的线程，为了提高性能可以这个变量值。通过比较 Connections 和 Threads_created 状态的变量，可以看到这个变量的作用<br />
	tmp_table_size = 256M<br />
	max_connections = 1000<br />
	指定MySQL允许的最大连接进程数。如果在访问论坛时经常出现Too Many Connections的错误提示，则需要增大该参数值。<br />
	max_connect_errors = 10000000<br />
	对于同一主机，如果有超出该参数值个数的中断错误连接，则该主机将被禁止连接。如需对该主机进行解禁，执行：FLUSH HOST;。<br />
	wait_timeout = 10<br />
	指定一个请求的最大连接时间，对于4GB左右内存的服务器可以设置为5-10。<br />
	thread_concurrency = 8<br />
	该参数取值为服务器逻辑CPU数量&times;2，在本例中，服务器有2颗物理CPU，而每颗物理CPU又支持H.T超线程，所以实际取值为4 &times; 2 = 8<br />
	skip-networking<br />
	开启该选项可以彻底关闭MySQL的TCP/IP连接方式，如果WEB服务器是以远程连接的方式访问MySQL数据库服务器则不要开启该选项!否则将无法正常连接!<br />
	long_query_time = 10<br />
	log-slow-queries =<br />
	log-queries-not-using-indexes<br />
	开启慢查询日志( slow query log )<br />
	慢查询日志对于跟踪有问题的查询非常有用。它记录所有查过long_query_time的查询，如果需要，还可以记录不使用索引的记录。下面是一个慢查询日志的例子：<br />
	开启慢查询日志，需要设置参数log_slow_queries、long_query_times、log-queries-not-using-indexes。<br />
	log_slow_queries指定日志文件，如果不提供文件名，MySQL将自己产生缺省文件名。long_query_times指定慢查询的阈值，缺省是10秒。log-queries-not-using-indexes是4.1.0以后引入的参数，它指示记录不使用索引的查询。设置long_query_time=10<br />
	外附上使用show status命令查看mysql状态相关的值及其含义：<br />
	使用show status命令<br />
	含义如下:<br />
	aborted_clients 客户端非法中断连接次数<br />
	aborted_connects 连接mysql失败次数<br />
	com_xxx xxx命令执行次数,有很多条<br />
	connections 连接mysql的数量<br />
	Created_tmp_disk_tables 在磁盘上创建的临时表<br />
	Created_tmp_tables 在内存里创建的临时表<br />
	Created_tmp_files 临时文件数<br />
	Key_read_requests The number of requests to read a key block from the cache<br />
	Key_reads The number of physical reads of a key block from disk<br />
	Max_used_connections 同时使用的连接数<br />
	Open_tables 开放的表<br />
	Open_files 开放的文件<br />
	Opened_tables 打开的表<br />
	Questions 提交到server的查询数<br />
	Sort_merge_passes 如果这个值很大,应该增加my.cnf中的sort_buffer值<br />
	Uptime 服务器已经工作的秒数<br />
	提升性能的建议:<br />
	1.如果opened_tables太大,应该把my.cnf中的table_cache变大<br />
	2.如果Key_reads太大,则应该把my.cnf中key_buffer_size变大.可以用Key_reads/Key_read_requests计算出cache失败率<br />
	3.如果Handler_read_rnd太大,则你写的SQL语句里很多查询都是要扫描整个表,而没有发挥索引的键的作用<br />
	4.如果Threads_created太大,就要增加my.cnf中thread_cache_size的值.可以用Threads_created/Connections计算cache命中率<br />
	5.如果Created_tmp_disk_tables太大,就要增加my.cnf中tmp_table_size的值,用基于内存的临时表代替基于磁盘的<br />
	===================================================================<br />
	存储引擎是什么？MySQL中的数据用各种不同的技术存储在文件(或者内 正确的编译方法固然重要，但它只是提高MySQL服务器性能工作的一部分。MySQL服务器的许多参数会影响服务器的性能表现，而且我们可以把这些参数保存到配置文件，使得每次MySQL服务器启动时这些参数都自动发挥作用。这个配置文件就是my.cnf。<br />
	MySQL服务器提供了my.cnf文件的几个示例，它们可以在/usr/local/mysql/share/mysql/目录下找到，名字分别为my-small.cnf、my-medium.cnf、my-large.cnf以及my-huge.cnf。文件名字中关于规模的说明描述了该配置文件适用的系统类型。例如，如果运行MySQL服务器的系统内存不多，而且MySQL只是偶尔使用，那么使用my-small.cnf配置文件最为理想，这个配置文件告诉mysqld daemon使用最少的系统资源。反之，如果MySQL服务器用于支持一个大规模的在线商场，系统拥有2G的内存，那么使用mysql-huge.cnf最为合适。<br />
	要使用上述示例配置文件，我们应该先复制一个最适合要求的配置文件，并把它命名为my.cnf。这个复制得到的配置文件可以按照如下三种方式使用：<br />
	全局：把这个my.cnf文件复制到服务器的/etc目录，此时文件中所定义的参数将全局有效，即对该服务器上运行的所有MySQL数据库服务器都有效。<br />
	局部：把这个my.cnf文件复制到[MYSQL-INSTALL-DIR]/var/将使该文件只对指定的服务器有效，其中[MYSQL-INSTALL-DIR]表示安装MySQL的目录。<br />
	用户：最后，我们还可以把该文件的作用范围局限到指定的用户，这只需把my.cnf文件复制到用户的根目录即可。<br />
	那么，如何设置my.cnf文件中的参数呢？或者进一步说，哪些参数是我们可以设置的呢？所有这些参数都对MySQL服务器有着全局性的影响，但同时每一个参数都和MySQL的特定部分关系较为密切。例如，max_connections参数属于mysqld一类。那么，如何才能得知这一点呢？这只需执行如下命令：<br />
	% &gt;/usr/local/mysql/libexec/mysqld &ndash;help<br />
	该命令将显示出和mysqld有关的各种选项和参数。要寻找这些参数非常方便，因为这些参数都在&ldquo;Possible variables for option &ndash;set-variable (-O) are&rdquo;这行内容的后面。找到这些参数之后，我们就可以在my.cnf文件中按照如下方式设置所有这些参数：<br />
	set-variable = max_connections=100<br />
	这行代码的效果是：同时连接MySQL服务器的最大连接数量限制为100。不要忘了在my.cnf文件[mysqld]小节加上一个set-variable指令，具体请参见配置文件中的示例。<br />
	==================================================================<br />
	正确的编译方法固然重要，但它只是提高MySQL服务器性能工作的一部分。MySQL服务器的许多参数会影响服务器的性能表现，而且我们可以把这些参数保存到配置文件，使得每次MySQL服务器启动时这些参数都自动发挥作用。这个配置文件就是my.cnf。<br />
	MySQL服务器提供了my.cnf文件的几个示例，它们可以在/usr/local/mysql/share/mysql/目录下找到，名字分别为my-small.cnf、my-medium.cnf、my-large.cnf以及my-huge.cnf。文件名字中关于规模的说明描述了该配置文件适用的系统类型。例如，如果运行MySQL服务器的系统内存不多，而且MySQL只是偶尔使用，那么使用my-small.cnf配置文件最为理想，这个配置文件告诉mysqld daemon使用最少的系统资源。反之，如果MySQL服务器用于支持一个大规模的在线商场，系统拥有2G的内存，那么使用mysql-huge.cnf最为合适。<br />
	要使用上述示例配置文件，我们应该先复制一个最适合要求的配置文件，并把它命名为my.cnf。这个复制得到的配置文件可以按照如下三种方式使用：<br />
	全局：把这个my.cnf文件复制到服务器的/etc目录，此时文件中所定义的参数将全局有效，即对该服务器上运行的所有MySQL数据库服务器都有效。<br />
	局部：把这个my.cnf文件复制到[MYSQL-INSTALL-DIR]/var/将使该文件只对指定的服务器有效，其中[MYSQL-INSTALL-DIR]表示安装MySQL的目录。<br />
	用户：最后，我们还可以把该文件的作用范围局限到指定的用户，这只需把my.cnf文件复制到用户的根目录即可。<br />
	那么，如何设置my.cnf文件中的参数呢？或者进一步说，哪些参数是我们可以设置的呢？所有这些参数都对MySQL服务器有着全局性的影响，但同时每一个参数都和MySQL的特定部分关系较为密切。例如，max_connections参数属于mysqld一类。那么，如何才能得知这一点呢？这只需执行如下命令：<br />
	% &gt;/usr/local/mysql/libexec/mysqld &ndash;help<br />
	该命令将显示出和mysqld有关的各种选项和参数。要寻找这些参数非常方便，因为这些参数都在&ldquo;Possible variables for option &ndash;set-variable (-O) are&rdquo;这行内容的后面。找到这些参数之后，我们就可以在my.cnf文件中按照如下方式设置所有这些参数：<br />
	set-variable = max_connections=100<br />
	这行代码的效果是：同时连接MySQL服务器的最大连接数量限制为100。不要忘了在my.cnf文件[mysqld]小节加上一个set-variable指令，具体请参见配置文件中的示例。</p>

