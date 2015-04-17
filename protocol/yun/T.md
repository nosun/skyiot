## Protocol T: Wifi & Server

    	Make By: Moore Huang, nosun,wyf
    	version: v1.2
    	
#####名词&缩写约定

+ sn: (Serial Number)，协议序号，用于匹配请求-应答对  
+ pid: (Product ID)，为产品ID，长度5，用于标示指定产品
+ pv: (Protocol Version)， 当前协议版本号 
+ sv: (firmware version) wifi固件版本号
+ pkey: (Product Key)，长度5，与当前产品相对的密钥
+ [xxx]: 被'[',']'包围的JSON项为传输中非必需包含项

#####协议要点

+ 长连接：

	为了保证Server反向控制Dev的实时性和高效率，Dev与Server之间的通信采用TCP长连接方式，而长连接的维持需要心跳机制来保证。
	
	Dev端作为TCP Client，主动向Server发起三次握手。断开时Dev端主动断开。（正常情况下一直保持连接，除非遇到异常等特殊情况才主断开）	

+ 心跳：

	设备主动向Server发送的用来维持TCP长连接的短包。一般维持在60s左右，防止传输过程中转发路由的NAT端口映射老化的问题。Server则根据此心跳包来判断Dev的在线情况。

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
	    "sv": "<hard version | software version>",
	    "data": [
	        "pid::12345",
	        "pkey::12345",
	        "usrver::<string>"
	    ]
	}

>`sv` 为WiFi模块版本号，其值为固定格式"硬件版本|软件版本"
>
>`usrver` 为用户主控板的版本号，其内容为一串字符串，由用户自己定义，通过串口设置给WiFi模块
>>此处WiFi模块设备激活前，需先向用户主控(MCU)发送激活命令（命令格式见串口协议文档），若3s内未收到主控发来的响应包（包含主控的vid, pid, pkey等相关信息），则以无"data"的JSON包激活，并立刻上传错误事件到服务器。此处若上传错误事件后，WiFi模块收到用户主控发来的响应包，则重新发送设备激活命令。若收到主控的响应包，则将响应包中的数据填充入"data"

**Response (Server to Dev):**

	{
	    "sn": 1020,
	    "cmd": "login",
	    "ret": 200,
	    "akey": "0123456789ABCDEF",
	    "heart": 30,
	    ["serverip"]: "ipaddr:port",
	}

>`ret` 此处可取值为(200, 403, 301, 302),其中200为正常相应，403为未授权阻止接入，301为永久重定向，302为临时重定向，当取值为301,302时，Server会同时下发 serverip，当wifi收到 serverip时会将向服务器发起连接的socket 地址改写，301为永久改写，302为临时改写，改写完毕之后，socket主动关闭连接并向收到的server地址发起新的连接。

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
 
`ret` 为当前协议的应答返回值，可选值为（200,201,601）,若为200则表明设备已接收到此协议包，但不表明已执行成功。wifi模块升级成功之后，返回201，表示更新固件成功，400表示更新固件失败。每次固件升级需要升级版本号，并提供相应的升级说明。

固件升级，可以通过小循环（局域网）下发，也可以通过server下发，谁下发的指令返回给谁200的结果，升级成功之后大循环，小循环都上报升级结果。

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

Server 发送指令给wifi，wifi收到指令后回复Server收到指令，并不代表指令执行成功。
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

Dev 上报自身状态，通过wifi模块上报给Server，Server收到后回复200,表示收到，如果没有发送成功，wifi继续发送。

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

Dev在登录过程中首先要请求mcu数据，如果没有获取到mcu数据，则上报给服务器错误，此处还可以定义其他错误。

## 超时重传机制
发送 `Request` 协议包后，等待制定时间，若没有收到 `Response` 协议包则进行重传，其中 `sn` 号每次发送时 `+1` ，上限为65535，溢出后归零重新计数，相应的应答包中，`sn` 保持与请求包相同，用来识别请求应答对

重传时 `sn` 号不变

## ret返回数字解释

<table style="font-size:14px">
	<tr><th>ret值</td>
		<th>解释</td></tr>
	<tr><td>200</td>
		<td>成功收到协议包</td></tr>
	<tr><td>301</td>
		<td>永久重定向</td></tr>
	<tr><td>302</td>
		<td>临时重定向</td></tr>
	<tr><td>403</td>
		<td>设备未授权</td></tr>
	<tr><td>404</td>
		<td>未接收到主控的串口数据</td></tr>
	<tr><td>501</td>
		<td width=200>不支持此协议</td></tr>
	<tr><td>503</td>
		<td>WiFi模块负载重，请设备稍后再发</td></tr>
	<tr><td>600</td>
		<td width=200>wifi升级成功</td></tr>
	<tr><td>602</td>
		<td>wifi升级失败</td></tr>
</table>