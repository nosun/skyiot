## 串口Smartlink && 联网状态查询指令


## 协议内容
**协议说明：** 设备通过串口下发指令 `0x07` 给wifi模块，wifi模块开启Smartlink模式，随后wifi模块进入Smartlink状态，并发送回复给Wifi，告知wifi已经收到指令，并进入Smartlink状态，Mcu可以在设备端进行必要的显示，以标识设备已经进入Smartlink状态。

此时手机App开始于Wifi进行Smartlink配置，如果配置超时，APP端进行显示，告知用户重新进行配置，此时需要重新通过点击按键的方式重新触发Smartlink，此动作，Mcu需要首先重启wifi模块以关闭当前的Smartlink功能，同时再次发送开启Smartlink指令，使Wifi模块回复到配置状态。

如果Wifi模块Smartlink配置联网成功的话，Smartlink模式会自动关闭，同时Wifi模块会给Mcu一个设备联网状态的回复，Mcu收到此回复后，可以关闭对外的标识显示，并通过显示常量的方式告知用户Smartlink配置成功。

##协议指令
#### Request(Mcu to Wifi)
发送开启Smartlink命令

    AA 00 01 07 0A 

#### Request(WiFi to Mcu)
进入Smartlink状态

    AA 00 03 07 C8 0A
   
### Wifi上报设备联网状态

#### Respond(WiFi to Mcu)
联网成功汇报

    AA 00 04 05 01 01 01 0A 
    
> 说明：

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


#### Respond(WiFi to Mcu)
每次wifi模块开启时，会检测是否能联网，同时联网检测汇报

    AA 00 04 05 01 01 01 0A 

### 串口查询联网状态
Mcu通过串口亦可以主动查询Wifi当前的工作状态



#### Rquest(Mcu to Wifi)
Mcu查询Wifi当前连接状况

    AA 00 01 05 0A
     
#### Respond(WiFi to Mcu)
联网成功汇报

    AA 00 04 05 01 01 01 0A
