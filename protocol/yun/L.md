## Protocol L: Wifi & App

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

L口作为T口的辅助，主要为了提高T口的用户体验，一般情况下不作为独立使用。

L口协议采用TCP短连接与长连接混合的通信方式，设备控制、上传状态、协议格式与T口相同，不做赘述，此处WiFi模块作为TCP Server，手机App作为TCP Client。

常态情况使用TCP长连接，即App与WiFi模块建立TCP连接，发送请求包，收到应答包，及相应的回报协议后，断开连接。

若App希望实时监控设备状态，则建立TCP连接后，只需按照指定频率发送心跳包维持连接不被WiFi模块收回，即可实时收到设备的上传状态协议包，退出实时监控状态时断开TCP连接。

用户主控的所有数据都将同时发送到服务器和本地，手机App可以使用最先收到的数据包(两条通道的sn号相同，便于判断不同通道的相同数据包，避免误决策)

APP主动发起的请求（登录，查询，命令，升级），APP 自己维持sn，wifi收到后的回复，sn必须相对应。

### 1. App登录 
Wifi作为Server，App作为client，向Wifi发起连接，并发出登录请求，Wifi收到登录指令，需立刻向Mcu下发`0x01` 设备查询指令，3s之内如果Mcu如果不上报，返回一个错误给App，否则返回设备的基本信息。此过程App一直等待Wifi返回信息，直到Wifi成功查询状态给App，成功或者失败。

**Request (App to WiFi):**

	{
	    "sn": 1020,
	    "cmd": "login",
	}

**Response (WiFi to App):**

	{
	    "sn": 1020,
	    "cmd": "login",
	    "ret": "200",
	    "mac": "AABBCCDDEEFF",
	    "pv": "1.0",
	    "sv": "<hard version | software version>",
	    "data": [
	        "pid::12345",
	        "pkey::12345",
	        "usrver::<string>"
	    ]
	}

`注:谁查给谁，不用给Server，wifi回复的Sn与App下发的sn相同。`

### 2. 设备状态查询 
App 与 Wifi建立连接之后，需要立刻获取设备的信息，在中间的过程中也可能会需要查询设备的所有状态信息，此时使用此接口，App下发查询指令，Wifi收到后回复200 表示收到，同时下发 `0x06` 查询指令，设备收到指令立刻上报当前所有状态，wifi收到后上报大小循环。

**Request (App to WiFi):**

	{
	    "sn": 1020,
	    "cmd": "info",
	}

**Request (WiFi to App):**
	{
	    "sn": 1020,
	    "cmd": "info",
	    "ret":200
	}


### 3. 发送心跳

    Request (App to WiFi):

    {
        "sn": 1026,
        "cmd": "heartbeat"
    }

    Response (WiFi to App):
    
    {
        "sn": 1026,
        "cmd": "heartbeat"
    }

此包只为在长连接时维持连接，无应答包，WiFi模块会对15s内无数据的连接进行回收，故App的心跳周期应小于此值。

### 4. 设备控制

参见T口协议

### 5. 上传状态

参见T口协议


**超时重传机制**

发送Request协议包后，等待制定时间，若没有收到Response协议包则进行重传，其中sn号每次发送时+1，上限为65535，溢出后归零重新计数，相应的应答包中，sn保持与请求包相同，用来识别请求应答对。

重传时sn号不变

### ret返回数字解释

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