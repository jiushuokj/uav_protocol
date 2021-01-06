## 实时制图服务(RTMAPPER)

### State消息
#### 定义
State消息表示实时制图软件的状态信息，由RTMAPPER端以一定频率发送，10秒检测一次是否有新的瓦片形成，当有新的地图瓦片形成后，会自动向Mqtt服务器发送。使用JSON做为消息数据序列化的工具。
#### 消息详情描述
1. Progress
   * 制图进度
   * 主题：RTMAPPER/0/State/Progress
   * 详细信息:

      名称|类型|必填|说明
      ---|:--:|:---:|---
      taskId|string|是|任务ID，该ID由用户启动实时制图时传入
      running|bool|是|任务在实时制图中
      level|int|是|当前实时制图瓦片级别
      renew|bool|是|是否需要更新，为true表示需要更新地图，否则不用更新

   
### Request/Response消息
#### 定义
Request消息表示CIS向RTMAPPER端下发的请求信息。在Request消息的请求参数中，默认都会有一个参数resTopicName，来表示回应(Res)消息对应的主题,如果用户不填则表示不需要Res消息。使用JSON作为数据交换的工具。
   * 默认请求参数说明：

      名称|类型|必填|说明
      ---|:--:|:---:|---
      resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息

Response消息是对Request消息的回应消息，消息的主题由Request消息中的默认请求参数resTopicName来指定。默认都有一个参数code，来表示回应的状态。使用JSON作为数据交换的工具。
   * 默认回应参数说明：

      名称|类型|必填|说明
      ---|:--:|:---:|---
      code|int|是|回应状态码

#### 消息详情描述
1. 启动实时制图
    * CIS向RTMAPPER发起离线制图请求
    * 该服务由RTMAPPER提供，信息流向为 CIS-->MAPPER
    * 主题：RTMAPPER/0/Req/Start
    * 请求参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
     taskId|string|是|该制图任务的ID，格式为'当前飞行任务ID'_'当前任务开始执行时时间'_'RT'
     flyDistance|double|是|本次飞行最大飞行半径，确保飞机距离home点不超出该半径，填0，采用默认2000米
     maxTileLevel|int|是|本次瓦片最大等级，填0，采用默认的20级
     useCere|bool|是|是否启用非线性优化，为true表示启用，默认启用

    * 回应参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     code|int|是|回应状态码
     
     状态码取值|说明
     ---|---
     0|开始启动执行
     1|已有任务在执行，无法执行本次任务
     2|无法获取飞机视频流
     3|输入参数无效
     
2. 停止实时制图任务
    * CIS向RTMAPPER发起离线制图请求
    * 该服务由RTMAPPER提供，信息流向为 CIS-->MAPPER
    * 主题：RTMAPPER/0/Req/Stop
    * 请求参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
     taskId|string|是|该制图任务的ID,格式为'当前飞行任务ID'_'当前任务开始执行时时间'_'RT'

    * 回应参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     code|int|是|回应状态码
     
     状态码取值|说明
     ---|---
     0|成功停止