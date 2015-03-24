#物联网云平台API V2.0.0

    Make: nosun
    Date: 2015-02-03
    ver :  V2.0.0
    
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
    argument   : login_id
    example    : http://c1.skyware.com.cn/api/login_id/18600364250
    
    return 
    if sucess  : {
                    "login_id": "18600364250",
                    "message": 200
                 }
                 
    if fail    :{
                    "login_id": "18600364250",
                    "message": 404
                }

###2、用户注册
	用户注册，在短信验证之后，将用户名密码发过来，进行注册，社会化登录不在此。

    url        : user
    methord    : post
    argument   : login_id  手机号码
				 login_pwd 用户密码
    example    : http://c1.skyware.com.cn/api/user
    
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
				 login_pwd 用户密码
    example    : http://c1.skyware.com.cn/api/token
    
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
    example    : http://c1.skyware.com.cn/api/user/$token
    
    return 
    if sucess  : 
				{
				    "result": [
				        {
				            "login_id": "18600364250",
				            "user_name": "nosun",
				            "user_img": "http://c1.skyware.com.cn/uploads/201502/99e521bfc7d2851f0f6cc7f7fcdc85bb.jpg",
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
    example    : http://c1.skyware.com.cn/api/user/$token
    需要修改的字段 放在put中，目前支持修改的字段如下：
			user_name
			user_phone
			user_email
			user_img
			notice_filter
			notice_pm
			notice_pm_value

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

    example    : http://c1.skyware.com.cn/api/passwd
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
    example    : http://c1.skyware.com.cn/api/passwd
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
				 file (file 的字段 name=avatar)
    example    : http://c1.skyware.com.cn/api/file
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

###1、扫描并查询设备

	设备在smartlink之前的第一步，是看设备是否在数据库中。

    url        : device
    methord    : get
    argument   : 
				 token	
				 device 
    example    : http://c1.skyware.com.cn/api/device/token/device_sn
    return 
    if sucess  : {
					"result": json
                    "message": 200
                 }
                 
    if fail    :{
                    "message": 404
                }
                
    message: 400：参数错误
			 500：已经绑定过
             404：设备不存在

###2、获取设备清单

    url        : devices
    methord    : get
    argument   : token [get 参数] 
    example    : http://c1.skyware.com.cn/api/devices/token
    return 
    if sucess  : {
                    "result": json
                    "message": 200
                 }
                 
    if fail    :{
                    "message": 500
                }
                
    message: 400：参数错误
             404：没有结果
             

###3、更新设备信息
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

    example    : http://c1.skyware.com.cn/api/device/device_mac
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


###4、绑定设备
	建立用户与设备的绑定关系，,目前尚未做设备和用户已经绑定的关系进行判断


    url        : bind
    methord    : post
    argument   : $device_id
				 $token 
    example    : http://c1.skyware.com.cn/api/bind
    return 
    if sucess  : {
                    "message": 200
                 }
                 
    if fail    :{
                    "message": 500
                }
                
    message: 400：参数错误
             500：服务器错误    
                
###5、解除绑定
	解除用户与设备的绑定关系,目前尚未做设备和用户已经绑定的关系进行判断

    url        : bind
    methord    : delete
    argument   : device_id [delete 参数] 1_2_4
				 token     [delete 参数]
    example    : http://c1.skyware.com.cn/api/bind/token/device_id
    return 
    if sucess  : {
                    "message": 200
                 }
                 
    if fail    :{
                    "message": 500 
                }
                
    message: 400：参数错误
             500：服务器错误

       
##运营相关

###1、根据app_id 下载最新的app version
    url        : app
    methord    : get
    argument   : app_id [get 参数] 
    example    : http://c1.skyware.com.cn/api/app
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
    example    : http://c1.skyware.com.cn/api/company
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
             