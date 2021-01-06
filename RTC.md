## 实时视频(RTC)

### Request/Response消息
#### 定义
Request消息表示CIS/APU向GCS端下发的请求信息。在Request消息的请求参数中，默认都会有一个参数resTopicName，来表示回应(Res)消息对应的主题,如果用户不填则表示不需要Res消息。使用JSON作为数据交换的工具。
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
1. 视频注册
    * APU向GCS注册视频源
    * 该服务由GCS提供，信息流向为 APU-->GCS
    * 主题：RTC/0/Req/Reg
    * 请求参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
     apuId|string|是|提供视频源的apuId, 飞控唯一编号

    * 回应参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     code|int|是|回应状态码

2. 查询当前的视频流状态
    * 查询当前GCS支持的视频源
    * 该服务由GCS提供，信息流 CIS-->GCS
    * 主题：RTC/0/Req/Ls
    * 请求参数说明：使用默认请求参数
    * 回应参数说明： 
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     code|int|是|回应状态码
     rtc|RTC结构 数组|是|当前GCS已经收集到的视频源
     
     > 其中RTC结构

     名称|类型|必填|说明
     ---|:--:|:---:|---
     apuId|string|是|提供视频源的apuId，若为'UDP'表示该视频流通过UDP6666端口接收的来自遥控器(RCT)的视频流
     recving|bool|是|该视频流是否在接收中
     recording|bool|是|该视频流的关键帧是否在保存中
     clientNumber|int|是|有几个客户端在接收该视频流

3. 请求播放某视频   
    * 请求播放该视频流
    * GCS和APU分别提供该能力:
        * GCS
            * 主题 RTC/0/Req/Play
            * 信息流向 CIS-->GCS 
        * APU
            * 主题 APU/使用apuId/Req/Rtc/Play
            * 信息流向 CIS-->APU 或者 GCS-->APU
    * 请求参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
     apuId|string|是|希望播放的视频流，使用apuId标示，若为'UDP'表示该视频流通过UDP6666端口接收的来自遥控器(RCT)的视频流
     cisId|string|是|视频接收方唯一标示符，即cis的Id
     offer|string|是|视频请求方的SDP信息
     ice|string|是|视频请求方的ICE信息
     sdpMid|string|是|视频请求方的sdpMid
     sdpMlineIndex|int|是|视频请求方的sdpMlineIndex
    
    * 回应参数说明： 
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     code|int|是|回应状态码
     apuId|string|是|提供该视频流的apuId，若为'UDP'表示该视频流通过UDP6666端口接收的来自遥控器(RCT)的视频流
     answer|string|是|视频提供方的SDP信息
     ice|string|是|视频发送方的ICE信息
     sdpMid|string|是|视频提供方的sdpMid
     sdpMlineIndex|int|是|视频提供方的sdpMlineIndex

4. 请求停止播放某视频   
    * 请求停止播放该视频流
    * GCS和APU分别提供该能力
        * GCS
            * 主题 RTC/0/Req/Stop
            * 信息流向 CIS-->GCS 
        * APU
            * 主题 APU/使用apuId/Req/Rtc/Stop
            * 信息流向 CIS-->APU 或者 GCS-->APU
    * 请求参数说明：
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
     apuId|string|是|希望停止播放的视频流，使用apuId标示,若为'UDP'表示该视频流通过UDP6666端口接收的来自遥控器(RCT)的视频流
     cisId|string|是|视频接收方唯一标示符，即cis的Id

    * 回应参数说明： 
    
     名称|类型|必填|说明
     ---|:--:|:---:|---
     code|int|是|回应状态码




