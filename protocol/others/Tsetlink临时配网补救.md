# T口协议 增加 上传SSID

### 协议说明

为解决配网成功率的问题，临时增加SSID信息上报方案，App通过SSID信息查询获取设备信息。

### 设置网络

**Request (Dev to Server):**

    {
        "cmd": "setlink",
        "mac": "AABBCCDDEEFF",
        "key": "**************************************"
    }

> 说明：key 指代 wifi的ssid 连接 ssid 密码做 md5 加密。


