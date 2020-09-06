.. figure:: https://i.loli.net/2019/07/20/5d32d776e715059620.jpg
   :alt: nmap.jpg

nmap支持很多扫描技术，例如：UDP、TCP connect()、TCP
SYN(半开扫描)、ftp代理(bounce攻击)、反向标志、ICMP、FIN、ACK扫描、圣诞树(Xmas
Tree)、null扫描。

nmap还提供了一些高级的特征，例如：通过TCP/IP协议栈特征探测操作系统类型，秘密扫描，动态延时和重传计算，并行扫描，通过并行ping扫描探测关闭的主机，诱饵扫描，避开端口过滤检测，直接RPC扫描(无须端口影射)，碎片扫描，以及灵活的目标和端口设定.nmap总会给出well
known端口的服务名(如果可能)、端口号、状态和协议等信息。

每个端口的状态有：open、filtered、
unfiltered。open状态意味着目标主机能够在这个端口使用accept()系统调用接受连接。

filtered状态表示：防火墙、包过滤和其
它的网络安全软件掩盖了这个端口，禁止 nmap探测其是否打开。

unfiltered表示：这个端口关闭，并且没有防火墙/包过滤软件来隔离nmap的探测企图。通常情况下，端口的状态基本都
是unfiltered状态，只有在大多数被扫描的端口处于filtered状态下，才会显示处于unfiltered状态的端口。

根据使用的功能选
项，nmap也可以报告远程主机的下列特征：使用的操作系统、TCP序列、运行绑定到每个端口上的应用程序的用户名、DNS名、主机地址是否是欺骗地址、
以及其它一些东西。

可以使用nmap -h快速列出功能选项的列表。

::

   [root@node1 ~]# nmap -h
   Nmap 6.40 ( http://nmap.org )
   Usage: nmap [Scan Type(s)] [Options] {target specification}
   TARGET SPECIFICATION:
   Can pass hostnames, IP addresses, networks, etc.
   Ex: scanme.nmap.org, microsoft.com/24, 192.168.0.1; 10.0.0-255.1-254
   -iL <inputfilename>: Input from list of hosts/networks
   -iR <num hosts>: Choose random targets
   --exclude <host1[,host2][,host3],...>: Exclude hosts/networks
   --excludefile <exclude_file>: Exclude list from file
   HOST DISCOVERY:
   -sL: List Scan - simply list targets to scan
   -sn: Ping Scan - disable port scan
   -Pn: Treat all hosts as online -- skip host discovery
   -PS/PA/PU/PY[portlist]: TCP SYN/ACK, UDP or SCTP discovery to given ports
   -PE/PP/PM: ICMP echo, timestamp, and netmask request discovery probes
   -PO[protocol list]: IP Protocol Ping
   -n/-R: Never do DNS resolution/Always resolve [default: sometimes]
   --dns-servers <serv1[,serv2],...>: Specify custom DNS servers
   --system-dns: Use OS's DNS resolver
   --traceroute: Trace hop path to each host
   SCAN TECHNIQUES:
   -sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
   -sU: UDP Scan
   -sN/sF/sX: TCP Null, FIN, and Xmas scans
   --scanflags <flags>: Customize TCP scan flags
   -sI <zombie host[:probeport]>: Idle scan
   -sY/sZ: SCTP INIT/COOKIE-ECHO scans
   -sO: IP protocol scan
   -b <FTP relay host>: FTP bounce scan
   PORT SPECIFICATION AND SCAN ORDER:
   -p <port ranges>: Only scan specified ports
       Ex: -p22; -p1-65535; -p U:53,111,137,T:21-25,80,139,8080,S:9
   -F: Fast mode - Scan fewer ports than the default scan
   -r: Scan ports consecutively - don't randomize
   --top-ports <number>: Scan <number> most common ports
   --port-ratio <ratio>: Scan ports more common than <ratio>
   SERVICE/VERSION DETECTION:
   -sV: Probe open ports to determine service/version info
   --version-intensity <level>: Set from 0 (light) to 9 (try all probes)
   --version-light: Limit to most likely probes (intensity 2)
   --version-all: Try every single probe (intensity 9)
   --version-trace: Show detailed version scan activity (for debugging)
   SCRIPT SCAN:
   -sC: equivalent to --script=default
   --script=<Lua scripts>: <Lua scripts> is a comma separated list of 
           directories, script-files or script-categories
   --script-args=<n1=v1,[n2=v2,...]>: provide arguments to scripts
   --script-args-file=filename: provide NSE script args in a file
   --script-trace: Show all data sent and received
   --script-updatedb: Update the script database.
   --script-help=<Lua scripts>: Show help about scripts.
           <Lua scripts> is a comma separted list of script-files or
           script-categories.
   OS DETECTION:
   -O: Enable OS detection
   --osscan-limit: Limit OS detection to promising targets
   --osscan-guess: Guess OS more aggressively
   TIMING AND PERFORMANCE:
   Options which take <time> are in seconds, or append 'ms' (milliseconds),
   's' (seconds), 'm' (minutes), or 'h' (hours) to the value (e.g. 30m).
   -T<0-5>: Set timing template (higher is faster)
   --min-hostgroup/max-hostgroup <size>: Parallel host scan group sizes
   --min-parallelism/max-parallelism <numprobes>: Probe parallelization
   --min-rtt-timeout/max-rtt-timeout/initial-rtt-timeout <time>: Specifies
       probe round trip time.
   --max-retries <tries>: Caps number of port scan probe retransmissions.
   --host-timeout <time>: Give up on target after this long
   --scan-delay/--max-scan-delay <time>: Adjust delay between probes
   --min-rate <number>: Send packets no slower than <number> per second
   --max-rate <number>: Send packets no faster than <number> per second
   FIREWALL/IDS EVASION AND SPOOFING:
   -f; --mtu <val>: fragment packets (optionally w/given MTU)
   -D <decoy1,decoy2[,ME],...>: Cloak a scan with decoys
   -S <IP_Address>: Spoof source address
   -e <iface>: Use specified interface
   -g/--source-port <portnum>: Use given port number
   --data-length <num>: Append random data to sent packets
   --ip-options <options>: Send packets with specified ip options
   --ttl <val>: Set IP time-to-live field
   --spoof-mac <mac address/prefix/vendor name>: Spoof your MAC address
   --badsum: Send packets with a bogus TCP/UDP/SCTP checksum
   OUTPUT:
   -oN/-oX/-oS/-oG <file>: Output scan in normal, XML, s|<rIpt kIddi3,
       and Grepable format, respectively, to the given filename.
   -oA <basename>: Output in the three major formats at once
   -v: Increase verbosity level (use -vv or more for greater effect)
   -d: Increase debugging level (use -dd or more for greater effect)
   --reason: Display the reason a port is in a particular state
   --open: Only show open (or possibly open) ports
   --packet-trace: Show all packets sent and received
   --iflist: Print host interfaces and routes (for debugging)
   --log-errors: Log errors/warnings to the normal-format output file
   --append-output: Append to rather than clobber specified output files
   --resume <filename>: Resume an aborted scan
   --stylesheet <path/URL>: XSL stylesheet to transform XML output to HTML
   --webxml: Reference stylesheet from Nmap.Org for more portable XML
   --no-stylesheet: Prevent associating of XSL stylesheet w/XML output
   MISC:
   -6: Enable IPv6 scanning
   -A: Enable OS detection, version detection, script scanning, and traceroute
   --datadir <dirname>: Specify custom Nmap data file location
   --send-eth/--send-ip: Send using raw ethernet frames or IP packets
   --privileged: Assume that the user is fully privileged
   --unprivileged: Assume the user lacks raw socket privileges
   -V: Print version number
   -h: Print this help summary page.
   EXAMPLES:
   nmap -v -A scanme.nmap.org
   nmap -v -sn 192.168.0.0/16 10.0.0.0/8
   nmap -v -iR 10000 -Pn -p 80
   SEE THE MAN PAGE (http://nmap.org/book/man.html) FOR MORE OPTIONS AND EXAMPLES
   [root@node1 ~]# 

`参考nmap中文网 <http://www.nmap.com.cn/doc/manual.shtm>`__

1.1 nmap使用
^^^^^^^^^^^^

1.1.1 Centos 系统安装nmap
'''''''''''''''''''''''''

::

   yum -y install nmap

1.1.2 查看安装版本
''''''''''''''''''

::

   [root@node1 ~]# nmap -version

   Nmap version 6.40 ( http://nmap.org )
   Platform: x86_64-redhat-linux-gnu
   Compiled with: nmap-liblua-5.2.2 openssl-1.0.2k libpcre-8.32 libpcap-1.5.3 nmap-libdnet-1.12 ipv6
   Compiled without:
   Available nsock engines: epoll poll select
   [root@node1 ~]# 

1.1.3 nmap常用命令
''''''''''''''''''

::

   nmap 172.31.8.8                             ---扫描单个主机
   nmap 172.31.8.0/24                          ---扫描整个子网
   nmap 172.31.8.8 172.31.8.13                 ---扫描多个目标
   nmap 172.31.8.8-20                          ---扫描一个指定范围内的目标
   nmap -iL pinglist.txt                       ---如果有一个IP地址列表，将这些IP保存在一个pinglist.txt文件内，使用nmap -iL iplist.txt 扫描指定文件内的目标 list
   例：
   [root@node1 ~]# cat pinglist.txt 
   172.31.8.2
   172.31.8.3
   172.31.8.75
   172.31.8.107
   [root@node1 ~]# nmap -iL pinglist.txt 

   Starting Nmap 6.40 ( http://nmap.org ) at 2019-07-20 17:13 CST
   Nmap scan report for 172.31.8.3
   Host is up (0.0034s latency).
   Not shown: 995 filtered ports
   PORT     STATE SERVICE
   135/tcp  open  msrpc
   139/tcp  open  netbios-ssn
   445/tcp  open  microsoft-ds
   3389/tcp open  ms-wbt-server
   5357/tcp open  wsdapi
   MAC Address: 50:7B:9D:99:AA:D9 (Unknown)

   Nmap scan report for node4 (172.31.8.75)
   Host is up (0.000022s latency).
   Not shown: 999 closed ports
   PORT   STATE SERVICE
   22/tcp open  ssh
   MAC Address: 00:0C:29:E3:5F:D2 (VMware)

   Nmap scan report for node3 (172.31.8.107)
   Host is up (0.000066s latency).
   Not shown: 999 closed ports
   PORT   STATE SERVICE
   22/tcp open  ssh
   MAC Address: 00:0C:29:9A:43:BC (VMware)

   Nmap done: 4 IP addresses (3 hosts up) scanned in 5.15 seconds
   [root@node1 ~]# 

   nmap 172.31.8.0/24 -exclude 172.31.8.8      ---扫描172.31.8.0/24 这个子网除172.31.8.8以外的所有IP
   nmap 172.31.8/0/24 -exclude iplist.txt      ---扫描172.31.8.0/24 这个子网所有的IP，排除iplist.txt这个文件内列出的IP
   nmap -p8080,22 172.31.8.8                   ---扫描特定主机上的8080，22端口

1.1.4 深入探究
''''''''''''''

::

   nmap -sS 172.31.8.8                         ---SYN扫描，指定IP/IP范围指定扫描端口:nmap -sS 172.31.8.8 -p 8080
   nmap -sP 172.31.8.0/24                      ---扫描存活主机,可以添加 | grep up 参数过滤存活主机:nmap -sP 172.31.8.0/24 | grep up
   nmap -sV 172.31.8.8 -p 8080                 ---扫描主机的8080端口的服务和服务版本
   nmap -O 172.31.8.8                          ---扫描目标主机的系统版本
   nmap -A 172.31.8.8                          ---扫描目标主机，-A参数包括:-sV、-O 系统全面检测、启动脚本检测、扫描等操作
   nmap -PO 172.31.8.8                         ---扫描之前不使用ping操作，适用于禁ping的系统和设备
   nmap -v 172.31.8.8                          ---显示目标主机上详细信息
   例：
   [root@node1 ~]# nmap -v 172.31.8.8 

   Starting Nmap 6.40 ( http://nmap.org ) at 2019-07-20 17:15 CST
   Initiating SYN Stealth Scan at 17:15
   Scanning node1 (172.31.8.8) [1000 ports]
   Discovered open port 8080/tcp on 172.31.8.8
   adjust_timeouts2: packet supposedly had rtt of 1428142253061189 microseconds.  Ignoring time.
   adjust_timeouts2: packet supposedly had rtt of 1428142253061189 microseconds.  Ignoring time.
   Discovered open port 22/tcp on 172.31.8.8
   adjust_timeouts2: packet supposedly had rtt of 1428142253061166 microseconds.  Ignoring time.
   adjust_timeouts2: packet supposedly had rtt of 1428142253061166 microseconds.  Ignoring time.
   Discovered open port 8009/tcp on 172.31.8.8
   adjust_timeouts2: packet supposedly had rtt of 1428142274531622 microseconds.  Ignoring time.
   adjust_timeouts2: packet supposedly had rtt of 1428142274531622 microseconds.  Ignoring time.
   Completed SYN Stealth Scan at 17:15, 0.01s elapsed (1000 total ports)
   Nmap scan report for node1 (172.31.8.8)
   Host is up (0.0000010s latency).
   Not shown: 997 closed ports
   PORT     STATE SERVICE
   22/tcp   open  ssh
   8009/tcp open  ajp13
   8080/tcp open  http-proxy

   Read data files from: /usr/bin/../share/nmap
   Nmap done: 1 IP address (1 host up) scanned in 0.03 seconds
           Raw packets sent: 1000 (44.000KB) | Rcvd: 2003 (84.132KB)
   [root@node1 ~]# 

   nmap -T4 -sP 172.31.8.0/24 && egrep "00:00:00:00:00:00" /proc/net/arp | grep Unknown     ---扫描子网上未使用的IP
   nmap -PN -T4 -p139,445 -n -v --script=smb-check-vulns --script-args safe=1 172.31.8.0/24  ---在子网中探测Conficker 蠕虫病毒
   nmap -F -O 172.31.8.0/24 | grep "Running: " > /tmp/os; echo "$(cat /tmp/os | grep Linux | wc -l) Linux device(s)"; echo "$(cat /tmp/os | grep Windows | wc -l) Window(s) device"
   ---扫描子网中存在的Linux、windows设备数量

|主机数量探测.png|
nmap是一款非常强大的扫描工具，以上的命令基本都是比较常用的命令，如果想深入了解nmap的功能，可以自己去研究一下。

.. |主机数量探测.png| image:: https://i.loli.net/2019/07/20/5d32d5a15ff3269908.png
