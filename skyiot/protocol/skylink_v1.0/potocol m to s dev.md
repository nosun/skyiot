#mobile to server —— 设备操作 #

    make by guohw@skyware.com.cn

info:

	2014-08-08 整理接口，提出修改意见

## 文档规约 ##

名词&缩写解释

+ **S口协议**：服务器与手机端协议
+ **T口协议**：服务器与设备端协议
+ **L口协议**：设备端与手机端协议


本文档用于说明S口协议，以及相关测试用例，URL采用REST风格，每个URL对应相应的资源，通过相应的操作，对资源进行查看，增加，修改，删除。

+ S口协议采用http协议进行通信
+ 平台的根目录为/mdot
+ 设备的接入点为/device/

## 1.扫描并查询设备 ##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/device/isExist</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###1.1描述###

扫描设备dsn号码，到服务器查询该号码是否合法，在设备表中是否存在。

###1.2参数###
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
      <td>deviceSn</td>
      <td>string 12</td>
      <td>设备序列号，条形码</td>
      <td>Required</td>
    </tr>
  </tbody>
</table>
###1.3 Request###

	devicesn="123456789123"

###1.4 Response###

	{"ret":true}

##2、更新设备信息##

<table>
	<tr>
		<td>URL</td>
		<td width=300>/device/configByMac</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###2.1 描述###

根据设备的mac地址查询，对设备的信息进行更新，将sn号，设备地址等信息更新至服务器

### 2.2 参数###

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
      <td>string 32</td>
      <td>用户id</td>
      <td>Required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>device</td>
      <td>json body</td>
      <td>设备信息</td>
      <td>Required</td>
    </tr>
  </tbody>
</table>
### 2.3 Request ###
	
	loginId ="987654321"
	device="{
		"mac":"ffeeddccdd",
		"district":"北京 海淀 温泉",
		"divice_name":"my home",
	}"

### 2.4 Response ###

	{"ret":true}

##3、绑定设备##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/device/bind</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

### 3.1 描述###
用户操作，建立用户id，设备id，百度云推送id的绑定关系

### 3.2 参数###
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
      <td class="alarm">bindId/loginId</td>
      <td>string</td>
      <td>用户id</td>
      <td>Required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>diviceId</td>
      <td>string</td>
      <td>设备id</td>
      <td>required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>meid</td>
      <td>string</td>
      <td>百度userid</td>
      <td>required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>channelId</td>
      <td>string</td>
      <td>百度channelId</td>
      <td>required</td>
    </tr>
  </tbody>
</table>

### 3.3 Request ###

	loginid =999999
	deviceid =999999
	meid	=9999999
	channelid=999999

### 3.4 Response ###
	{"ret":1}

##4、解除绑定##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/device/unbind</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

<p class="alarm">解除绑定也需要百度推送的账号信息</p>

###4.1 描述###

### 4.2 参数###
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
      <td>dicFrom</td>
      <td>string</td>
      <td>指向用户id</td>
      <td>Required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>dicTo</td>
      <td>string</td>
      <td>指向设备id</td>
      <td>required</td>
    </tr>
  </tbody>
</table>

### 4.3 Request###
	[
		{
			dicFrom:"661654640",
			dicTo:"606834246"
		},
		{
			dicFrom:"661654640",
			dicTo:"606834246"
		}
	]
### 4.4 Response###
	
	{"ret"：1}


## 5绑定百度推送id ##
<table>
	<tr>
		<td>URL</td>
		<td width=300>message/mobileBindDevice</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###5.1 描述###

### 5.2 参数###
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
      <td class="alarm">loginId</td>
      <td>string</td>
      <td>用户id</td>
      <td>Required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>mobile_type</td>
      <td>string</td>
      <td>手机操作系统类型</td>
      <td>Required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>deviceIds</td>
      <td>string</td>
      <td>设备id</td>
      <td>required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>meid</td>
      <td>string</td>
      <td>百度userid</td>
      <td>required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>channelId</td>
      <td>string</td>
      <td>百度channelId</td>
      <td>required</td>
    </tr>
  </tbody>
</table>

### 5.3 Request###
	[
		{
			dicFrom:"661654640",
			dicTo:"606834246"
		},
		{
			dicFrom:"661654640",
			dicTo:"606834246"
		}
	]
### 5.4 Response###
	
	{"ret"：1}
