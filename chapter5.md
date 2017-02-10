## Journaling日志简介

Journaling日志是MongoDB中一个非常重要的功能，他保证了数据库服务器在意外断电、自然灾害下数据的完整性 。该功能类似于RDB中的事务日志，使得数据库在意外故障后快速回复，该功能默认打开

## 内存视图

Journaling功能的两个重要内存视图：private view 与 shared view ，这两个视图都是通过MMAP（内存映射）来实现的，对private view的映射的内存修改不会影响到磁盘上，而对shared中数据的变化则会影响到磁盘上的文件，系统会周期性的刷新shared view数据到磁盘上。

shared view在MongoDB启动的过程中，操作系统会将磁盘上的数据文件映射到内存中的shared view，注意，操作系统只是映射，并未加载数据到内存，MongoDB稍后会更具需要加载数据到shared view；

private view内存视图是为读操作保存数据的位置，是MongoDB保存新的写操作的第一个地方，一旦journal日志提交完成，MongoDB会复制private视图中的改变到shared视图，再通过shared视图将数据刷入到磁盘数据文件。

磁盘山的Journaling日志文件是实现写操作持久化保存的地方，MongoDB实例启动时会读取该文件。

## 工作原理

    MongoDB进程启动后，首先将数据文件映射到shared视图中，假如数据文件大小为4000字节，MongoDB会将此大小的数据文件映射到内存中，地址可能为1000000~1004000。如果直接读取地址为100060的内存，我们会得到数据文件中第60个字节的内容，该步骤只加载映射信息，不实际加载数据到内存。
    

## 参考资料



