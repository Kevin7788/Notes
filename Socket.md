	MAC		网卡地址，全球唯一性，在路由器中做IP地址映射
	IP		公网地址，标识一个连接公网的路由器
	TCP/UDP	封装端口

通过IP找到路由器，通过数据包MAC信息与路由器进行映射找到具体的设备，通过端口与应用通信(通过IP+端口定位一个网络进程)

ARP数据格式(局域网通信查找MAC地址)，192.168.1.100发送一个广播包，与这个路由器相连的所有设备都会收到，对应目标设备的IP会回复自己的MAC地址，反之丢弃。

	FFFFFFFF | . . . | 自己的MAC | 自己的IP | . . .  | 目标IP

三次握手(SYN 请求建立连接，ACK 应答标志)

	client							server
	SYN 1000(0) <mss 1460>  ------>
							<------ SYN 8000(0) ACK 1001 <mss 1024>
	ACK 8001				------>

##### 穿透 

两台各自独立的局域网设备通信,借助公网服务器中转

##### Socket 

应用抽象层，抽象成文件通过read/write接口来进行数据传输
	
	AF_INET		ipv4套接字
	AF_UNIX		本地套执接字
	
