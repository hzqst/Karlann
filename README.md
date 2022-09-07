# KARLANN
## It's a kernel-based keylogger for Windows x64. <a href="https://github.com/hkx3upper/Karlann/wiki">English</a>  
## Foreword：
**Karlann**是一个Windows内核键盘记录器，Poc驱动拦截Win32k发送到Kbdclass的IRP，获取键盘的Scancode，并通过UDP将Scancode转换成的字符发送到服务端。  
## Description：
#### 演示：  
获取notepad的键盘输入  
![Demo](https://user-images.githubusercontent.com/41336794/188859281-62d303b8-78bf-4973-861f-33ba7acbdf0e.gif)
#### 原理：
![Karlann](https://user-images.githubusercontent.com/41336794/188293026-0bfcdd72-3e2d-47fe-b604-5458d8a710f9.jpg)    
```
1.将Win32k驱动用于读键盘数据的hKeyboard->FileObject->DeviceObject替换为Poc驱动的DeviceObject，
由Poc驱动充当中间层驱动，过滤Win32k和Kbdclass的IRP。  
2.重点在于获取这个FileObject，这个FileObject由ZwReadFile填在Irp->IrpSp->FileObject中，
并且Kbdclass会在没有键盘数据时将IRP保存在它的DeviceExtension->ReadQueue链表中，
虽然Kbdclass的DeviceExtension结构体没有公开，但其中大部分结构的偏移自从Windows 8开始都是不变的，
所以可以找到ReadQueue链表，使用KeyboardClassDequeueRead函数取出IRP，也就取出了FileObject。  
3.为支持PNP，Poc驱动会在IoCancelIrp时将FileObject->DeviceObject还原，以便于之后设备卸载。
4.使用libwsk库(下方References)，把它的C++库做了一些调整，libwsk.h所有函数声明加前缀extern "C"后编译，
实现了通过UDP传输键盘数据的功能。
```
#### 未公开的结构体和函数（kbdclass.sys）：
```
DeviceExtension->RemoveLock（DeviceExtension + REMOVE_LOCK_OFFET_DE）
DeviceExtension->SpinLock（DeviceExtension + SPIN_LOCK_OFFSET_DE）  
DeviceExtension->ReadQueue（DeviceExtension + READ_QUEUE_OFFSET_DE）  
kbdclass!KeyboardClassDequeueRead（在驱动内实现）  
```
## Build & Installation：
1.建议在Windows 8 x64 6.2（9200） - Windows 10 x64 21H1（19043.1889）环境运行  
```
已测试系统版本:                  0903        0905        0906
Windows 8  x64 6.2(9200)        NOTESTED    PASS        PASS
Windows 8.1x64 6.3(9600)        PASS        NOTESTED    NT
Windows 10 x64 1511(10586.164)  PASS        PASS        PASS
Windows 10 x64 1607(14393.447)  PASS        PASS        PASS
Windows 10 x64 1703(15063.0)    PASS        PASS        NT
Windows 10 x64 1709(16299.15)   PASS        PASS        PASS
Windows 10 x64 1809(17763.2928) PASS        PASS        PASS
Windows 10 x64 21H1(19043.1889) PASS        PASS        PASS
```
2.修改global.h中的POC_IP_ADDRESS（SocketTest所在电脑的IP）和POC_UDP_PORT，使用Visual Studio 2019编译Release x64 Poc驱动  
```
不能编译Debug驱动，IO_REMOVE_LOCK在Debug和Release下的定义不同
```
3.使用OsrLoader加载驱动  
4.在局域网使用SocketTest监听global.h里的端口  
![SocketTest](https://user-images.githubusercontent.com/41336794/188532624-a1cb49bf-748e-4fe9-ae2a-f7c3f41f2996.JPG)
## License：
**Karlann**, and all its submodules and repos, unless a license is otherwise specified, are licensed under **GPLv3** LICENSE.  
Dependencies are licensed by their own.  
## Warning：
For educational purposes only, use at your own responsibility.  
And using this program might render your computer into an unstable state.  
## References：
https://github.com/Aekras1a/Labs/tree/master/Labs/WinDDK/7600.16385.1/src/input/kbdclass  
https://github.com/ZoloZiak/WinNT4/tree/master/private/ntos/dd/kbdclass  
https://github.com/ZoloZiak/WinNT4/tree/master/private/ntos/dd/i8042prt  
https://github.com/reactos/reactos/tree/master/drivers/hid/kbdhid  
https://github.com/ZoloZiak/WinNT4/tree/master/private/ntos/w32/ntuser/kernel  
https://github.com/HighSchoolSoftwareClub/Windows-Research-Kernel-WRK-  
https://download.microsoft.com/download/1/6/1/161ba512-40e2-4cc9-843a-923143f3456c/translate.pdf  
https://github.com/ParkHanbum/HypervisorKeylogger  
https://github.com/minglinchen/WinKernelDev/tree/master/Kb_sniff_Mp  
https://github.com/MiroKaku/libwsk  
https://github.com/akshath/SocketTest  
