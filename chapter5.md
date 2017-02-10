## Journaling日志简介

Journaling日志是MongoDB中一个非常重要的功能，他保证了数据库服务器在意外断电、自然灾害下数据的完整性 。该功能类似于RDB中的事务日志，使得数据库在意外故障后快速回复，该功能默认打开

## 内存视图

Journaling功能的两个重要内存视图：private view 与 shared view ，这两个视图都是通过MMAP（内存映射）来实现的，对private view的映射的内存修改不会影响到磁盘上，而对shared中数据的变化则会影响到磁盘上的文件，系统会周期性的刷新shared view数据到磁盘上。

shared view在MongoDB启动的过程中，操作系统会将磁盘上的数据文件映射到内存中的shared view，注意，操作系统只是映射，并未加载数据到内存，MongoDB稍后会更具需要加载数据到shared view；

private view内存视图是为读操作保存数据的位置，是MongoDB保存新的写操作的第一个地方，一旦journal日志提交完成，MongoDB会复制private视图中的改变到shared视图，再通过shared视图将数据刷入到磁盘数据文件。

磁盘山的Journaling日志文件是实现写操作持久化保存的地方，MongoDB实例启动时会读取该文件。

## 工作原理

MongoDB进程启动后，首先将数据文件映射到shared视图中，假如数据文件大小为4000字节，MongoDB会将此大小的数据文件映射到内存中，地址可能为1000000~1004000。如果直接读取地址为100060的内存，我们会得到数据文件中第60个字节的内容，该步骤只加载映射信息，不实际加载数据到内存。

当写操作或修改操作发生时，MongoDB进程首先修改内存中的数据，此时磁盘上的文件数据就与内存中的数据不一致了，如果Mongod启动时没有打开Journaling功能，系统将每60s自动刷新shared视图对应的内存变化数据到磁盘（如果打开了Journaling日志，Mongod将额外产生private 视图，MongoDB将shared视图同步到private view中，该步骤在启动时即进行）；

下面是写操作的过程：以db.colection.insert({key,value})为例，

写操作发生时，MongoDB首先将数据写入到private view中，private view并不予磁盘文件相连接，数据不会刷新到磁盘上；

下一步，MongoDB将private view中的数据批量复制到Journal(该操作周期性完成，启动MongoDB时可通过journalCOmmitInterval来控制，默认100ms)，journal会将写操作记录存储到磁盘上的文件上，进行持久化保存，journal日志文件上的每一条都描述了写操作更改了数据文件上的哪些字节（此时由于数据被写入到journal中，即便MongoDB服务器崩溃了，写操作仍然是安全的，当数据库重启时，Mongod会先读取Journal日志，将写操作引起的变化重新同步到数据文件中）；

当上面的步骤完成后，下面MongoDB会利用journal日志中的写操作记录引起的数据文件变化更新shared view 中的数据；

更新shared view完成后，MongoDB重新利用shared view来对 private view进行映射，防止private view变脏，使其占用的内存空间恢复到初始值（大小基本为0），此时shared view中的数据与磁盘变得不一致了，MongoDB周期性（默认60s，可在启动时通过syncdelay控制）的将shared view中的数据flush到磁盘，进行数据同步；

执行完 data flush后，会通知journal日志已经刷入，一旦journal日志文件只包含全部刷入的写操作，不再用于恢复，MongoDB会将它删除或者作为一个新的日志文件再次使用。

执行


```
> db.serverStatus()

```

可以看到有这么一段：

```
"dur" : {
		"commits" : 30,
		"journaledMB" : 0,
		"writeToDataFilesMB" : 0,
		"compression" : 0,
		"commitsInWriteLock" : 0,
		"earlyCommits" : 0,
		"timeMs" : {
			"dt" : 3067,
			"prepLogBuffer" : 0,//从privateView映射到Logbuffer的时间
			"writeToJournal" : 0,//从logbuffer刷新到journalfile的时间
			"writeToDataFiles" : 0,//从journalbuffer映射到MMF，然后从MMF刷新到磁盘的时间
			"remapPrivateView" : 0//重新映射数据到PrivateView的时间
		}
	},

```



