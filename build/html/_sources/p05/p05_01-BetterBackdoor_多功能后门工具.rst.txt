.. contents::
   :depth: 3
..

5.1 BetterBackdoor 多功能后门工具
=================================

5.1.1 功能列表
--------------

::

   [cmd] Run Command Prompt commands                                        运行命令提示符命令
   [ps] Run a PowerShell script                                             运行一个PowerShell脚本
   [ds] Run a DuckyScript                                                   运行一个DuckyScript
   [exfiles] Exfiltarte files based on extension                            基于扩展名的Exfiltarte文件
   [expass] Exfiltrate Microsoft Edge and WiFi passwords                    获取微软Edge和WiFi密码
   [filesend] Send a file to victim's computer                              发送一个文件到受害者的电脑
   [filerec] Receive a file from victim's computer                          从受控的电脑收到一个文件
   [keylog] Start a KeyLogger on victim's computer                          在受控的电脑上启动一个键盘记录程序 
   [ss] Get a screenshot of vitim's computer                                获取受控电脑的的屏幕截图
   [cb] Get text currently copied to victim's clipboard                     获取当前复制到受控电脑剪贴板的文本
   [cat] Get contents of a file on victim's computer                        获取受控电脑上的文件内容
   [zip] Compress a directory to a ZIP file                                 将目录压缩为zip文件
   [unzip] Decompress a ZIP file                                            解压缩压缩文件
   [remove] Remove backdoor and all backdoor files from victim's computer   从受控的电脑中移除后门和所有后门文件
   [exit] Exit                                                              退出

5.1.2 BetterBackdoor 简介
-------------------------

BetterBackdoor创建的后门由一个客户端和一个服务器组成，并且双方通过套接字链接进行通信。
渗透测试的发起者需要启动服务器，而目标设备则需要以客户端的形式与该服务器建立连接。
成功建立连接后，渗透测试员可以将控制命令从服务器发送到目标设备，以管理和控制后门程序。

5.1.3 BetterBackdoor操作机制
----------------------------

首先，BetterBackdoor创建一个“
run.jar”文件（后门jar文件），然后将其复制到“ backdoor”目录中。
接下来，将包含服务器IP地址的文本文件添加到“
run.jar”文件中，其中IP地址以明文形式写入。

如果需要，还可以将Java运行时环境复制到“backdoor”目录，并创建批处理文件“
run.bat”以在打包的Java运行时环境中运行后门。

BetterBackdoor支持在单个网络，局域网或Internet（广域网）下工作。
如果要在WAN上使用BetterBackdoor，则必须进行端口转发。

要使用WAN，必须在服务器主机上启用TCP，并使用端口1025和1026进行端口转发。
完成此操作后，即使目标设备和渗透发起程序位于不同的网络上，渗透测试人员也可以控制后门。

要在目标设备上启动后门，将“backdoor”目录中的所有文件传输到目标设备。
如果JRE环境打包在后门文件中，则直接运行run.bat，否则运行run.jar文件。
运行完成后，后门将在目标设备上启动。

5.1.4 工具使用条件
------------------

::

   1.Java JDK> = 8
   2.生成后门和控制后门的设备必须相同，并且IP地址必须是静态的。
   3.控制后门的设备必须关闭本地防火墙。 如果在类似Unix的操作系统下运行，则需要使用“ sudo”权限来运行BetterBackdoor。

兼容性

::

   BetterBackdoor支持在Windows，macOS和Linux平台上运行，但是生成的后门程序当前仅支持在Windows平台上运行。

5.1.5 工具下载与安装
--------------------

使用以下命令在本地克隆项目源代码：

::

   git clone https://github.com/ThatcherDev/BetterBackdoor.git

转到项目所在的工作目录：

::

   cd BetterBackdoor

使用Maven构建BetterBackdoor。 对于Windows，运行以下命令：

::

   mvnw.cmd clean package

对于Linux和macOS环境，运行以下命令以完成BetterBackdoor：

::

   sh mvnw clean package

工具使用

::

   java -jar betterbackdoor.jar

|image0|

::

   Select:
   [0] Create backdoor   创建一个后门程序
   [1] Open backdoor shell    #打开后门shell

   选择 0 创建一个后门程序
   选择 0 创建一个LAN（内网环境）下的后门程序
   两次回车后 开始创建后门程序

|image1|

::

   输入 help 可以查看支持的功能
   输入 ss 获取受控主机的屏幕截图

   输入remove 移除受控主机上的所有后门文件，并退出程序

`文章参考自：
https://www.freebuf.com/sectool/225119.html <https://www.freebuf.com/sectool/225119.html>`__

.. |image0| image:: https://s1.ax1x.com/2020/06/12/tONmNT.png
.. |image1| image:: https://s1.ax1x.com/2020/06/12/tONwgH.png
