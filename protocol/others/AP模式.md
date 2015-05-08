## Ap 模式协议

    author nosun
    date   2015-04-20
    version 1.0
    
#### 说明
本协议用于App通过Ap模式为wifi模块配网，大致流程如下：

1. 用户通过硬件触发wifi模块开启AP模式。
2. App选择当前环境下的路由器账号及加密方式，并输入路由器密码，App尝试判断WiFi账号密码，如果成功则将账号消息通过TCP连接发送至WiFi模块。（中间App会自动切网）
3. App与WiFi模块建立连接。
4. WiFi收到信息，设置自身的SockA，Wmode为STA,请根据是否设置成功回复App消息。重启，之后尝试连接路由器，并回复App是否能正常连接路由器。
5. App 收到Wifi成功或失败的状态值，如果成功，则尝试向服务器请求设备的状态，反复尝试三次。如果失败则反馈给用户相应的错误消息。
6. 如果WiFi模块连接不到用户指定的WiFi，要及时**切回AP模式**。然后App等待的时间内发现此Ap后认为配置失败，提示用户重新回到步骤2.


#### 账号设置

##### App to Wifi
    
	{"sn":1,"cmd":"config","wsssid":"XXX","wskey":"WPA2PSK,AES,*****"}
    
#### Wifi to App
    {"sn":1,"cmd":"config","ret":200}

#### ret返回数字解释

<table style="font-size:14px">
	<tr><th>ret值</td>
		<th>解释</td></tr>
	<tr><td>200</td>
		<td>设置成功</td></tr>
	<tr><td>400</td>
	    <td>参数不对</td></tr>
</table>