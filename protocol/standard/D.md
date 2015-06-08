##  Protocol D: Wifi & Mcu

	Make By: Moore Huang, nosun, wangyf
	version: v1.4

###一、设计原则与目标

本协议为WiFi与MCU之间的通信协议

+ **指令下发**：WiFi下发指令给MCU
+ **状态上报**：MCU状态变化后，及时上报WiFi模块
+ **传输稳定**：保证串口通信的稳定性

###二、协议要点

+ **协议组成：**
	
	协议分为协议头，协议长度，协议体（包含键值对与分隔符，或者二进制串），结束符

+ **协议长度与分隔符：**

	包分隔符为 `\n` (0x0A)，不计入协议长度。两个键值对之间的分割符为 `\0` (0x00)，计入协议长度。

+ **协议体与编码：**
	
	采用键值对的形式表示状态或者命令，默认采用 ASCII 编码。
 	
	但如果考虑传输效率，需要采用二进制方式传输协议体的内容。具体内容格式由MCU与云端协商，WiFi模块无需解析，但需要将T口和L口传输的 `data` 字段进行**BASE64**解码后转发给MCU，将MCU上报的二进制body进行BASE64加密，封装到`data` 字段中上报给T口和L口。

	MCU如果有二进制传输的需求，需要在 `基础信息` 中增加 `isbin` 的字段，值赋值为1。这样WiFi模块在login登录之后查到此字段，便对MCU上报与向MCU下发的数据body进行BASE64编解码

	

###三、通用说明

所有串口协议前增加plen，长度2字节，后面接实际协议，如下：

<table>
  <thead>
    <tr>
      <th>协议头</th>
      <th>协议长度</th>
      <th>协议体</th>
      <th>结束符</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0xAA</td>
      <td>plen</td>
      <td>body</td>
      <td>0x0A</td>
    </tr>
  </tbody>
</table>

> plen为协议体长度，字长2字节。整条协议包长度上限为512字节   
> 
> 每条协议包中键值对("xxx::xxx")的数量上限为30个
> 
> 如果有多个键值对，两两之间通过 `\0` (0x00) 分隔
> 
> 最后一个键值对后用`\n` (0x0A) 作为结束符
> 
> `\n` 不计入plen, `\0`计入plen


#####名词&缩写约定

+ pid: (Product ID): 为产品ID，长度5，用于标示指定产品
+ pv: (Protocol Version): 当前协议版本号 
+ sv: (firmware version): WiFi固件版本号
+ pkey: (Product Key): 长度5，与当前产品相对的密钥
+ [xxx]: 被'[',']'包围的JSON项为传输中非必需包含项

###四、协议内容

####1. 查询设备基础信息

**Request (WiFi to MCU)**

<table>
  <thead>
    <tr>
      <th>命令号</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x01</td>
    </tr>
  </tbody>
</table>

**Response (MCU to WiFi)**

<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>变量名</th>
      <th>隔离符</th>
      <th>值</th>
      <th>当前结束符</th>
	  <th>...</th>
	  <th>结束符</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x01</td>
      <td>key</td>
      <td>:</td>
      <td>value</td>
      <td>0x00</td>
	  <td>...</td>
	  <td>0x0A</td>
    </tr>
  </tbody>
</table>


>此协议WiFi上电后出发，目的一是为测试串口通信是否正常，二是为获取主控设备的基础信息，用于向服务器注册.  
>
>重要：回复此响应包后，MCU需立刻上传全部基础信息（见下表）  
>
>此处多个键值对由`\0`来分隔，即重复变量名到`\0`隔离符的格式  
>除`命令号0x01`和结束符，其余皆为ASCII字符值

键值对可包括下表所列信息：

<table>
  <thead>
    <tr>
      <th>变量key</th>
	  <th>value</th>	
      <th>描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>pid</td>
      <td>num [1~99999]</td>   
	  <td>产品ID</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>pkey</td>
      <td>num 5位定长</td>
	  <td>产品对应注册密钥</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>dsn</td>
      <td>char 32位以内</td>
	  <td>mcu串号，可选</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>mv</td>
      <td>float</td>
	  <td>mcu版本号，可选</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>bin</td>
      <td>num 1位 0/1</td>
	  <td>键值是否采用二进制传输，可选，默认为0</td>
    </tr>
  </tbody>
</table>

>1. 厂家可根据需求自定义添加键值对，不过类型必须为字符，此数据将直接经WiFi模块封包后发给服务器  
>2. 其中pid, pkey为云平台服务商提供，响应包中必需包括，dsn,usrver,isbin可选择。  
>3. 此处WiFi模块中"data"项目数上限为30个
	
####2. 设备控制

**Request (WiFi to MCU)**
<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>键值对</th>
      <th>分隔符</th>
      <th>...</th>
	  <th>结束符</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x02</td>
	  <td>key : value</td>
	  <td>0x00</td>
      <td>...</td>
	  <td>0x0A</td>
    </tr>
  </tbody>
</table>

> 此处将服务器的命令抽象为变量，如服务器要控制一个开关量，则可定义一个变量，通过下载此变量的值，达到控制设备的命令，例：switch:1，则可理解为打开插座，key value 为**ASCII码**。
    
Or

<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>二进制串</th>
      <th>结束符</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x02</td>
	  <td>binary</td>
	  <td>0x0A</td>
    </tr>
  </tbody>
</table>
>采用二进制传输，将 `data` 经过BASE64解码后下发给MCU

####3. 上传状态

**Request (MCU to WiFi)**

<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>键值对</th>
      <th>分隔符</th>
      <th>...</th>
	  <th>结束符</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x03</td>
	  <td>key : value</td>
	  <td>0x00</td>
      <td>...</td>
	  <td>0x0A</td>
    </tr>
  </tbody>
</table>

> 当主控设备MCU的状态发生变化时，需立刻发送变化后的状态给WiFi模块，从而上传到服务器。

Or

<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>二进制串</th>
      <th>结束符</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x03</td>
	  <td>binary</td>
	  <td>0x0A</td>
    </tr>
  </tbody>
</table>

>采用二进制传输，将 `data` 经过BASE64编码后放在`data`中上报给T口和L口

####4. 错误事件

**Request (WiFi to MCU)**
<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>错误号</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x04</td>
	  <td>ret</td>
    </tr>
  </tbody>
</table>

> 当WiFI模块与服务器间的连接断开，即上传状态命令所上传的数据无法传到服务器时，WiFi模块将向MCU发送此错误事件，通知MCU网络已断开，用户可根据实际情况选择，非必须。

ret的返回值如下：

<table>
  <thead>
    <tr>
      <th>ret值</th>
      <th>解释</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>200</td>
	  <td>成功收到协议包</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>500</td>
	  <td>服务器与WiFi模块间的连接断开，无法完成客户请求</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>501</td>
	  <td>不支持此协议</td>
    </tr>
  </tbody>
</table>

####5. 查询网络连接状态
**Request (MCU to WiFi)**
<table>
  <thead>
    <tr>
      <th>命令号</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x05</td>
    </tr>
  </tbody>
</table>

**Response (WiFi to MCU)**

<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>配置状态</th>
	  <th>连接状态</th>
	  <th>在线状态</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x05</td>
	  <td>config</td>
	  <td>link</td>
	  <td>online</td>
    </tr>
  </tbody>
</table>

**参数说明&取值范围：**
<table>
	<thead>
		<th>参数名</th>
		<th>取值范围</th>
		<th>说明</th>
	</thead>
	<tbody>
		<tr>
			<td>config</td>
			<td>1/0(已配置/未配置过)</td>
			<td>用于指示当前WiFi模块是否已配置到路由器</td>
		</tr>
		<tr>
			<td>link</td>
			<td>1/0(已连接/未连接)</td>
			<td>用于指示当前WiFi模块是否已连接到路由器</td>
		</tr>
		<tr>
			<td>online</td>
			<td>1/0(在线/离线)</td>
			<td>用于指示当前WiFi模块是否连接到服务器</td>
		</tr>
	</tbody>
</table>


####6.查询设备状态信息

**Request (WiFi to Mcu)**
<table>
  <thead>
    <tr>
      <th>命令号</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x06</td>
    </tr>
  </tbody>
</table>

**Request (MCU to WiFi)**


<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>键值对</th>
      <th>分隔符</th>
      <th>...</th>
	  <th>结束符</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x06</td>
	  <td>key : value</td>
	  <td>0x00</td>
      <td>...</td>
	  <td>0x0A</td>
    </tr>
  </tbody>
</table>
Or
<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>二进制串</th>
      <th>结束符</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x03</td>
	  <td>binary</td>
	  <td>0x0A</td>
    </tr>
  </tbody>
</table>


> 此接口用于Server或App查询设备的状态量时使用。设备MCU收到WiFi传过来的查询指令，需立刻上报设备的当前所有的状态量。

####7. 通过串口开启Smartlink

**Request(Mcu to Wifi)**

<table>
  <thead>
    <tr>
      <th>命令号</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x07</td>
    </tr>
  </tbody>
</table>

**Request(WiFi to Mcu)**

进入Smartlink状态

<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>结果</th>
      <th>结束符</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x07</td>
	  <td>0xC8</td>
	  <td>0x0A</td>
    </tr>
  </tbody>
</table>

配网的结果，通过 0x05 指令上报

<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>配置状态</th>
	  <th>连接状态</th>
	  <th>在线状态</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x05</td>
	  <td>config</td>
	  <td>link</td>
	  <td>online</td>
    </tr>
  </tbody>
</table>