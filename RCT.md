## RCT消息
### State消息
#### 定义
State消息表示无人机的状态信息，主要包括飞控（Flight Controller）、电池（Battery）、万向节（Gimbal）、RTK等部件的状态信息，由Android端以一定频率（通常是10Hz）向Mqtt服务器发送。使用Protobuf做为消息数据序列化的工具。
#### 消息详情描述
1. FlightController
   * 飞控的状态信息主要包括：飞行模式、飞行状态、无人机的维度、经度和高度、无人机的Pitch/Roll/Yaw角度、飞行时间、GPS状态、剩余飞行时间等
   * 主题：RCT/{SerialNumber}/State/FlightController
   * 其详细信息可以查看对应的[FlightController.proto文件](proto/state/FlightController.proto)
2. Battery
   * 电池的状态信息主要包括：剩余电量，当前电压和电流
   * 主题：RCT/{SerialNumber}/State/Battery
   * 其详细信息可以查看对应的[Battery.proto文件](proto/state/Battery.proto)
3. Gimbal
   * 万向节的状态信息主要包括：万向节模式以及万向节的Pitch/Roll/Yaw角度
   * 主题：RCT/{SerialNumber}/State/Gimbal
   * 其详细信息可以查看对应的[Gimbal.proto文件](proto/state/Gimbal.proto)
4. RTK
   * RTK的状态信息主要包括：RTK的连接信息、RTK的使用信息以及Heading Solution
   * 主题：CT/{SerialNumber}/State/RTK
   * 其详细信息可以查看对应的[RTK.proto文件](proto/state/RTK.proto)
5. 多媒体文件上传状态消息
   * 在航点飞行任务完成后，App从摄像机的SD卡将本次飞行任务中拍摄的多媒体文件下载到本地，然后上传到Ftp服务器。在将媒体文件上传到Ftp服务器后，发送一个消息来通知多媒体文件存放在Ftp服务器上的路径，如果上传没有成功，即没有收到消息或者在Ftp服务器上没有相应的文件（文件在Ftp服务器上存放的位置为："/WaypointMission/{WaypointMissionId}/{任务开始时间：格式为yyyy_MM_dd_HH_mm_ss}），可以到平板或者手机存储中的位置（/mnt/internal_sd/WaypointMission/{WaypointMissionId}/{任务开始时间：格式为yyyy_MM_dd_HH_mm_ss}）或者相机的SD卡上查找
   * 主题：RCT/{SerialNumber}/State/FtpUpload
   * 回应内容的说明：

      名称|类型|必填|说明
      ---|:--:|:---:|---
      resStr|string|是|ftp路径

### Request/Response消息
#### 定义
Request消息表示地面站向Android端下发的请求信息，主要包括对无人机参数和配置信息的查询指令、无人机的基本控制指令（比如起飞、降落、自动返航等指令）、云台的控制指令、相机的控制指令、无人机任务指令（包括航点任务、**任务等）等。该类信息一般由地面站发向Mqtt服务器，再由Android端从Mqtt服务器接收。在Request消息的请求参数中，默认都会有一个参数resTopic，来表示回应(Res)消息对应的主题,如果用户不填则表示不需要Res消息。使用JSON作为数据交换的工具。
   * 默认请求参数说明：

      名称|类型|必填|说明
      ---|:--:|:---:|---
      resTopic|string|否（但是必须要有该字段）|回应(Res)消息对应的主题,不填则表示不需要Res消息

Response消息是对Request消息的回应消息，消息的主题由Request消息中的默认请求参数resTopic来指定。默认都有一个参数code，来表示回应的状态。使用JSON作为数据交换的工具。
   * 默认回应参数说明：

      名称|类型|必填|说明
      ---|:--:|:---:|---
      code|int|是|回应状态码

   * 回应状态码说明：

      名称|状态码|说明
      ---|:---:|--- 
      CODE_SUCCESS|0|接收成功
      CODE_FAIL|111111|接收失败
      CODE_FLY_TO_STOPPED|111119|FlyTo任务停止
      CODE_FLY_TO_FINISHED|111118|FlyTo任务正常执行结束
      CODE_FLY_TO_RUNNING|111117|FlyTo任务正在运行，在FlyTo任务执行过程中，再次收到另外一个FlyTo任务时，会停止当前任务的执行，此时会返回该状态码，接着会返回CODE_FLY_TO_STOPPED，说明当前任务停止成功了，但是另外一个FlyTo任务不会执行，需要在任务停止成功后，再次发送才能执行
      CODE_FLY_TO_START_ERROR|111116|FlyTo任务开始错误
      CODE_FLY_TO_STOP_ERROR|111115|FlyTo任务停止错误
      CODE_FLY_TO_PROCESSED|111114|FlyTo任务执行中，在FlyTo任务执行过程中，会返回多个该状态码，如果在收到CODE_FLY_TO_STARTED并且接着收到CODE_FLY_TO_PROCESSED，说明任务开始成功了
      CODE_FLY_TO_STARTED|111110|FlyTo任务开始执行


#### 消息详情描述
1. 无人机信息查询指令
   * 查询无人机(Aircraft)、飞控(FlightController)、相机(Camera)等部件的参数和配置信息。
   * 查询无人机信息主题：RCT/{SerialNumber}/Req/Aircraft
      * 请求参数说明：使用默认请求参数
      * 回应参数说明：

         名称|类型|必填|说明
         ---|:--:|:---:|---
         code|int|是|回应状态码
         name|string|是|无人机的名字
         firmwarePackageVersion|strings|是|固件包版本

   * 查询飞控信息主题：RCT/{SerialNumber}/Req/FlightController
      * 请求参数说明：使用默认请求参数
      * 回应参数说明： 

         名称|类型|必填|说明
         ---|:--:|:---:|---
         code|int|是|回应状态码
         isHomeLocationSet|boolean|是|是否设置了Home点
         homeLocationLatitude|double|是|Home点的纬度
         homeLocationLongitude|double|是|Home点的经度
         maxFlightHeight|int|是|最大飞行高度
         maxFlightRadius|int|是|最大飞行半径
      
   * 查询相机信息主题：RCT/{SerialNumber}/Req/Camera
      * 请求参数说明：使用默认请求参数
      * 回应参数说明： 

         名称|类型|必填|说明
         ---|:--:|:---:|---
         code|int|是|回应状态码
         很多|

      
2. 无人机基本控制指令
   * 包括电机加锁(Lock)、电机解锁(Unlock)、设置自动返航高度(SetReturnHeight)、自动起飞(TakeOff)、自动降落(Land)、取消降落（CancelLanding）、确定降落（ConfirmLanding）、即点即飞(FlyTo)、自动返航(ReturnHome)等指令
   * 电机加锁主题：RCT/{SerialNumber}/Req/Lock
      * 请求参数说明：使用默认请求参数
      * 回应参数说明：使用默认回应参数
   * 电机解锁主题：RCT/{SerialNumber}/Req/Unlock
      * 请求参数说明：使用默认请求参数
      * 回应参数说明：使用默认回应参数
   * 设置自动返航高度主题:RCT/{SerialNumber}/Req/Unlock/SetReturnHeight
      * 请求参数说明：

         名称|类型|必填|说明
         ---|:--:|:---:|---
         resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
         height|float|是|指定返航高度，The valid range for the altitude is from 20m to 500m.
      * 回应参数说明：

         名称|类型|必填|说明
         ---|:--:|:--:|---
         code|int|是|回应状态码
         height|float|是|当前返航高度
   * 自动起飞主题：RCT/{SerialNumber}/Req/TakeOff
      * 请求参数说明：使用默认请求参数
      * 回应参数说明：使用默认回应参数
   * 自动降落主题：RCT/{SerialNumber}/Req/Land
      * 请求参数说明：使用默认请求参数
      * 回应参数说明：使用默认回应参数
   * 自动降落主题：RCT/{SerialNumber}/Req/CancelLanding
      * 请求参数说明：使用默认请求参数
      * 回应参数说明：使用默认回应参数
   * 确认降落主题：RCT/{SerialNumber}/Req/ConfirmLanding
      * 请求参数说明：使用默认请求参数
      * 回应参数说明：使用默认回应参数
   * 即点即飞主题：RCT/{SerialNumber}/Req/FlyTo
      * 请求参数说明：

         名称|类型|必填|说明
         ---|:--:|:---:|---
         resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
         latitude|double|是|指定经度
         longitude|double|是|指定纬度
      * 回应参数说明：使用默认回应参数
   * 自动返航主题：RCT/{SerialNumber}/Req/ReturnHome
      * 请求参数说明：使用默认请求参数
      * 回应参数说明：使用默认回应参数

3. 云台控制指令
   * 包括云台的Pitch/Roll/Yaw三个角度的设置
   * 主题:RCT/{SerialNumber}/Mobile/Req/GimbalControl
   * 请求参数说明：

      名称|类型|必填|说明
      ---|:--:|:---:|---
      resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
      mode|enum|是|万向节转动的模式，3种：RELATIVE_ANGLE（ relative to the current angle）、ABSOLUTE_ANGLE（ relative to the aircraft heading）、SPEED（Rotate the gimbal's pitch, roll, and yaw in SPEED Mode. The direction can either be set to clockwise or counter-clockwise.）
      pitch|float|是|指定云台的Pitch角度
      roll|float|是|指定云台的Roll角度
      yaw|float|是|指定云台的yaw角度
      time|double|是| the completion time in seconds to complete an action to control the gimbal. 
   * 回应参数说明：使用默认回应参数
4. 相机控制指令
   * 包括对设置相机参数(CameraSetting)、切换相机模式（CameraMode）、拍照(TakePhoto)、开始录制视频(StartRecordVideo)、停止录制视频(StopRecordVideo)等指令
   * 相机参数设置主题：RCT/{SerialNumber}/Mobile/Req/CameraSetting
      * 请求参数说明：

         名称|类型|必填|说明
         ---|:--:|:---:|---
         resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
         aperture|enum|是|相机的光圈大小
         等等|
      * 回应参数说明：使用默认回应参数
   * 切换相机模式：RCT/{SerialNumber}/Mobile/Req/CameraMode
      * 请求参数说明：

         名称|类型|必填|说明
         ---|:--:|:---:|---
         resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
         mode|int|是|相机模式，拍照或者录像
      * 回应参数说明：

         名称|类型|必填|说明
         ---|:--:|:--:|---
         code|int|是|回应状态码
         mode|int|是|相机模式，拍照或者录像
   
   * 相机放大缩小重置主题：RCT/{SerialNumber}/Mobile/Req/{CammandType Name}，包括放大（ZoomIn）、缩小（ZoomOut）、重置（ZoomReset）
      * 请求参数说明：使用默认请求参数
      * 回应参数说明：

         名称|类型|必填|说明
         ---|:--:|:--:|---
         code|int|是|回应状态码
         magnification|int|是|放大倍数
         
   * 相机的其他控制主题：RCT/{SerialNumber}/Mobile/Req/{CammandType Name}，包括拍照(TakePhoto)、开始录制视频(StartRecordVideo)、停止录制视频(StopRecordVideo)、等
      * 请求参数说明：使用默认请求参数
      * 回应参数说明：使用默认回应参数
  
5. 无人机任务指令
   * 支持的任务包括航点任务（Waypoint Mission）、热点环绕任务（Hotpoint Mission）等。
   * 航点任务
      * 包括航点任务的装订(WaypointLoad)、上传(WaypointUpload)、启动(WaypointStart)、暂停(WaypointPause)、恢复(WaypointResume)、停止(WaypointStop)等指令
      * 航点任务的装订主题：RCT/{SerialNumber}/Mobile/Req/WaypointLoad
         * 请求参数说明：
            >WaypointMission
            >
            名称|类型|必填|说明
            ---|:--:|:---:|---
            resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
            id|string|是|任务的id
            missionName|string|是|任务的名称
            autoFlightSpeed|float|是|任务的自动飞行速度,不大于最大飞行速度
            maxFlightSpeed|float|是|任务的最大飞行速度，不大于15m/s
            missionAltitude|float|是|任务的飞行高度
            headingMode|enum|是|飞行器偏航角，包含AUTO(沿航线方向), USING_INITIAL_DIRECTION(使用最初的方向), CONTROL_BY_REMOTE_CONTROLLER(手动控制),        USING_WAYPOINT_HEADING, TOWARD_POINT_OF_INTEREST(依照每个航点设置)
            gimbalPitchRotationEnabled|boolean|是|if the gimbal pitch rotation can be controlled during the waypoint mission. When true, gimbalPitch in Waypoint is used to control gimbal pitch.
            finishedAction|enum|是|NO_ACTION, GO_HOME, AUTO_LAND, GO_FIRST_WAYPOINT, CONTINUE_UNTIL_END
            waypoints|Waypoint|是|任务中的航点，使用WayPoint列表来表示
            >Waypoint

            名称|类型|必填|说明
            ---|:--:|:---:|---
            latitude|double|是|航点的纵坐标
            longitude|double|是|航点的横坐标
            usingMissionAltitude|boolean|是|是否使用任务的飞行高度作为航点的飞行高度
            altitude|float|是|如果using_mission_altitude值为true，则航点的飞行高度等于mission_altitude；若为false，则航点的飞行高度等于该值
            heading|int|否|无人机的朝向，如果“飞行器偏航角”选择“依照每个航点设置”起作用
            turnMode|enum|否|无人机的转向方式，如果“飞行器偏航角”选择“依照每个航点设置”起作用，其值包含CLOCKWISE和COUNTER_CLOCKWISE
            gimbalPitch|float|否|根据gimbalPitchRotationEnabled字段来确定需不需要填写 范围-90~0
            shootPhotoTimeInterval|float|否|执行拍照动作时间间隔，当照片格式为JEPG时，范围为[2,6000]秒，当为RAW时，范围为[10,6000]秒
            shootPhotoDistanceInterval|float|否|执行拍照动作距离间隔，当照片格式为JEPG时，范围为[2,6000]*autoFlightSpeed米，当为RAW时，范围为[10,6000]*autoFlightSpeed米
            actions|WaypointAction|否|无人机到达航点时做的动作
            >WaypointAction
            >
            名称|类型|必填|说明
            ---|:--:|:---:|---
            actionType|enum|是|动作类型，包含STAY（actionParam表示悬停的时间，单位为毫秒，范围为[0, 32767]）, START_TAKE_PHOTO, START_RECORD, STOP_RECORD, ROTATE_AIRCRAFT（actionParam表示无人机旋转的yaw角度，范围为[-180, 180]）, GIMBAL_PITCH（actionParam表示云台的pitch角度，范围为 [-90, 0] ）, CAMERA_ZOOM, CAMERA_FOCUS, FINE_TUNE_GIMBAL_PITCH, RESET_GIMBAL_YAW
            actionParam|int|否|动作的参数，根据动作类型来使用
         * 回应参数说明：使用默认回应参数
         * Warning: 'speed', shootPhotoTimeInterval and shootPhotoDistanceInterval relate to behavior between this waypoint and the next waypoint in the mission. In comparison, turnMode, altitude and heading relate to behavior between the last waypoint and this waypoint in the waypoint mission.
      * 航点任务的其他控制主题：RCT/{SerialNumber}/Mobile/Req/{CammandType Name}，包括上传(WaypointUpload)、启动(WaypointStart)、暂停(WaypointPause)、恢复(WaypointResume)、停止(WaypointStop)等
         * 请求参数说明：使用默认请求参数
         * 回应参数说明：使用默认回应参数
   * 热点环绕任务
      * 包括热点环绕任务的启动(HotpointStart)、暂停(HotpointPause)、恢复(HotpointResume)、停止(HotpointStop)等指令
      * 热点环绕任务的装订主题：RCT/{SerialNumber}/Mobile/Req/HotpointLoad
         * 请求参数说明：
         
            名称|类型|必填|说明
            ---|:--:|:---:|---
            resTopic|string|否|回应(Res)消息对应的主题,不填则表示不需要Res消息
            latitude|double|是|纬度
            longitude|double|是|经度
            startPoint|HotpointStartPoint|是|航点任务的开始点，枚举，NORTH,    //Start from the North.SOUTH,    //Start from the South. WEST,    //Start from the West.EAST,    //Start from the East. NEAREST,    //Start the circle surrounding the hotpoint at the nearest point on the circle to the aircraft's current location.
            heading|HotpointHeading|是|飞行方向，枚举，ALONG_CIRCLE_LOOKING_FORWARDS,    //Heading is in the forward direction of travel along the circular path. ALONG_CIRCLE_LOOKING_BACKWARDS, //Heading is in the backward direction of travel along the circular path. TOWARDS_HOT_POINT,    //Heading is toward the hotpoint.AWAY_FROM_HOT_POINT,    //Heading of the aircraft is looking away from the hotpoint. It is in the direction of the vector defined from the hotpoint to the aircraft.CONTROLLED_BY_REMOTE_CONT, //ROLLER	Heading is controlled by the remote controller. USING_INITIAL_HEADING,  //The heading remains the same as the heading of the aircraft when the mission started.
            altitude|double|是|高度，[5,500]
            radius|double|是|半径，[5,500]
            angularVelocity|float|是|环绕速度，默认20度/s
            clockwise|boolean|是|环绕方向，true为顺时针，false为逆时针
         * 回应参数说明：使用默认回应参数
      * 热点环绕任务的其他控制主题：RCT/{SerialNumber}/Mobile/Req/{CammandType Name}，暂停(HotpointPause)、恢复(HotpointResume)、停止(HotpointStop)等
         * 请求参数说明：使用默认请求参数
         * 回应参数说明：使用默认回应参数




