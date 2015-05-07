## Protocol T: WiFi & Server

    	Make By: Moore Huang, nosun,wyf
    	version: v1.3 

###一、设计原则与目标

+ **远程监控**：云端可实时监控设备的状态
+ **远程控制**：云端可随时控制在网的设备
+ **响应及时**：设备状态的上报和控制结果必须响应及时
+ **链路可靠**：设备与云端之间的连接链路必须可靠
+ **节约流量**：减少不必要的流量消耗
###二、协议要点

+ **TCP长连接**：

	为了保证Server反向控制WiFi的实时性和高效率，WiFi与Server之间的通信采用TCP长连接方式，而长连接的维持需要心跳机制来保证。
	
	WiFi端作为TCP Client，主动向Server发起三次握手。断开时Dev端主动断开。（正常情况下一直保持连接，除非遇到异常等特殊情况才主断开）	

+ **心跳维持**：

	设备主动向Server发送的用来维持TCP长连接的短包。一般维持在30s-60s左右，防止传输过程中转发路由的NAT端口映射老化的问题。Server则根据此心跳包来判断WiFi的在线情况。

+ **异步上报**：

	Server -> Dev 控制命令的发送是通过TCP长连接推送过去，而Dev -> Server的状态回馈是通过TCP长连接主动上报。Dev接到命令后立即给server反馈 `ret:200`，告知已收到并执行。然后Dev等待MCU执行完毕后将操作后的状态异步反馈给server

+ **固件升级**：

	采用内网和远程两种升级方式。两者从指令层面相同。WiFi模块需要在升级前进行同步回复来确认收到，升级后进行异步回复来告知升级的结果。详见 `协议内容 -> 7.WiFi固件升级` 一节


+ **流量控制**：

	物联系统的并发与流量控制是云端性能优化的关键。在很多场景下，并不需要对设备的实时监控，故需要由云端对WiFi模块的状态上报行为进行约束与控制，来保证整个系统的稳定，减少不必要的流量和带宽损耗。详见 `协议内容 -> 6. 上传限制` 一节


###三、协议通用说明

+ **域名与端口**：Server端的域名和端口预先协商好并写死在WiFi固件中。目前云平台的域名为 *yun.skyware.com.cn* ，端口为9999

+ **编码与格式**：编码统一采用ASCII字符集编码，格式采用JSON标准格式。

+ **帧结束符**：每个JSON包应以 ASCII字符 `\n` 作为结尾，以便应用层程序从流中解析成帧。此规则适用于后面所有协议包

+ **MCU编码格式**：MCU上报的数据按照D口协议，通过串口传输到WiFi模块，默认是ASCII字符格式。每一个状态键值对，按照`"key::value"`的格式，放到Key为`"data"`的JsonArray之中。如果客户MCU有特殊要求采用二进制传输，则`"data"`中的二进制数据采用 `BASE64` 方式转码。

+ **sn号规则**：

>1、App主动发起的请求（登录，查询，命令，升级），sn由APP 自己维护。WiFi的回复包，sn必须相对应。

>2、WiFi主动发起的上报（状态与错误上报），sn由WiFi模块维护，每次上报+1递增，App根据sn的大小判断次序防止网络原因导致的乱序。

+ **名词&缩写约定**

> sn: (Serial Number)，协议序号，用于匹配请求-应答对  
> 
> pid: (Product ID)，为产品ID，长度5，用于标示指定产品
> 
> pv: (Protocol Version)， 当前协议版本号 
> 
> sv: (firmware version) wifi固件版本号
> 
> pkey: (Product Key)，长度5，与当前产品相对的密钥
> 
> [xxx]: 被'[',']'包围的JSON项为传输中非必需包含项

###四、协议内容

#### 1. 设备激活/登录

**Request (WiFi to Server):**

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
>
>此处WiFi模块设备激活前，需先向用户主控(MCU)发送激活命令（命令格式见串口协议文档），若3s内未收到主控发来的响应包（包含主控的vid, pid, pkey等相关信息），则以无"data"的JSON包激活，并立刻上传错误事件到服务器。此处若上传错误事件后，WiFi模块收到用户主控发来的响应包，则重新发送设备激活命令。若收到主控的响应包，则将响应包中的数据填充入"data"

**Response (Server to WiFi):**

	{
	    "sn": 1020,
	    "cmd": "login",
	    "ret": 200,
	    "akey": "0123456789ABCDEF",
	    "heart": 30,
	    ["serverip"]: "ipaddr:port",
	}

>`ret` 此处可取值为(200, 403, 301, 302)
>
>其中200为正常响应，403为未授权阻止接入，301为永久重定向，302为临时重定向
>
>当取值为301、302时，Server会同时下发 serverip，当wifi收到 serverip时会将向服务器发起连接的socket 地址改写，301为永久改写，302为临时改写，改写完毕之后，socket主动关闭连接并向收到的server地址发起新的连接。

>`heart` 设置默认心跳包间隔时间，单位为秒(s)

>`serverip` 为重定向模块连接到指定服务器

#### 2. 设备状态查询 

**Request (Server to WiFi):**

	{
	    "sn": 1021,
	    "cmd": "info",
	}

**Request (WiFi to Server):**

	{
	    "sn": 1021,
	    "cmd": "info",
	    "ret": 200
	}

> 此接口用于Server向WiFi模块查询当前MCU的状态，保证Server的数据与MCU状态保持同步
> 
> WiFi收到后回复200 表示收到，同时通过串口下发 `0x06` 查询指令，设备MCU收到指令立刻上报当前所有状态，WiFi收到后通过`cmd: upload`（见下方条目5）上报Server。

#### 3. 发送心跳

**Request (WiFi to Server):**

	{
	    "sn": 1022,
	    "cmd": "heartbeat",
	    "mac": "AABBCCDDEEFF"
	}

**Response (Server to WiFi):**

	{
	    "sn": 1022,
	    "cmd": "heartbeat",
	    ["period"]: 30
	}

>`period` 为心跳周期，即服务器可以动态控制设备的心跳间隔，从而便于合理分配服务器资源，若响应包无此项数据则WiFI模块使用上次的历史间隔时间

#### 4. 设备控制

**Request (Server to WiFi):**

	{
	    "sn": 1023,
	    "cmd": "download",
	    "data": [
	        "switch::on",
	        "windspeed::high"
	    ]
	}

**Response (WiFi to Server):**

	{
	    "sn": 1023,
	    "cmd": "download",
	    "ret": 200
	}

>Server 发送指令给wifi，wifi收到指令后回复Server收到指令，并不代表指令执行成功。
>
>注意:此处WiFi模块中限制"data"列表项目数上限30个

#### 5. 上传状态

**Request (WiFi to Server):**

	{
	    "sn": 1024,
	    "cmd": "upload",
	    "mac": "AABBCCDDEEFF",
	    "data": [
	        "switch::on",
	        "windspeed::high"
	    ]
	}

**Response (Server to WiFi):**

	{
	    "sn": 1024,
	    "cmd": "upload",
	    "ret": 200
	}

>MCU 上报自身状态，通过WiFi模块上报给Server，Server收到后回复200，表示收到。如果收到错误码回复，或3s内没有收到Server的200回复，WiFi应继续发送。

#### 6. 上传限制

**Request (Server to WiFi):**

	{
	    "sn": 1025,
	    "cmd": "silence",
	    ["switch"]: "on/off"
		["period"]: 30
	}

**Response (WiFi to Server):**

	{
	    "sn": 1025,
	    "cmd": "silence",
	    "ret": 200
	}

>此指令用于Server端的流量控制，通过对WiFi的上报开关以及频次的管理。在对实时性要求不高的场景下，Server端不需要MCU的频繁上报，可以通过下发`"switch":"off"`和`"switch":"on"`来关闭/开启WiFi的上报行为，以及通过`"period":"xx"`(xx为秒数)来设置最小上报间隔，WiFi模块的上报间隔不得低于此值。此值不设置代表没有间隔的限制。

#### 7. WiFi固件升级

**Request (Server to WiFi):**

	{
	    "sn": 1026,
	    "cmd": "updatewifi",
	    "url": "http://www.xxx.com/path/to/update.bin"
	}

**Response (WiFi to Server):**

	{
	    "sn": 1026,
	    "cmd": "updatewifi",
	    "ret": 200
	}
 
> `ret` 为当前协议的应答返回值，可选值为（200,201,400）
> 
>>**200**： 为立即回复，表明设备已接收到此协议包，但不表明已执行成功。
> 
>>**201**： WiFi模块升级成功之后上报，表示更新固件成功。
> 
>>**401**： 表示更新固件失败。每次固件升级需要升级版本号，并提供相应的升级说明。

> 固件升级，可以通过小循环（局域网）下发，也可以通过Server下发，谁下发的指令返回给谁200的结果，升级成功之后大循环，小循环都上报升级结果。

#### 8. 错误事件

**Request (WiFi to Server):**

	{
	    "sn": 1027,
		"mac": "AABBCCDDEEFF",
	    "cmd": "error",
	    "ret": 404
	}

**Response (Server to WiFi):**

	{
	    "sn": 1027,
	    "cmd": "error",
	    "ret": 200
	}

> WiFi模块在登录过程中首先要请求MCU数据，如果没有获取到MCU数据，则上报给服务器错误，此处还可以定义其他错误。

### 超时重传机制
发送 `Request` 协议包后，等待制定时间，若没有收到 `Response` 协议包则进行重传，其中 `sn` 号每次发送时 `+1` ，上限为65535，溢出后归零重新计数，相应的应答包中，`sn` 保持与请求包相同，用来识别请求应答对

重传时 `sn` 号不变

### 五、ret 错误码

<table style="font-size:14px">
	<tr><th>ret值</td>
		<th>解释</td></tr>
	<tr><td>200</td>
		<td>成功收到协议包</td></tr>
	<tr><td>201</td>
		<td width=200>WiFi固件升级成功</td></tr>
	<tr><td>301</td>
		<td>永久重定向</td></tr>
	<tr><td>302</td>
		<td>临时重定向</td></tr>
	<tr><td>400</td>
		<td>请求参数错误</td></tr>
	<tr><td>401</td>
		<td>WiFi固件升级失败</td></tr>
	<tr><td>403</td>
		<td>设备未授权</td></tr>
	<tr><td>404</td>
		<td>未接收到主控的串口数据</td></tr>
	<tr><td>501</td>
		<td width=200>不支持此协议</td></tr>
	<tr><td>503</td>
		<td>WiFi模块负载重，请稍后再发</td></tr>
</table>