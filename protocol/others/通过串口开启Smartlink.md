## 补充协议 —— 通过串口开启Smartlink


## 协议内容

### 通过串口开启Smartlink 模式
**协议说明：** 设备通过串口下发指令 `0x07` 给wifi模块，wifi模块开启Smartlink模式，随后wifi模块进入Smartlink状态，并与Mcu进行通信，具体的数据包格式（此处略过……，请史工补充具体的细节），用来表示wifi模块配网的过程，以及配网是否成功或者失败的结果。

#### Request(WiFi to Mcu)
<table>
  <thead>
    <tr>
      <th>命令号</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0x07</td>
    </tr>
  </tbody>
</table>

#### Request(MCU to WiFi)

> 请史工补充具体的细节，是否考虑所有smartlink的回复包都带上包头 `0x07` 以供mcu识别处理。