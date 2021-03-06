## 概述
这是一个基于“兴趣”的陌生人间的社交网络。

具有瀑布流式的UI, 对于其上的每条消息(动态/广播), 可以标记“喜欢”或“不喜欢”，也可以进行评论，与发布者进行(公开式)的互动。
瀑布流上的消息全部由系统根据用户的喜好自动推进，用户只能通过标记“喜欢”和“不喜欢”进行干涉。

该系统的目的在于帮助用户发现感兴趣的话题，发现志同道合的朋友，起到基于兴趣的陌生人社交的作用。

## 架构
出于尝试新技术的目的，这次打算做到前后端彻底分离，仅通过下文制定的JSON API进行沟通。

## API 约定
所有的API都是POST方式请求，除个别请求外，所有API的访问需要附带token。

公共返回格式：

    {"error": <true/false>, "error_msg": <错误ID>, "data": <实际数据>}  

如果未出错，`data`中是返回数据，否则`data`中是错误的详情，两种情况都可能为空。  
`error_msg`是类似"not_login"的错误ID.

术语：

* 消息(post)，信息流中的一条消息，等价于动态、微博、广播的概念
* 回复(reply)，对消息的一条回复，等价于评论的概念

## API
### /api/account/signup  
INPUT: {"username": <用户名>, "email": <邮箱>, "passwd": <密码>}  
ERROR: "bad_username", "bad_email", "bad_passwd", "duplicate_username"
不需要token

### /api/account/login  
INPUT: {"username": <用户名>, "passwd": <密码>}  
OUTPUT: {"token": < 64字符长的token > }  
ERROR: "failure"  
不需要token

### /api/account/logout  
INPUT: {"token": < token >}  
ERROR: "unknown_token"

### /api/account/get_user
INPUT: {"username": <用户名>}  
OUTPUT: {'created_at': <UNIX时间戳>, 'email': <邮箱>, 'contact': <JSON>}  
ERROR: "failure"  
不需要token

### /api/account/check_token
INPUT: {"token": < 64字符长的token >}  
OUTPUT: {"username": <用户名>}  
ERROR: "failure"  

### /api/post/new (发新消息)   
INPUT: {"content": <消息内容(HTML)>}  
OUTPUT: {"id": <新消息的ID>}


### /api/post/reply  (回复消息)  
INPUT: {"content": <消息内容(MD格式)>, "reply_post": <回复所在的消息>, "reply_to": <如果是在回复另一条回复，这里填ID，否则为零>}  
OUTPUT: {"id": <新回复的ID>}

 
### /api/post/fetch_by_number  (以数量抓取信息流)  
INPUT: {"num": <数量>}  
OUTPUT: {"result": [<消息ID的数组>]}

### /api/post/fetch_by_last_post  (以最后一条ID抓取信息流)  
INPUT: {"id": <ID>}  
OUTPUT: {"result": [<消息ID的数组>]}

### /api/post/get_post  (获取一个消息的详情)  
INPUT：{"id": <消息ID>}  
OUTPUT: 

     {
         "content": <评论内容(HTML)>, 
         "time": <发表时间>, 
         "replys": <评论数>, 
         "author":{
             "id": <用户ID>, 
             "username": <用户名>, 
             "avatar": <用户头像>}
         }
    }

### /api/post/get_replys  (获取一个消息下的评论)  
INPUT: {"id": <消息ID>}  
OUTPUT: 
     {
        "result": [{
            "content": <评论内容(HTML)>, 
            "time": <发表时间>, 
            "author":{
                "id": <用户ID>, 
                "username": <用户名>, 
                "avatar": <用户头像>}
            }, <更多评论>
        ]
    }

### /api/post/markup   (标记消息)
INPUT: {"id": <消息ID>, "type": <类型，取值：like, dislike, spam>}  