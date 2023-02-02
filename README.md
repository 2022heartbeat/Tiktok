# TikTok Backend Basic Implementation

## 1.  基础接口

抖音最基础的功能实现，支持所有用户刷抖音视频，同时允许用户注册账号，发布自己拍摄的视频，发布后的视频能够被其他人刷到。

### /douyin/feed/ - 视频流接口

不限制登录状态，返回按投稿时间倒序的视频列表，视频数由服务端控制，单次最多30个。

接口类型 GET

接口定义

```proto2
syntax = "proto2";
package douyin.core;

message douyin_feed_request {
  optional int64 latest_time = 1; // 可选参数，限制返回视频的最新投稿时间戳，精确到秒，不填表示当前时间
  optional string token = 2； // 可选参数，登录用户设置
}

message douyin_feed_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  repeated Video video_list = 3; // 视频列表
  optional int64 next_time = 4; // 本次返回的视频中，发布最早的时间，作为下次请求时的latest_time
}

message Video {
  required int64 id = 1; // 视频唯一标识
  required User author = 2; // 视频作者信息
  required string play_url = 3; // 视频播放地址
  required string cover_url = 4; // 视频封面地址
  required int64 favorite_count = 5; // 视频的点赞总数
  required int64 comment_count = 6; // 视频的评论总数
  required bool is_favorite = 7; // true-已点赞，false-未点赞
  required string title = 8; // 视频标题
}

message User {
  required int64 id = 1; // 用户id
  required string name = 2; // 用户名称
  optional int64 follow_count = 3; // 关注总数
  optional int64 follower_count = 4; // 粉丝总数
  required bool is_follow = 5; // true-已关注，false-未关注
}
```

### /douyin/user/register/ - 用户注册接口

新用户注册时提供用户名，密码，昵称即可，用户名需要保证唯一。创建成功后返回用户 id 和权限token.

接口类型 POST

接口定义

```proto2
syntax = "proto2";
package douyin.core;

message douyin_user_register_request {
  required string username = 1; // 注册用户名，最长32个字符
  required string password = 2; // 密码，最长32个字符
}

message douyin_user_register_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  required int64 user_id = 3; // 用户id
  required string token = 4; // 用户鉴权token
}
```

### /douyin/user/login/ - 用户登录接口

通过用户名和密码进行登录，登录成功后返回用户 id 和权限 token.

接口类型 POST

接口定义

```proto2
syntax = "proto2";
package douyin.core;

message douyin_user_login_request {
  required string username = 1; // 登录用户名
  required string password = 2; // 登录密码
}

message douyin_user_login_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  required int64 user_id = 3; // 用户id
  required string token = 4; // 用户鉴权token
}
```

### /douyin/user/ - 用户信息
获取登录用户的 id、昵称，如果实现社交部分的功能，还会返回关注数和粉丝数。

接口类型 GET

接口定义

```proto2
syntax = "proto2";
package douyin.core;

message douyin_user_request {
  required int64 user_id = 1; // 用户id
  required string token = 2; // 用户鉴权token
}

message douyin_user_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  required User user = 3; // 用户信息
}

message User {
  required int64 id = 1; // 用户id
  required string name = 2; // 用户名称
  optional int64 follow_count = 3; // 关注总数
  optional int64 follower_count = 4; // 粉丝总数
  required bool is_follow = 5; // true-已关注，false-未关注
}
```

### /douyin/publish/action/ - 视频投稿

登录用户选择视频上传。

接口类型 POST

接口定义

```proto2
syntax = "proto2";
package douyin.core;

message douyin_publish_action_request {
  required string token = 1; // 用户鉴权token
  required bytes data = 2; // 视频数据
  required string title = 3; // 视频标题
}

message douyin_publish_action_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
}
```

### /douyin/publish/list/ - 发布列表

登录用户的视频发布列表，直接列出用户所有投稿过的视频。

接口类型 GET

接口定义

```proto2
syntax = "proto2";
package douyin.core;

message douyin_publish_list_request {
  required int64 user_id = 1; // 用户id
  required string token = 2; // 用户鉴权token
}

message douyin_publish_list_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  repeated Video video_list = 3; // 用户发布的视频列表
}

message Video {
  required int64 id = 1; // 视频唯一标识
  required User author = 2; // 视频作者信息
  required string play_url = 3; // 视频播放地址
  required string cover_url = 4; // 视频封面地址
  required int64 favorite_count = 5; // 视频的点赞总数
  required int64 comment_count = 6; // 视频的评论总数
  required bool is_favorite = 7; // true-已点赞，false-未点赞
  required string title = 8; // 视频标题
}

message User {
  required int64 id = 1; // 用户id
  required string name = 2; // 用户名称
  optional int64 follow_count = 3; // 关注总数
  optional int64 follower_count = 4; // 粉丝总数
  required bool is_follow = 5; // true-已关注，false-未关注
}
```


## 2.  互动接口

每个登录用户支持点赞，同时维护用户自己的点赞视频列表，在个人信息页中查看。

所有用户能够查看视频的评论列表，但是只有登录用户能够对视频进行评论。

### /douyin/favorite/action/ - 赞操作

登录用户对视频的点赞和取消点赞操作。

接口类型 POST

接口定义

```proto2
syntax = "proto2";
package douyin.extra.first;

message douyin_favorite_action_request {
  required string token = 1; // 用户鉴权token
  required int64 video_id = 2; // 视频id
  required int32 action_type = 3; // 1-点赞，2-取消点赞
}

message douyin_favorite_action_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
}
```

### /douyin/favorite/list/ - 喜欢列表

登录用户的所有点赞视频。

接口类型 GET

接口定义

```proto2
syntax = "proto2";
package douyin.extra.first;

message douyin_favorite_list_request {
  required int64 user_id = 1; // 用户id
  required string token = 2; // 用户鉴权token
}

message douyin_favorite_list_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  repeated Video video_list = 3; // 用户点赞视频列表
}

message Video {
  required int64 id = 1; // 视频唯一标识
  required User author = 2; // 视频作者信息
  required string play_url = 3; // 视频播放地址
  required string cover_url = 4; // 视频封面地址
  required int64 favorite_count = 5; // 视频的点赞总数
  required int64 comment_count = 6; // 视频的评论总数
  required bool is_favorite = 7; // true-已点赞，false-未点赞
  required string title = 8; // 视频标题
}

message User {
  required int64 id = 1; // 用户id
  required string name = 2; // 用户名称
  optional int64 follow_count = 3; // 关注总数
  optional int64 follower_count = 4; // 粉丝总数
  required bool is_follow = 5; // true-已关注，false-未关注
}
```

### /douyin/comment/action/ - 评论操作

登录用户对视频进行评论。

接口类型 POST

接口定义

```proto2
syntax = "proto2";
package douyin.extra.first;

message douyin_comment_action_request {
  required string token = 1; // 用户鉴权token
  required int64 video_id = 2; // 视频id
  required int32 action_type = 3; // 1-发布评论，2-删除评论
  optional string comment_text = 4; // 用户填写的评论内容，在action_type=1的时候使用
  optional int64 comment_id = 5; // 要删除的评论id，在action_type=2的时候使用
}

message douyin_comment_action_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  optional Comment comment = 3; // 评论成功返回评论内容，不需要重新拉取整个列表
}

message Comment {
  required int64 id = 1; // 视频评论id
  required User user =2; // 评论用户信息
  required string content = 3; // 评论内容
  required string create_date = 4; // 评论发布日期，格式 mm-dd
}
```

### /douyin/comment/list/ - 视频评论列表

查看视频的所有评论，按发布时间倒序。

接口类型 GET

接口定义

```proto2
syntax = "proto2";
package douyin.extra.first;

message douyin_comment_list_request {
  required string token = 1; // 用户鉴权token
  required int64 video_id = 2; // 视频id
}

message douyin_comment_list_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  repeated Comment comment_list = 3; // 评论列表
}

message Comment {
  required int64 id = 1; // 视频评论id
  required User user =2; // 评论用户信息
  required string content = 3; // 评论内容
  required string create_date = 4; // 评论发布日期，格式 mm-dd
}

message User {
  required int64 id = 1; // 用户id
  required string name = 2; // 用户名称
  optional int64 follow_count = 3; // 关注总数
  optional int64 follower_count = 4; // 粉丝总数
  required bool is_follow = 5; // true-已关注，false-未关注
}
```


## 3.  社交接口

实现用户之间的关注关系维护，登录用户能够关注或取关其他用户，同时自己能够看到自己关注过的所有用户列表，以及所有关注自己的用户列表。

### /douyin/relation/action/ - 关系操作

登录用户对其他用户进行关注或取消关注。

接口类型 POST

接口说明

```proto2
syntax = "proto2";
package douyin.extra.second;

message douyin_relation_action_request {
  required string token = 1; // 用户鉴权token
  required int64 to_user_id = 2; // 对方用户id
  required int32 action_type = 3; // 1-关注，2-取消关注
}

message douyin_relation_action_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
}
```

### /douyin/relatioin/follow/list/ - 用户关注列表

登录用户关注的所有用户列表。

接口类型 GET

接口说明

```proto2
syntax = "proto2";
package douyin.extra.second;

message douyin_relation_follow_list_request {
  required int64 user_id = 1; // 用户id
  required string token = 2; // 用户鉴权token
}

message douyin_relation_follow_list_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  repeated User user_list = 3; // 用户信息列表
}

message User {
  required int64 id = 1; // 用户id
  required string name = 2; // 用户名称
  optional int64 follow_count = 3; // 关注总数
  optional int64 follower_count = 4; // 粉丝总数
  required bool is_follow = 5; // true-已关注，false-未关注
}
```

### /douyin/relation/follower/list/ - 用户粉丝列表

所有关注登录用户的粉丝列表。

接口类型 GET

接口说明

```proto2
syntax = "proto2";
package douyin.extra.second;

message douyin_relation_follower_list_request {
  required int64 user_id = 1; // 用户id
  required string token = 2; // 用户鉴权token
}

message douyin_relation_follower_list_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  repeated User user_list = 3; // 用户列表
}

message User {
  required int64 id = 1; // 用户id
  required string name = 2; // 用户名称
  optional int64 follow_count = 3; // 关注总数
  optional int64 follower_count = 4; // 粉丝总数
  required bool is_follow = 5; // true-已关注，false-未关注
}
```

### /douyin/relation/friend/list/ - 用户好友列表

所有关注登录用户的粉丝列表。

接口类型 GET

接口说明

```proto2
syntax = "proto2";
package douyin.extra.second;

message douyin_relation_friend_list_request {
  required int64 user_id = 1; // 用户id
  required string token = 2; // 用户鉴权token
}

message douyin_relation_friend_list_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  repeated FriendUser user_list = 3; // 用户列表
}

message User {
  required int64 id = 1; // 用户id
  required string name = 2; // 用户名称
  optional int64 follow_count = 3; // 关注总数
  optional int64 follower_count = 4; // 粉丝总数
  required bool is_follow = 5; // true-已关注，false-未关注
  required string avatar = 6; // 用户头像Url
}

message FriendUser extends User {
    optional string message = 1; // 和该好友的最新聊天消息
    required int64 msgType = 2; // message消息的类型，0 => 当前请求用户接收的消息， 1 => 当前请求用户发送的消息
}
```

## 4. 消息

客户端通过定时轮询服务端接口查询消息记录

### /douyin/message/chat/ - 聊天记录

当前登录用户和其他指定用户的聊天消息记录

接口类型 GET

接口说明

```proto2
syntax = "proto2";
package douyin.extra.second;

message douyin_message_chat_request {
  required string token = 1; // 用户鉴权token
  required int64 to_user_id = 2; // 对方用户id
}

message douyin_message_chat_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
  repeated Message message_list = 3; // 消息列表
}

message Message {
  required int64 id = 1; // 消息id
  required int64 to_user_id = 2; // 该消息接收者的id
  required int64 from_user_id =3; // 该消息发送者的id
  required string content = 4; // 消息内容
  optional string create_time = 5; // 消息创建时间
}
```

### /douyin/message/action/ - 消息操作
登录用户对消息的相关操作，目前只支持消息发送
接口类型 POST
接口说明

```proto2
syntax = "proto2";
package douyin.extra.second;

message douyin_relation_action_request {
  required string token = 1; // 用户鉴权token
  required int64 to_user_id = 2; // 对方用户id
  required int32 action_type = 3; // 1-发送消息
  required string content = 4; // 消息内容
}

message douyin_relation_action_response {
  required int32 status_code = 1; // 状态码，0-成功，其他值-失败
  optional string status_msg = 2; // 返回状态描述
}
```
