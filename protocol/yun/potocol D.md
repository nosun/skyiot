#云服务平台——串口协议文档

	Make By: Moore Huang & nosun

Log：

	2014-07-26    更改ret值原"503"为“501”
	2014-07-28    更新"设备登录"的测试用例
	              增加对设备版本号长度的限制，不超过10个字节
	2014-07-29    更改"设备激活"命令为"查询版本"，并完善协议说明
	              大幅修改第一条命令，并增加"查询网络连接状态"命令
	2014-07-30    修改"查询设备信息"中键值对"应包括"为"可包括"
	              在"查询设备信息"的PS中强调必需包括的项
	              增加"查询设备信息"中"data"列表上限数的限制
	              增加plen字长说明
	              增加所有协议包中键值对数量的限制
	2014-08-13    增加"设备控制"命令中，无键值对时上传全部状态的逻辑


##串口协议

###约定

所有串口协议前增加plen，长度2字节，后面接实际协议，如下：

<table>
  <thead>
    <tr>
      <th>协议头</th>
      <th>协议长度</th>
      <th>协议体</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0xAA</td>
      <td>plen</td>
      <td>protocol</td>
    </tr>
  </tbody>
</table>

> plen为协议体长度，字长2字节   
> 整条协议包长度上限为512字节   
> 每条协议包中键值对("xxx::xxx\0")的数量上限为30个  

###1. 查询设备信息

####Request(WiFi to MCU)

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

####Response(MCU to WiFi)

<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>变量名</th>
      <th>隔离符</th>
      <th>值</th>
      <th>当前结束符</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x01</td>
      <td>key</td>
      <td>"::"</td>
      <td>value</td>
      <td>'\0'</td>
    </tr>
  </tbody>
</table>


>此协议WiFi上电后出发，主要目的为测试串口通信是否正常，同时获取主控设备信息，用于向服务器注册.  
>重要：回复此响应包后，MCU需立刻上传全部状态信息（包括`控制量`和`状态量`）  
>此处多个键值对由`\0`来分隔，即重复变量名到`\0`隔离符的格式  
>除`命令`和结束符，其余皆为字符值

键值对可包括下表所列信息：

<table>
  <thead>
    <tr>
      <th>变量</th>
	  <th>value</th>	
      <th>描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>vid</td>
	  <td>num [1~99999]</td>
	  <td>厂家ID</td>
    </tr>
  </tbody>

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
	  <td>设备的唯一串行码</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>usrver</td>
      <td>float</td>
	  <td>设备版本号</td>
    </tr>
  </tbody>
</table>

>1. 厂家可自定义添加键值对，不过类型必须为字符，此数据将直接经WiFi模块封包后发给服务器  
>2. 其中vid, pid, pkey为云平台服务商提供，响应包中必需包括  
>3. 此处WiFi模块中"data"项目数上限为30个
	
###2. 设备控制

####Request(WiFi to MCU)
<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>变量名</th>
      <th>隔离符</th>
      <th>值</th>
      <th>当前结束符</th>
      <th>...</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x02</td>
	  <td>key</td>
      <td>"::" </td>
	  <td>value</td>
	  <td>\0</td>
      <td>...</td>
    </tr>
  </tbody>
</table>

    此处将服务器的命令抽象为变量，如服务器要控制一个开关量，则可定义一个变量，通过下载此变量的值，达到控制设备的命令，例：变量switch，服务器下传value＝"1"，则可理解为打开插座
    若此处命令号后不存在键值对，则用户主控需上传全部状态

###3. 上传状态

####Request(MCU to WiFi)

<table>
  <thead>
    <tr>
      <th>命令号</th>
      <th>变量名</th>
      <th>隔离符</th>
      <th>值</th>
      <th>当前结束符</th>
      <th>...</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x03</td>
	  <td>key</td>
      <td>"::" </td>
	  <td>value</td>
	  <td>\0</td>
      <td>...</td>
    </tr>
  </tbody>
</table>



> 此处与 `下载数据` 命令相同，当主控设备的状态发生变化时，需立刻发送变化后的状态给WiFi模块，
> 从而上传到服务器，可以理解为上传数据为一种事件，当变化发生时需立刻产生此事件将变化后状态发送给WiFi模块

###4. 错误事件

####Request(WiFi to MCU)
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

>当WiFI模块与服务器间的连接断开，即上传状态命令所上传的数据无法传到服务器时，WiFi模块将向MCU发送此错误事件，通知MCU网络已断开

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

###5. 查询网络连接状态
####Request(MCU to WiFi)
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

####Response(WiFi to MCU)

命令号 	配置状态 	连接状态 	在线状态
0x05 	config 	link 	online

####参数说明&取值范围：
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


    PS: 上述参数字长均为1字节




##测试用例

###1. 单命令（查询版本/错误事件）

####查询版本
	
	WiFi to MCU: 0xAA 1 0x01
	
	MCU to WiFi: 0xAA 53 0x01 "vid" "::" "12345" \0 "pid" "::" "12345" \0 "pkey" "::" "12345" \0 "dsn" "::" "XXXXXXXXXXXX" \0

####错误事件

	WiFi to MCU: 0xAA 11 0x04 500

###2. 单命令（控制量）


###3. 单命令（状态量）
滤芯工作时间

MCU to WiFi: 0xAA 14 0x03 "filter" "::" "3000" \0

    滤芯工作时间单位为min

UV灯

MCU to WiFi: 0xAA 10 0x03 "uv" "::" "good" \0
电机

MCU to WiFi: 0xAA 13 0x03 "motor" "::" "good" \0
温度

MCU to WiFi: 0xAA 10 0x03 "temp" "::" "28" \0
湿度

MCU to WiFi: 0xAA 10 0x03 "humi" "::" "40" \0
PM2.5

MCU to WiFi: 0xAA 10 0x03 "pm25" "::" "57" \0
甲醛

MCU to WiFi: 0xAA 13 0x03 "forma" "::" "0.08" \0
VOC

MCU to WiFi: 0xAA 11 0x03 "voc" "::" "0.09" \0

###4. 组合命令

设备控制 上传状态均可分别进行组合，组合方式视需求而定
控制量

    命令解释：童锁开，风速高，模式自动，除菌开

WiFi to MCU: 0xAA 46 0x02 "lock" "::" "on" \0 "windspeed" "::" "high" \0 "mode" "::" "auto" \0 "kill" "::" "on" \0

MCU to WiFi: 0xAA 46 0x03 "lock" "::" "on" \0 "windspeed" "::" "high" \0 "mode" "::" "auto" \0 "kill" "::" "on" \0

    PS:所有控制命令 上传状态协议中的MCU to WiFi均为当主控设备(MCU)状态确实发生改变后，上传改变后状态

状态量

    命令解释：滤芯状态优、UV灯状态优、电机状态优、温度28度、湿度40%、PM2.5为57、甲醛浓度0.08、VOC浓度0.09

MCU to WiFi: 0xAA 84 0x03 "filter" "::" "3000" \0 "uv" "::" "good" \0 "motor" "::" "good" \0 "temp" "::" "28" \0 "humi" "::" "40" \0 "pm25" "::" "57" \0 "forma" "::" "0.08" \0 "voc" "::" "0.09" \0

    在模块上电发送完登录协议包后，应立刻上传全部状态信息（控制量和状态量），可分别发送控制量和状态量的上传状态命令，也可将两条命令中的键值合并为一条，之后只在状态量发生变化时发送变化后的单一状态量即可