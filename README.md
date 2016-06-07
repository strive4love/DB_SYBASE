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
