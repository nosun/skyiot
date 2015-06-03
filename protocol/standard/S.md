## Protocol S: Server & App

    Make: nosun
    Date: 2015-02-03
    ver : V1.2
    
##说明

    本接口文档为思佳维云平台移动客户端接口文档，采用RESTFUL理念进行设计,
	使用http协议中的GET,POST,PUT,DELETE等方式对资源进行操作，实现对资源的增删查改。

##错误码说明

    400：请求参数不正确
    200：响应成功
    404：请求无结果
    500：服务器错误

##用户相关
###1、验证用户id

    用户在注册之前，验证用户id是否存在，系统采用电话号码作为用户id，变量为login_id

    url        : login_id
    methord    : get
    argument   : app_id/login_id
    example    :/api/login_id/1/18600364250
    
    return 
    if sucess  : {
                    "message": 200
                 }
                 
    if fail    :{
                    "message": 404
                }

###2、用户注册

	用户注册，在短信验证之后，将用户名密码发过来，进行注册，社会化登录不在此。

    url        : user
    methord    : post
    argument   : login_id  手机号码
                 app_id    app_id
				 login_pwd 用户密码
    example    :/api/user
    
    return 
    if sucess  : {
                    "login_id": "18600364250",
                    "message": 200
                 }
                 
    if fail    :{
                    "login_id": "18600364250",
                    "message": 500
                }

    message: 500：未能成功添加用户
    
###3、用户登录
	
	用户进行登录，这里考虑常规登录，尚未做社会化登录，用户登录之后，给返还一个token，登录之后所有的与服务器之间的交互，均需要发送token过来。

    url        : token
    methord    : post
    argument   : login_id  手机号码
                 app_id    app_id
				 login_pwd 用户密码
    example    :/api/token
    
    return 
    if sucess  : {
                    "token": "*****",
                    "message":200
                 }
                 
    if fail    :{
                    "message":404
                }
    
    message: 404：用户名不存在
             500：用户名密码错误            

###4、用户退出

	用户退出，在客户端做判断即可，删除本地token，然后调回登录页面，如果找不到token，自动跳转到登录页面。

###5、获取用户信息

	用户发送token，获取用户的基本信息，目前可能有部分信息缺少，请补充一下。

    url        : user/$token
    methord    : get
    argument   : $token 登录时服务器给返回的token
    example    :/api/user/$token
    
    return 
    if sucess  : 
				{
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
                 
    if fail    :{
                    "message": 404
                }
                
    message: 404：用户名不存在    

###6、修改用户信息

	修改用户的基本信息。

    url        : user/$token
    methord    : put
    argument   : $token 登录时服务器给返回的token
    example    :/api/user/$token
    需要修改的字段 放在put中，目前支持修改的字段如下：
			user_name
			user_phone
			user_email
			user_img
			user_perfer {json 对象，用来存储用户偏好}

    return 
    if sucess  : {
                    "result": Json,
                    "message": 200
                 }
                 http code :200;
                 
    if fail    :{
                    "message": 500
                }
                 http code :200;
    
    message: 400：参数错误
             500：服务器错误    

###7、修改用户密码

	修改用户密码，将token和用户密码传过来

    url        : passwd
    methord    : put
    argument   : login_pwd     [put 参数]
                 token         [put 参数]
				 login_pwd_old [put 参数]

    example    :/api/passwd
    return 
    if sucess  : {
                    "message": 200
                 }
                 
    if fail    :{
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
    example    :/api/passwd
    return 
    if sucess  : {
                    "message": 200
                 }
                 
    if fail    :{
                    "message":500
                }

    message: 400：参数错误
             500：服务器错误    
                
###9、上传用户头像

	上传用户头像，文件的类型是图片，大小为 不超过512k，字段的名称为file，token需要同时传过来

    url        : file
    methord    : post
    argument   : token 
	        file (file 的字段 name=file)
    example    :/api/file
    return 
    if sucess  : {
					"url": the url of the img
                    "message": 200
                 }
                 
    if fail    :{
                    "message": 500
                }
                
    message: 400：参数错误
             500：服务器错误    
             501：未能成功创建文件夹


##设备管理

###1、查询设备
	设备在smartlink之前的第一步，是看设备是否在数据库中。

    url        : device
    methord    : get
    argument   : 
				 token	
				 sn|mac|id|link 均可以，以键值对形式请求 
				 (P.S.:link为ssid+key的MD5加密串)
    example    :/api/device/token/sn/$sn
                /api/device/token/mac/$mac
                /api/device/token/id/$id
				/api/device/token/link/$link
    return 
    if sucess  : {
                    "result": {
                        "device_id": "100027",
                        "device_mac": "ACCF232C6F38",
                        "device_sn": "999999999",
                        "device_name": "nosun2",
                        "product_id": "4",
                        "device_protocol_ver": "1.0",
                        "device_wifi_version": "1.0|1.0",
                        "device_lock": "0",
                        "device_online": "1",
                        "add_time": "1428639836",
                        "update_time": "1428641905",
                        "device_address": "北京市北京市",
                        "device_data": {
                            "pw": "0",
                            "lc": "0",
                            "io": "0",
                            "": "fa0",
                            "fa": "3",
                            "tm": "000",
                            "fi": "59166",
                            "pm": "5006",
                            "th": "2626"
                        }
                    },
                    "message": 200
                }
    if fail    :{
                    "message": 404
                }
                
    message: 400：参数错误
			 200：已经绑定过
             404：设备不存在

### 2、验证Sn是否合法

验证 Sn号码是否合法。

    url        : deviceSn
    methord    : get
    argument   : app_id        
				 sn   
				 				 				 
    example    :/api/deviceSn/1/100
    
    return 
    
        if sucess  : {
                        "message": 200
                     }
                     
        if fail    :{
                        "message": 404 
                    }
                    
        message: 400:参数错误
                 404:Sn没查到
                 
### 3、获取设备清单

    url        : devices
    methord    : get
    argument   : token [get 参数] 
    example    :/api/devices/token
    return 
    if sucess  : {
                    "result": [
                        {
                            "device_id": "100027",
                            "device_mac": "ACCF232C6F38",
                            "device_sn": "999999999",
                            "device_name": "nosun2",
                            "product_id": "4",
                            "device_protocol_ver": "1.0",
                            "device_wifi_version": "1.0|1.0",
                            "device_lock": "0",
                            "device_online": "1",
                            "add_time": "1428639836",
                            "update_time": "1428641905",
                            "device_address": "北京市北京市",
                            "device_data": {
                                "pw": "0",
                                "lc": "0",
                                "io": "0",
                                "": "fa0",
                                "fa": "3",
                                "tm": "000",
                                "fi": "59166",
                                "pm": "5006",
                                "th": "2626"
                            }
                        },
                        {
                            "device_id": "100028",
                            "device_mac": "ACCF232C7D3A",
                            "device_sn": "1234",
                            "device_name": "airpal",
                            "product_id": "1",
                            "device_protocol_ver": "1.0",
                            "device_wifi_version": "1.0|1.0",
                            "device_lock": "1",
                            "device_online": "1",
                            "add_time": "1428652781",
                            "update_time": "1428653569",
                            "device_address": "北京市北京市",
                            "device_data": {
                                "pw": "1",
                                "lc": "0",
                                "uv": "0",
                                "io": "0",
                                "mo": "0",
                                "fa": "3",
                                "tm": "000",
                                "fi": "58866",
                                "pm": "0853",
                                "th": "2626"
                            },
                            "area_id": "101010100"
                        }
                    ],
                    "message": 200
                }
                                 
    if fail    :{
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
    argument   : $token	需要带上
                
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

    example    :/api/device/device_mac
    return 
    if sucess  : {
					"result": json  
                    "message": 200
                 }
                 
    if fail    :{
                    "message": 500
                }
    message: 400：参数错误
             500：服务器错误    


### 5、绑定设备
	建立用户与设备的绑定关系，,目前尚未做设备和用户已经绑定的关系进行判断

    url        : bind
    methord    : post
    argument   : $device_id || $device_mac
				 $token 
    example    :/api/bind
    return 
    if sucess  : {
                    "message": 200
                 }
                 
    if fail    :{
                    "message": 500
                }
                
    message: 400：参数错误
             404:未找到该设备，请重试~
             500：服务器错误    
                
### 6、解除绑定
	解除用户与设备的绑定关系,目前尚未做设备和用户已经绑定的关系进行判断

    url        : bind
    methord    : delete
    argument   : device_id [delete 参数] 1_2_4
				 token     [delete 参数]
    example    :/api/bind/token/device_id
    return 
    if sucess  : {
                    "message": 200
                 }
                 
    if fail    :{
                    "message": 500 
                }
                
    message: 400：参数错误
             500：服务器错误

### 7、发送指令

	App 通过 http post 方式向设备发送指令，控制设备

    url        : cmd
    methord    : post
    argument   : device_id  
				 token        
				 commandv   {"sn":1,"cmd":"download","data":["pw::1"]}
				 
    example    :/api/cmd
    
    return 
    
    if sucess  : {
                    "message": 200
                 }
                 
    if fail    :{
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

    {
        "result": {
            "temperature": "15",
            "humidity": "32",
            "pm": "114"
        },
        "message": 200
    }
                
    message: 200：成功
			 401：天气查询失败
			 402：pm查询失败
             404：数据不存在
             400：参数错误

##运营相关

###1、根据app_id 下载最新的app version
    url        : app
    methord    : get
    argument   : app_id [get 参数] 
    example    :/api/app
    return 
    if sucess  : {
                    "result": json
                    "message": 200
                 }
                 
    if fail    :{
                    "message": 404
                }
                
    message: 400：参数错误
             404：没有结果 
             
###2、调取厂家信息

    url        : company
    methord    : get
    argument   : company_id [get 参数] 
    example    :/api/company
    return 
    if sucess  : {
                    "result": json
                    "message": 200
                 }
                 
    if fail    :{
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
    example    :/api/testSpeed
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
    example    :/api/testHttp
    return 
    if sucess  : {
                    "result": ok
                 }
                 http code

- log上传

    url        : log
    methord    : post formdata
    argument   : 
                 token  
                 file   log文件
    require    : fileSize limit: 1024k
                 fileType limit: *.log
     
    example    :/api/log

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
				 				 
    example    :/api/mac
    
    return 
    
        if sucess  : {
                        "message": 200
                     }
                     
        if fail    :{
                        "message": 500 
                    }
                    
        message: 400:参数错误
                 500:插入失败
                 

### server 服务地址

根据 app_id 返回 server 的地址

    url        : appHost
    methord    : get
    argument   : app_id
				 				 
    example    :/api/appHost
    
    return 
    
        if success
        {
            "result": {
                "server_login": "serverip:serverport",
                "server_api": "serverip:serverport",
                "server_mq": "serverip:serverport"
            },
            "message": 200
        }
                     
        if fail    :{
                        "message": 404 
                    }
                    
        message: 400:参数错误
                 404:查询无结果
