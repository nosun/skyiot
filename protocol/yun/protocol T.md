	
##协议概述

#####名词&缩写约定

+ sn: (Serial Number)，协议序号，用于匹配请求-应答对
+ vid: (Vender ID)，为厂商ID，长度5，用于标示指定厂家
+ pid: (Product ID)，为产品ID，长度5，用于标示指定产品
+ pv: (Protocol Version)， 当前协议版本
+ pkey: (Product Key)，长度5，与当前产品相对的密钥
+ akey: (AES Key)，长度16，通信数据加密所用密钥，默认采用AES CBC加密方式，IV==Key，设备激活/登录时使用默认密钥0123456789ABCDEF，数据长度不足时Padding为不足长度个不足长度值（不清楚可自行谷歌）
+ [xxx]: 被'['，']'包围的JSON项为传输中非必需包含项

#####协议要点

+ 长连接：

	为了保证Server反向控制Dev的实时性和高效率，Dev与Server之间的通信采用TCP长连接方式，而不建议采用HTTP长轮询的方式。而长连接的维持需要心跳机制来保证。
	
	Dev端作为TCP Client，主动向Server发起三次握手。断开时Dev端主动断开。（正常情况下一直保持连接，除非遇到异常等特殊情况才主断开）	

	可参考Android端推送服务的实现：
	
	<http://www.cnblogs.com/manuosex/p/3660727.html> 
	<http://www.cnblogs.com/hanyonglu/archive/2012/03/04/2378971.html>
	

+ 心跳：

	设备主动向Server发送的用来维持TCP长连接的短包。一般维持在30s左右，防止传输过程中转发路由的NAT端口映射老化的问题。Server则根据此心跳包来判断Dev的在线情况。

+ 编码格式：

	本协议为字符协议，编码方式采用ASCII（UTF-8可兼容）。

	协议文本采用JSON格式封装，具体内容见下

+ 异步上报：

	Server -> Dev 控制命令的发送是通过TCP长连接推送过去，而Dev -> Server的状态回馈是通过TCP长连接主动上报。Dev接到命令后立即给server反馈 `ret:200`，告知已收到并执行。然后Dev等待MCU执行完毕后将操作后的状态异步反馈给server

##协议内容

**1. 设备激活/登录**

**Request (Dev to Server):**

	{
	    "sn": 1020,
	    "cmd": "login",
	    "mac": "AABBCCDDEEFF",
	    "pv": "1.0",
	    "hfver": "<hardware version>|<software version>",
	    "data": [
	        "vid::12345"
	        "pid::12345",
	        "pkey::12345",
	        "usrver::<string>"
	    ]
	}

>`hfver` 为WiFi模块版本号，其值为固定格式"硬件版本|软件版本"
>
>`usrver` 为用户主控板的版本号，其内容为一串字符串，由用户自己定义，通过串口设置给WiFi模块
>>此处WiFi模块设备激活前，需先向用户主控(MCU)发送激活命令（命令格式见串口协议文档），若10s内未收到主控发来的响应包（包含主控的vid, pid, pkey等相关信息），则以无"data"的JSON包激活，并立刻上传错误事件到服务器。此处若上传错误事件后，WiFi模块收到用户主控发来的响应包，则重新发送设备激活命令。若收到主控的响应包，则将响应包中的数据填充入"data"

**Response (Server to Dev):**

	{
	    "sn": 1020,
	    "cmd": "login",
	    "ret": 200,
	    "akey": "0123456789ABCDEF",
	    "heart": 30,
	    ["serverip"]: "xxx.xxx.xxx.xxx:xxxxx"
	}

>`ret` 此处可取值为(200, 403, 501)

>`heart` 设置默认心跳包间隔时间，单位为秒(s)

>`serverip` 为重定向模块连接到指定服务器

**2. 发送心跳**

**Request (Dev to Server):**

	{
	    "sn": 1021,
	    "cmd": "heartbeat",
	    "mac": "AABBCCDDEEFF"
	}

**Response (Server to Dev):**

	{
	    "sn": 1021,
	    "cmd": "heartbeat",
	    ["heart"]: 30
	}

`heart` 为心跳包间隔时间，即服务器可以动态控制设备的心跳间隔，从而便于合理分配服务器资源，若响应包无此项数据则WiFI模块使用上次的历史间隔时间

**3. WiFi固件升级**

**Request (Server to Dev):**

	{
	    "sn": 1022,
	    "cmd": "updatewifi",
	    "url": "http://www.xxx.com/path/to/update.bin"
	}

**Response (Dev to Server):**

	{
	    "sn": 1022,
	    "cmd": "updatewifi",
	    "ret": 200
	}
 
`ret` 为当前协议的应答返回值，若为200则表明设备已接收到此协议包，但不表明已执行成功，执行结果将通过上传数据，设备登录等命令体现，如此处结果将体现在模块升级重启后，设备登录中的hfver软件版本

`ret` 所对应的意思将在后面章节中列表说明

**4. 设备控制**

**Request (Server to Dev):**

	{
	    "sn": 1023,
	    "cmd": "download",
	    "data": [
	        "switch::on",
	        "windspeed::high"
	    ]
	}

**Response (Dev to Server):**

	{
	    "sn": 1023,
	    "cmd": "download",
	    "ret": 200
	}

PS:此处WiFi模块中限制"data"列表项目数上限30个

**5. 上传状态**

**Request (Dev to Server):**

	{
	    "sn": 1024,
	    "cmd": "upload",
	    "mac": "AABBCCDDEEFF",
	    "data": [
	        "switch::on",
	        "windspeed::high"
	    ]
	}

**Response (Server to Dev):**

	{
	    "sn": 1024,
	    "cmd": "upload",
	    "ret": 200
	}

**6. 错误事件**

**Request (Dev to Server):**

	{
	    "sn": 1025,
		"mac": "AABBCCDDEEFF",
	    "cmd": "error",
	    "ret": 404
	}

**Response (Server to Dev):**

	{
	    "sn": 1025,
	    "cmd": "error",
	    "ret": 200
	}

## 超时重传机制
发送 `Request` 协议包后，等待制定时间，若没有收到 `Response` 协议包则进行重传，其中 `sn` 号每次发送时 `+1` ，上限为65535，溢出后归零重新计数，相应的应答包中，`sn` 保持与请求包相同，用来识别请求应答对

重传时 `sn` 号不变

## ret返回数字解释

<table style="font-size:14px">
	<tr><th>ret值</td>
		<th>解释</td></tr>
	<tr><td>200</td>
		<td>成功收到协议包</td></tr>
	<tr><td>403</td>
		<td>设备未授权</td></tr>
	<tr><td>404</td>
		<td>未接收到主控的串口数据</td></tr>
	<tr><td>501</td>
		<td width=200>不支持此协议</td></tr>
	<tr><td>503</td>
		<td>WiFi模块负载重，请设备稍后再发</td></tr>
</table>