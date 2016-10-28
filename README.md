## sybase简介/版本/产品首页
http://infocenter.sybase.com/help/index.jsp
## sybase版本简介
Sybas ASE 15.7 (Adaptive Server Enterprise) 。早期版本叫SYBASE 10 及SYABSE 11，后来改为Sybase ASE。


安装Sybase
下载安装包（如32位linux版本）解压到安装目录（如/pos/opt/sybase）安装；开发版不需要license有效期为2个月，过期后可重新安装再继续学习。
划设备(device_fragments),建默认数据库(database),分配设备给数据库, 建实例(instance)。

（master主数据库/model/sybsystemprocs系统过程数据库/sybsystemdb系统临时数据库）
（如主设备master、系统过程设备sysprocsdev和systemdbdev）



## sybase简介、安装
Sybas ASE，全称为Adaptive Server Enterprise，是全球著名的基础架构供货商Sybase公司提供的数据库产品。Sybase ASE的早期版本为SYBASE 10 及SYABSE 11，随后，改为Sybase ASE。
安装Sybase =>  下载安装包（如32位linux版本）解压到安装目录（如/pos/opt/sybase）安装；开发版不需要license，有效期为2个月，时间一过，可以重新安装再行学习。安装一个Sybase的主要事项就是，划分出一些设备（如主设备master、系统过程设备sysprocsdev和systemdbdev）,再创建四个默认的数据库（master主数据库/model/sybsystemprocs系统过程数据库/sybsystemdb系统临时数据库），将刚创建的设备分配给指定的数据库,通常master(master设备)/model(master设备)/sybsystemprocs(sysprocsdev设备)/sybsystemdb(master设备&systemdbdev设备)， 再创建一组服务（也称数据库实例），这组服务中以adaptive server为首，来管理这些创建好的设备（也可以理解为管理占用了这些设备的数据库）。 

## Sybase数据库设备（databasedevices）
用于存储组成数据库的对象。在创建设备后，可将它分配到某个数据库，也可以将它分配到设备的缺省池，以便新建数据库或更改现有数据库。名称为Master , sybprocs 和 systemdbdev的设备都是安装完Sybase后默认默认创建的设备。 sp_helpdevices命令可查询主机上所有的设备名称或查询指定设备的信息。
## 数据类型

## store procedure
1. 变量声明： declare @variableName varchar(30)

## sp_helpdevice.txt
在ASE15.x之前的所有版本中（包括ASE12.5.4），存储过程sp_helpdevcie无法显示设备的剩余空间以及设备上各个数据库的具体分配情况。

比如：在ASE12.5中，执行sp_helpdevice master的结果为：

1> sp_helpdevice master
2> go
device_name  physical_name                   description                                                     status    cntrltype     device_number  low    high

master       e:\sybase125\data\master.dat    special, dsync on, default disk, physical disk, 100.00 MB       3         0             0               0      51199

(1 row affected)
(return status = 0)


上面输出结果中加粗标记出来的100.00MB表示master设备的总大小。至于master设备还剩余多少空间？master设备都分配给哪些数据库使用了？ASE15.x之前版本的存储过程sp_helpdevice不能给出答案。

在ASE15.0之后版本的系统存储过程sp_helpdevice增加了上述功能。比如：ASE15.0.3中执行sp_helpdevice master的结果如下：
1> sp_helpdevice master
2> go
 device_name  physical_name                 description                                                                                                                                                                                            status    cntrltype   vdevno   vpn_low    vpn_high
------------- ---------------------------   ---------------------------------------------                                                                                                                                                          ------    ---------   ------   -------    --------
 master       D:\sybase\data\master.dat     file system device, special, MIRROR ENABLED, mirror = 'D:\sybase\data\master_mirr.dat', serial writes, dsync on, directio off, reads mirrored,default disk, physical disk, 80.00 MB, Free: 42.00 MB     739         0           0       0        40959

(1 row affected)
 dbname               size          allocated           vstart lstart
 -----------      -------------     ------------------- ------ ------
 master           26.00 MB Dec  2 2009  6:58PM      4      0
 model             6.00 MB Dec  2 2009  6:58PM  13316      0
 sybsystemdb       6.00 MB Dec  2 2009  6:58PM  19460      0

(1 row affected)
(return status = 0)


那么，如何在ASE15.x之前的版本中查看设备的剩余空间以及设备上的数据库分配信息呢？

方法有两种：

1.使用sybase central这个sybase ASE自带的客户端工具。个人感觉ASE15以来自带的Sybase Central功能比较丰富，更加易用了。

在ASE12.5自带的Sybase Central上查看各设备的剩余空间：



在master设备上点右键，选择“属性”，切换到databases能够看到在master设备上各个数据库所分配的空间。



2.自己编写一个存储过程，比如：sp_helpdevice2。

我参考了ASE12.5.4和ASE15.0.3中的sp_helpdevice的语法完成该过程sp_helpdevice2的编写。分别在ASE v11.0.1, ASE v11.5.1， ASE v11.9.2, ASE v12.5, v12.5.0.3, v12.5.4 平台上进行了测试。

/*
* 此存储过程sp_helpdevice2适用于 ASE v11.x, v12.x,不能用于ASE15.x。实际上ASE15.x中的sp_helpdevice包含设备剩余空间以及设备上所分配的数据库的功能！
* ASE v11.x版本中系统表 sysusages中没有crdate这个表示设备段分配时间的字段，考虑到支持ASEv11.x，为了简单处理，没有在Allocation information 中列出设备段的具体分配时间！
*/

/*
* 此存储过程在ASE v11.0.1, ASE v11.5.1，ASE v11.9.2, ASE v12.5, v12.5.0.3, v12.5.4 平台测试通过！适用于 ASE v11.x, v12.x,不能用于ASE15。实际上ASE15.x中的sp_helpdevice完全能够实现该功能！
* ASE v11.x版本中系统表 sysusages中没有crdate这个表示设备段分配时间的字段，考虑到支持ASEv11.x为了简单处理，没有在Allocation information 中列出设备段的具体分配时间！
*/
use sybsystemprocs
go

if exists(select 1 from dbo.sysobjects where type='P' and name='sp_helpdevice2')
  drop procedure sp_helpdevice2
go

create procedure sp_helpdevice2
@devname varchar(30) = "%"
as

declare @numpgsmb float
declare @numpgsmb2 float
declare @Major_Version int

set nocount on 
select @numpgsmb = (1048576. / @@pagesize)
select @numpgsmb2 = (1048576. / @@maxpagesize)
--select @version_as_num = @@version_as_integer
select @Major_Version= convert(int, right(substring(@@version,1,charindex('.',@@version)-1),2) )

if @Major_Version >= 15 or @Major_Version < 11
begin 
    print "this procedure is available for ASE versions from v11.x to v12.5.x, not for ASE15.x!"
    return (1)
end

/*  See if the device exists.*/
if not exists (select *
            from master.dbo.sysdevices
                where name like @devname)
begin
    /* 17610, "No such i/o device exists." */
    raiserror 17610
    return (1)
end

/* total size of device */
select d.name,
    totalsizeMB = (1. + (d.high - d.low)) / @numpgsmb 
  into #totalsize
    from master.dbo.sysdevices d
        where d.status & 2 = 2
            and name like @devname
        group by d.name

/* Calculate used size in MB */
select d.name, 
    usedsizeMB = isnull(sum(u.size) / @numpgsmb2,0)
  into #usedsize                 
    from master.dbo.sysdevices d, master.dbo.sysusages u 
        where u.vstart >= d.low and u.vstart <= d.high                           
            and d.status & 2 = 2
            and d.name like @devname
        group by d.name
union 
select d.name, 0.
from master.dbo.sysdevices d 
  where not exists ( select 1 from master.dbo.sysusages u where u.vstart >= d.low and u.vstart <= d.high )
      and d.status & 2 = 2
      and d.name like @devname

set nocount off
/* Calculate the free size of device */
select d.name ,TotalSize = str(#totalsize.totalsizeMB,10,2), UsedSize = str(#usedsize.usedsizeMB,10,2),FreeSize = str(#totalsize.totalsizeMB - #usedsize.usedsizeMB,10,2),phyname = convert(varchar(50),d.phyname) 
  from master.dbo.sysdevices d, #totalsize, #usedsize
  where    d.name = #totalsize.name
    and #totalsize.name = #usedsize.name
  order by low,high

if (select count(*) from master.dbo.sysdevices where name like @devname) = 1 
begin 
    print ""
    print "========================== Allocate Information =========================="
    /*if @Major_Version = 12
        select dbname = db_name(dbid), "size(MB)"=str(size/@numpgsmb2,10,2), allocated = u.crdate, vstart, lstart 
          from master.dbo.sysusages u, master.dbo.sysdevices d
            where d.status & 2 = 2 
              and d.name like @devname
              and (u.vstart >= d.low and u.vstart <= d.high )
           order by dbname,vstart
     else if @Major_Version = 11 
     */
        select dbname = db_name(dbid), "size(MB)"=str(size/@numpgsmb2,10,2), vstart, lstart 
          from master.dbo.sysusages u, master.dbo.sysdevices d
            where d.status & 2 = 2 
              and d.name like @devname
              and (u.vstart >= d.low and u.vstart <= d.high )
           order by dbname,vstart

end

drop table #totalsize
drop table #usedsize
go

/* grant the execute privilege to public */
grant execute on sp_helpdevice2 to public
go

 

存储过程sp_helpdevice2的语法在此下载：sp_helpdevice2

使用存储过程sp_helpdevice2的输出结果见下图所示：



最后，存储过程sp_helpdevice2仅输出了一些基本信息，像设备的状态信息等还请结合自带的sp_helpdevice进行查看。

在chinaunix的sybase板块有讨论此博文主题的帖子：怎么查看设备文件上还有多少空间没有被分配出去

有什么建议欢迎在博文后面留言！



## sybase ct_lib接口
Sybase ct-lib API说明

一．Sybase的二种应用接口        2
二．Open Client / Server的编程接口        2
三．Open Client的函数库        2
1．头文件        3
2．库文件        3
3．控制结构        3
四．数据类型        4
五．接口处理流程        6
1．流程图        6
2. 查询操作流程        6
3. 插入操作流程        7
六．接口函数说明        7
1.分配环境结构空间cs_ctx_alloc        7
2.释放环境结构空间cs_ctx_drop        7
3. 初始化ct_init()        8
4. 分配联接结构ct_con_alloc        8
5. 设置/获取连接connection属性ct_con_props()        8
6. 释放一个CONNECTION结构ct_con_drop()        9
7. 设置或获取context属性ct_config()        9
8. 关闭联接ct_exit()        10
9. 分配command结构ct_cmd_alloc()        10
10. 设置/获取CS_COMMAND结构属性值ct_cmd_props        10
11. 初始化SQL串ct_command()        11
12. 向一个初始化好的SQL命令传递参数Ct_param()/ct_setparam()        12
14. 发送SQL命令ct_send()        13
15. 准备结果集ct_results()        13
16. 释放一个COMMAND结构Ct_cmd_drop        16
17. 获取结果集或命令信息ct_res_info()        16
18. 获取数据每一列的描述ct_describe()        17
19. Ct_options        18
20. 将结果集绑定给输出变量ct_bind()        18
21. 向下取一条记录ct_fetch()        18
22. 安装回叫函数Ct_callback()/cs_config()        19
23. 取消一个命令或清除一个结果集Ct_cancel()        20
24. 关闭一个连接ct_close()        20
25. 初始化一个cursor：ct_cursor()        20



一．Sybase的二种应用接口
SqlServer：是Sybase的数据库服务器，用它可管理一个或多个存储在DB中的信息。接口为T_SQL。
      Isql –Uxxx –Pxxxx -Sxxxx
OpenServer：是提供建立一个用户服务器所需的工具与接口。接口为一组C/S的调用函数。又称为OpenServer应用。
二．Open Client / Server的编程接口
    见图一

CT-LIB：Client-lib接口, 它需要CS-LIB支持
CS-LIB：Common-lib：包括：数据类型转换，算术运算，字符集转换，时间操作，排序操作，本地错误处理。
DB-LIB：是Sybase早期版本的接口:，它不需要CS-LIB支持
BulK-LIB：批量拷贝库。支持把数据批量存入数据库中。DB-LIB有自己的批量拷贝接口，所以不需要Bulk-Libs支持。
Sybase的库目录：
$(SYBASE)/devlib     # for debug library versions
$(SYBASE)/lib       # Sybase library directory
三．Open Client的函数库
1．        Client-Lib：是提供对C/S体系结构定义的所有服务的完全访问。，这种访问以一种与查询语言和数据库无关的方式进行。可提供真正的异步接口。是Sybase提供的最新版本的API。以CT-作前缀
DB-Lib：提供建立SqlServer与OpenServer客户应用程序的模块。它提供对Sybase特有的应用服务的完全访问。它与SQL有关，但与Sybase Sql Server无关。        是sybase的早期接口，库文件为Libsybdb.a
2．        CS-Lib：定义了实现数据类型转换，设置字符集，语言类型及核心的数据结构，所有这引起定义与Ct-lib/Server lib都是共享的。以CS-作前缀
3．        Bulk-Lib：批量操作接口。
1．头文件
#include <ctpublic.h>
#include <cspublic.h>
#include < bkpublic.h>   ---批量操作
2．库文件
Libct.a：ct-lib库
Libcs.a：Common-lib库
Libblk.a：批量操作库
Libtcl.a：通用网络库
Libinsck.a：TCP/IP Socket库
Libcomn.a：共享库
   --------后三个是Sybase内部库，用户无法存取。
3．控制结构
控制结构是指应用程序在与SqlServer建立联接和执行命令之前必须分配一些结构。程序通过设置结构的属性值存储相关的环境配置、联接信息等以实现与Sybase的通讯，并控制程序对DB的操作动作。
1．        Context 结构：上下文环境结构CS_CONTEXT
在应用程序初始化ct-lib时必须分配CS_CONTEXT结构，它存储了操作环境的配置信息，如语言类型等。
2．        Connection结构：联结结构CS_CONNECTION
一个联接结构存储了与Sybase SQLServer一次联接的信息,如用户名，密码等。一个CS_CONNECTION结构与SqlServer一个进程对应。在一次联接中，一条命令的执行结果必须处理完才能发送另一条命令如果想要处理一条命令过程中执行另一条命令，则必须建立二个联接。
3．        Command结构：命令结构CS_COMMAND
命令结构存储的是一系列送往Sybase Sql Server命令。
    见图二


    见图三


CS_CONTEXT      ----> ct_config()/cs_config()
CS_CONNECTION  ----> ct_con_props()
CS_COMMAND     ----> ct_command() /ct_cmd_props()  
控制的属性包括：
CS_MAX_CONNECTION：一个context结构中最多与Server建立联接的最大个数。
CS_NETIO： 指定context结构控制的联接是同步还是异步。
CS_TIMEOUT：建立联接的时限。
CS_USERNAME：连接用户名
CS_PASSWORD：用户名密码
CS_APPNAME：应用名

四．数据类型
类型        Open Client类型        Sybase类型        C类型        描述
Binary        CS_BINARY        Binary/varbinary                二进制类型
Bit        CS_BIT        boolean        unsigned char BYTE        位类型
Character        CS_CHAR        Char/varchar        Char        字符型
Datetime        CS_DATETIME        Datetime                8位日期
        CS_DATETIME4        Smalldatetime                4位日期
数字类型        CS_TINYINT        tinyint        unsigned char BYTE        1位整型
        CS_SMALLINT        Smallint (最大32767)        unsigned short int WORD        2位整型
        CS_INT        int        unsigned long int DWORD        4位整型
        CS_DECIMAL        decimal                
        CS_NUMERIC        Numeric                
        CS_FLOAT        float                8位浮点型
        CS_REAL        real                4位浮点型
money        CS_MONEY        money                8位money型
        CS_MONEY4        smallmoney                4位money型
text        CS_TEXT        text                文本型
        CS_IMAGE        image                图像型

五．接口处理流程
1．流程图
见图四

2. 查询操作流程
  为处理一个select返回的结果，在程序中应增加以下功能
A．        建立一个循环，处理从server返回的结果
B．        描述server返回的不同类型的结果
C．        对于server返回的不同类型的结果分别进行处理
D．        把结果集中的数据捆绑到程序变量中输出

3. 插入操作流程
六．接口函数说明
1.分配环境结构空间cs_ctx_alloc
CS_RETCODE cs_ctx_alloc(CS_INT version, CS_CONTEXT **ctx_ptr);
说明：
Version：CT_LIB版本
   CS_VERSION_100      sybase 10.0
   CS_VERSION_110      sybase 11.1
   CS_VERSION_120      sybase 12.0
   CS_VERSION_125      sybase 12.5
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
CS        _MEM_ERROR：失败，内存不足
2.释放环境结构空间cs_ctx_drop
CS_RETCODE cs_ctx_drop(CS_CONTEXT *ctx_ptr);
说明：
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
3. 初始化ct_init()
CS_RETCODE cs_init(CS_CONTEXT *ctx_ptr, CS_INT version);
说明：
Version：CT_LIB版本
   CS_VERSION_100      sybase 10.0
   CS_VERSION_110      sybase 11.1
   CS_VERSION_120      sybase 12.0
   CS_VERSION_125      sybase 12.5
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
CS        _MEM_ERROR：失败，内存不足

4. 分配联接结构ct_con_alloc
    CS_RETCODE ct_con_alloc(CS_CONTEXT *ctx_ptr, CS_CONNECTION **con_ptr)
        返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败

5. 设置/获取连接connection属性ct_con_props()
CS_RETCODE ct_con_props(con_ptr, action, property , buffer，buflen, outlen)
  CS_CONNECTION *con_ptr ：连接指针
         CS_INT  action ：动作
  CS_INT  property：属性名
  CS_VOID *buffer：属性值变量缓冲
  CS_INT  buflen ：属性值缓冲长度
  CS_INT  *outlen ：
说明：
Action：
   CS_SET：设置属性
   CS_GET：获取属性
   CS_CLEAR：设置本属性为默认值
Property:
                      
属性名           buffer值             设置级别          说明
CS_ANSI_BINDS        CS_TRUE/CS_FALSE        Context /. connection        是否使用ansi格式绑定
CS_APPNAME        实际值串        Context /. connection        登录server的应用名
CS_BULK_LOGIN        CS_TRUE/CS_FALSE        connection        是否使用批量功能
CS_CON_STATUS        CS_INT类型的变量        connection        获取连接状态
CS_HOSTNAME        实际值串        connection        客户端机器的主机名
CS_PASSWORD        实际值串        connection        连接用户的密码
CS_USERNAME        实际值串        connection        连接用户名
CS_SERVERNAME        实际值串        connection        连接的服务器名
CS_NETIO        CS_SYNC_IO/CS_ASYNC_IO/
CS_DEFER_IO        Context 
connection        网络I/O是否是同步还是异步
CS_LOGIN_STATUS        CS_TRUE/CS_FALSE        connection        获取登录状态

Buflen：如果属性值buffer是个固定长度或一个变量时，则Buflen=CS_NULLTERM或CS_UNUSED
Outlen：如果是设置属性操作，=NULL
                        如果是获取属性操作，则为本属性的输出值字节数   
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知。
6. 释放一个CONNECTION结构ct_con_drop()
CS_RETCODE ct_con_drop(CS_CONNECTION *conn_ptr)
返回值：
CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知。

7. 设置或获取context属性ct_config()
CS_RETCODE ct_config (cntx_ptr, action, property , buffer，buflen, outlen)
  CS_CONTEXT * cntx _ptr ：连接指针
         CS_INT  action ：动作
  CS_INT  property：属性名
  CS_VOID *buffer：属性值变量缓冲
  CS_INT  buflen ：属性值缓冲长度
  CS_INT  *outlen ：
说明：
Action：
   CS_SET：设置属性
   CS_GET：获取属性
   CS_CLEAR：设置本属性为默认值
   CS_SUPPORTED：
Property：同上
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
8. 关闭联接ct_exit()
CS_RETCODE ct_exit(CS_CONTEXT *cntx , CS_INT option)
说明：
  Option：操作选项
    CS_UNUSED：发送logout命令给server。
如果指定的context中还有尚存结果的联接，则返回CS_FAIL
                          如果没有尚存结果的联接，则关闭所有联接并释放所有资源。                
                CS_FORCE_EXIT：强制关闭所有联接，释放所有的资源。
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
9. 分配command结构ct_cmd_alloc()
CS_RETCODE ct_cmd_alloc(CS_CONNEXTION *conn_ptr,  CS_COMMAND **cmd_ptr)
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知
10. 设置/获取CS_COMMAND结构属性值ct_cmd_props
CS_RETCODE ct_cmd_props(cmd_ptr, action, property, buffer, buflen, outlen)
  CS_COMMAND * cmd _ptr ：连接指针
          CS_INT  action ：动作
  CS_INT  property：属性名
  CS_VOID *buffer：属性值变量缓冲
  CS_INT  buflen ：属性值缓冲长度
  CS_INT  *outlen ：
说明：
Action：
   CS_SET：设置属性
   CS_GET：获取属性
   CS_CLEAR：设置本属性为默认值
Property：
属性名           buffer值             设置级别          说明
CS_CUR_ID        一整值        command        读取Cursor的ID
CS_CUR_NAME        串         Command         读取Cursor的名 
CS_CUR_ROWCOUNT        一整值        Command        读取cursor的行数
CS_CUR_STATUS        一CS_INT值        Command        读取Cursor的状态
CS_HAVE_BINDS        CS_TRUE/CS_FAIL        command        结果集是否绑定
CS_HAVE_CMD        CS_TRUE/CS_FAIL        Command         命令结构是否存在要发送的命令
CS_USERDATA        用户数据        Command        
                Command        
                command        



11. 初始化SQL串ct_command()
CS_RETCODE ct_command(cmd_ptr, type, buffer, buflen, option)
  CS_COMMAND *cmd_ptr
  CS_INT type
  CS_VOID *buffer
  CS_INT buflen 
  CS_INT option
说明：
  Type：初始化的命令类型,，TSQL命令，RPC命令，批数据命令等
          Buffer：命令串。对于RPC，则为存储过程的名(不带参数)
  Buflen：命令串长或CS_NILLTERM
  Option：如果是TSQL命令则为CS_UNUSED
Type类型名         Option值            说明
CS_LANG_CMD
普通的TSQL命令        CS_MORE        Buffer中的命令是全部命令的一部分
        CS_END         Buffer中的命令中全部命令的最后部分
        CS_UNUSED        同CS_END
CS_RPC_CMD
执行存储过程命令        CS_RECOMPILE        在执行前重新编译此存储过程
        CS_NO_RECOMPILE        在执行前不重新编译此存储过程
        CS_UNUSED        同CS_NO_RECOMPILE
CS_SEND_DATA_CMD        CS_COLUMN_DATA        用于文本或图像数据
        CS_BULK_DATA        批量
CS_SEND_BULK_DATA
        CS_BULK_INIT        初始化批量操作命令
        CS_BULK_CONT        继续执行批量操作命令
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知。

12. 向一个初始化好的SQL命令传递参数Ct_param()/ct_setparam()
CS_RETCODE ct_param(cmd_ptr, datafmt, data, datalen, indicator)
   CS_COMMAND *cmd_ptr
   CS_DATAFMT *datafmt  ---数据格式
          CS_VOID *data   --数据
   CS_INT datalen     ---数据长度
   CS_SMALLINT indicator   ---指示器
CS_RETCODE ct_setparam(cmd_ptr, datafmt, data, datalenp, indp)
   CS_COMMAND *cmd_ptr
   CS_DATAFMT *datafmt  ---数据格式
          CS_VOID *data   --数据
   CS_INT *datalenp     ---数据长度
   CS_SMALLINT *indp   ---指示器

                说明：
1．        在ct_send前调用。
2．        如果datafmt->datatype指示本变量是固定长度类型，则datalen可忽略.CS_VARBINARY和CS_VARCHAR可以认为是固定长度类型
3．        Indicator：表示数据是否为空。=-1时表示为空值，此时data及datalen可以忽略
命令类型        Ct_param的作用        Datafmt->status值        Data,  datalen is
Cursor定义        修改列        CS_UPDATECOL        修改列的名与长度
Language         传递参数值        CS_INPUTVALUE        值及值长度
RPC        传递参数值        CS_RETURN        返回值
                CS_INPUTVALUE        非返回值
4．        Datafmt的结构设置
结构成员名        设置值
Name             变量名
Namelen          变量名长度
Datatype          数据类型.
    Status                                CS_INPUTVALUE
  其他字段可忽略

返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知
Ct_param与ct_setparam的区别
  1．Ct_param()是直接将变量中的值传递给SQL，ct_setparam是传递地址，所以在再次ct_send时允许改变参数的值。
14. 发送SQL命令ct_send()
CS_RETCODE ct_send(CS_COMMAND *cmt_ptr)
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_CANCEL：命令被取消
  CS_PENDING：异步操作时状态不可知
         CS_BUSY：在异步操作时表示本连接状态不可知


15. 准备结果集ct_results()
CS_RETCODE ct_results(CS_COMMAND *cmd_ptr, CS_INT *result_type)
说明：
  Result_type：输出变量，表示结果类型。包括：
结果集的类型        Result_type值        说明        结果集内容
命令执行状态        CS_CMD_DONE        表示一个命令已经正常结束        无
        CS_CMD_FAIL        命令执行有错误        无结果集
        CS_CMD_SUCCEED        成功执行一个无结果集的命令，如insert命令        无结果集
Fetch状态        CS_COMPUTE_RESULT        统计结果集行数        结果集行数
        CS_CURSOR_RESULT        Cursor的处理结果        结果集行数
        CS_PARAM_RESULT        返回存储过程的出参        出参的数值
        CS_STATUS_RESULT        返回存储过程的状态        状态
        CS_ROW_RESULT        返回记录行供处理        0或多条记录
Eg:  select …. From  …
      Order by …
      Compute sum() by …


返回值
CS_SUCCEED：结果集已准备好，应用可用
CS_END_RESULTS：所有的结果都已处理完毕
CS_FAIL：执行出错，结果集中的结果无效。
CS_CANCELED：结果集被取消
CS_PENDING/CS_BUSY：
处理的方式为：
  while ((result_ok=ct_results(cmd_ptr,&result_type)) == CS_SUCCEED)
  {
        switch ((int)result_type) 
        {
                case CS_ROW_RESULT:  /*普通行*/
1.        绑定输出列 ct_bind
2.        取出结果集 ct_fetch
                        break;
                case CS_CMD_DONE: /*所有结果处理完毕*/
                        break;
                case CS_CMD_SUCCEED: /*命令执行无错误*/
                        break;
                case CS_CMD_FAIL: /*命令执行有错误*/
                        break;
                case CS_PARAM_RESULT: /*对于rpc，捆绑及取出输出参数*/
1．        绑定输出列 ct_bind
2．        取出结果集 ct_fetch
                        break;
                case CS_STATUS_RESULT: /*对于rpc，捆绑及取出返回状态*/
1．        绑定返回状态（只有一列）ct_bind
2．        取出返回状态值ct_fetch
                        break;
                case CS_COMPUTE_RESULT: /*取出计算数据*/
                        break;
                case CS_CURSOR_RESULT: : /*游标结果集数据*/
1．        绑定输出列 ct_bind
2．        取出结果集 ct_fetch
                        break;
                default:
                        break;
        }
}
/*命令结束处理*/
switch (result_ok)
        {
          case  CS_END_RESULTS: /*所有结果处理完毕*/
                {
                        ret_code=CS_SUCCEED;
                        break;
                }
        case  CS_FAIL:
                {
                        ret_code=CS_FAIL;
                        break;
                }
        default:
                {
                        ret_code=CS_FAIL;
                        break;
                }
        }
各种操作的ct_results返回值与返回类型的各值次序
      result=ct_results(cmd_ptr,&result_type)
        正确操作        说明
        Result_type        Result        
Select操作        1.CS_ROW_RESULT        1CS_SUCCEED        可取结果集:ct_bind/ct_fetch
        2.CS_CMD_DONE        2CS_SUCCEED        命令执行成功
                3CS_END_RESULTS        命令结束
Insert操作        1.CS_CMD_SUCCEED        1CS_SUCCEED        Update与delete同insert一样
        2. CS_CMD_DONE        2CS_SUCCEED        
                3CS_END_RESULTS        
RPC        1.CS_STATUS_RESULT        1. CS_SUCCEED        先得到RPC的返回值:
ct_bind/ct_fetch
        2.CS_PARAM_RESULT        2 CS_SUCCEED        再得到RPC出参值
:ct_bind/ct_fetch
        3. CS_CMD_SUCCEED        3. CS_SUCCEED        
        4 CS_CMD_DONE        4. CS_SUCCEED        
                5CS_END_RESULTS        
CURSOR        1 CS_CMD_SUCCEED        1. CS_SUCCEED        定义cursor
        2 CS_CMD_DONE        2 CS_SUCCEED        
        3 CS_CMD_SUCCEED        3. CS_SUCCEED        打开cursor
        4 CS_CMD_DONE        4. CS_SUCCEED        
        5CS_CURSOR_RESULT        5. CS_SUCCEED        处理结果集: :ct_bind/ct_fetch
        6 CS_CMD_DONE        6. CS_SUCCEED        
                7CS_END_RESULTS        
错误的操作都是一样：
Result_type        Result
1.CS_CMD_FAIL        CS_SUCCEED
2.CS_CMD_DONE        CS_SUCCEED
        CS_END_RESULTS

16. 释放一个COMMAND结构Ct_cmd_drop
CS_RETCODE ct_cmd_drop(CS_COMMAND *cmd_ptr)
返回值
CS_SUCCEED：结果集已准备好，应用可用
CS_FAIL：执行出错，结果集中的结果无效
CS_BUSY：
17. 获取结果集或命令信息ct_res_info()
CS_RETCODE ct_res_info(cmd_ptr,  type, buffer, buflen , outlen )
   CS_COMMAND *cmd_ptr
   CS_INT type
   CS_VOID *buffer
   CS_INT buflen
   CS_INT *outlen
说明：
  Type：
Value of type        返回给buffer的值        Ct_results后的result_type参数值        Buffer类型
CS_CMD_NUMBER        产生结果集的命令条数        任意值        CS_INT
CS_NUM_COMPUTERS        当前命令的computer子句个数        CS_COMPUTER_RESULT        CS_INT
CS_NUMDATA        结果集的项目个数或列个数        CS_COMPUTER_RESULT
CS_ROW_RESULT
CS_PARAM_RESULT
CS_STATUS_RESULT
CS_ROWFMT_RESULT        CS_INT
CS_NUMBERCOLS        Order-by的列数        CS_ROW_RESULT        CS_INT
CS_ROW_COUNT        本次命令涉及的记录行数        CS_CMD_DONE
CS_CMD_FAIL
CS_CMD_SUCCEED        CS_INT
CS_TRANS_STATE        当前的事务状态        任何值        CS_INT
Eg:
/*** Find out how many columns there are in this result set. */
        retcode = ct_res_info(cmd, CS_NUMDATA, &num_cols, CS_UNUSED, NULL);
        if (retcode != CS_SUCCEED)
        {
                ex_error("FetchComputeResults: ct_res_info() failed");
                return retcode;
        }
for (i = 0; i < num_cols; i++)
        {
                /*
                ** Get the column description.  ct_describe() fills the
                ** datafmt parameter with a description of the column.
                */
                retcode = ct_describe(cmd, (i + 1), &datafmt);
                if (retcode != CS_SUCCEED)
                {
                        ex_error("FetchComputeResults: ct_describe() failed");
                        break;
                }
retcode = ct_bind(cmd, (i + 1), &datafmt,
                                coldata.value, &coldata.valuelen,
                                (CS_SMALLINT *)&coldata.indicator);
                if (retcode != CS_SUCCEED)
                {
                        ex_error("FetchComputeResults: ct_describe() failed");
                        break;
                }
        }
18. 获取数据每一列的描述ct_describe()
CS_RETCODE ct_describe(cmd_ptr , item, *datafmt)
CS_COMMAND *cmd_ptr
CS_INT item                ------列或返回值的序号，从1开始
CS_DATAFMT *datafmt;     ----数据属性格式
说明：
  Datafmt会填写以下字段：
字段名        类型        值
name                列名
namelen                
Datatype        列类型        CS_XXX_TYPE
Format        Not used         
Maxlength                最大长度
Scale                
Precision                浮点
Status                浮点
Count                
usertype                
Locale                

19. Ct_options

20. 将结果集绑定给输出变量ct_bind()
CS_RETCODE ct_bind(cmd_ptr, item, datafmt, buffer ,  copied , ind)
  CS_COMMAND *cmd_pte
  CS_INT item  ----绑定的列或参数位置，从1开始，存储过程返回值则按出参顺序从1开始
  CS_DATAFMT *datafmt
  CS_VOID *buffer  ---绑定的输出变量数级组（按列绑定 char aa[col][列的最大长度]）
  CS_INT  *copied   ----拷贝的地址或NULL
  CS_SMALLINT *ind   ----指示器或NULL
说明：
  Datafmt成员设置：
    Name：不设
    Namelen：不设
    Datatype：设置成对应的类型CS_XXX_TYPE
    Format：除了binary, text, image外设置为CS_FMT_UNUSED
                Maxlength：对于固定长度的列可忽略
    Scale：用于浮点数
    Precision：用于浮点数
    Status：不设
    Count：每次绑定的记录个数，一般为1
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知

21. 向下取一条记录ct_fetch()
CS_RETCODE ct_fetch(cmd_ptr, type, offset , option , rows_read)
                        CS_COMMAND *cmd_ptr
            CS_INT type        ---CS_UNUSED 
                        CS_INT offset   ---CS_UNUSED
                        CS_INT option    ---CS_UNUSED
                        CS_INT        *rows_read   ---一次fetch的行数
        返回值：
  CS_SUCCEED：本次fetch执行成功,读出的记录数为roaw_read
  CS_END_DATA：全部数据fetch结束，应用程序应调用ct_results取下一个结果集
         CS_ROW_FAIL： 一次处理中有部分记录是错误的，常用于数组绑定
  CS_FAIL：操作失败
  CS_CANCELED：        取消
CS_BUSY/CS_PENDING：在异步操作时表示本连接状态不可知
    Eg：
         While (((retcode = ct_fetch(cmd,CS_UNUSED, CS_UNUSED, CS_UNUSED,&rowsread)) == CS_SUCCEED)|| (retcode ==CS_ROW_FAIL))
{
          If (retcode==CS_ROW_FAIL){printf(“error on row: %d”,rowsread);}
    For (i=0; i<num_cols ; i++ )
        {
按列取得每一行记录;
}
}
Switch (retcode)
{
        Case  CS_END_DATA:  retcode=CS_SUCCEED; break;
        Default : retcode   CS_FAIL; break;
}
22. 安装回叫函数Ct_callback()/cs_config()
CS_RETCODE ct_callback(cntx_ptr, conn_ptr, action , type , func)
        CS_CONTEXT *cntx_ptr
CS_CONNECTIOn *conn_ptr
CS_INT action   == CS_SET / CS_GET
CS_INT type
CS_VOID *func ----回叫函数名
CS_RETCODE cs_config(cntx, action ,  property, buffer , buflen, outlen )
        CS_CONTEXT *cntx;
CS_INT action ;   == CS_SET / CS_GET/CS_CLEAR
    CS_INT property ：设置的属性值
    CS_VOID *buffer  ----回叫函数名
    CS_INT buflen;  ---CS_UNUSED
    CS_INT *outlen;  ---- NULL
说明
Ct_callback用来设置ct_lib的回叫函数
Type名        说明
CS_CLIENTMSG_CB        客户端的消息处理
CS_SERVERMSG_CB        server端的消息处理
        
Cs_config用来设置cs_lib的回叫函数
Property值         说明
CS_MESSAGE_CB        CS的消息处理
        

返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知

23. 取消一个命令或清除一个结果集Ct_cancel()
CS_RETCODE ct_cancel(conn_ptr, cmd_ptr, type)
CS_CONNECTION *conn_ptr
CS_COMMAND *cmd_ptr
CS_INT type
说明：
1.        如果type= CS_CANCEL_CURRENT则conn_ptr=NULL
2.        如果type= CS_CANCEL_ALL则conn_ptr和cmd_ptr中有一个必须=NULL
Type 值
  CS_CANCEL_CURRENT：仅取消当前的结果集，而不是整批数据。取消命令并没有被送到SQLServer。
  CS_CANCEL_ALL：：取消批命令中的所有结果。Server一收到取消命令，它就停止相应的处理工作。

24. 关闭一个连接ct_close()
CS_RETCODE ct_close(CS_CONNECTION *conn_ptr,  CS_INT option )
Option:
        =CS_FORCE_CLOSE：强制关闭
　　=CS_UNUSED：关闭前发消息给server等当前操作结束后关闭连接　
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知
CS_PENDING
25. 初始化一个cursor：ct_cursor()
CS_RETCODE ct_cursor(cmd_ptr, type, name,  namelen, text ,  textlen , option)
CS_COMMAND *cmd_ptr
CS_INT type ：初始化操作类型
CS_CHAR  name　　：cursor名
CS_INT namelen　　：CS_NULLTERM
CS_CHAR *text　　----cursor对应的ＳＱＬ操作语句
CS_INT textlen  ：CS_NULLTERM
CS_INT option
Type值        说明        Name值        Text值        Option值
CS_CURSOR_DECLARE        定义一个cursor        Cursor名        对应的SQL语句        CS_UNUSED:是否修改则服务器决定
CS_READ_ONLY：只读cursor
CS_FOR_UPDATE:可修改cursor
CS_CURSOR_OPEN        打开一个cursor        NULL        NULL        首次打开时CS_UNUSED
CS_CURSOR_CLOSE        关闭        NULL        NULL        CS_UNUSED：只关闭cursor
CS_DEALLOC：关闭cursor同时释放资源
CS_CURSOR_OPTION        设置cursor属性        NULL         NULL        CS_UNUSED
CS_CURSOR_ROWS        设置cursor一次处理的行数        NULL        NULL        一次处理的行数
CS_CURSOR_DELETE        Cursor执行删除操作        对应的表名        NULL        CS_UNUSED
CS_CURDOR_UPDATE        Cursor执行修改操作        对应的表名        SQL语句        CS_UNUSED
CS_CURSOR_DEALLOC        释放一个cursor        NULL        NULL        CS_UNUSED续 
15. 准备结果集ct_results()
CS_RETCODE ct_results(CS_COMMAND *cmd_ptr, CS_INT *result_type)
说明：
  Result_type：输出变量，表示结果类型。包括：
结果集的类型        Result_type值        说明        结果集内容
命令执行状态        CS_CMD_DONE        表示一个命令已经正常结束        无
        CS_CMD_FAIL        命令执行有错误        无结果集
        CS_CMD_SUCCEED        成功执行一个无结果集的命令，如insert命令        无结果集
Fetch状态        CS_COMPUTE_RESULT        统计结果集行数        结果集行数
        CS_CURSOR_RESULT        Cursor的处理结果        结果集行数
        CS_PARAM_RESULT        返回存储过程的出参        出参的数值
        CS_STATUS_RESULT        返回存储过程的状态        状态
        CS_ROW_RESULT        返回记录行供处理        0或多条记录
Eg:  select …. From  …
      Order by …
      Compute sum() by …


返回值
CS_SUCCEED：结果集已准备好，应用可用
CS_END_RESULTS：所有的结果都已处理完毕
CS_FAIL：执行出错，结果集中的结果无效。
CS_CANCELED：结果集被取消
CS_PENDING/CS_BUSY：
处理的方式为：
  while ((result_ok=ct_results(cmd_ptr,&result_type)) == CS_SUCCEED)
  {
        switch ((int)result_type) 
        {
                case CS_ROW_RESULT:  /*普通行*/
1.        绑定输出列 ct_bind
2.        取出结果集 ct_fetch
                        break;
                case CS_CMD_DONE: /*所有结果处理完毕*/
                        break;
                case CS_CMD_SUCCEED: /*命令执行无错误*/
                        break;
                case CS_CMD_FAIL: /*命令执行有错误*/
                        break;
                case CS_PARAM_RESULT: /*对于rpc，捆绑及取出输出参数*/
1．        绑定输出列 ct_bind
2．        取出结果集 ct_fetch
                        break;
                case CS_STATUS_RESULT: /*对于rpc，捆绑及取出返回状态*/
1．        绑定返回状态（只有一列）ct_bind
2．        取出返回状态值ct_fetch
                        break;
                case CS_COMPUTE_RESULT: /*取出计算数据*/
                        break;
                case CS_CURSOR_RESULT: : /*游标结果集数据*/
1．        绑定输出列 ct_bind
2．        取出结果集 ct_fetch
                        break;
                default:
                        break;
        }
}
/*命令结束处理*/
switch (result_ok)
        {
          case  CS_END_RESULTS: /*所有结果处理完毕*/
                {
                        ret_code=CS_SUCCEED;
                        break;
                }
        case  CS_FAIL:
                {
                        ret_code=CS_FAIL;
                        break;
                }
        default:
                {
                        ret_code=CS_FAIL;
                        break;
                }
        }
各种操作的ct_results返回值与返回类型的各值次序
      result=ct_results(cmd_ptr,&result_type)
        正确操作        说明
        Result_type        Result        
Select操作        1.CS_ROW_RESULT        1CS_SUCCEED        可取结果集:ct_bind/ct_fetch
        2.CS_CMD_DONE        2CS_SUCCEED        命令执行成功
                3CS_END_RESULTS        命令结束
Insert操作        1.CS_CMD_SUCCEED        1CS_SUCCEED        Update与delete同insert一样
        2. CS_CMD_DONE        2CS_SUCCEED        
                3CS_END_RESULTS        
RPC        1.CS_STATUS_RESULT        1. CS_SUCCEED        先得到RPC的返回值:
ct_bind/ct_fetch
        2.CS_PARAM_RESULT        2 CS_SUCCEED        再得到RPC出参值
:ct_bind/ct_fetch
        3. CS_CMD_SUCCEED        3. CS_SUCCEED        
        4 CS_CMD_DONE        4. CS_SUCCEED        
                5CS_END_RESULTS        
CURSOR        1 CS_CMD_SUCCEED        1. CS_SUCCEED        定义cursor
        2 CS_CMD_DONE        2 CS_SUCCEED        
        3 CS_CMD_SUCCEED        3. CS_SUCCEED        打开cursor
        4 CS_CMD_DONE        4. CS_SUCCEED        
        5CS_CURSOR_RESULT        5. CS_SUCCEED        处理结果集: :ct_bind/ct_fetch
        6 CS_CMD_DONE        6. CS_SUCCEED        
                7CS_END_RESULTS        
错误的操作都是一样：
Result_type        Result
1.CS_CMD_FAIL        CS_SUCCEED
2.CS_CMD_DONE        CS_SUCCEED
        CS_END_RESULTS

16. 释放一个COMMAND结构Ct_cmd_drop
CS_RETCODE ct_cmd_drop(CS_COMMAND *cmd_ptr)
返回值
CS_SUCCEED：结果集已准备好，应用可用
CS_FAIL：执行出错，结果集中的结果无效
CS_BUSY：
17. 获取结果集或命令信息ct_res_info()
CS_RETCODE ct_res_info(cmd_ptr,  type, buffer, buflen , outlen )
   CS_COMMAND *cmd_ptr
   CS_INT type
   CS_VOID *buffer
   CS_INT buflen
   CS_INT *outlen
说明：
  Type：
Value of type        返回给buffer的值        Ct_results后的result_type参数值        Buffer类型
CS_CMD_NUMBER        产生结果集的命令条数        任意值        CS_INT
CS_NUM_COMPUTERS        当前命令的computer子句个数        CS_COMPUTER_RESULT        CS_INT
CS_NUMDATA        结果集的项目个数或列个数        CS_COMPUTER_RESULT
CS_ROW_RESULT
CS_PARAM_RESULT
CS_STATUS_RESULT
CS_ROWFMT_RESULT        CS_INT
CS_NUMBERCOLS        Order-by的列数        CS_ROW_RESULT        CS_INT
CS_ROW_COUNT        本次命令涉及的记录行数        CS_CMD_DONE
CS_CMD_FAIL
CS_CMD_SUCCEED        CS_INT
CS_TRANS_STATE        当前的事务状态        任何值        CS_INT
Eg:
/*** Find out how many columns there are in this result set. */
        retcode = ct_res_info(cmd, CS_NUMDATA, &num_cols, CS_UNUSED, NULL);
        if (retcode != CS_SUCCEED)
        {
                ex_error("FetchComputeResults: ct_res_info() failed");
                return retcode;
        }
for (i = 0; i < num_cols; i++)
        {
                /*
                ** Get the column description.  ct_describe() fills the
                ** datafmt parameter with a description of the column.
                */
                retcode = ct_describe(cmd, (i + 1), &datafmt);
                if (retcode != CS_SUCCEED)
                {
                        ex_error("FetchComputeResults: ct_describe() failed");
                        break;
                }
retcode = ct_bind(cmd, (i + 1), &datafmt,
                                coldata.value, &coldata.valuelen,
                                (CS_SMALLINT *)&coldata.indicator);
                if (retcode != CS_SUCCEED)
                {
                        ex_error("FetchComputeResults: ct_describe() failed");
                        break;
                }
        }
18. 获取数据每一列的描述ct_describe()
CS_RETCODE ct_describe(cmd_ptr , item, *datafmt)
CS_COMMAND *cmd_ptr
CS_INT item                ------列或返回值的序号，从1开始
CS_DATAFMT *datafmt;     ----数据属性格式
说明：
  Datafmt会填写以下字段：
字段名        类型        值
name                列名
namelen                
Datatype        列类型        CS_XXX_TYPE
Format        Not used         
Maxlength                最大长度
Scale                
Precision                浮点
Status                浮点
Count                
usertype                
Locale                

19. Ct_options

20. 将结果集绑定给输出变量ct_bind()
CS_RETCODE ct_bind(cmd_ptr, item, datafmt, buffer ,  copied , ind)
  CS_COMMAND *cmd_pte
  CS_INT item  ----绑定的列或参数位置，从1开始，存储过程返回值则按出参顺序从1开始
  CS_DATAFMT *datafmt
  CS_VOID *buffer  ---绑定的输出变量数级组（按列绑定 char aa[col][列的最大长度]）
  CS_INT  *copied   ----拷贝的地址或NULL
  CS_SMALLINT *ind   ----指示器或NULL
说明：
  Datafmt成员设置：
    Name：不设
    Namelen：不设
    Datatype：设置成对应的类型CS_XXX_TYPE
    Format：除了binary, text, image外设置为CS_FMT_UNUSED
                Maxlength：对于固定长度的列可忽略
    Scale：用于浮点数
    Precision：用于浮点数
    Status：不设
    Count：每次绑定的记录个数，一般为1
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知

21. 向下取一条记录ct_fetch()
CS_RETCODE ct_fetch(cmd_ptr, type, offset , option , rows_read)
                        CS_COMMAND *cmd_ptr
            CS_INT type        ---CS_UNUSED 
                        CS_INT offset   ---CS_UNUSED
                        CS_INT option    ---CS_UNUSED
                        CS_INT        *rows_read   ---一次fetch的行数
        返回值：
  CS_SUCCEED：本次fetch执行成功,读出的记录数为roaw_read
  CS_END_DATA：全部数据fetch结束，应用程序应调用ct_results取下一个结果集
         CS_ROW_FAIL： 一次处理中有部分记录是错误的，常用于数组绑定
  CS_FAIL：操作失败
  CS_CANCELED：        取消
CS_BUSY/CS_PENDING：在异步操作时表示本连接状态不可知
    Eg：
         While (((retcode = ct_fetch(cmd,CS_UNUSED, CS_UNUSED, CS_UNUSED,&rowsread)) == CS_SUCCEED)|| (retcode ==CS_ROW_FAIL))
{
          If (retcode==CS_ROW_FAIL){printf(“error on row: %d”,rowsread);}
    For (i=0; i<num_cols ; i++ )
        {
按列取得每一行记录;
}
}
Switch (retcode)
{
        Case  CS_END_DATA:  retcode=CS_SUCCEED; break;
        Default : retcode   CS_FAIL; break;
}
22. 安装回叫函数Ct_callback()/cs_config()
CS_RETCODE ct_callback(cntx_ptr, conn_ptr, action , type , func)
        CS_CONTEXT *cntx_ptr
CS_CONNECTIOn *conn_ptr
CS_INT action   == CS_SET / CS_GET
CS_INT type
CS_VOID *func ----回叫函数名
CS_RETCODE cs_config(cntx, action ,  property, buffer , buflen, outlen )
        CS_CONTEXT *cntx;
CS_INT action ;   == CS_SET / CS_GET/CS_CLEAR
    CS_INT property ：设置的属性值
    CS_VOID *buffer  ----回叫函数名
    CS_INT buflen;  ---CS_UNUSED
    CS_INT *outlen;  ---- NULL
说明
Ct_callback用来设置ct_lib的回叫函数
Type名        说明
CS_CLIENTMSG_CB        客户端的消息处理
CS_SERVERMSG_CB        server端的消息处理
        
Cs_config用来设置cs_lib的回叫函数
Property值         说明
CS_MESSAGE_CB        CS的消息处理
        

返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知

23. 取消一个命令或清除一个结果集Ct_cancel()
CS_RETCODE ct_cancel(conn_ptr, cmd_ptr, type)
CS_CONNECTION *conn_ptr
CS_COMMAND *cmd_ptr
CS_INT type
说明：
1.        如果type= CS_CANCEL_CURRENT则conn_ptr=NULL
2.        如果type= CS_CANCEL_ALL则conn_ptr和cmd_ptr中有一个必须=NULL
Type 值
  CS_CANCEL_CURRENT：仅取消当前的结果集，而不是整批数据。取消命令并没有被送到SQLServer。
  CS_CANCEL_ALL：：取消批命令中的所有结果。Server一收到取消命令，它就停止相应的处理工作。

24. 关闭一个连接ct_close()
CS_RETCODE ct_close(CS_CONNECTION *conn_ptr,  CS_INT option )
Option:
        =CS_FORCE_CLOSE：强制关闭
　　=CS_UNUSED：关闭前发消息给server等当前操作结束后关闭连接　
返回值：
  CS_SUCCEED：执行成功
         CS_FAIL：执行失败
         CS_BUSY：在异步操作时表示本连接状态不可知
CS_PENDING
25. 初始化一个cursor：ct_cursor()
CS_RETCODE ct_cursor(cmd_ptr, type, name,  namelen, text ,  textlen , option)
CS_COMMAND *cmd_ptr
CS_INT type ：初始化操作类型
CS_CHAR  name　　：cursor名
CS_INT namelen　　：CS_NULLTERM
CS_CHAR *text　　----cursor对应的ＳＱＬ操作语句
CS_INT textlen  ：CS_NULLTERM
CS_INT option
Type值        说明        Name值        Text值        Option值
CS_CURSOR_DECLARE        定义一个cursor        Cursor名        对应的SQL语句        CS_UNUSED:是否修改则服务器决定
CS_READ_ONLY：只读cursor
CS_FOR_UPDATE:可修改cursor
CS_CURSOR_OPEN        打开一个cursor        NULL        NULL        首次打开时CS_UNUSED
CS_CURSOR_CLOSE        关闭        NULL        NULL        CS_UNUSED：只关闭cursor
CS_DEALLOC：关闭cursor同时释放资源
CS_CURSOR_OPTION        设置cursor属性        NULL         NULL        CS_UNUSED
CS_CURSOR_ROWS        设置cursor一次处理的行数        NULL        NULL        一次处理的行数
CS_CURSOR_DELETE        Cursor执行删除操作        对应的表名        NULL        CS_UNUSED
CS_CURDOR_UPDATE        Cursor执行修改操作        对应的表名        SQL语句        CS_UNUSED
CS_CURSOR_DEALLOC        释放一个cursor        NULL        NULL        CS_UNUSED续

##  Sybase 命令
1、关闭sybase主服务               shutdown with nowait go
2、关闭sybase某一服务             shutdown SYB_BACKUP（服务名） go
3、查看服务名                     sp_helpserver go
4、查看sybase版本                 select @@version go
5、查看sybase的数据设备信息       sp_helpdevice/select *from master..sysdevices go
6、设备管理
(1)创建
use master go
disk init
name = 'test',physname='/opt/sybase/data/test.dat',vdevno=2,size='1024m',vstart=0,cntrltype=0,dsync=true
go
(2)删除
use master go
exec sp_dropdevice 'test'
go

(3)修改最大的虚拟设备号
sp_configure 'number of devices',25 go
7、数据库管理
(1)创建
use master go
create database test on test='1024M' go
use test go
exec sp_changedbowner 'sa' go
(2)查看当前数据库                 select db_name() go
(3)查看数据库信息                 sp_helpdb syk go
(4)删除                           drop database syk go
(5)空间使用情况
use syk go
sp_spaceused  go
8、默认排序方式、字符集等信息     sp_helpsort
9、执行数据库脚本
isql -Usa -P -SABC -i /opt/sybase/test.sql -o /opt/sybase/test.log
isql -Usa(用户名) -P(密码) -SABC(服务名)
isql参数详解
usage: isql [-b] [-e] [-F] [-p] [-n] [-v] [-X] [-Y] [-Q]
        [-a display_charset] [-A packet_size] [-c cmdend] [-D database]
        [-E editor [-h header [-H hostname [-i inputfile]
        [-I interfaces_file] [-J client_charset] [-K keytab_file]
        [-l login_timeout] [-m errorlevel] [-M labelname labelvalue]
        [-o outputfile] [-P password] [-R remote_server_principal]
        [-s col_separator] [-S server_name] [-t timeout] [-U username]
        [-V [security_options]] [-w column_width] [-z localename]
        [-Z security_mechanism]
(2)执行 isql –Usa –Ppasswd –Sservername –i bcpout.sh –o bcpout.txt
10、查看用户信息                  sp_helpuser
11、用户锁定操作                  sp_locklogin /sp_locklogin username,’lock/unlock’
12、查看登录用户                  sp_displaylogin [loginname]/sp_who

13、bcp
bcp dbname..tablename out /opt/sybase/test.bcp –Usa –P –Sservername –c ————数据备份
bcp dbname..tablename in  /opt/sybase/test.bcp –Usa –P –Sservername –c ————数据还原
一次性导出所有表
(1)建立导出脚本文件(bcpout.sh):
use test go
select ‘bcp test..’+name+’out /opt/sybase/test.txt’+’-Usa –P –Sservername -c’from sysobjects where type=’U’U表示为用户表。
14、列出所有库                    sp_helpdb
15、查看表信息                    sp_help [tablename]

16、数据库死进程                  select * from master..syslogshold

---------------------------------


sybase事务日志已满，怎么清除？
建立索引的时候，提示：
Can't allocate space for object 'POWER_FILE' in database 'dms' because 'default' segment is full/has no free extents. If you ran out of space in syslogs, dump the transaction log. Otherwise, use ALTER DATABASE to increase the size of the segment.
翻译了下，貌似是日志已经满了，于是我在数据库的属性里，又添加了300MB事务日志，还是不行，以前有个人交过我的，他给了一个sql语句让我执行一下就OK了，里面有dump的，但是不记得 
新建一个设备， alter database 数据库名 on 设备名=分配大小 
default段满了？除了增加一个设备，还可以用其他方法吗？增加设备的话不是要浪费存储空间吗 


你的日志满了，设备空间不够，只要在加个数据库设备，然后在执行就应该可以；
数据库属性 日志是否截断啊
我以前也遇到过日志满的问题，不过是在导数据的时候，先将索引删除，禁用触发器，然后在导入；
但如果是 default 和 system 段满的话，也只能加设备了吧，我都是加设备；
还有 可以 rog rebulid table_name 是不是也可以减少空间；
有篇文章不知能否用上，可以看看；（希望对你有所帮助）
*****************************************
*****************************************
1. Sybase数据库日志详解
Sybase　SQL Server用事务（Transaction）来跟踪所有数据库的变化。事务是SQL　Server的工作单元。一个事务包含一条或多条作为整体执行的T－SQL语句。每个数据库都有自己的事务日志（Transaction　Log），即系统表（Syslogs）。事务日志自动记录每个用户发出的每个事务。日志对于数据库的数据安全性、完整性至关重要，我们进行数据库开发和维护必须熟知日志的相关知识。 

（1）Sybase　SQL Server 如何记录和读取日志信息 

Sybase　SQL Server是先记Log的机制。每当用户执行将修改数据库的语句时，SQL　Server就会自动地把变化写入日志。一条语句所产生的所有变化都被记录到日志后，它们就被写到数据页在缓冲区的拷贝里。该数据页保存在缓冲区中，直到别的数据页需要该内存时，该数据页才被写到磁盘上。若事务中的某条语句没能完成，SQL　Server将回滚事务产生的所有变化。这样就保证了整个数据库系统的一致性和完整性。 

（2）日志设备 

Log和数据库的Data一样，需要存放在数据库设备上，可以将Log和Data存放在同一设备上，也可以分开存放。一般来说，应该将一个数据库的Data和Log存放在不同的数据库设备上。这样做有如下好处：一是可以单独地备份Backup　事务日志；二是防止数据库溢满；三是可以看到Log的空间使用情况。 

所建Log设备的大小，没有十分精确的方法来确定。一般来说，对于新建的数据库，Log的大小应为数据库大小的20-30%左右。Log的大小还取决于数据库修改的频繁程度。如果数据库修改频繁，则Log的增长十分迅速。所以说Log空间大小依赖于用户是如何使用数据库的。此外，还有其它因素影响Log大小，我们应该根据实际操作情况估计Log大小，并间隔一段时间就对Log进行备份和清除。 

（3）日志的清除 

随着数据库的使用，数据库的Log是不断增长的，必须在它占满空间之前将它们清除掉。清除Log有两种方法： 

1.自动清除法 

开放数据库选项 Trunc Log on Chkpt，使数据库系统每隔一段时间自动清除Log。此方法的优点是无须人工干预，由SQL　Server自动执行，并且一般不会出现Log溢满的情况；缺点是只清除Log而不做备份。

2.手动清除法 

执行命令“dump transaction”来清除Log。以下两条命令都可以清除日志： 

　　dump transaction with truncate_only
　　dump transaction with no_log

通常删除事务日志中不活跃的部分可使用“dump transaction with trancate_only”命令，这条命令写进事务日志时，还要做必要的并发性检查。SYBASE提供“dump transaction with no_log”来处理某些非常紧迫的情况，使用这条命令有很大的危险性，SQL　Server会弹出一条警告信息。为了尽量确保数据库的一致性，你应将它作为“最后一招”。 

以上两种方法只是清除日志，而不做日志备份，若想备份日志，应执行“dump transaction database_name to dumpdevice”命令。

2. Sybase日志管理 
在创建用户数据库的时候，应尽量为事务日志创建独立的日志设备，这样可以单独备份事务日志、防止数据库溢满、可以看到事务日志的占用情况及可以镜像等。 
dump transaction db_name  with truncate_only      //不备份事务日志，直接清除。 
dump transaction db_name  with no log 
dump transaction db_name to “路径/名字”     //备份事务日志 
检查log大小? 
dbcc checktable(syslogs) 
go 

3. 提示：Can"t allocate space for object "syslogs" in database "csbt?" because the "logsegment" segment is full……

提示数据库的日志已满。

（1） 重启数据库（最笨的但最有效的）。 
（2） 给数据库日志加空间，但必须有足够的空间。 
（3） 找出执行大事物SESSION的ID，KILL它，但也会回滚，而且不定可以KILL得掉。
（4） 截断日志：

如果从未dump transaction过，transactionlog将最终会满。 SQLServer使用log（日志）是出于恢复目的的。当数据库日志满时，服务器将停止事务的继续进行，因为服务器将不能将这些事务写进日志，而服务器不能运行大多数的dump tran命令，因为SQL Server也需在日志中记录这些命令。这就是为什么当其它dump tran命令不能执行时no_log可执行的原因。但是想一下dump transaction with no_log被设计执行的环境，所有对不做并发性检查。如果在对数据库的修改发生时使用dump transaction with no_log，你就会冒整个数据库崩溃的风险。在多数情况下，它们被反映成813或605错误。为了在数据库被修改时，删除transaction log中的不活跃部分可使用dump transaction with trancate_only。这条命令写进transaction log时，并且它还做不要的并发性检查。这两条命令都有与其相关的警告，在命令参考手册中会看到这些警告。请确保在使用其中任一条命令以前，你已理解这些警告和指示。 Sybase提供dump transaction with no_log来处理某些非常紧迫的情况。为了尽量确保你的数据库的一致性，你应将其作为“最后一招”。 

查看是否截断，可用如下方法解决：
（1）如果是 master 库：
dump tran master with no_log 
（2）如果是用户数据库（如：csbt）：
可以等待自动清理，过5分钟后，再重启SQLSERVER；否则：
use master
go


http://sybase-addict.com/re-sync-a-warm-standby-database-with-dump-marker/
sp_dboption csbt,"trunc. log on chkpt",false
go
checkpoint
go
dump tran csbt with no_log
go
sp_dboption csbt,"trunc. log on chkpt",true
go
checkpoint
go


4. 当设置truncate log on checkpoint，对于日志备份就没有意义，建议在生成环境上不要打开truncate log on checkpoint这个选项。

5. Sybase日志无法清除或截断。在使用dump tran.....后清除不了日志，且重启动sybase也无法清除，（已设置了trunc log on chkpt选项）。
（1） 应用系统给SQL Server发送了大量的用户自定义事务，一直未提交，这些最早活跃事务阻碍系统截断日志。要督促用户退出系统或者提交事务，便可清掉日志。因为给SQL Server发送Dump transaction with no-log或者with truncate-only，它截掉事务日志的非活跃部分。所谓非活跃部分是指服务器检查点之间的所有已提交或回退的事务。而从最早的未提交的事务到最近的日志记录之间的事务日志记录被称为活跃的。从此可以看明，打开的事务能致使日志上涨，因为在最早活跃事务之后的日志不能被截除。
（2） 二是客户端向SQL Server发送了一个修改数量大的事务，清日志时，该事务还正在执行之中，此事务所涉及的日志只能等到事务结束后，才能被截掉。在处理它时，需慎重从事。如果这个大事务已运行较长时间，应尽量想法扩大数据库日志空间，保证该事务正常结束。若该事务被强行回滚，SQL Server需要做大量的处理工作，往往是正向执行时间的几倍，系统恢复时间长，可能会影响正常使用的时间。
（3） 是否使用了复制服务器？主数据库是否增加了表或者字段？
（4） 如果使用了sybase复制服务器，会存在第二截断点问题。如果复制进程由于某种原因无法正常工作，那么会导致ASE的日志充满的问题，你可以使用下面的命令来忽略第二截断点，但是这样做的时候，会导致复制数据不能同步，需要手工同步。设置数据库为单用户，使用 dbcc tablealloc(syslogs,full,fix) 检查，修复一下数据库日志空间，看是否有问题。如果忽略复制的第二截断点，使用
dbcc settrunc(‘item’,’ignore’)
go

如果复制的第二截断点没有清除，使用
use csbt
go
sp_config_rep_agent csbt,'disable'
go
checkpoint
go

（5） 如果tempdb满了的话，是无法使用dump tran tempdb with truncate_only 或no_log清除掉的，而且用select * from master..syslogshold这个命令查看是哪个进程，在这种状态下只能kill 其他进程，直到可以使用dump tran。

--------------------------------
sp_help是要为其提供信息的数据库名称。name 的数据类型为 sysname，无默认值。如果没有指定 name，则 sp_helpdb 报告 master.dbo.sysdatabases 中的所有数据库。
-------
sp_spaceused 
reserved	data	        index_size	unused
270274692 KB	228917992 KB	36193192 KB	5163508 KB
------------
syabse 中，查看设备的空闲空间，是采用sp_spaceused还是sp_helpdb？采用这两个命令得到的结果不一致？

通过sp_helpdb 看到数据段占用2044+2044= 4088M空间，剩余空间（free kbytes）是1343632+1971264=3314896k=3237m
那么数据段占用了4088-3237=851m

通过sp_helpdb算出来的数据段占用的空间是接近用sp_spaceused得到的结果的。 
sp_spaceused 显示reserved为851120k=831m


这2者的误差为：851-831=20m

如果你想了解清楚为什么有20m的误差，那么建议你详细研究一下sp_helpdb & sp_spaceused这2个存储过程的语法。 
毕竟计算的算法不同，得到的计算结果也不同了。 
-----------
sp_helpdb 一台主机服务器上安装了多少个数据库。解析：当客户端已经连接了该主机某个实例（i.e. mx7a_125_sql），然后执行
sp_helpdb命令，该实例（进程）不仅能提供它所管辖下的数据库的信息（数据库名、id、owner和创建时间），还可以与其他实例通信？提供该主机上其他实例所管辖的数据库信息。


## Sybase basic
## Sybas ASE，全称为Adaptive Server Enterprise，是全球著名的基础架构供货商Sybase公司提供的数据库产品。Sybase ASE的早期版本为SYBASE 10 及SYABSE 11，随后，改为Sybase ASE。														
														
## 安装Sybase =>  下载安装包（如32位linux版本）解压到安装目录（如/pos/opt/sybase）安装；开发版不需要license，有效期为2个月，时间一过，可以重新安装再行学习。下图为windows版本安装完成后的显示页面（具体安装过程可参考http://blog.csdn.net/wgw335363240/article/details/8240951）可见安装一个Sybase的主要事项就是，划分出一些设备（如主设备master、系统过程设备sysprocsdev和systemdbdev）,再创建四个默认的数据库（master主数据库/model/sybsystemprocs系统过程数据库/sybsystemdb系统临时数据库），将刚创建的设备分配给指定的数据库,通常master(master设备)/model(master设备)/sybsystemprocs(sysprocsdev设备)/sybsystemdb(master设备&systemdbdev设备)， 再创建一组服务（也称数据库实例），这组服务中以adaptive server为首，来管理这些创建好的设备（也可以理解为管理占用了这些设备的数据库）。														
														
														
														
														
														
														
														
														
														
														
														
														S
														
														
														
														
														
														
														
														
														
														
														
														
														
														
														
														
## Sybase数据库设备=> 英文名叫databasedevices，用于存储组成数据库的对象。在创建设备后，可将它分配到某个数据库，也可以将它分配到设备的缺省池，以便新建数据库或更改现有数据库。名称为Master , sybprocs 和 systemdbdev的设备都是安装完Sybase后默认默认创建的设备。 sp_helpdevices命令可查询主机上所有的设备名称或查询指定设备的信息。														
														
## adaptive server（Sybase数据库实例）=>  1.  Create an Adaptive Server为其命名（见## 命名规则）2.为 Adaptive server设置页大小（如16K）3. 指定其主设备（已创建完）的路径和大小														
														
														
														
														
														
														
														
														
														
														S
														
														
														
														
														
														
														
														
														
														
														
														
														
## Sybase服务（也叫做实例）=> 服务名称、绑定的IP地址和端口号标识一个Sybase 服务。其本质是由一组内存结构和访问数据库文件的后台进程组成。一台主机上可以安装多个数据库服务器（实例），但各自占用的端口一定不同；一个数据库服务器（实例）可以管理多个数据库。一个数据库就是一组数据集合，物理上表现为一组数据文件。 														
														
														
														
														
														
## 数据库实例命名规则 => 需要体现出实例在哪个主机上（例如SCB公司某实例命名为mx10a_sql，而magdb10是该实例所属的主机名，登陆magdb10.uk后用hostname命令可查询当前主机的名称）														
														
## 														
														
														
##  查询实例的端口														
如果是windows，查看 %SYBASE%\ini\sql.ini														
如果是linux/unix，cat $SYBASE/interfaces														
														
														
														
Sybase基础知识														
http://blog.csdn.net/s04023083/article/details/6795831														
														
http://blog.sina.com.cn/s/blog_7d6fac110100x9r3.html														
														
http://blog.csdn.net/wangwenwen/article/details/43084841														
														
														
														
														
create procedure p_convert_num_to_char 														
@cint numeric, @outchar varchar(10) output 														
as 														
if (@cint=4)														
   select @outchar='Four'														
if (@cint=5)														
   select @outchar='Five'														
if (@cint=6)														
   select @outchar='Six'   														
(1 row affected)														
(return status = 0)														
														
18、 oracle中查看源码，查看该sources表														
19、 在一个过程中调用另外一个过程，														
例如：														
create procedure test_one @test_proc_name varchar(255)														
as exec @test_proc_name														
20、 记住在sybase中执行sql是用'go'不是以'；'为结束兼执行														
21、创建数据库用户:sa用户 master库中，sp_addlogin user,password,dbname														
22、变更数据属主：:sa用户进入要变更的数据库执行 sp_changedbowner user,dbname														
23、设置用户的默认登陆数据库：:sa用户进入要设定的数据库执行：														
sp_defaultdb user,dbname														
24、以#开头的临时表只是在某一过程或sql操作中存在，一旦过程或sql操作结束，则临时表就不存在了，如果再要访问就回出错。解决是不建立以#为preffix的表。														
25、要想直接手工插入值到表中identity字段，需要打开该表的identity_insert选项。														
Set identity_insert 表名 on/off														
如：														
set identity_insert t_dns_rezo_gs on														
这样insert into t_dns_rezo_gs(rzgs_id,rzgs_name) values(999,'12121')才会成功。														
26、指定某个过程什么时候执行后用waitfor delay "hh24:mi:ss",并且用了这种方式后该connection不会有什么响应直到过程被执行完成。														
如半个小时后执行过程test_p														
begin														
waitfor delay "0:30:00"														
    exec test_p														
end 														
27、 调用带返回参数的过程完整例子														
create procedure p_convert_num_to_char 														
@cint numeric, @outchar varchar(10) output 														
as 														
if (@cint=4)														
   select @outchar='Four'														
if (@cint=5)														
   select @outchar='Five'														
if (@cint=6)														
   select @outchar='Six'   														
go														
														
declare @getchar varchar(10) 														
exec p_convert_num_to_char 4,@outchar=@getchar output														
28、 过程中有返回参数时，如果预先设定参数值，最终都会改变														
如:														
declare @First int														
select @First=123														
exec test_p @second=@First output														
//运行结果为999														
则@second和@First 都为999														
29、过程名改名sp_rename oldname, newName														
30、ct-library编程，在sybase提供的linux中有，环境搭建要点，要确定$SYBASE设定了，$SYBASE_OCS设定open client所在目录即可不要是全目录，还要设定平台$SYBPLATFORM=linux; 具有这三个环境变量，open client提供的sample才可大部分编译通过；其中有几个程序由于找不到-lsrv，而编译通不过。推测可能涉及了open server的东西，所以没有通过。														
例子程序可以单独编译，如make 例子名    [F.E   make firstapp														
 ]														
设定LD_LIBRARY_PATH=$SYBASE/$SYBASE_OCS/lib														
														
编译形式例如firstapp.c														
$SYBASE=/opt/sybase-12.5														
$SYBASE_OCS= OCS-12_5														
cc -o fist firstapp.c -I. -I/opt/sybase-12.5/OCS-12_5/include -L/opt/sybase-12.5/OCS-12_5/lib -rdynamic -ldl -lnsl -lm  -lct -lcs -lsybtcl -lcomn -lintl														
31、 db-library编译语句：														
cc  -I. -I/opt/sybase-12.5/OCS/include example1.c /opt/sybase-12.5/OCS/lib/libsybdb.a -lm -o example1														
   同样要设好SYBASE SYBASE_OCS SYBPLATFORM														
   并且要保证interface文件中连接服务器是对的。														
   同时对于想要连接的服务器名要在环境变量DSQUERY中设好。														
  如：														
  export DSQUERY=accunetsvr														
														
  注意，用hostname作为连接名时，确保/etc/hosts中的IP和hostname有对应且对应正确。														
														
32、db-library 经实验数据库连接结构支持线程间的传递，将函数打包用下列样式：														
gcc -c -I. -I/opt/sybase-12.5/OCS/include s_fcts.c														
ar r libleon.a s_fcts.o														
rm -rf *.o														
打包完毕														
调用函数进行编译样式：														
cc -I. -I/opt/sybase-12.5/OCS/include ss_nip_checkUGandUser.c libleon.a														
         /opt/sybase-12.5/OCS/lib/libsybdb.a -lm -o leon4 														
或者														
cc  -I. -I/opt/sybase-12.5/OCS/include -o leon4 ss_nip_checkUGandUser.c libleon.a														
 /opt/sybase-12.5/OCS/lib/libsybdb.a  -lm														
33、DB-library的应用程序运行其他机上访问另一台机（数据库所在的机器）.在客户机上需要装sybase-comom 和syabse-openclient组件。需要设定对SYBASE / DSQUERY. 其中sybase要设定为指向interfaces文件的路径,DSQUERY要设定为要远程访问的主机名(adapative_server_name).将远程主机的interfaces拷贝到客户机上SYBASE指定的目录即可。注意如果没有设定DSQUERY，则程序默认去找sybase的数据库，这时如果没有该数据库名字在interfaces文件中，程序就会不能运行。														
   [实际只需设定好DSQUERY环境变量和interface文件即可]														
34、DB-library应用在多线程中每次都要重新连接数据库，否则一定时间后连接会被操作系统重置掉。  Connection reset by peer														
35、DB-library的错误捕捉。系统提供一种系统错误信息函数自动在出现错误时去捕捉显示错误信息。														
   int msg_handler(dbproc, msgno, msgstate, severity, msgtext, 														
                srvname, procname, line)														
DBPROCESS       *dbproc;														
DBINT           msgno;														
int             msgstate;														
int             severity;														
char            *msgtext;														
char            *srvname;														
char            *procname;														
int     line; 														
{};														
int err_handler(dbproc, severity, dberr, oserr, dberrstr, oserrstr)														
DBPROCESS       *dbproc;														
int             severity;														
int             dberr;														
int             oserr;														
char            *dberrstr;														
char            *oserrstr;														
{};														
														
　　dbmsghandle(msg_handler); 														
dberrhandle(err_handler);														
														
除此之外，用户如果想要自己控制错误信息可在dbsqlexec() 调用后并且while()处理完后，调用dbcount(dbproc)进行错误信息判断，如果等于-1则出现错误。特例调用没有select的过程::dbnocount设为on::没有select的sql语句::sql写错::sql执行错误等都可以出现-1所以要小心判断处理。														
36、Jdbc连接sybase。首先需要jconn2.jar和jTDS2.jar文件，在环境变量CLASSPATH设定好。														
Class.forName("com.sybase.jdbc2.jdbc.SybDriver");														
url结构为: jdbc:sybase:Tds:dbserver_ip/dbserver_hostname:dbserver_port/dbname														
dbserver的端口为数据库服务器上的interfaces文件中对应的数据库端口。														
例子：														
192.168.0.6														
interfaces														
[root@accunetsvr sybase-12.5]# more interfaces 														
accunetsvr_text														
        master tcp ether accunetsvr 4500														
        query tcp ether accunetsvr 4500														
														
														
accunetsvr														
        master tcp ether accunetsvr 4100														
        query tcp ether accunetsvr 4100														
														
														
accunetsvr_back														
        master tcp ether accunetsvr 4200														
        query tcp ether accunetsvr 4200														
														
														
accunetsvr_mon														
        master tcp ether accunetsvr 4300														
        query tcp ether accunetsvr 4300														
														
														
ACCUNETSVR_XP														
        master tcp ether accunetsvr 4400														
        query tcp ether accunetsvr 4400														
														
可知dbserver_name是accunetsvr 														
  dbserver_ip    是192.168.0.6														
dbserver_port  是4100														
dbname为nextip														
  url为  jdbc:sybase:Tds:192.168.0.6:4100/nextip														
37、创建identity列，如果是create table 时一定是numeric型。如果想要创建数据库自动为新建的所有表增加一个隐藏的identity字段，用[sp_dboption database_name, "auto identity", "true"]。在检索数据的时候必须隐式加上SYB_IDENTITY_COL作为隐藏的identity列，例如select SYB_IDENTITY_COL, sn_name from t-subnet														
默认的隐藏精度大小为10如果用户想要增大其精度，可用[sp_configure  "size of  auto  identity", 新的精度]，例如: sp_configure "size of auto identity",15														
38、从select into 创建一个新的idenetity列，这在sql语句分页检索用。														
Select idenetity_name=identity(精度) , *  into new_table from old_table;														
如：														
select id0=identity(18),* into #subnets from t_subnet where sn_type=10;														
39、  实现用sql语句进行分页查询方法：														
A.创建一个临时表带identity字段 select id=indentity(20), * into #table_anme from table_name where 条件														
B.然后根据id进行检索第n条到m条数据 (也可用between and)														
C. 最后Drop掉该临时表														
D.注意要打开数据库的select into /bulk copy属性 sp_dboption database_name, "select into/bulk copy", "true"才能进行select into操作														
E.mssql中格式为select identity(int)  id, * from #table_name from table_name where 条件														
40、linux下访问sql-server用db-library与sybase相同要素。只是远程访问端口为sql-server指定的1433														
														
41、 JDBC访问MS-SQLSERVER 														
连接数据库：[需要这三个jar文件msbase.jar msutil.jar mssqlserver.jar]														
JDBC DRIVER：com.microsoft.jdbc.sqlserver.SQLServerDriver														
URL：jdbc:microsoft:sqlserver://Ip Or Name:1433;DatabaseName=XXX														
42、Oracle中的外连接符为=(+) 或(+)= 在Sybase中为=* 或 *=														
43、执行sybase过程中会有日志满了或存储空间不够了，出现supsend状态，可用														
isql  -Usa -Ppassword -Sdbservername														
进去执行dump tran db_name with truncate_only进行清空操作														
或者dump tran db_name to 'path/file'进行备份在执行清空。														
44、ms-sql中的substring(string, start, length)函数参数，start和length为INT型不能为numeric型。														
45、select * into 在oracle中的用法在ms-sql和sybase中的用法为select @变量=column 														
from 表名 where 条件														
46、游标在ms-sql和sybase中差别主要为while 判断的全局变量不同。														
Sybase中为@@sqlstatus = 0														
Ms-sql中为@@FETCH_STATUS = 0														
   关闭游标时在ms-sql中除了[close 游标名] 还要增加[deallocate 游标名]														
   														
47、 MS-SQL对于sql语句大小写不敏感，sybase对大小写敏感。														
														
48、 oracle同sybase和mssql的常用函数对比														
ORACLE SYBASE MS-SQLSERVER														
SysdateTo_char(, '格式')格式:yyyymmddhh24miss任意组合getdate()可用year() month() day()分解获得年月日 convert(varchar, getdate(), 108)是hh:mm:ssgetdate()可用year() month() day()分解获得年月日convert(varchar, getdate(), 108)108是hh:mm:ss120是yyyy-mm-dd														
Length() Datalength() Datalength()														
Ltrim() rtrim()Ltrim() rtrim() Ltrim() rtrim()														
Substr() Substring()Substring(varchar, INT, INT)														
Replace('123', '2', 'A)Replace('123', '2', 'A)														
instr														
Lpad() rpad() Replicate('0', 32)Replicate('0', 32)														
Upper() lower() Upper() lower()Upper() lower()														
														
49、 日期计算在ms-sql中														
dateadd(日期代码，日期值， 日期)														
select convert(varchar,getdate(),120) as year,convert(varchar,dateadd(ss, 1200,getdate()),120) as nYear														
go														
日期部分           简写               值														
year                yy                1753--9999														
quarter             qq                1--4														
month               mm                1--12														
day of year         dy                1--366														
day                 dd                1--31														
week                wk                1--53														
weekday             dw                1--7(Sunday--Saturday)														
hour                hh                0--23														
minute              mi                0--59														
second              ss                0--59														
milisecond          ms                0-999														
														
也可														
select convert(varchar, getdate(), 111)+' '+convert(varchar,getdate(),108) as oldtime,convert(varchar,dateadd(ss,1200,getdate()),111)+' '+convert(varchar,dateadd(ss,1200,getdate()),108) as date														
也可														
select convert(varchar,convert(datetime,'20031223'),111)														
go														
														
														
50、PostgreSQL中的lib编程时，select 和update /delete的成功失败条件判断是不同的。														
Update/delete:: strcmp(PQcmdStatus(temp_res), "")==0														
select :: !temp_res||PQresultStatus(temp_res) != PGRES_TUPLES_OK														
51、 日期各格式的引用模式ms-sql和sybase中都通用														
yyyy/mm/dd hh24:mi:ss														
select convert(varchar, getdate(), 111)+' '+convert(varchar,getdate(),108) as oldtime,convert(varchar,dateadd(ss,1200,getdate()),111)+' '+convert(varchar,dateadd(ss,1200,getdate()),108) as date														
select name,text from all_source where type='FUNCTION' and name='F_CHECK_IB_SCOPE';														
select convert(varchar,convert(datetime,'20031223'),111)														
go														
Select count(ipgs_host_name)||',host-'||f_ipad_change_dec4('%s','1') From t_ipdev_gs														
														
yyyy/mm/ddhh24miss														
Select CO_DESC,to_char(sysdate - 1/48,'yyyymmddhh24miss') co_date from T_LICENSE														
dateadd(ss, -1800, getdate())														
select co_desc, convert(varchar, dateadd(ss, -1800, getdate()),111)+convert(varchar,dateadd(ss, -1800, getdate()),108) as co_date from t_license														
select convert(varchar, getdate(), 111)+' '+convert(varchar,getdate(),108) as oldtime,convert(varchar, dateadd(ss, -1800, getdate()),111)+convert(varchar,dateadd(ss, -1800, getdate()),108) as co_date														
														
yymmddhh:mi:ss														
select convert(varchar,getdate(),112)+convert(varchar,getdate(),8)														
go														
52、显示sql执行时间用: isql  -Unextip -Pnextip  -Sleon1 -p														
        进入即可，加小写-p参数														
53、 MS-SQL中创建数据库														
create database NEXTIPDB														
on primary														
(														
name = DEVDB,														
filename = 'd:\mssql_data\devdb.mdf',														
size = 30MB,														
maxsize = 100MB,														
filegrowth = 10MB														
),														
(														
name = DEVDB1,														
filename = 'd:\mssql_data\devdb1.ndf',														
size = 10MB,														
filegrowth = 10MB 														
)														
Log on														
(														
name = DEVDBLOG,														
filename = 'e:\mssql_log\devdblog.ldf',														
size = 10MB,														
maxsize = 100MB,														
filegrowth = 10MB														
)														
go														
54、 细小区别														
Sybase::														
IF NOT EXISTS (SELECT * FROM master..syslogins, master..sysdatabases														
WHERE master..syslogins.suid = master..sysdatabases.suid														
AND master..syslogins.name = 'nextip')														
EXEC sp_changedbowner nextip, NEXTIPDB														
GO														
														
Ms-sqlserver::														
IF NOT EXISTS (SELECT * FROM master..syslogins, master..sysdatabases														
WHERE master..syslogins.sid = master..sysdatabases.sid														
AND master..syslogins.name = 'nextip')														
EXEC sp_changedbowner nextip, NEXTIPDB														
GO														
														
														
														
														
一、 连接isql														
														
mo:/sybase/NIC # source SYBASE.sh														
mo:/sybase/NIC # export LANG=en_US														
mo:/sybase/NIC # isql -U sapsa -P xxx -H 10.1.1.1:4901  -S NIC -X														
														
														
二、 显示当前版本														
														
1> select @@version														
2> go														
														
或														
1> sp_version														
2> go														
														
														
														
三、显示所有数据库名														
														
1> sp_helpdb														
2> go														


## Sybase不公开存储过程
系统存储过程sp_MSforeachtable和sp_MSforeachdb,是微软提供的两个不公开的存储过程。从mssql6.5开始，存放在SQL Server的MASTER数据库中。可以用来对某个数据库的所有表或某个SQL服务器上的所有数据库进行管理，下面将对此进行详细介绍								
								
## sp_helpdb 列出当前数据库（实例）所管理的所有数据库的名称（如下）。 sp_helpdb MXG_FDR ,输出MXG_FDR数据库的基本信息								
	name	db_size	owner	dbid	created	status		
	MXGDM_FDR	356500MB	sa	6	Jul 14,2013	select into/bulkcopy/pllsort, trunc log on chkpt, abort tran on log full		
	MXG_FDR	321200MB	sa	4	Jul 14,2014	select into/bulkcopy/pllsort, trunc log on chkpt, abort tran on log full		
	MXG_VAR	321200MB	sa	5	Jul 14,2015	select into/bulkcopy/pllsort		
	master	50MB	sa	1	Jul 14,2016	mixed log and data		
	model	4MB	sa	3	Jul 14,2017	mixed log and data		
	sybsystemdb	54MB	sa	31513	Jul 14,2018	mixed log and data		
	sybsystemprocs	200MB	sa	31514	Jul 14,2019	trunc log on chkpt, mixed log and data		
	tempdb	32004MB	sa	2	Nov 20,2015	select into/bulkcopy/pllsort, trunc log on chkpt, mixed log and data		
								
## 客户端连接MXG_FDR数据库，后输入sp_help  命令,输出10036行的内容（如下），sp_help object_name,输出object_name的相关信息。								
	Name	Owner	Object_type					
	。。。							
	TRI_TABLE_LIST_COM_SWF__DBF	MUREXDB	trigger					
	UPDCHINESEWALL	MUREXDB	trigger					
	sysalternates	dbo	system table					
	sysattributes	dbo	system table					
	syscolumns	dbo	system table					
	syscomments	dbo	system table					
	sysconstraints	dbo	system table					
	sysdepends	dbo	system table					
	sysgams	dbo	system table					
	sysindexes	dbo	system table					
	sysjars	dbo	system table					
	syskeys	dbo	system table					
	syslogs	dbo	system table					
	sysobjects	dbo	system table					
	syspartitions	dbo	system table					
	sysprocedures	dbo	system table					
	sysprotects	dbo	system table					
	sysqueryplans	dbo	system table					
	sysreferences	dbo	system table					
	。。。							
								
## 已经连接了MXG_FDR(或use MXG_FDR)后,执行sp_stored_procedures，查看当前数据库里所有的存储过程和函数								
	procedure_qualifier	procedure_owner	procedure_name	num_input_params	num_output_params	num_result_sets	remarks	
	MXG_FDR	MUREXDB	CR02_INS_BSK_ALLOC;1	-1	-1	-1		
	MXG_FDR	MUREXDB	CR02_INS_SCN;1	-1	-1	-1		
	MXG_FDR	MUREXDB	CR02_INS_SNG_SCN;1	-1	-1	-1		
	MXG_FDR	MUREXDB	CRD_DFLT_TEMPLATE;1	-1	-1	-1		
	。。。							
								
## sp_helptext   CR02_INS_BSK_ALLOC，查看存储过程CR02_INS_BSK_ALLOC的源代码								
								
## 已连接上了MXG_FDR数据库（或use MXG_FDR go 切换到MXG_FDR数据库）后，执行sp_spaceused，输出当前数据库的空间使用情况。sp_spaceused TABLE#LIST#SWIFT_CT_DBF,输出TABLE#LIST#SWIFT_CT_DBF表的空间使用情况。								
	reserved	data	index_size	unused				
	277609240KB	232690512KB	39556104KB	5362624KB				
计算关系： reserved（已分配空间）=data+index_size+unused ， 剩余空间= Datebase Size（总空间，通过sp_helpdb得到）—已分配空间（resrved） = 321200M - 277609.240MB = 43590.76MB								
http://sqlserverplanet.com/dba/using-sp_spaceused								
								
##  sp_depends TABLE#DATA#DEALCOM_DBF，命令了解对象所关联的触发器是有好处的，该命令能列出触发器影响的所有对象、表和视图等。在定义几类数据库对象的时候，对存储过程、索引和触发器要给予特别的注意，尤其存储过程，它设计的好坏对数据库性能的影响很大。								
	object	type		Column	Type	Object Names or Column Names		
	MUREXDB.DID_TRAD_DBF	view		M_CONFO_ID	index	DEALCOM_ND3 (M_CONFO_ID)		
	MUREXDB.PAY_LIEN_MONIT_VW_DBF	view		M_CONFO_ID	statistics	(M_CONFO_ID)		
	MUREXDB.SCB_VW_ETL_DEALCOM_CLIVE_REP	view		M_EXT_REF_NB	index	DEALCOM_ND4 (M_EXT_REF_NB,M_SRC_SYSTEM)		
	MUREXDB.VIEW_CONFIRMATIONID_DBF	view		M_EXT_REF_NB	statistics	(M_EXT_REF_NB)		
	MUREXDB.VIEW_DID_DBF	view		M_EXT_REF_NB	statistics	(M_EXT_REF_NB,M_SRC_SYSTEM)		
	MUREXDB.VIEW_ETPDID_DBF	view		M_IDENTITY	index	DEALCOM_ND1 (M_IDENTITY)		
	MUREXDB.VIEW_ETP_CONFIRMATIONID_DBF	view		M_IDENTITY	statistics	(M_IDENTITY)		
	MUREXDB.VIEW_STP_CONFIRMATIONID_DBF	view		M_NB	index	DEALCOM_ND0 (M_NB)		
	MUREXDB.VIEW_STP_DID_DBF	view		M_NB	index	DEALCOM_ND2 (M_STRUCT_ID,M_NB)		
	MUREXDB.mxg_NackTradeAudSum_sp	stored procedure		M_NB	statistics	(M_NB)		
	MUREXDB.scb_mxg_dailyMktInfo	stored procedure		M_NB	statistics	(M_STRUCT_ID,M_NB)		
	MUREXDB.scb_mxg_dailyTradeInfo	stored procedure		M_ND02ID	index	DEALCOM_ND5 (M_ND02ID)		
	MUREXDB.sp_Process_SID_DID	stored procedure		M_ND02ID	statistics	(M_ND02ID)		
	MUREXDB.sp_S_GenereicSID_PRU	stored procedure		M_SRC_SYSTEM	index	DEALCOM_ND4 (M_EXT_REF_NB, M_SRC_SYSTEM)		
	MUREXDB.sp_S_GenereicSID_PRU_Main	stored procedure		M_SRC_SYSTEM	statistics	(M_EXT_REF_NB, M_SRC_SYSTEM)		
	MUREXDB.sp_S_GenericDID_ETP_PRU1	stored procedure		M_STRUCT_ID	index	DEALCOM_ND2 (M_STRUCT_ID, M_NB)		
	MUREXDB.sp_S_GenericDID_PRU	stored procedure		M_STRUCT_ID	statistics	(M_STRUCT_ID)		
	MUREXDB.sp_S_GenericDID_PRU1	stored procedure		M_STRUCT_ID	statistics	(M_STRUCT_ID, M_NB)		
	MUREXDB.sp_conservative_monit	stored procedure						
	MUREXDB.sp_ext_purge_deals	stored procedure						
	MUREXDB.sp_ext_purge_deals_com	stored procedure						
	MUREXDB.sp_ext_purge_deals_jan21_old	stored procedure						
	MUREXDB.sp_ext_purge_deals_typo	stored procedure						
	dbo.STRCTVUE_DBF	view						
								
## sp__dbfree MXG_FDR ， 								
	Database	Type	Size(MB)	Freespace(MB)	%Free			
	MXG_FDR	data only	313200	40873	13			
	MXG_FDR	log only	8000	7983	99			
								
## sp_helpdevice								
在ASE15.x之前的所有版本中（包括ASE12.5.4），存储过程sp_helpdevcie无法显示设备的剩余空间以及设备上各个数据库的具体分配情况。								
								
比如：在ASE12.5中，执行sp_helpdevice master的结果为：								
								
1> sp_helpdevice master								
2> go								
device_name  physical_name                   description                                                     status    cntrltype     device_number  low    high								
								
master       e:\sybase125\data\master.dat    special, dsync on, default disk, physical disk, 100.00 MB       3         0             0               0      51199								
								
(1 row affected)								
(return status = 0)								
								
								
上面输出结果中加粗标记出来的100.00MB表示master设备的总大小。至于master设备还剩余多少空间？master设备都分配给哪些数据库使用了？ASE15.x之前版本的存储过程sp_helpdevice不能给出答案。								
								
在ASE15.0之后版本的系统存储过程sp_helpdevice增加了上述功能。比如：ASE15.0.3中执行sp_helpdevice master的结果如下：								
1> sp_helpdevice master								
2> go								
 device_name  physical_name                 description                                                                                                                                                                                            status    cntrltype   vdevno   vpn_low    vpn_high								
------------- ---------------------------   ---------------------------------------------                                                                                                                                                          ------    --								
 master       D:\sybase\data\master.dat     file system device, special, MIRROR ENABLED, mirror = 'D:\sybase\data\master_mirr.dat', serial writes, dsync on, directio off, reads mirrored,default disk, physical disk, 80.00 MB, Free: 42.00 MB     739         0           0       0        40959								
								
(1 row affected)								
 dbname               size          allocated           vstart lstart								
 -----------      -------------     ------------------- ------ ------								
 master           26.00 MB Dec  2 2009  6:58PM      4      0								
 model             6.00 MB Dec  2 2009  6:58PM  13316      0								
 sybsystemdb       6.00 MB Dec  2 2009  6:58PM  19460      0								
								
(1 row affected)								
(return status = 0)								
								
								
那么，如何在ASE15.x之前的版本中查看设备的剩余空间以及设备上的数据库分配信息呢？								
								
方法有两种：								
								
1.使用sybase central这个sybase ASE自带的客户端工具。个人感觉ASE15以来自带的Sybase Central功能比较丰富，更加易用了。								
								
在ASE12.5自带的Sybase Central上查看各设备的剩余空间：								
								
								
								
在master设备上点右键，选择“属性”，切换到databases能够看到在master设备上各个数据库所分配的空间。								
								
								
								
2.自己编写一个存储过程，比如：sp_helpdevice2。								
								
我参考了ASE12.5.4和ASE15.0.3中的sp_helpdevice的语法完成该过程sp_helpdevice2的编写。分别在ASE v11.0.1, ASE v11.5.1， ASE v11.9.2, ASE v12.5, v12.5.0.3, v12.5.4 平台上进行了测试。								
								
/*								
* 此存储过程sp_helpdevice2适用于 ASE v11.x, v12.x,不能用于ASE15.x。实际上ASE15.x中的sp_helpdevice包含设备剩余空间以及设备上所分配的数据库的功能！								
* ASE v11.x版本中系统表 sysusages中没有crdate这个表示设备段分配时间的字段，考虑到支持ASEv11.x，为了简单处理，没有在Allocation information 中列出设备段的具体分配时间！								
*/								
								
/*								
* 此存储过程在ASE v11.0.1, ASE v11.5.1，ASE v11.9.2, ASE v12.5, v12.5.0.3, v12.5.4 平台测试通过！适用于 ASE v11.x, v12.x,不能用于ASE15。实际上ASE15.x中的sp_helpdevice完全能够实现该功能！								
* ASE v11.x版本中系统表 sysusages中没有crdate这个表示设备段分配时间的字段，考虑到支持ASEv11.x为了简单处理，没有在Allocation information 中列出设备段的具体分配时间！								
*/								
use sybsystemprocs								
go								
								
if exists(select 1 from dbo.sysobjects where type='P' and name='sp_helpdevice2')								
  drop procedure sp_helpdevice2								
go								
								
create procedure sp_helpdevice2								
@devname varchar(30) = "%"								
as								
								
declare @numpgsmb float								
declare @numpgsmb2 float								
declare @Major_Version int								
								
set nocount on 								
select @numpgsmb = (1048576. / @@pagesize)								
select @numpgsmb2 = (1048576. / @@maxpagesize)								
--select @version_as_num = @@version_as_integer								
select @Major_Version= convert(int, right(substring(@@version,1,charindex('.',@@version)-1),2) )								
								
if @Major_Version >= 15 or @Major_Version < 11								
begin 								
    print "this procedure is available for ASE versions from v11.x to v12.5.x, not for ASE15.x!"								
    return (1)								
end								
								
/*  See if the device exists.*/								
if not exists (select *								
            from master.dbo.sysdevices								
                where name like @devname)								
begin								
    /* 17610, "No such i/o device exists." */								
    raiserror 17610								
    return (1)								
end								
								
/* total size of device */								
select d.name,								
    totalsizeMB = (1. + (d.high - d.low)) / @numpgsmb 								
  into #totalsize								
    from master.dbo.sysdevices d								
        where d.status & 2 = 2								
            and name like @devname								
        group by d.name								
								
/* Calculate used size in MB */								
select d.name, 								
    usedsizeMB = isnull(sum(u.size) / @numpgsmb2,0)								
  into #usedsize                 								
    from master.dbo.sysdevices d, master.dbo.sysusages u 								
        where u.vstart >= d.low and u.vstart <= d.high                           								
            and d.status & 2 = 2								
            and d.name like @devname								
        group by d.name								
union 								
select d.name, 0.								
from master.dbo.sysdevices d 								
  where not exists ( select 1 from master.dbo.sysusages u where u.vstart >= d.low and u.vstart <= d.high )								
      and d.status & 2 = 2								
      and d.name like @devname								
								
set nocount off								
/* Calculate the free size of device */								
select d.name ,TotalSize = str(#totalsize.totalsizeMB,10,2), UsedSize = str(#usedsize.usedsizeMB,10,2),FreeSize = str(#totalsize.totalsizeMB - #usedsize.usedsizeMB,10,2),phyname = convert(varchar(50),d.phyname) 								
  from master.dbo.sysdevices d, #totalsize, #usedsize								
  where    d.name = #totalsize.name								
    and #totalsize.name = #usedsize.name								
  order by low,high								
								
if (select count(*) from master.dbo.sysdevices where name like @devname) = 1 								
begin 								
    print ""								
    print "========================== Allocate Information =========================="								
    /*if @Major_Version = 12								
        select dbname = db_name(dbid), "size(MB)"=str(size/@numpgsmb2,10,2), allocated = u.crdate, vstart, lstart 								
          from master.dbo.sysusages u, master.dbo.sysdevices d								
            where d.status & 2 = 2 								
              and d.name like @devname								
              and (u.vstart >= d.low and u.vstart <= d.high )								
           order by dbname,vstart								
     else if @Major_Version = 11 								
     */								
        select dbname = db_name(dbid), "size(MB)"=str(size/@numpgsmb2,10,2), vstart, lstart 								
          from master.dbo.sysusages u, master.dbo.sysdevices d								
            where d.status & 2 = 2 								
              and d.name like @devname								
              and (u.vstart >= d.low and u.vstart <= d.high )								
           order by dbname,vstart								
								
end								
								
drop table #totalsize								
drop table #usedsize								
go								
								
/* grant the execute privilege to public */								
grant execute on sp_helpdevice2 to public								
go								
								
 								
								
存储过程sp_helpdevice2的语法在此下载：sp_helpdevice2								
								
使用存储过程sp_helpdevice2的输出结果见下图所示：								
								
								
								
最后，存储过程sp_helpdevice2仅输出了一些基本信息，像设备的状态信息等还请结合自带的sp_helpdevice进行查看。								
								
在chinaunix的sybase板块有讨论此博文主题的帖子：怎么查看设备文件上还有多少空间没有被分配出去								
								
有什么建议欢迎在博文后面留言！								
								
								
## sp_helpdevice								
								
sp_rename [ @objname = ] 'object_name' , [ @newname = ] 'new_name' [ , [ @objtype = ] 'object_type' ]								
								
## sp_MxLock / sp_lock								
SQL Server数据库引擎为了保证每一次只有一个线程同时访问同一个资源的对象而采用的一种锁定机制，系统有大量锁时就产生了“数据阻塞”。因此你的数据库设计和程序编制应该科学和合理，以便让sql server涉及的锁定的数量降到最少。当你的系统的反应迟缓时就应该注意数据库是否产生了阻塞。sp_lock是SQL Server 2000 的一个系统存储过程，EXECUTE sp_lock 执行这个存储过程，可以查看当前阻塞的数据表，以便分析程序，解决问题。								
								
## sp_rename								
	sp_rename tb1,tb2    , then re-create tb1 and feed it. The other object(view, store procedure) still point to tb2. not sure, lisson to Journey's cource voice record in my phone
								
								
## sp_dboption  / sp_helpdb								
	查看有哪些option / 查看指定的option是否被打开							
								
	"sp_dboption [ [ @dbname = ] 'database' ] 
    [ , [ @optname = ] 'option_name' ] 
    [ , [ @optvalue = ] 'value' ] "	"参数
[@dbname =] 'database'

在其中设置指定选项的数据库的名称。database 的数据类型为 sysname，默认值为 NULL。"	"[@optname =] 'option_name'

要设置的选项的名称。没有必要输入完整的选项名称。Microsoft&reg; SQL Server&#8482; 可识别名称中任何独有的部分。如果选项名称包含空格或者关键字，请将选项名称用引号引起来。如果省略此参数，sp_dboption 将列出处于打开状态的选项。option_name 的数据类型为 varchar(35)，默认值为 NULL。 
"	"[@optvalue =] 'value'

option_name 的新设置。如果省略此参数，sp_dboption 将返回当前设置。value 可以是 true、false、on 或 off。value 的数据类型为 varchar(10)，默认值为 NULL。
"				
								
								
##  sp_helpuser								
	Users_name	ID_in_db	Group_name	Login_name				
	A1291887         	23	readonly					
	A1313066         	32	readonly					
	A1471985         	25	readonly	test				
	A1475135         	21	readonly					
	A1486530         	24	readonly					
	DMDAPRW          	20	dmreadwrite	DMDAPRW				
	DMDBO            	1	public	DMDBO				
	INSTAL           	39	readonly	INSTAL				
	MUREXDB          	38	readonly	MUREXDB				
	a1289528         	37	readonly					
	a1416208         	18	readonly					
	a1462140         	22	readonly					
	a1486017         	26	readonly					
	a1493277         	36	public					
	a1497060         	34	readonly					
	a1499748         	33	readonly					
	a1512571         	35	readonly					
	asindhu          	30	readonly					
	dbo              	5	public					
	dmro             	9	readonly					
	drajiv           	17	readonly					
	etluser          	7	readonly	etluser				
	ggonza           	14	readonly	ggonza				
	mstruser_ro      	11	readonly					
	paruna           	27	readonly	jobuser				
	patrolmonuk      	3	public					
	pfabri           	31	readonly					
	rgupta           	19	readonly	rgupta				
	schua            	28	readonly					
	tvan             	8	readonly	BO_EOD1				
	tvictor          	13	readonly					
	yadegbo          	29	readonly					
								
								
								
# sp_who								
								
								
sp_chgattribute 'DM_UDF_IRD_REP','identity_burn_max',0,'0'								
go


## Sybase系统表
syscomments	一行或者多行记录了每一视图、规则、缺省值、触发器和存储过程	
	## select distinct object_name(id) from syscomments where text like '%@str%' ，查看包含某个字符串@str的数据对象名称	
systypes	一行纪录了每一个由系统提供的和用户定义的数据类型	
sysusers	一行记录了一个数据库的合法用户	
sysconfigures	一行纪录了用户可以设置的配置参数	
syscurconfigs	有关SQL	Server当前正使用的配置参数情况
sysdatabases	一行纪录SQL	Server中的一个数据库
sysdevices	一行纪录数据库每一个磁带转储设备，盘转储设备，数据库设备和磁盘分区	
syslocks	有关动态锁的情况	
syslogins	一行纪录了每一个有效的SQL	Server的用户
sysmessages	一行记录了每一个系统错误或者警告	
sysprocesses	有关server进程的情况	
sysremotelogins	一行记录了一个远程用户	
sysservers	一行记录了一个远程server	
systypes	一行纪录了每一个由系统提供的和用户定义的数据类型	
sysusers	一行记录了一行记录了一个数据库的合法用户	
sysconfigures	一行纪录了用户可以设置的配置参数	
syscurconfigs	有关SQL	Server当前正使用的配置参数情况
sysdatabases	一行纪录SQL	Server中的一个数据库
sysdevices	一行纪录数据库每一个磁带转储设备，盘转储设备，数据库设备和磁盘分区	
syslocks	有关动态锁的情况	
syslogins	一行纪录了每一个有效的SQL	Server的用户
sysmessages	一行记录了每一个系统错误或者警告	
sysprocesses	有关server进程的情况	
sysremotelogins	一行记录了一个远程用户	
sysservers	一行记录了一个远程server	
sysusages	一行记录了分配给每个数据库的每个磁盘分片	
sysatterrates	一行记录了分配给SQL	Server用户在当前数据库的标识
syscolumns	一行记录了一个表或视图的每一列，一个存储过程的每一个参数	
sysdepends	一行记录了由一个过程、视图或者触发器所参照的每一个过程、视图或者表	
sysindexes	一行记录了每一个聚集或者非聚集索引，每一个不带索引的表，含有text或者image列的表	
syskeys	一行记录了每一个主玛、外玛或者公用玛	
syslogs	事务日志	
sysobjects	纪录表、视图、存储过程、规则、缺省值、触发器和临时表（在tempdb中）	
sysprocedures	纪录视图、规则、缺省值、触发器和过程	
sysprocts	纪录用户权限信息	
syssegments	纪录每一个片段（命名的磁盘）	

##  Sybase SQL 函数
http://blog.csdn.net/ljd_1986413/article/details/8794295
## ASCII 返回表达式中第一个字符的ASCII代码
select ASCII('Aennet') => 结果65
select ASCII('Bennet') => 结果66

## db_name返回指定数据库的名称,
select db_name() => 结果MXG_FDR
select db_name(4) => 返回ID为4的数据库名称，结果MXGDM_FDR

## next_identity 返回insert可用的下一个标识值即下一个自增的ID
select next_identity('TABLE#LIST#SWIFT_CT_DBF') =〉结果13446，注意如果这个表不是自增，则返回null

##  obeject_id(object_name)，参数object_name是数据库对象（表、视图、过程、触发器、缺省值或规则）的名称,函数返回指定对象的对象ID，对象ID存储在sysobjects的ID列中。
select object_id('TABLE#LIST#SWIFT_CT_DBF') => 结果 1945170719

##  obeject_name(object_id)，参数object_id是数据库对象（表、视图、过程、触发器、缺省值或规则）的对象ID,对象ID存储在sysobjects的ID列中，函数返回指定对象的对象名称
select object_name(1945170719) => 结果TABLE#LIST#SWIFT_CT_DBF

## host_id 返回当前Adaptive Server客户端机操作系统进程ID,
select host_id() => 结果6092

## host_name 返回当前Adaptive Server客户端机操作系统进程名称,在客户端可自定义该名称，一般用local计算机名称。
select host_name() => 结果CNNPC03BSQR

## reverse(@char_var) 反写@char_var中的文本

## substring(@char_var,start_index,length)

## datalength(@char_var) 返回@char_var字符串的长度值，忽略尾空 

## charindex('and',@sql_conditon_str) 返回'and'字符串在@sql_conditon_str 变量中的开始位置（即index，首字符的index为1），否则为0

## patindex('%pattern%',@sql_conditon_str),  查询模式字符串“%pattern%”首次出现的位置变量中的开始位置（即index，首字符的index为1），否则为0

## 小写转换函数 ，  lower(char_expr)


##   stuff(param1, startIndex, length, param2)
说明：将param1中自startIndex(SQL中都是从1开始，而非0)起，删除length个字符，然后用param2替换删掉的字符。*/


## Sybase备份恢复
Sybase数据库的dump和load	
1.     划分空间，为新建的数据库划分足够（大于等于需要备份的数据库）的空间，注意新数据库的版本要大于等于老数据库的版本。	
2.     检查老库接口文件，确认老库的interface文件的backupserver指向的地址和端口，通常会是本机的端口，但不排除使用远程的备份服务器。注意端口不要和已使用的端口重合。	
3.     在老库所在的机器上启动备份服务器，使用sowserver确认服务器已经启动。	
4.     DUMP	
use master	
sp_dboption 要备份的数据库,’single user’,ture  #将数据库置为单用户模式	
sp_flushstats  #等待页面刷新	
checkpoint	
use 要备份的数据库	
sp_flushstats	
checkpoint	
dump database pms to ‘data_dump/pms_dump.dat’   #备份数据库pms到指定的文件中	
5.     将DUMP完毕的文件传送至新库所在的主机中（参考步骤1）	
6.     启动新库的备份服务（参考步骤3）	
7.     LOAD	
load database pms from ‘/lxq/data/pppp’ 	
dbcc reindex(xxx)  	
sp_recompile  #存储过程编译和一致性检查	
8.     联机数据库，整个数据库load完毕后，还需要将数据库联机	
Online database pms 	
	
How to complete above steps in SCB	
1.     划分空间: load dump 之前，应先检查空间是否足够大（take mxg_var as example）。	
	[mx12b_sql]
	master=TCP,ukspddmrx02a.uk,8202
	query=TCP,ukspddmrx02a.uk,8202


##  Sybase 命令
## DUMP TRANSACTION [数据库名] WITH NO_LOG
这个命令的目的是把数据库的日志清空而不是停止记录。执行完后,日志会清空但是数据库依然会继续记录。
The transaction log in database MXG_IRDFX_DEV9 is almost full.  Your transaction is being suspended until space is made available in the log.



Below is coming from Yahui
/dev/vx/rdsk/ukspddmrx01a_hraw1 $ mx11a
1> use MXG_IRDFX_DEV9
2> go
1> dump tran MXG_IRDFX_DEV9 with no_log
2> go


1> sp__dbfree MXG_IRDFX_DEV9
2> go
 Database                       Type                         Size (MB)   Freespace (MB) % Free     
 ------------------------------ ---------------------------- ----------- -------------- -----------
 MXG_IRDFX_DEV9                 data only                         400000         111153          27
 MXG_IRDFX_DEV9                 log only                           20000          19921          99
done


##  SET NOCOUNT ON	
	在存储过程中，经常用到SET NOCOUNT ON；作用：阻止在结果集中返回显示受T-SQL语句或则usp影响的行计数信息(如： 549 行受影响)
	当SET ONCOUNT ON时候，不返回计数，当SET NOCOUNT OFF时候，返回计数；即使当SET NOCOUNT ON 时候，也更新@@RowCount；
	当SET NOCOUNT on时候，将不向客户端发送存储过程每个语句的DONE_IN_proc消息，如果存储过程中包含一些并不返回实际数据的语句，网络通信流量便会大量减少，可以显著提高应用程序性能；
	SET NOCOUNT 指定的设置时在执行或运行时候生效，分析时候不生效。

