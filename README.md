# 游云 iOS MediaPlusSDK 开发指南

## 简介

游云通讯VoIP

## 集成说明

在使用`MediaPlusSDK`之前，请前往 [官方网站](http://www.17youyun.com) 注册开发者帐号。注册时，您需要提供真实的邮箱和手机号，以方便我们向您发送重要通知并在紧急时刻能够联系到您。如果您没有提供正确可用的邮箱和手机号，我们随时可能关闭您的应用。

注册了开发者账号之后，在进行开发 App 之前，您需要请前往 [开发者控制台](http://www.17youyun.com) 创建应用。您创建完应用之后，在您的应用中，会自动创建两套的环境，即：开发环境和生产环境。创建应用成功后会生成对应开发环境的App唯一的 `ClientID`和 `Secret`。

### 包含文件

- include/MediaPlusSDK.h
- include/MediaPlusSDK+IM.h
- include/MediaPlusSDK+VoIP.h
- include/MediaPlusSDKDelegate.h
- include/WChatCommon.h
- include/public.der
- include/Resource/hold.wav // 等待提示音
- include/Resource/msg.caf // 提醒
- include/Resource/msg.wav // 提醒
- include/Resource/ring.caf // 来电
- include/Resource/ring.wav // 来电
- include/Resource/ringback.wav // 播出等待提示音
- libMediaPlusSDK.a

### 要求

- iOS 7.0 +

- 依赖库

  如果您使用的是 Xcode 6.x 版本，则需要将上面的动态库 *.tbd 的后缀改为 *.dylib。

  - Security.framework
  - AVFoundation.framework
  - CoreVideo.framework
  - CoreMedia.framework
  - libresolv.tbd
  - libsqlite3.tbd
  - CoreTelephony.framework
  - QuartzCore.framework
  - OpenGLES.framework
  - CoreGraphics.framework
  - CFNetwork.framework
  - SystemConfiguration.framework
  - AudioToolbox.framework
  - MediaPlayer.framework
  - UIKit.framework
  - Foundation.framework

- 在 `Xcode` 选中工程，在 `Build Settings` 中搜索 `Other Linker Flags`，增加 `-ObjC` 链接选项。

- 支持HTTP

  iOS 9 中，Apple 引入了新特性 App Transport Security (ATS)，默认要求 App 必须使用 https 协议。详情：[What's New in iOS 9](https://developer.apple.com/library/prerelease/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS9.html#//apple_ref/doc/uid/TP40016198-DontLinkElementID_13)。SDK 在 iOS9 上需要使用 HTTP，您需要设置在 App 中使用 HTTP。在 App 的 *Info.plist 中添加 NSAppTransportSecurity 类型Dictionary。在 NSAppTransportSecurity 下添加 NSAllowsArbitraryLoads 类型 Boolean，值设为 YES。

- Enable Bitcode -> NO

# 接口调用

## 单例对象

在 `MediaPlusSDK.h`文件中 ，SDK 单例对象的初始化以及释放。

```objective-c
/**
 *  初始化单例
 *
 *  @return 单例生成的对象
 */
+ (MediaPlusSDK *)sharedInstance;

/**
 *  @brief 释放单例
 */
+ (void)purgeSharedInstance;
```

## 授权登录

对于生产环境和测试环境拆分两个方法进行授权登录，调用此方法要保证网络正常，app启动调用一次即可，调用结果会通过  `delegate` 反馈。

```objective-c
/**
 *  SDK使用的文件存储路径
 */
@property (nonatomic, copy, readonly) NSString *cachePath;
/**
 *  是否登录成功
 */
@property (nonatomic, assign, readonly) BOOL isAuth;
/**
 *  授权成功后游云ID
 */
@property (nonatomic, copy, readonly) NSString *userID;
```

```objective-c
/**
 *  @method 初始化SDK，登录生产环境服务器
 *
 *  @param udid     用于标识用户、设备的UDID
 *  @param client   开发者后台获取到生产环境的clientID
 *  @param secret   开发者后台获取到生产环境的key(secret)
 *
 */
- (void)startWithUDID:(NSString *)udid
             clientID:(NSString *)client
               secret:(NSString *)secret;
/**
 *  @method 初始化SDK，登录开发环境服务器
 *
 *  @param udid     用于标识用户、设备的UDID
 *  @param client   开发者后台获取到开发环境的clientID
 *  @param secret   开发者后台获取到开发环境的key(secret)
 *
 */
- (void)testStartWithUDID:(NSString *)udid
                 clientID:(NSString *)client
                   secret:(NSString *)secret;
/*! @method
 *  退出登录
 *
 */
- (void)logout;
```

其中UDID可以使用自己的规则，用于标识iOS设备，如果UDID变化，对应游云平台的用户信息也会变化。SDK也提供了UDID方法，使用的是[OpenUDID](https://github.com/ylechelle/OpenUDID)开源项目，不保证每台设备的唯一性。

```objective-c
/**
 *  获取游云提供的UDID
 *
 *  @return UDID
 */
+ (NSString *)getUDID;
```

## APNs

关于 `APNs` ，SDK 默认不使用游云推送，如果使用游云推送，请设置以下信息。关于`APNs`设置详情，请移步[游云推送](https://github.com/youyundeveloper/PushSDK_iOS)，需要注意的是不需要额外集成`SHPushSDK`。

```objective-c
/**
 *  设置是否适用APNs功能，默认不使用，初始化SDK之前调用
 *
 *  @param enable Y/N
 */
- (void)setEnablePush:(BOOL)enable;

/*! @method
 *  设置设备token。
 *
 *  @note 需要在UIApplicationDelegate的回调
 *   - (void)application:didRegisterForRemoteNotificationsWithDeviceToken:中调用。
 */
- (NSString *)setDeviceToken:(NSData *)deviceToken;

/*! @method
 *  当前设备注册推送. push时段，需要登录成功后才能有效注册push.
 *
 *  @param pushToken ios注册的推送token
 *  @param startTime push时段开始时间(0~24),默认0,  如: 开始时间为9,  结束时间为20, push时段从当天9 点到 当天  20点. 传-1 上次开始时间
 *  @param endTime   push时段结束时间(0~24),默认24, 如: 开始时间为20, 结束时间为9,  push时段从当天20点到 第二天 9点. 传-1 上次结束时间
 *  @param handler   回调block (是否操作成功, 如果错误则返回错误信息)
 *
 */
- (void)deviceRegisterPush:(NSString *)pushToken
             pushStartTime:(NSInteger)startTime
                   endTime:(NSInteger)endTime
         completionHandler:(void (^)(BOOL isRegister, NSError* requestError))handler;

/*! @method
 *  取消push服务.
 *
 *  @param handler 回调block (设备信息注册信息, 如果错误则返回错误信息)
 */
- (void)deviceUnRegisterPush:(void (^)(BOOL isUnRegister, NSError* requestError))handler;

/*! @method
 *  获取设备信息.
 *
 *  @param handler 回调block (设备信息注册信息, 如果错误则返回错误信息)
 */
- (void)deviceInfoWithCompletionHandler:(void (^)(NSDictionary *deviceInfo, NSError* requestError))handler;
```

## VoIP接口

`MediaPlusSDK+VoIP.h`文件中

### 单对单语音

目前SDK不支持视频，`enableVideo = NO`。

```objective-c
/**
 *  给某个人员拨打语音消息
 *
 *  @param userID      人员ID
 *  @param enableVideo 是否视频（当前版本不支持）
 *  @param error       错误句柄
 */
- (void)callUser:(NSString *)userID
     enableVideo:(BOOL)enableVideo
           error:(NSError *__autoreleasing *)error;
```

### 接受／挂断

接受或者挂断单对单语音、会议语音。

```objective-c
/**
 *  接受来电／会议请求
 *
 *  @param error 错误句柄
 */
- (void)acceptCall:(NSError *__autoreleasing *)error;

/**
 *  挂断离开单人／会议
 */
- (void)terminateCall;
```

### 会议语音

#### 申请一个房间

开始会议语音要首先申请一个房间

```objective-c
/**
 *  根据群组ID，申请创建一个电话会议的房间
 *
 *  @param goupID 申请房间的群组ID（自行管理群ID）
 *  @param error 错误句柄
 *
 *  @return 操作是否成功
 */
- (BOOL)conferenceRequestRoomGroupID:(NSString *)goupID
                               error:(NSError *__autoreleasing *)error;
```

其中`groupID`参数开发者自己管理业务。结果以`MediaPlusSDKDelegate`形式返回。

cfcallbackType 是枚举类型，有关会议会通过这个回调通知客户端。

```objective-c
typedef NS_ENUM(NSInteger, cfcallbackType) {
    cfcallbackTypeNone              = 0,    //不是任何请求类型 用于默认设置
    cfcallbackTypeRoomRequest       = 1,    //请求房间
    cfcallbackTypeSendInvite        = 2,    //发送邀请
    cfcallbackReceiveInvite         = 3,    //收到他人的电话会议邀请
    cfcallbackTypeKick              = 4,    //踢人
    cfcallbackTypeMute              = 5,    //禁言
    cfcallbackTypeUnmute            = 6,    //解禁
    cfcallbackTypeFetch             = 7,    //获取加入电话会议的成员
    cfcallbackTypeState             = 8,    //收到的是状态通知
};
```

```objective-c
/**
 * Called when receive conference 电话会议 房间 的 创建 和邀请 message
 **/
- (void)onReceiveConfeneceCallback:(MediaPlusSDK *)instance
                              type:(cfcallbackType)type
                          fromUser:(NSString *)fromUid
                           groupID:(NSString *)groupID
                            roomID:(NSString *)roomID
                               key:(NSString *)key
                             users:(NSArray *)users
                         startTime:(NSString *)startTime
                           endTime:(NSString *)endTime
                             error:(NSError *)error;
```

#### 加入房间

成功获取房间号(roomID)和房间密钥(key)后，申请进入房间，拨通房间。

```objective-c
/**
 *  拨通某个电话会议房间
 *
 *  @param Room  房间ID
 *  @param key   房间key
 *  @param error 错误句柄
 */
- (void)callRoom:(NSString *)Room
         withKey:(NSString *)key
           error:(NSError *__autoreleasing *)error;
```

#### 接受会议邀请

`MediaPlusSDKDelegate` 中`onReceiveConfeneceCallback` 的 type ==  cfcallbackReceiveInvite 时，是收到他人的会议邀请。

接受他人的会议邀请，先调用加入房间接口(步骤2)，如果加入房间成功，调用SDK接受来电、会议接口，接口与接受单对单语音接口一致。

```objective-c
/**
 *  接受来电／会议请求
 *
 *  @param error 错误句柄
 */
- (void)acceptCall:(NSError *__autoreleasing *)error;
```

#### 拒绝会议邀请

接口与拒绝单对单语音接口一致。

#### 邀请

邀请多人参加会议接口。

```objective-c
/**
 *  邀请多个成员进入申请好的电话会议房间
 *
 *  @param users   被邀请人员的ID列表
 *  @param groupID 群组ID
 *  @param roomID  房间ID
 *  @param key     房间key
 *
 *  @return 操作是否成功
 */
- (BOOL)conferenceInviteUsers:(NSArray *)users
                      groupID:(NSString *)groupID
                       roomID:(NSString *)roomID
                          key:(NSString *)key
                        error:(NSError *__autoreleasing *)error;
```

#### 业务相关

SDK提供`禁言／解禁`、`成员`、`踢人`接口。

```objective-c
/**
 *  根据群组ID和房间ID查询房间当前成员列表
 *
 *  @param roomID  房间ID
 *  @param groupID 群组ID
 *
 *  @return 操作是否成功
 */
- (BOOL)conferenceFetchUsersinRoom:(NSString *)roomID
                           groupID:(NSString *)groupID
                             error:(NSError *__autoreleasing *)error;

/**
 *  根据房间ID踢人（一个或多个）
 *
 *  @param users  被踢人的人员ID列表
 *  @param roomID 房间ID
 *
 *  @return 操作是否成功
 */
- (BOOL)conferenceKickUsers:(NSArray *)users
                     roomID:(NSString *)roomID
                      error:(NSError *__autoreleasing *)error;

/**
 *  根据房间ID给某些人禁言
 *
 *  @param users  被禁言人员ID列表
 *  @param roomID 当前房间ID
 *
 *  @return 操作是否成功
 */
- (BOOL)conferenceMuteUsers:(NSArray *)users
                     roomID:(NSString *)roomID
                      error:(NSError *__autoreleasing *)error;

/**
 *  根据房间ID给某些人解除禁言
 *
 *  @param users  被解除禁言的人员ID
 *  @param roomID 当前房间ID
 *
 *  @return 操作是否成功
 */
- (BOOL)conferenceUnmuteUsers:(NSArray *)users
                       roomID:(NSString *)roomID
                        error:(NSError *__autoreleasing *)error;
```

可以关闭／开始麦克风、切换听筒和外放。

```objective-c
/**
 *  是否外放
 *
 *  @param enable Y/N
 */
- (void)enableSpeaker:(BOOL)enable;
/**
 *  是否无声
 *
 *  @param enable Y/N
 */
- (void)enableMute:(BOOL)enable;
```

会议室中成员、状态的变化也会收到delegate回调

```objective-c
/**
 * Called when conference 电话会议 有人加入
 **/
- (void)conferenceJoinedWith:(NSString *)roomID
                     groupID:(NSString *)groupID
                       users:(NSArray *)users;

/**
 * Called when conference 电话会议 有人被禁言
 **/
- (void)conferenceMutedWith:(NSString *)roomID
                    groupID:(NSString *)groupID
                    fromUid:(NSString *)fromUid
                      users:(NSArray *)users;

/**
 * Called when conference 电话会议 有人被解禁
 **/
- (void)conferenceUnmutedWith:(NSString *)roomID
                      groupID:(NSString *)groupID
                      fromUid:(NSString *)fromUid
                        users:(NSArray *)users;

/**
 * Called when conference 电话会议 有人被踢
 **/
- (void)conferenceKickedWith:(NSString *)roomID
                     groupID:(NSString *)groupID
                     fromUid:(NSString *)fromUid
                       users:(NSArray *)users;

/**
 * Called when conference 电话会议 有人离开
 **/
- (void)conferenceLeftWith:(NSString *)roomID
                   groupID:(NSString *)groupID
                     users:(NSArray *)users;

/**
 * Called when conference 电话会议 即将关闭的通知提示
 **/
- (void)conferenceWillbeEndWith:(NSString *)roomID
                        groupID:(NSString *)groupID
                         intime:(NSInteger )second;

/**
 * Called when conference 电话会议 验证失败 房间已经过期
 **/
- (void)conferenceExpiredWithRoomID:(NSString *)roomID
                                key:(NSString *)key;
```

### 错误相关

单对单、多人会议语音SDK相关状态会通过delegate和Notification两种方式通知客户端。

```objective-c
/**
 *  来电回调
 *
 *  @param userId 通话对方用户id
 */
- (void)mediaCallIncomingByUser:(NSString*)userId;

/**
 *  来电接通回调
 *
 *  @param userId 通话对方用户id
 */
- (void)mediaCallConnectedByUser:(NSString*)userId;

/**
 *  单人电话下，对方挂断 或 直播的时候本机来电话（live_self）  或 直播的时候主播失去连接超时
 *
 *  @param userId 通话对方用户id
 */
- (void)mediaCallEndByUser:(NSString*)userId;

/**
 *  挂起(被优先级更高任务打断)
 *
 *  @param userId 通话对方用户id
 */
- (void)mediaCallRemotePauseByUser:(NSString*)userId;

/**
 *  错误回调
 *
 *  @param error  错误信息
 *  @param userId 通话对方用户id
 */
- (void)mediaCallError:(NSError*)error fromUser:(NSString*)userId;

/**
 * 未接到的来电
 **/
- (void)missCallFromUser:(NSString *)fromUid
                  atTime:(NSInteger)time;
```

Notification:

```objective-c
extern NSString *const NotificationMediaCallIncoming;     //来电
extern NSString *const NotificationMediaCallConnected;    //来电接通
extern NSString *const NotificationMediaCallEnd;          //单人电话下，对方挂断 或 直播的时候本机来电话（live_self） 或 直播的时候主播失去连接超时
extern NSString *const NotificationMediaCallRemotePause;  //挂起(被优先级更高任务打断)

//出错
extern NSString *const NotificationMediaCallError;        //错误
```

## IM接口

SDK提供基础的IM相关接口

#### 发送文本消息

单对单、群组发送文本消息接口

```objective-c
/**
 *  @brief 发送文本消息
 *
 *  @param toid       收消息人、群组、聊天室id
 *  @param content    消息内容
 *  @param extContent 扩展消息内容
 *  @param tag        消息标示, 用于回调
 *  @param type       消息类型
 *                      YYWChatFileTypeCustom : 自定义消息(服务器不作处理)
 *                      YYWChatFileTypeText : (服务器业务处理(e:敏感词...))
 *  @param target     消息对象类型
 *  @param timeout    调用超时时间
 *  @param errPtr     错误句柄
 *
 *  @return 消息是否正常发送, YES是, NO否
 */
- (void)socialSendMsg:(NSString *)toid
                 body:(NSData *)content
              extBody:(NSData *)extContent
              withTag:(NSInteger)tag
             withType:(YYWChatFileType)type
           targetType:(WChatMsgTargetType)target
          withTimeout:(NSTimeInterval)timeout
                error:(NSError **)errPtr;
```

#### 发送语音消息

发送语音消息首先获取语音的标识

```objective-c
/**
 *  @brief 发送音频消息
 *
 *  @param toid       收消息人、群组、聊天室id
 *  @param spanId     语音唯一标示
 *  @param sequenceNo 语音分片编号, 如 1, 2, 3, ... -1, -1 表示结束
 *  @param content    语音消息内容
 *  @param ext        扩展消息内容
 *  @param tag        消息标示, 用于回调
 *  @param target     消息对象类型
 *  @param timeout    调用超时时间
 *  @param errPtr     错误句柄
 *
 *  @return 消息是否正常发送, YES是, NO否
 */
- (void)socialSendVoice:(NSString *)toid
                 spanId:(NSString *)spanId
             sequenceNo:(NSInteger)sequenceNo
                content:(NSData *)content
                    ext:(NSData *)ext
                withTag:(NSInteger)tag
             targetType:(WChatMsgTargetType)target
            withTimeout:(NSTimeInterval)timeout
                  error:(NSError **)errPtr;

/**
 *  @brief 获取语音唯一标示
 *
 *  @param tuid 收消息人Uid
 *
 *  @return 语音消息唯一标示
 */
- (NSString *)getVoiceSpanId:(NSString *)tuid;
```

#### 发送图片消息

发送图片可以断点续传和非断点续传两种，图片缩略图最大50KB，原图最大50MB，超出不发送。

```objective-c
/**
 *  @brief 发送文件(图片)给个人、群组, 带缩略图
 *
 *  @param toid       收消息人、群组、聊天室id
 *  @param filepath   文件路径
 *  @param nailpath   缩略图路径
 *  @param extContent 扩展消息内容
 *  @param tag        消息标示, 用于回调
 *  @param fileType   文件类型
 *  @param target     消息对象类型
 *  @param timeout    调用超时时间
 *  @param errPtr     错误句柄
 *
 *  @return 文件id
 */

- (NSString *)socialSendFileWithThumbnail:(NSString *)toid
                                     path:(NSString *)filepath
                                 nailpath:(NSString *)nailpath
                                  extBody:(NSData *)extContent
                                  withTag:(NSInteger)tag
                                 filetype:(YYWChatFileType)fileType
                               targetType:(WChatMsgTargetType)target
                              withTimeout:(NSTimeInterval)timeout
                                    error:(NSError **)errPtr;

/**
 *	@brief 发送文件给个人、群组, 带缩略图, 断点续传
 *
 *  @param toid       收消息人、群组、聊天室id
 *  @param fid        文件id
 *  @param filepath   文件路径
 *  @param nailpath   缩略图路径
 *  @param extContent 扩展消息内容
 *  @param index      文件片数索引
 *  @param tag        消息标示, 用于回调
 *  @param fileType   文件类型
 *  @param target     消息对象类型
 *  @param timeout    调用超时时间
 *  @param errPtr     错误句柄
 *
 *  @return 文件id
 */
- (NSString *)socialSendFileWithThumbnail:(NSString *)toid
                                   fileId:(NSString *)fid
                                     path:(NSString *)filename
                                 nailpath:(NSString *)nailfile
                                  extBody:(NSData *)extContent
                                withIndex:(UInt32)index
                                  withTag:(NSInteger)tag
                                 filetype:(YYWChatFileType)fileType
                               targetType:(WChatMsgTargetType)target
                              withTimeout:(NSTimeInterval)timeout
                                    error:(NSError **)errPtr;
```

#### 发送文件消息

发送文件可以断点续传和非断点续传两种，文件最大50MB，超出不发送。

```objective-c
/**
 *  获取发送文件的文件ID
 *
 *  @param targetID 对方的ID
 *
 *  @return 文件ID
 */
- (NSString *)getFileIdWithTargetID:(NSString *)targetID;
/**
 *  @brief 发送文件给个人、群组
 *
 *  @param toid       收消息人、群组、聊天室id
 *  @param filepath   文件路径
 *  @param extContent 扩展消息内容
 *  @param tag        消息标示, 用于回调
 *  @param fileType   文件类型
 *  @param target     消息对象类型
 *  @param timeout    调用超时时间
 *  @param errPtr     错误句柄
 *
 *  @return 文件id
 */
- (NSString *)socialSendFile:(NSString *)toid
                        path:(NSString *)filepath
                     extBody:(NSData *)extContent
                     withTag:(NSInteger)tag
                    filetype:(YYWChatFileType)fileType
                  targetType:(WChatMsgTargetType)target
                 withTimeout:(NSTimeInterval)timeout
                       error:(NSError **)errPtr;

/**
 *	@brief 发送文件给个人、群组, 断点续传
 *
 *  @param toid       收消息人、群组、聊天室id
 *  @param fid        文件id
 *  @param filepath   文件路径
 *  @param extContent 扩展消息内容
 *  @param index      文件片数索引
 *  @param tag        消息标示, 用于回调
 *  @param fileType   文件类型
 *  @param target     消息对象类型
 *  @param timeout    调用超时时间
 *  @param errPtr     错误句柄
 *
 *  @return 文件id
 */
- (NSString *)socialSendFile:(NSString *)toid
                     withFid:(NSString *)fid
                        path:(NSString *)filename
                     extBody:(NSData *)extContent
                   withIndex:(UInt32)index
                     withTag:(NSInteger)tag
                    filetype:(YYWChatFileType)fileType
                  targetType:(WChatMsgTargetType)target
                 withTimeout:(NSTimeInterval)timeout
                       error:(NSError **)errPtr;
```

#### 消息未读数

游云服务器会存储每位用户的消息未读数，可以设置用户的未读数消息

```objective-c
/**
 *  @brief 设置消息未读数
 *
 *  @param number 未读数数量
 *  @param tag    消息标示, 用于回调
 *  @param errPtr 错误句柄
 *
 */
- (void)socialSetUnreadNumber:(NSInteger)number
                      withTag:(NSInteger)tag
                        error:(NSError **)errPtr;

/**
 *  @brief 设置消息未读数 - number
 *
 *  @param number 减掉的消息未读数
 *  @param tag    消息标示, 用于回调
 *  @param errPtr 错误句柄
 *
 */
- (void)socialMinusUnreadNumber:(NSInteger)number
                        withTag:(NSInteger)tag
                          error:(NSError **)errPtr;
```

#### 获取文件

根据文件ID获取文件／图片／视频，可以分片获取和一次性获取，获取成功后，文件保存在`Document`路径下。

```objective-c
/**
 *  @brief 根据文件id获取文件, 分片获取
 *
 *  @param fid     文件id
 *  @param length  文件长度
 *  @param size    分片长度
 *  @param tag     消息标示, 用于回调
 *  @param index   分片索引
 *  @param timeout 调用超时时间
 *  @param errPtr  错误句柄
 *
 */
- (void)socialGetFile:(NSString *)fid
           filelength:(UInt64)length
            pieceSize:(UInt32)size
              withTag:(NSInteger)tag
                index:(UInt32)index
          withTimeout:(NSTimeInterval)timeout
                error:(NSError **)errPtr;

/**
 *  @brief 根据文件id获取文件
 *
 *  @param fid     文件id
 *  @param length  文件长度
 *  @param tag     消息标示, 用于回调
 *  @param timeout 调用超时时间
 *  @param errPtr  错误句柄
 *
 */
- (void)socialGetFile:(NSString *)fid
           filelength:(UInt64)length
              withTag:(NSInteger)tag
          withTimeout:(NSTimeInterval)timeout
                error:(NSError **)errPtr;
```

#### 获取历史聊天记录

历史聊天记录返回的是NSDictionary的数组。

```objective-c
/**
 *  单聊聊天历史消息.
 *
 *  @param userId    聊天对方uid(非当前登陆用户)
 *  @param timestamp 时间戳(精确到秒)
 *  @param size      数据条数(服务器默认一次最多取20)
 *  @param handler   回调block (历史消息数据, 如果错误则返回错误信息)
 */
- (void)getHistoryByUser:(NSString *)userId timestamp:(NSInteger)timestamp size:(NSInteger)size completionHandler:(void (^)(NSArray *history, NSError* requestError))handler;
```

## 群组接口

SDK 提供基础的群组相关接口以支持IM功能

#### 群组创建

创建一个临时群组

```objective-c
/**
 *  创建群组
 *
 *  @param handler 回调block (创建成功的群组id, 如果错误则返回错误信息)
 */
- (void)groupCreateHandler:(void(^)(NSString *groupId, NSError* requestError))handler;
```

创建一个条件群组

```objective-c
/**
 *  创建一个群组
 *
 *  @param name    群名称
 *  @param desc    群简介
 *  @param cate    群类别
 *  @param handler 回调
 */
- (void)groupCreateName:(NSString *)name
            description:(NSString *)desc
               categary:(WChatGroupCategary)cate
             completion:(void (^)(NSDictionary *response, NSError *err))handler;
```

#### 群组添加成员

群主和群管理员可以添加群成员

```objective-c
/**
 *  群组加人
 *
 *  @param groupId 群组id
 *  @param userIds 用户id数组
 *  @param handler 回调block (是否操作成功, 如果错误则返回错误信息)
 */
- (void)group:(NSString *)groupId
      addUser:(NSArray *)userIds
completionHandler:(void (^)(BOOL isAdd, NSError* requestError))handler;
```

群组删除成员

群主和管理员可以删除群成员

```objective-c
/**
 *  群组踢人
 *
 *  @param groupId 群组id
 *  @param userIds 用户id数组
 *  @param handler 回调block (是否操作成功, 如果错误则返回错误信息)
 */
- (void)group:(NSString *)groupId
      delUser:(NSArray *)userIds
completionHandler:(void (^)(BOOL isDel, NSError* requestError))handler;
```

#### 退出群组

群主是不可以推出群的

```objective-c
/**
 *  退出群组
 *
 *  @param groupId 群组id
 *  @param handler 回调block (是否操作成功, 如果错误则返回错误信息)
 */
- (void)groupExit:(NSString *)groupId
completionHandler:(void(^)(BOOL isExit, NSError* requestError))handler;
```

#### 获取群成员ID

```objective-c
/**
 *  获取群组成员
 *
 *  @param groupId 群组id
 *  @param handler 回调block (群组成员数据, 如果错误则返回错误信息)
 */
- (void)groupGetUsers:(NSString *)groupId
    completionHandler:(void (^)(NSArray *users, NSError* requestError))handler;
```

#### 获取群列表

获取当前用户加入／拥有的群列表

```objective-c
/**
 *  获取当前用户的群组
 *
 *  @param handler 回调block (用户的群组数据, 如果错误则返回错误信息)
 */
- (void)getUserGroupsHandler:(void (^)(NSArray *groups, NSError* requestError))handler;
```

#### 销毁群组

```objective-c
/**
 *  销毁房间
 *
 *  @param gid     房间ID
 *  @param handler 回调
 */
- (void)groupDelete:(NSString *)gid completion:(void (^)(BOOL sucess, NSError *err))handler;
```

#### 获取群信息

```objective-c
/**
 *  获取群信息
 *
 *  @param gid     群组ID
 *  @param handler 回调
 */
- (void)groupGetInfo:(NSString *)gid completion:(void (^)(NSDictionary *response, NSError *err))handler;
/**
 *  获取群组内的所有成员
 *
 *  @param gid     群id
 *  @param handler 回调
 */
- (void)groupGetMember:(NSString *)gid completion:(void (^)(NSArray *members, NSError *err))handler;
```

更新群名称、简介

```objective-c
/**
 *  更新群组信息
 *
 *  @param gid     群ID
 *  @param name    群名称
 *  @param desc    群简介
 *  @param cate    群类别
 *  @param handler 回调
 */
- (void)groupUpdate:(NSString *)gid name:(NSString *)name description:(NSString *)desc categary:(WChatGroupCategary)cate completion:(void (^)(NSDictionary *response, NSError *err))handler;
```

#### 申请入群

```objective-c
/**
 *  申请加入群组
 *
 *  @param gid        群组ID
 *  @param extContent 附言
 *  @param handler    回调
 */
- (void)groupApplyJoinin:(NSString *)gid extContent:(NSString *)extContent completion:(void (^)(BOOL success, NSError *err))handler;
```

#### 获取群组历史聊天记录

```objective-c
/**
 *  单聊聊天历史消息.
 *
 *  @param userId    聊天对方uid(非当前登陆用户)
 *  @param timestamp 时间戳(精确到秒)
 *  @param size      数据条数(服务器默认一次最多取20)
 *  @param handler   回调block (历史消息数据, 如果错误则返回错误信息)
 */
- (void)getHistoryByUser:(NSString *)userId timestamp:(NSInteger)timestamp size:(NSInteger)size completionHandler:(void (^)(NSArray *history, NSError* requestError))handler;
```

## Delegate 

SDK Delegate协议

#### SDK网络状态回调

```objective-c
/**
 *  @brief 连接成功回调
 *
 *  @param instance 实例
 */
- (void)onConnected:(MediaPlusSDK *)instance;

/**
 *  @brief 连接断开回调
 *
 *  @param instance 实例
 *  @param error    如连接出错断开, 则返回错误消息
 */
- (void)onDisconnect:(MediaPlusSDK *)instance withError:(NSError *)error;

/**
 *  @brief 向服务器发送断开连接的消息回调
 *
 *  @param instance 实例
 *  @param error    如果设置失败, 则返回错误信息
 */
- (void)onClose:(MediaPlusSDK *)instance withError:(NSError *)error;

/**
 *  @brief 退出登陆回调
 *
 *  @param instance 实例
 *  @param error    如登陆出错, 则返回错误消息
 */
- (void)onLogout:(MediaPlusSDK *)instance withError:(NSError *)error;

/**
 *  @brief 超时回调
 *
 *  @param instance 实例
 *  @param tag      消息标示
 *  @param error    如操作超时, 则返回错误消息
 */
- (void)onTimeout:(MediaPlusSDK *)instance withTag:(NSInteger)tag withError:(NSError *)error;

/**
 *  @brief 连接状态回调
 *
 *  @param instance 实例
 *  @param state    连接状态
 */
- (void)onConnectState:(MediaPlusSDK *)instance state:(WChatConnectState)state;
```

#### 前后台状态

iOS 有前台和后台的区分，所有SDK根据前台后台会有一些操作，

```objective-c
#pragma mark - 前后台切换
/**
 *  @brief 客户端退到后台, 关闭服务器消息notice下发, 开启推送回调
 *
 *  @param instance 实例
 *  @param error    如果设置失败, 则返回错误信息
 */
- (void)onPreClose:(MediaPlusSDK *)instance withError:(NSError *)error;

/**
 *  @brief 客户端回到前台, 开启服务器消息notice下发, 关闭推送
 *
 *  @param instance 实例
 *  @param error    如果设置失败, 则返回错误信息
 */
- (void)onKeepAlive:(MediaPlusSDK *)instance withError:(NSError *)error;
```

#### 发送消息(文本、文件、图片)消息后的回调

```objective-c
/**
 *  @brief 消息已送达到服务器, 但服务器还未下发相应, sdk预先返回, 随后服务器会下发相应, 以及时间戳.
 *  可理解为发送消息成功, 前端可根据此状态, 预先显示消息发送成功, 后台处理服务器下发.
 *
 *  @param instance 实例
 *  @param tag      消息标示
 */
- (void)onSendPreBack:(MediaPlusSDK *)instance withTag:(NSInteger)tag;

/**
 *  @brief 发送文本消息回调
 *
 *  @param instance  实例
 *  @param tag       消息标示
 *  @param time      消息时间
 *  @param messageId 消息id
 *  @param error     如发送出错, 则返回错误消息
 */
- (void)onSendMsg:(MediaPlusSDK *)instance
          withTag:(NSInteger)tag
         withTime:(NSInteger)time
    withMessageId:(NSString *)messageId
        withError:(NSError *)error;

/**
 *  @brief 发送文件回调
 *
 *  @param instance  实例
 *  @param tag       消息标示
 *  @param time      消息时间
 *  @param messageId 消息id
 *  @param error     如发送出错, 则返回错误消息
 */
- (void)onSendFile:(MediaPlusSDK *)instance
           withTag:(NSInteger)tag
          withTime:(NSInteger)time
     withMessageId:(NSString *)messageId
         withError:(NSError *)error;
```

#### 设置消息数量的回调

```objective-c
/**
 *  未读数设置回调
 *
 *  @param instance   实例
 *  @param callbackId 消息标示
 */
-(void)onUnreadNoticeCallback:(MediaPlusSDK*)instance withCallbackId:(NSInteger)callbackId;

/**
 *  @brief 获取消息未读数
 *
 *  @param user  用户消息未读数, 字典格式 @{ @"用户id": @{ @"num": NSNumber 未读数, @"time": NSNumber 消息时间 }, @"用户id": @{ @"num": NSNumber 未读数, @"time": NSNumber 消息时间 } }
 *  @param group 群组消息未读数, 字典格式 @{ @"群组id": @{ @"num": NSNumber 未读数, @"time": NSNumber 消息时间 }, @"群组id": @{ @"num": NSNumber 未读数, @"time": NSNumber 消息时间 } }
 */
- (void)onRecvUnreadNumber:(MediaPlusSDK *)instance
                  withUser:(NSDictionary *)user
                 withGroup:(NSDictionary *)group;

/**
 *  @brief http链接请求回调
 *
 *  @param instance   实例
 *  @param response   返回消息
 *  @param callbackId 回调id
 *  @param error      如收消息出错, 则返回错误信息
 */
- (void)onShortResponse:(MediaPlusSDK *)instance
           withResponse:(NSString *)response
         withCallbackId:(NSInteger)callbackId
              withError:(NSError *)error;
```

#### 接收消息回调

```objective-c
/**
 *  @brief 接收文本消息回调
 *
 *  @param instance   实例
 *  @param messageId  消息id
 *  @param fromUid    发消息人Uid
 *  @param toUid      收消息人Uid
 *  @param type       消息类型
 *  @param timevalue  消息时间
 *  @param content    消息内容
 *  @param extContent 消息扩展内容
 *  @param error      如收消息出错, 则返回错误信息
 */
- (void)onRecvMsg:(MediaPlusSDK *)instance
    withMessageId:(NSString *)messageId
          fromUid:(NSString *)fromUid
            toUid:(NSString *)toUid
         filetype:(YYWChatFileType)type
             time:(NSInteger)timevalue
          content:(NSData *)content
          extBody:(NSData *)extContent
        withError:(NSError *)error;

/**
 *  @brief 接收群组文本消息回调
 *
 *  @param instance   实例
 *  @param messageId  消息id
 *  @param gid        群id
 *  @param fromUid    发消息人Uid
 *  @param type       消息类型
 *  @param timevalue  消息时间
 *  @param content    消息内容
 *  @param extContent 消息扩展内容
 *  @param error      如收消息出错, 则返回错误信息
 */
- (void)onRecvGroupMsg:(MediaPlusSDK *)instance
         withMessageId:(NSString *)messageId
           withGroupId:(NSString *)gid
               fromUid:(NSString *)fromUid
              filetype:(YYWChatFileType)type
                  time:(NSInteger)timevalue
               content:(NSData *)content
               extBody:(NSData *)extContent
             withError:(NSError *)error;

/**
 *  @brief 接收聊天室消息回调
 *
 *  @param instance      实例
 *  @param messageId     消息id
 *  @param rid           房间id
 *  @param fromUid       发消息人Uid
 *  @param type          消息类型
 *  @param spanId        语音唯一标识
 *  @param sequenceNo    语音分片编号, 如 1, 2, 3, ... -1, -1 表示结束
 *  @param fileid        文件id
 *  @param thumbnailData 缩略图二进制数据
 *  @param length        文件长度
 *  @param size          分片大小
 *  @param timevalue     消息时间
 *  @param content       消息内容
 *  @param extContent    消息扩展内容
 *  @param error         如收消息出错, 则返回错误信息
 */
- (void)onRecvChatRoomMsg:(MediaPlusSDK *)instance
            withMessageId:(NSString *)messageId
               withRoomId:(NSString *)rid
                  fromUid:(NSString *)fromUid
                 filetype:(YYWChatFileType)type
                   spanId:(NSString *)spanId
               sequenceNo:(NSInteger)sequenceNo
                   fileId:(NSString *)fileid
                thumbnail:(NSData *)thumbnailData
               filelength:(UInt64)length
                pieceSize:(UInt32)size
                     time:(NSInteger)timevalue
                  content:(NSData *)content
                  extBody:(NSData *)extContent
                withError:(NSError *)error;

/**
 *  @brief 接收语音消息回调
 *
 *  @param instance   实例
 *  @param messageId  消息id
 *  @param fromUid    发消息人Uid
 *  @param toUid      收消息人Uid
 *  @param spanId     语音唯一标识
 *  @param sequenceNo 语音分片编号, 如 1, 2, 3, ... -1, -1 表示结束
 *  @param timevalue  消息时间
 *  @param content    消息内容
 *  @param extContent 消息扩展内容
 *  @param error      如收消息出错, 则返回错误信息
 */
- (void)onRecvVoice:(MediaPlusSDK *)instance
      withMessageId:(NSString *)messageId
            fromUid:(NSString *)fromUid
              toUid:(NSString *)toUid
             spanId:(NSString *)spanId
         sequenceNo:(NSInteger)sequenceNo
               time:(NSInteger)timevalue
            content:(NSData *)content
            extBody:(NSData *)extContent
          withError:(NSError *)error;

/**
 *  @brief 接收群组语音消息回调
 *
 *  @param instance   实例
 *  @param messageId  消息id
 *  @param gid        群id
 *  @param fromUid    发消息人Uid
 *  @param spanId     语音唯一标识
 *  @param sequenceNo 语音分片编号, 如 1, 2, 3, ... -1, -1 表示结束
 *  @param timevalue  消息时间
 *  @param content    消息内容
 *  @param extContent 消息扩展内容
 *  @param error      如收消息出错, 则返回错误信息
 */
- (void)onRecvGroupVoice:(MediaPlusSDK *)instance
           withMessageId:(NSString *)messageId
             withGroupId:(NSString *)gid
                 fromUid:(NSString *)fromUid
                  spanId:(NSString *)spanId
              sequenceNo:(NSInteger)sequenceNo
                    time:(NSInteger)timevalue
                 content:(NSData *)content
                 extBody:(NSData *)extContent
               withError:(NSError *)error;

/**
 *  @brief 接收文件消息回调
 *
 *  @param instance      实例
 *  @param messageId     消息id
 *  @param fromUid       发消息人Uid
 *  @param toUid         收消息人Uid
 *  @param type          消息类型
 *  @param timevalue     消息时间
 *  @param fileid        文件id
 *  @param thumbnailData 缩略图二进制数据
 *  @param extContent    消息扩展内容
 *  @param length        文件长度
 *  @param size          文件分片大小
 *  @param error         如收文件出错, 则返回错误信息
 */
- (void)onRecvFile:(MediaPlusSDK *)instance
     withMessageId:(NSString *)messageId
           fromUid:(NSString *)fromUid
             toUid:(NSString *)toUid
          filetype:(YYWChatFileType)type
              time:(NSInteger)timevalue
            fileId:(NSString *)fileid
         thumbnail:(NSData *)thumbnailData
           extBody:(NSData *)extContent
        filelength:(UInt64)length
         pieceSize:(UInt32)size
         withError:(NSError *)error;

/**
 *  @brief 接收群组文件消息回调
 *
 *  @param instance      实例
 *  @param messageId     消息id
 *  @param gid           群id
 *  @param fromUid       发消息人Uid
 *  @param type          消息类型
 *  @param timevalue     消息时间
 *  @param fileid        文件id
 *  @param thumbnailData 缩略图二进制数据
 *  @param extContent    消息扩展内容
 *  @param length        文件长度
 *  @param size          文件分片大小
 *  @param error         如收文件出错, 则返回错误信息
 */
- (void)onRecvGroupFile:(MediaPlusSDK *)instance
          withMessageId:(NSString *)messageId
            withGroupId:(NSString *)gid
                fromUid:(NSString *)fromUid
               filetype:(YYWChatFileType)type
                   time:(NSInteger)timevalue
                 fileId:(NSString *)fileid
              thumbnail:(NSData *)thumbnailData
                extBody:(NSData *)extContent
             filelength:(UInt64)length
              pieceSize:(UInt32)size
              withError:(NSError *)error;

/**
 *  @brief 系统下发的notice消息, 踢人回调
 *
 *  @param instance 实例
 *  @param fuid     发消息人Uid
 *  @param type     Notice类型
 *  @param content  消息内容
 */
- (void)onRecvNoticeMessage:(MediaPlusSDK *)instance
                    fromUid:(NSString *)fuid
                   withType:(WChatNoticeType)type
                withContent:(NSString *)content;

/**
 *  @brief 订阅消息回调
 *
 *  @param instance      实例
 *  @param messageId     消息id
 *  @param fromUid       发消息人Uid
 *  @param toUid         收消息人Uid
 *  @param type          消息类型
 *  @param spanId        语音唯一标识
 *  @param sequenceNo    语音分片编号, 如 1, 2, 3, ... -1, -1 表示结束
 *  @param fileid        文件id
 *  @param thumbnailData 缩略图二进制数据
 *  @param length        文件长度
 *  @param size          分片大小
 *  @param timevalue     消息时间
 *  @param content       消息内容
 *  @param extContent    消息扩展内容
 *  @param error         如收消息出错, 则返回错误信息
 */
- (void)onRecvSubscribeMsg:(MediaPlusSDK *)instance
             withMessageId:(NSString *)messageId
                   fromUid:(NSString *)fromUid
                     toUid:(NSString *)toUid
                  filetype:(YYWChatFileType)type
                    spanId:(NSString *)spanId
                sequenceNo:(NSInteger)sequenceNo
                    fileId:(NSString *)fileid
                 thumbnail:(NSData *)thumbnailData
                filelength:(UInt64)length
                 pieceSize:(UInt32)size
                      time:(NSInteger)timevalue
                   content:(NSData *)content
                   extBody:(NSData *)extContent
                 withError:(NSError *)error;
```

#### 获取文件进度回调

获取文件分分片获取和一次性获取，开发者需要通过tag和fileid自己区分不同的文件和操作

```objective-c
/**
 *  @brief 获取文件数据回调
 *
 *  @param instance 实例
 *  @param fileid   文件id
 *  @param tag      消息标示
 *  @param error    如获取文件出错, 则返回错误信息
 */
- (void)onGetFile:(MediaPlusSDK *)instance
           fileId:(NSString *)fileid
          withTag:(NSInteger)tag
        withError:(NSError *)error;

/**
 *  @brief 发送和接收文件进度的回调
 *
 *  @param instance 实例
 *  @param tag      消息标示
 *  @param index    文件分片索引
 *  @param limit    文件分片总数
 *  @param error    如获取进度出错, 则返回错误信息
 */
- (void)onFileProgress:(MediaPlusSDK *)instance
               withTag:(NSInteger)tag
             withIndex:(UInt32)index
             withLimit:(UInt32)limit
             withError:(NSError *)error;
```

