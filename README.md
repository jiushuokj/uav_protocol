## 1. 系统组成及命名规范
系统包括无人机机载信息处理单元、遥控器、地面站、应用客户端，各部分功能如下：
* 机载信息处理单元：
  * 下发飞机状态
  * 通过WEBRTC推送视频
  * 目标识别与环境理解
  * 飞机控制
* 遥控器（远程操控终端）
  * 无人机操控
  * 无人机视频展现与转发
  * 无人机状态展现与转发
  * 无人机控制转发
* 地面站
  * 消息中转中心
  * 视频服务器
  * 实时制图服务器
* 业务信息系统
  * 接收飞机状态信息，综合展现
  * 任务规划，远程控制
  * 视频监控
  
各部分简称如下：
* 机载信息处理单元（APU，Airborne processing unit）
* 远程操控终端（RCT，Remote control terminal），俗称遥控器
* 地面站（GCS，Ground Control System）
* 指挥信息系统（CIS，Command information system）

## 2. 协议概述
>使用MQTT协议作为通讯协议。其中，在RCT消息中，使用ProtoBuf作为State数据交换的技术，使用JSON作为Request/Response数据交换的格式。

>主题名称的构成 {模块}/{SerialNumber}/{State|Req|Res}/{Product/Component/CommandType Name}

* 各模块由自己唯一的代号：
  * RCT: andriod平板APP
  * APU: 机载智能处理程序
  * CIS: 无人机指控系统
  * RTC: GCS中实时视频推送服务
  * MAPPER: GCS中离线制图服务
  * RTMAPPER:GCS中实时制图服务
  * DETECTER:GCS中实时目标检测服务

* 各模块的实体必须有唯一的编号SerialNumber
  * RCT，SerialNumer采用飞机编号，表示无人机飞行控制器的序列号，用于作为无人机的标识
  * APU，SerialNumer采用飞机编号，表示无人机飞行控制器的序列号，用于作为无人机的标识
  * GCS[各具体服务]，SerialNumer采用GCS设备编号
  * CIS，SerialNumer由应用系统自己编号
* State|Req|Res: 表示消息的类型。其中，State表示从无人机发出的状态消息，用于实时获取无人机的状态。Req表示从外部发往无人机的请求信息，主要用于获取和设置无人机的参数和配置信息，以及对无人机的控制。Res表示对Req消息的回应信息。
* Product/Component/CommandType Name：表示消息对应的无人机的部位或者命令类型。
* 各部分启动后，必须订阅如下mqtt 主题
  * {模块}/编号/Req/#: 通过该主题接收下达的命令
    * 例如 收到 RCT/飞机编号/Req/Takeoff 的主题消息后，表示请求飞机起飞 
  * {模块}/编号/Res/#: 通过该主题接收其他部分的回复
 
## 3. [飞行状态及飞行控制](RCT.md)

## 4. [实时视频RTC](RTC.md)

## 5. [离线制图服务](MAPPER.md)

## 6. [实时制图服务](RTMAPPER.md)





