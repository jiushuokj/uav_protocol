## 离线制图服务(MAPPER)

### State消息
#### 定义
State消息表示制图软件的状态信息，由MAPPER端以一定频率（通常是0.5Hz）向Mqtt服务器发送。使用JSON做为消息数据序列化的工具。
#### 消息详情描述
1. Progress
   * 制图进度
   * 主题：MAPPER/0/State/Progress
   * 详细信息:

      名称|类型|必填|说明
      ---|:--:|:---:|---
      taskId|string|是|任务ID
      execTime|string|是|任务执行时间
      running|bool|是|任务是否在执行
      succeed|bool|是|任务是否执行成功了
      rate|int|是|已完成比例，100表示完成，0表示未开始
      passTime|int|是|已用时，单位s

   
### Request/Response消息
#### 定义
Request消息表示CIS向MAPPER端下发的请求信息。在Request消息的请求参数中，默认都会有一个参数resTopicName，来表示回应(Res)消息对应的主题,如果用户不填则表示不需要Res消息。使用JSON作为数据交换的工具。
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
1. 启动离线制图
    * CIS向MAPPER发起离线制图请求
    * 该服务由MAPPER提供，信息流向为 CIS-->MAPPER
    * 主题：MAPPER/0/Req/Start
    * 请求参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
     taskId|string|是|该制图任务的ID，需要制图的照片放在/WaypointMission/taskId/execTime 下边
     execTime|string|是|该制图任务的执行时间，，需要制图的照片放在/WaypointMission/taskId/execTime 下边


    * 回应参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     code|int|是|回应状态码
     
     状态码取值|说明
     ---|---
     0|开始启动执行
     1|已有任务在执行，无法执行本次任务
     2|无法找到照片
     
2. 取消离线制图任务
    * CIS向MAPPER发起离线制图请求
    * 该服务由MAPPER提供，信息流向为 CIS-->MAPPER
    * 主题：MAPPER/0/Req/Stop
    * 请求参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
     taskId|string|是|该制图任务的ID

    * 回应参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     code|int|是|回应状态码
     
     状态码取值|说明
     ---|---
     0|成功停止

3. 获取制图任务结果
    * 查询MAPPER生成的制图结果
    * 该服务由MAPPER提供，信息流向为 CIS-->MAPPER
    * 主题：MAPPER/0/Req/GetMap
    * 请求参数说明:
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
     taskId|string|是|该制图任务的ID


    * 回应参数说明： 
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     code|int|是|回应状态码
     taskId|string|是|该制图任务的ID
     domTile|string|是|DOM瓦片数据路径
     dsmTile|string|是|DSM瓦片数据路径
     domTiff|string|是|DOMTIFF格式数据路径
     dsmTile|string|是|DSMTIFF格式数据路径
     pose|string|是|pose点数据路径
     

