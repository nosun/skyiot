#mobile to server —— 设备控制&状态获取#

    make by guohw@skyware.com.cn

info:

	2014-08-08 整理接口，提出修改意见

## 文档规约 ##

名词&缩写解释

+ **S口协议**：服务器与手机端协议
+ **T口协议**：服务器与设备端协议
+ **L口协议**：设备端与手机端协议
+ **ret值**：ret是服务器收到请求后的http状态回复

<table>
	<tr>
		<th>值</td>
		<th width=200>说明</td>
	</tr>
	<tr>
		<td>200</td>
		<td width=200>成功处理</td>
	</tr>
	<tr>
		<td>302</td>
		<td width=200>连接重定向</td>
	</tr>
	<tr>
		<td>401</td>
		<td width=200>拒绝访问</td>
	</tr>
	<tr>
		<td>404</td>
		<td width=200>页面不存在</td>
	</tr>
	<tr>
		<td>500</td>
		<td>服务器错误</td>
	</tr>
	<tr>
		<td>504</td>
		<td>连接超时</td>
	</tr>

</table>


本文档用于说明S口协议，以及相关测试用例，URL采用REST风格，每个URL对应相应的资源，通过相应的操作，对资源进行查看，增加，修改，删除。

+ S口协议采用http协议进行通信
+ 平台的根目录为/mdot
+ 设备的接入点为/device/

## 1.设备控制 ##

<table>
	<tr>
		<td>URL</td>
		<td width=300>/message/sendMg</td>
	</tr>
	<tr>
		<td class="alarm">new URL</td>
		<td width=300>/device/command</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###1.1 描述###

用户通过手机向指定设备发送指令，控制设备的运行状态。

###1.2 body argument###
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
    <tr>
      <td>loginId</td>
      <td>String</td>
      <td>用户id</td>
      <td>是</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td class="alarm">id (device_mac)</td>
      <td>String</td>
      <td>设备id，考虑换成mac地址</td>
      <td>是</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>commandv</td>
      <td>json obj</td>
      <td>发送指令消息体</td>
      <td>是</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td class="alarm">command</td>
      <td>json obj</td>
      <td>发送指令消息体</td>
      <td>是</td>
    </tr>
  </tbody>
</table>

### 1.3 body example ###

	"loginid"="131311233";
	"devicid"="995753324";
	"commondv"="{"sn":1023,"cmd":"download","data":["mode::1"]}";

### 1.4 response example ###

	{"ret":"200"}

### 1.5 控制指令###

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Description</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>power</td>
      <td>int</td>
      <td>开关</td>
      <td>0：关；1：开</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>klock</td>
      <td>int</td>
      <td>童锁 kidClock</td>
      <td>0：关；1：开</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>wind</td>
      <td>int</td>
      <td>风速</td>
      <td>0：关；1~5档</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>uv</td>
      <td>int</td>
      <td>uv杀菌</td>
      <td>0：关；1：开</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>anion</td>
      <td>int</td>
      <td>负离子</td>
      <td>0：关；1：开</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>mode</td>
      <td>int</td>
      <td>组合命令，设备收到命令后，响应一组命令</td>
      <td>0：关；1：auto;2:sleep</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>timer</td>
      <td>int</td>
      <td>定时功能</td>
      <td>0~12 小时</td>
    </tr>
  </tbody>
</table>

## 2.发送心跳 ##

<table>
	<tr>
		<td>URL</td>
		<td width=300>/message/sendMg</td>
	</tr>
	<tr>
		<td class="alarm">new URL</td>
		<td width=300>/device/command</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###2.1 描述###

手机在不同的界面对服务器与设备间保持心跳的频率做出控制

###2.2 body argument###
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>heart</td>
      <td>int</td>
      <td>心跳频率</td>
      <td>是</td>
    </tr>
  </tbody>
</table>

### 2.3 body example ###

	"loginid"="131311233";
	"devicid"="995753324";
	"commondv"="{"sn":1023,"cmd":"heartbeat","heart":30}";

### 2.4 response example ###

	{"ret":"200"}

## 3.从云推送获取设备状态 ##

###3.1 描述###

设备状态的改变通过云推送传至手机

### 3.2 body example ###

	运行状态
	{
	"cmd": "download",
	"deviceId":"1313131",
    "data":{
        "switch":"on",
        "winds ":"1",
        "lock":"1"
    	}
	}

	传感器状态
	{
	"cmd": "upload",
	"deviceId":"1313131",
	"time":"12321321321321321",
    "data":{
        "tem":"30",
        "hum ":"10",
        "pm":"15",
        "net_time":"15",
    	}
	}
	
	滤网过期提醒
	
	{
	"net_time":"15",
	"isOverdue":"0",
	"deviceId":"1313131"
	}
	
	如果值为1：过期，0：不过期

	设备不在线推送提醒
   	
	{
		"deviceOnline":"0",
		"deviceId":"1313131"
	}

	0:不在线；1：为在线


## 4.获取所有设备的状态##

### 4.1 描述###
<table>
	<tr>
		<td>URL</td>
		<td width=300>device/alldevicebind</td>
	</tr>
	<tr>
		<td class="alarm">new URL</td>
		<td width=300>device/{userid}</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

该接口用于调取用户绑定的设备的所有的状态值

###4.2 body argument###
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Description</th>
      <th>Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>loginId</td>
      <td>String</td>
      <td>用户id</td>
      <td>是</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>page</td>
      <td>int</td>
      <td>页数</td>
      <td>是</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>max</td>
      <td>int</td>
      <td>每页数量</td>
      <td>是</td>
    </tr>
  </tbody>
</table>

### 4.3 body example ###
	"loginid"="131311233";
	"page"="1";
	"max"="1";

### 4.4 response example ###

	{	"total":"1",
		"respCode":"200",
		"result":[
		{"deviceData":
		"{
			"tem":"4",
			"pw":"7",
			"hum":"22"
		 }",
			"deviceDesc1":"1407722795798",
			"deviceDesc2":"
			{	
			"power":"0",
			"wind":"2",
			"time":"3",
			"anion":"0",
			"mode"："2"
			}",
		"deviceLocation":"重庆",
		"deviceLock":"1",
		"deviceMac":"ACCF23216434",
		"deviceName":"WOWtrd",
		"deviceSn":"6922374562004",
		"id":"984433027"}
		]
	}

### 4.5 设备参数 ###

#### 4.5.1设备管理参数 ####

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Description</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>province</td>
      <td>string</td>
      <td>省</td>
      <td>0-999</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>city</td>
      <td>string</td>
      <td>市</td>
      <td>0-999</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>district</td>
      <td>string</td>
      <td>地区</td>
      <td>0-999</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>device_mac</td>
      <td>string</td>
      <td>mac地址</td>
      <td></td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>device_name</td>
      <td>string</td>
      <td>名称</td>
      <td></td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>device_sn</td>
      <td>string</td>
      <td>序列号</td>
      <td></td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>locked</td>
      <td>int</td>
      <td>是否锁定</td>
      <td>0：未锁，1：锁定</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>online</td>
      <td>int</td>
      <td>是否在线</td>
      <td>0，离线；1：在线</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>addtime</td>
      <td>int</td>
      <td>设备添加时间</td>
      <td></td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>updatetime</td>
      <td>int</td>
      <td>最后更新时间</td>
      <td></td>
    </tr>
  </tbody>
</table>

#### 4.5.2 设备状态参数 ####

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Description</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>power</td>
      <td>int</td>
      <td>开关</td>
      <td>0：关；1：开</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>klock</td>
      <td>int</td>
      <td>童锁</td>
      <td>0：关；1：开</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>wind</td>
      <td>int</td>
      <td>风速</td>
      <td>0：关；1~5档</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>uv</td>
      <td>int</td>
      <td>uv杀菌</td>
      <td>0：关；1：开</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>anion</td>
      <td>int</td>
      <td>负离子</td>
      <td>0：关；1：开</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>mode</td>
      <td>int</td>
      <td>组合命令，设备收到命令后，响应一组命令</td>
      <td>0：关；1：auto;2:sleep</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>timer</td>
      <td>int</td>
      <td>定时功能</td>
      <td>0~12 小时</td>
    </tr>
  </tbody>
</table>
#### 4.5.3 设备传感器参数 ####

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Description</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>temp</td>
      <td>int</td>
      <td>温度</td>
      <td>0-60</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>hum</td>
      <td>int</td>
      <td>湿度</td>
      <td>10%-95%</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>pm</td>
      <td>int</td>
      <td>Pm值</td>
      <td>0-999</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>net</td>
      <td>int</td>
      <td>滤网剩余时间</td>
      <td>0-4320</td>
    </tr>
  </tbody>
</table>
