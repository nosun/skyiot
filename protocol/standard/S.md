# Protocol S: Server & App

    Make: nosun
    Date: 2015-07-20
    ver : V2.0
    
## 说明

本接口文档为思佳维云平台移动客户端接口文档，采用RESTFUL理念进行设计,使用http协议中的GET,POST,PUT,DELETE等方式对资源进行操作，实现对资源的增删查改。

### json格式

所有返回值均为json格式。

### header

header 中有三个必填参数，键名统一用小写；

- apiver（必填）   ：这里对应的是api接口的版本号，例如 v1；
- token（接口相关） ：这里对应的使用户的身份认证token，但不是所有的接口都有；
- signature（必填）: 访问签名信息；

### signature key

应签名加密的要求，这里设置一个客户端和服务器约定好的key。

### signature

分为两种情况，构造方式如下：

- 接口不需要token  
对url中`api`之后的所有字符串`拼接`后进行`MD5加密`，然后`拼接`指定 `key` 第二次`MD5加密`，如：
    
        访问/api/login_id/1/18600364250
        
        key = 'skyware'; 
        apiver = 'v1';
        
        signature = MD5(MD5('login_id'.'1'.'18600364250').'v1'.'skyware')。

> 其中`.`为字符串拼接符号，请根据具体的语言进行处理.    

- 接口需要token

        访问/api/devices
        
        key = 'skyware'; 
        apiver = 'v1';
        token = '19234'
    
        signature = MD5(MD5('devices').'v1'.'19234'.'skyware')。

### 响应状态 

    200：响应成功
    400：签名不正确或请求参数不正确
	401：token不存在
	403：禁止访问（黑名单）
    404：请求无结果
	405：方法不存在
	406：必须https
	429：访问速度过快
    500：服务器错误
	501：创建文件夹失败

##用户相关

###1、验证用户id

    用户在注册之前，验证用户id是否存在，使用电话号码作为用户id，变量为login_id

    url        : login_id
    methord    : get
    argument   : app_id/login_id
    example    : /api/login_id/1/18600364250
    
    return 
    if sucess  : {
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 404
                 }

###2、用户注册

	短信验证之后，将用户名密码发过来，进行注册。

    url        : user
    methord    : post
    argument   : login_id  手机号码
                 app_id    app_id
				 login_pwd 用户密码
    example    : /api/user
    
    return 
    if sucess  : {
                     "login_id": "18600364250",
                     "message": 200
                 }
                 
    if fail    : {
                     "login_id": "18600364250",
                     "message": 500
                 }

    message: 500：未能成功添加用户
    
###3、用户登录
	
	用户进行登录，这里考虑常规登录，尚未做社会化登录，用户登录之后，给返还一个token，登录之后所有的与服务器之间的交互，均需要发送token过来。(token会在两个小时之后过期)

    url        : token
    methord    : post
    argument   : login_id  手机号码
                 app_id    app_id
				 login_pwd 用户密码
    example    : /api/token
    
    return
    if sucess  : {
                     "token": "*****",
                     "message":200
                 }
                  
    if fail    : {
                     "message":404
                 }
    
    message: 404：用户名不存在
             500：用户名密码错误

###4、用户退出

	用户退出，删除token，然后调回登录页面，如果找不到token，自动跳转到登录页面。

    url        : token
    methord    : delete
    argument   : (需要token)
    example    : /api/token
    
    return
    if sucess  : {
                     "message":200
                 }
                  
    if fail    : {
                     "message":500
                 }
    
    message: 200：退出成功
             500：退出失败

###5、获取用户信息

	用户发送token，获取用户的基本信息，目前可能有部分信息缺少，请补充一下。

    url        : user
    methord    : get
    argument   : (需要token)
    example    : /api/user
    
    return
    if sucess  : {
				     "result": [
				         {
				             "login_id": "18600364250",
				             "user_name": "nosun",
				             "user_img": "http://ccc.cn/uploads/201502/99e521bfc7d2851f0f6cc7f7fcdc85bb.jpg",
				             "user_email": "nosun@nosun.cn",
				             "user_phone": "18600364250",
				             "notice_pm": "1",
				             "notice_pm_value": "80",
				             "notice_filter": "1"
				         }
				     ],
				     "message": 200
				 }
                  
    if fail    : {
                     "message": 404
                 }
                
    message: 404：用户名不存在    

###6、修改用户信息

	修改用户的基本信息。

    url        : user
    methord    : put
    argument   : (需要token)
    example    : /api/user
    需要修改的字段 放在put中，目前支持修改的字段如下：
			user_name
			user_phone
			user_email
			user_img
			user_perfer {json 对象，用来存储用户偏好}

    return 
    if sucess  : {
                     "message": 200
                 }
                 http code :200;
                 
    if fail    : {
                     "message": 500
                 }
                 http code :200;
    
    message: 400：参数错误
             500：服务器错误    

###7、修改用户密码

	修改用户密码，将token和用户密码传过来

    url        : passwd
    methord    : put
    argument   : (需要token)
				 login_pwd     [put 参数]
				 login_pwd_old [put 参数]

    example    : /api/passwd

    return
    if sucess  : {
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 500
                 }
                
    message: 400：参数错误
             500：服务器错误
			 404：旧密码不正确    


###8、找回用户密码

	找回用户密码，将login_id和用户密码传过来

    url        : passwd
    methord    : post
    argument   : login_pwd  [post 参数]
                 login_id   [post 参数]
                 app_id     
    example    : /api/passwd

    return
    if sucess  : {
                     "message": 200
                 }
                 
    if fail    : {
                     "message":500
                 }

    message: 400：参数错误
             500：服务器错误    
                
###9、上传用户头像

	上传用户头像，文件的类型是图片，大小为 不超过512k，字段的名称为file，token需要同时传过来

    url        : file
    methord    : post
    argument   : (需要token) 
	        	 file (file 的字段 name=file)
    example    : /api/file

    return
    if sucess  : {
					 "url": the url of the img
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 500
                 }
                
    message: 400：参数错误
             500：服务器错误    
             501：未能成功创建文件夹

###10、用户反馈

	添加用户反馈，需要将反馈主题，反馈内容，产品id，反馈类型，token需要同时传过来

    url        : feedback
    methord    : post
    argument   : (需要token)
	        	 title
				 content
				 product_id
				 category		类型（int）
    example    : /api/feedback

    return
    if sucess  : {
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 500
                 }
                
    message: 200：添加成功
			 400：参数错误
             404：该反馈已存在    
             500：添加失败

###11、用户反馈回复

	添加用户反馈回复，需要将反馈id，回复内容，token需要同时传过来

    url        : feedback_reply
    methord    : post
    argument   : (需要token) 
	        	 fid
				 content
    example    : /api/feedback_reply

    return
    if sucess  : {
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 500
                 }
                
    message: 200：添加成功
			 400：参数错误
             404：该回复已存在
             500：添加失败

##设备管理

###1、查询设备
	设备在smartlink之前的第一步，是看设备是否在数据库中。

    url        : device
    methord    : get
    argument   : (需要token)
				 sn|mac|id|link 均可以，以键值对形式请求 
				 (P.S.:link为ssid+key的MD5加密串)
    example    : /api/device/sn/$sn
                 /api/device/mac/$mac
                 /api/device/id/$id
				 /api/device/link/$link

    return
    if sucess  : {
				     "result": {
				         "device_id": "100387",
				         "device_mac": "ACCF234DB278",
				         "device_sn": "1001",
				         "device_link": "48ce0de8678079185c3c5180438c7cba",
				         "device_state": "0",
				         "device_name": "上海展会",
				         "product_id": "1",
				         "device_protocol_ver": "2.0",
				         "device_wifi_version": "<1.0.9|1.3>",
				         "device_mcu_version": "V5107",
				         "device_lock": "0",
				         "device_online": "0",
				         "add_time": "1433744329",
				         "update_time": "1436950392",
				         "setlink_time": "1433744412",
				         "longitude": "0",
				         "latitude": "0",
				         "radius": "136",
				         "device_address": "上海市上海市",
				         "device_data": {
				             "pw": "1",
				             "lc": "0",
				             "uv": "1",
				             "io": "0",
				             "mo": "0",
				             "fa": "4",
				             "tm": "000",
				             "fi": "56706",
				             "pm": "0201",
				             "th": "2362"
				         },
				         "area_id": "101020100",
				         "province": "上海市",
				         "city": "上海市",
				         "pm_id": "上海",
				         "product_name": "AP500",
				         "app_name": "Airpal"
				     },
				     "message": 200,
				     "time": 1437446690
				 }
    if fail    : {
                     "message": 404
                 }
                
    message: 400：参数错误
			 200：已经绑定过
             404：设备不存在

### 2、验证Sn是否合法

验证 Sn号码是否合法。

    url        : deviceSn
    methord    : get
    argument   : (需要token)
				 app_id     
				 sn   
				 				 				 
    example    : /api/deviceSn/1/100
    
    return
    if sucess  : {
             		 "message": 200
                 }
                 
    if fail    : {
                     "message": 404 
                 }
                
    message: 400:参数错误
             404:Sn没查到
                 
### 3、获取设备清单

    url        : devices
    methord    : get
    argument   : (需要token)
    example    : /api/devices

    return
    if sucess  : {
				     "result": [
				         {
				             "device_id": "100387",
				             "device_mac": "ACCF234DB278",
				             "device_sn": "1001",
				             "device_link": "48ce0de8678079185c3c5180438c7cba",
				             "device_state": "0",
				             "device_name": "上海展会",
				             "product_id": "1",
				             "device_protocol_ver": "2.0",
				             "device_wifi_version": "<1.0.9|1.3>",
				             "device_mcu_version": "V5107",
				             "device_lock": "0",
				             "device_online": "0",
				             "add_time": "1433744329",
				             "update_time": "1436950392",
				             "setlink_time": "1433744412",
				             "longitude": "0",
				             "latitude": "0",
				             "radius": "136",
				             "device_address": "上海市上海市",
				             "device_data": {
				                 "pw": "1",
				                 "lc": "0",
				                 "uv": "1",
				                 "io": "0",
				                 "mo": "0",
				                 "fa": "4",
				                 "tm": "000",
				                 "fi": "56706",
				                 "pm": "0201",
				                 "th": "2362"
				             },
				             "area_id": "101020100",
				             "province": "上海市",
				             "city": "上海市",
				             "pm_id": "上海"
				         },
				         {
				             "device_id": "100419",
				             "device_mac": "ACCF234DAF0E",
				             "device_sn": "1021",
				             "device_state": "0",
				             "device_name": "壶",
				             "product_id": "3",
				             "device_protocol_ver": "2.0",
				             "device_wifi_version": "<1.0.9|1.3>",
				             "device_mcu_version": "1",
				             "device_lock": "1",
				             "device_online": "1",
				             "add_time": "1435567730",
				             "update_time": "1437387536",
				             "longitude": "0",
				             "latitude": "0",
				             "device_data": {
				                 "p": "0",
				                 "m": "00100",
				                 "t": "025",
				                 "f": "0",
				                 "h": "0",
				                 "e": "0",
				                 "ut": "1435299050",
				                 "t1": "1437387740",
				                 "t2": "1437387633",
				                 "t3": "1437387635"
				             }
				         }
				     ],
				     "message": 200,
				     "time": 1437447201
				 }
                                 
    if fail    : {
                     "message": 500
                 }
                
    message: 400：参数错误
             404：没有结果
             

### 4、更新设备信息
	设备一经连接云平台，立刻进行登录操作，上报设备的基本信息，此时进行入库操作。
	用户通过客户端登录后，对设备进行修改，绑定等操作。
	首次添加某设备，尚无法查出device_id,通过device_mac 对设备信息进行修改。
    
	url        : device/device_mac
    methord    : put
    argument   : (需要token)
				 province                
				 city                    
				 district                   
				 device_name
				 device_mac 
				 device_sn
				 device_lock
				 longitude
				 latitude  
				 radius  
				 device_address
				 area_id

    example    : /api/device/device_mac

    return
    if sucess  : {
				 	 "result": json  
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 500
                 }
    message: 400：参数错误
             500：服务器错误    


### 5、绑定设备
	建立用户与设备的绑定关系，,目前尚未做设备和用户已经绑定的关系进行判断

    url        : bind
    methord    : post
    argument   : (需要token)
				 $device_id || $device_mac 
    example    : /api/bind

    return
    if sucess  : {
					 "result" : num(用户绑定设备数量)
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 500
                 }
                
    message: 400：参数错误
             404:未找到该设备，请重试~
             500：服务器错误
                
### 6、解除绑定
	解除用户与设备的绑定关系,目前尚未做设备和用户已经绑定的关系进行判断

    url        : bind
    methord    : delete
    argument   : (需要token)
				 device_id [delete 参数] 1_2_4
    example    : /api/bind/device_id

    return
    if sucess  : {
					 "result" : num(解绑成功数量)
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 500 
                 }
                
    message: 400：参数错误
             500：服务器错误

### 7、发送指令

	App 通过 http post 方式向设备发送指令，控制设备

    url        : cmd
    methord    : post
    argument   : (需要token)
				 device_id
				 commandv  {"sn":1,"cmd":"download","data":["pw::1"]}
				 
    example    : /api/cmd
    
    return    
    if sucess  : {
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 500 
                 }
                
    message: 400:参数错误
             404:用户和该设备无绑定关系
             500:连接服务器失败
             501:发送失败


## 公共API接口

### 1、天气接口
	根据获取到的地址信息查询天气和pm，并返回

    url        : wpm
    methord    : post
    argument   : $province | 省
				 $city	   | 市
				 $district | 区
	example    : /api/wpm

    return
    if sucess  : {
			         "result": {
			             "temperature": "15",
			             "humidity": "32",
			             "pm": "114"
			         },
			         "message": 200
			     }
                 
    if fail    : {
                     "message": 404 
                 }

    message: 200：成功
             404：查询失败
             400：参数错误

##运营相关

###1、根据app_id 下载最新的app version
    url        : app
    methord    : get
    argument   : app_id [get 参数] 
    example    : /api/app

    return
    if sucess  : {
                     "result": json
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 404
                 }
                
    message: 400：参数错误
             404：没有结果
             
###2、调取厂家信息

    url        : company
    methord    : get
    argument   : company_id [get 参数]  需要token
    example    : /api/company

    return
    if sucess  : {
                     "result": json
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 404
                 }
                
    message: 400：参数错误
             404：没有结果 
             
##  测试相关

### 网速测试
给客户端提供指定大小的数据包，提供给客户端下载，从而测定网络环境。

    url        : testSpeed
    methord    : get
    argument   : (int)num [get 参数] 数据包大小，默认为1k
    example    : /api/testSpeed

    return
    if sucess  : {
                     "result": 指定大小的数据包
                     "message": 200
                 }
                
    加载时间需要客户端来做。


### Http码测试
给客户端提供http错误码，看客户端框架的响应。

    url        : testHttp
    methord    : get
    argument   : (int) code [get 参数] http 错误码，比如 200 404…… 
    example    : /api/testHttp

    return
    if sucess  : {
                     "result": ok
                 }
                 http code
- log上传

    url        : log
    methord    : post formdata
    argument   : (需要token)  
                 file   log文件
    require    : fileSize limit: 1024k
                 fileType limit: *.log
     
    example    : /api/log

    return
    if sucess  : {
                     "result": url
                     "message": 200
                 }
                 
    message: 200：成功上传
             400：参数错误
             500：服务器错误    
             501：未能成功创建文件夹

## 运营相关

### mac地址上传

通过 App 扫描 Mac 二维码 入库mac。

    url        : deviceMac
    methord    : post
    argument   : pid   设备pid  
				 pass  密码验证      
				 mac   mac地址
				 				 
    example    : /api/mac
    
    return
    if sucess  : {
                     "message": 200
                 }
                 
    if fail    : {
                     "message": 500 
                 }
                
    message: 400:参数错误
             500:插入失败
                 

### server 服务地址

根据 app_id 返回 server 的地址

    url        : appHost
    methord    : get
    argument   : app_id
				 				 
    example    : /api/appHost
    
    return
    if success : {
		             "result": {
		                 "server_login": "serverip:serverport",
		                 "server_api": "serverip:serverport",
		                 "server_mq": "serverip:serverport"
		             },
		             "message": 200
		         }
                 
    if fail    : {
                     "message": 404 
                 }
                
    message: 400:参数错误
             404:查询无结果
