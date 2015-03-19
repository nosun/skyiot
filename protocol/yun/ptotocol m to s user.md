#mobile to server —— 用户操作 #

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
+ 用户接口的接入点为/user

## 1.用户注册 ##

<table>
	<tr>
		<td>URL</td>
		<td width=300>/user/register</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###1.1 描述###

用户通过手机向指定api发送注册信息，注册成功，服务器返回成功或者错误状态。

###1.2 body argument###
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
      <td>userPhone</td>
      <td>String 14</td>
      <td>用户手机号</td>
      <td>是</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>loginPwd</td>
      <td>String 20</td>
      <td>用户密码 </td>
      <td>是</td>
    </tr>
  </tbody>
</table>

### 1.3 body example ###

	userPhone="+8618999009878"
	loginPwd="nicaibudao"

### 1.4 response example ###

	{"ret":"200"}

##2、用户登录##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/user/login</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###2.1 描述 ###

用户名登录，第三方登陆，手机登陆

###2.2 参数 ###
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
      <td>loginType</td>
      <td>string 32</td>
      <td>0为普通登录，1为qq登录，2为sina登录,3为手机号登录</td>
      <td>Required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>loginName</td>
      <td>string 32</td>
      <td>登录id</td>
      <td>loginType为0时必填</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>loginPwd</td>
      <td>string 32</td>
      <td>登录密码</td>
      <td>loginType为0时必填</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>userQq</td>
      <td>string 32</td>
      <td>qq联合登录授权码</td>
      <td>loginType为1时必填</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td class="alarm">userSina</td>
      <td>string 32</td>
      <td>新浪联合登录授权码</td>
      <td>loginType为2时必填</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>userPhone</td>
      <td>string 32</td>
      <td>用户手机号</td>
      <td>loginType为3时必填</td>
    </tr>
  </tbody>
</table>

##3、用户退出##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/user/loginout</td>
	</tr>
	<tr>
		<td class="alarm">new URL</td>
		<td width=300>/user/logout</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###3.1描述###

用户注销，退出系统

###3.2参数###
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
      <td>LoginId</td>
      <td>String 20</td>
      <td>用户名</td>
      <td>Required</td>
    </tr>
  </tbody>
</table>
###3.3 Request###

	id="134"

###3.4 Response###

	{"ret":"200"}

##4、获取用户信息##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/user/detail</td>
	</tr>
	<tr>
		<td  class="alarm">new URL</td>
		<td width=300>/user/{userid}</td>
	</tr>
	<tr>
		<td>method</td>
		<td>get</td>
	</tr>
</table>

###4.1描述###

获取指定用户的详细信息

###4.2参数###
	none
###4.3 Request###

	none

###4.4 Response###

	{
		"result":
		{
			"id":"756181644",
			"loginName":"test1",
			"loginPwd":"202cb962ac59075b964b07152d234b70", 
			"userEmail":"123@qq.com",
			"userName":"ceshi1",
			"userRegtime":"1404141161359"，
			"userPhone":"18612349875"
		},
		"respCode":"1"
	}

<p class="alarm">* 密码不该传出来,部分无用的值无需传出来</p>

##5、修改用户信息##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/user/modifiy</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
	<tr>
		<td  class="alarm">new URL</td>
		<td width=300>/user/{userid}</td>
	</tr>
	<tr>
		<td  class="alarm">new method</td>
		<td>put</td>
	</tr>
</table>

###5.1描述###

修改指定用户的某项值

###5.2参数###
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
      <td>Required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>userName</td>
      <td>String</td>
      <td>用户名</td>
      <td>optional</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>loginPwd</td>
      <td>String</td>
      <td>密码</td>
      <td>optional</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>userEmail</td>
      <td>String</td>
      <td>电子邮件</td>
      <td>optional</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>userPhone</td>
      <td>String 20</td>
      <td>手机号码</td>
      <td>optional</td>
    </tr>
  </tbody>
</table>
###5.3 Request###

	none

###5.4 Response###

	{"ret":1}

##6、获取用户头像##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/share/img/{userid}</td>
	</tr>
	<tr>
		<td>method</td>
		<td>get</td>
	</tr>
	<tr>
		<td class="alarm">new URL</td>
		<td width=300>/user/img/{userid}</td>
	</tr>
</table>

###6.1描述###

修改用户头像

###6.2参数###
	none
###.3 Request###

	none

###6.4 Response###

	{"ret":1}

##7、修改用户头像##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/user/modifyuserimg</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
	<tr>
		<td class="alarm"> new URL</td>
		<td width=300>/user/img/{userid}</td>
	</tr>
	<tr>
		<td class="alarm">new method</td>
		<td width=300>put</td>
	</tr>
</table>

###7.1描述###

修改用户头像

###7.2参数###
	none
###7.3 Request###

	none

###7.4 Response###

	{"ret":1}

##8、获取短信验证码##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/shared/sendMessage</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###8.1描述###

用户注册或者找回密码过程中，发送手机号，获取短信验证码。

###8.2参数###
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
      <td>userPhone</td>
      <td>string 11</td>
      <td>用户手机号</td>
      <td>Required</td>
    </tr>
  </tbody>
</table>
###8.3 Request###

	user_phone="13800000000"

###8.4 Response###

	{"ret":1}

##9、验证短信验证码##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/shared/validationUser</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###9.1描述###

用户注册或者找回密码过程中，填写收到的短信，发送给服务器，服务器做出验证。

###9.2参数###
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
      <td>userPhone</td>
      <td>string 11</td>
      <td>用户手机号</td>
      <td>Required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>validationCode</td>
      <td>string 11</td>
      <td>短信验证码</td>
      <td>Required</td>
    </tr>
  </tbody>


</table>
###9.3 Request###

	user_phone="13800000000"
	validationCode="123456"

###9.4 Response###

	{"ret":true}

##10、邮箱验证##
<table>
	<tr>
		<td>URL</td>
		<td width=300>/shared/sendEmailCode</td>
	</tr>
	<tr>
		<td>method</td>
		<td>post</td>
	</tr>
</table>

###9.1描述###

用户填写的电子邮箱验证 
<p class="alarm">此接口尚未测试！</a>

###9.2参数###
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
      <td>userID</td>
      <td>string</td>
      <td>用户id</td>
      <td>Required</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>userEmail</td>
      <td>string</td>
      <td>电子邮箱</td>
      <td>Required</td>
    </tr>
  </tbody>
</table>

###9.3 Request###

	userID="982878989"
	userEmail="fafa@126.com"

###9.4 Response###

	{"ret":1}