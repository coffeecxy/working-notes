zoomApi
===

zoom.us提供了一个可以使用的rest api来完成我们想要的视频聊天的操作。

## meeting api
在提供的所有类型的api中，对于Meeting的API是最重要的。

在zoom中，meeting分成三类，用`type`表示，`1`表示立即的meeting，`2`表示预约的，`3`表示可以重复的。

购买的zoom的API是有要求的，一个用户只能同时有200个meeting在进行中，为了最大化的使用meeting资源，我们只在会议要进行的时候才打开一个会议

最开始想使用可以重复的会议，但是使用的时候发现其给出的startUrl在一个新的浏览器窗口中打开之后，虽然会不断的被更新，但是如果有网络的抓包工具，那么从最开始就被抓下来了信息就可以得到这个url，而这个url可以被重新打开，这样就造成了一个漏洞。

而对于立即的meeting，只要被结束了以后，在一个新的浏览器窗口中用startUrl去打开，那么会说这个meeting已经不存在了。

所以，都使用立即的meeting。

使用立即的meeting会遇到的问题是，这些meeting不能够被/meeting/list给显示出来，如果需要对这些meeting进行管理，那么需要使用/meeting/get来得到。

在下面的所有样例中，默认都给出了api key, api secret, data type的

### 新建一个Meeting

	https://api.zoom.us/v1/meeting/create
可以给出的参数。
host_id: 必须给出。给出这个meeting的host id，这个host id可以是当前这个API partner账户创建的任何用户。**创建之后就不可以被更改了**。

topic: 必须给出。 这个meeting的主题。**最多可以使用300个字符**，一般的，我们不会给出超过300个字符的主题。

type： 必须给出。 会议的类型。 上面有给出解释，我们使用1类型，也就是立即类型。

start_time,duration,timezone: 这三个只能用在预约的会议中，表示开始时间，会议持续的时间，用户的时区。

password： 如果对会议进行了加密，那么只有输入了正确密码的才能进入。

option_jbh: 默认是false。 是否允许在host开启了会议之前就加入会议，因为zoom都是使用其客户端进行聊天的，这个设置是不是在host连入了网络才准许其他的用户连入，我们就使用默认值，也就是只有提供服务的人先连接进入了，才让接收服务的人连入。

option_start_type: 默认为video。 会议开始之后使用的默认的服务方式，可以使用视频video，也可以是桌面分享screen_share。 这个属性是过时的了，所以不给出就是了。 现在zoom的客户端会自动识别了，如果使用的主机上面使用摄像头的，那么其会开启视频，如果没有摄像头，那么其会进行桌面共享，所以是智能的了。

option_no_video_host: 默认为false。 当Host加入会议时，是否开启其视频。默认时要开启的，在我们的场景中，使用默认设置。

option_no_video_participant: 默认为false。 当Host加入会议时，是否开启其视频。默认时要开启的，在我们的场景中，使用默认设置。

option_audio: 默认为false。 会议中使用的音频，可以使用both，表示都使用，telephony或者是voip。这个就使用默认。

	{
    	"uuid": "dPl7dtK3QhCcSNzueLqCNw==",
	    "id": 167148267,
	    "host_id": "uYNWNMS0ReO9e0HKbpcJrg",
	    "topic": "adf",
	    "password": "",
	    "status": 0,
	    "option_jbh": false,
	    "option_start_type": "video",
	    "option_host_video": true,
	    "option_participants_video": true,
	    "option_audio": "both",
	    "type": 1,
	    "start_time": "",
	    "duration": 0,
	    "timezone": "",
	    "start_url": "https://www.zoom.us/s/167148267?zpk=p4RvqbfbXBYXr_ifMHClReg8zmNEXY5nYYx-z7ptWvs.BwUAAAFLIVbV4AAAHCAkMmI4MWFkNzQtMTY5ZC00MDE1LTg1NDktYjNkYWI0ZDMyZTE2FnVZTldOTVMwUmVPOWUwSEticGNKcmcWdVlOV05NUzBSZU85ZTBIS2JwY0pyZw1YaW55b25nIENoZW5nZACzXzRFSUJnU05OUkZQTXloMjA5Xy0yRm1maTlwb0d6NC1DYUFTMXotUDlWWS5CZ0lnUTA1SVZVdHZVMkpaYm5JMlNHczNRamwzVDFkMlRFRnBObG8yVWxCUFNYcEFOVFJoTlRrd01tVmpZV000TVRWbE16RXlZelprWVdabU5tUXpNVE5oTWpCa05tRXdaVFkyTURNellUUmhOR1EyTm1Jd1lUazBPREl4TldFd016ZzNOUUEAABZ2WGFJVFU0NlMyV2d1Nm1CZ2dCX19RAQI",
	    "join_url": "https://www.zoom.us/j/167148267",
	    "created_at": "2015-01-25T13:45:13Z"
	}

这个rest api返回的是如上的json，注意这个是成功返回的，如果是返回失败了其中其只有一个`error`字段。

在返回的JSON中，最重要的几个字段是

`id`，表示这个meeting的标识符。

`start_url`和`join_url`是要开始这个会议的Host和一般用户需要打开的窗口。

`status`表示当前会议的状态，`0`表示还没有开始，`1`表示正在进行中，`2`表示已经结束了。
对于立即的会议，如果结束了，那么其就不能再被开启了。如果需要的话，只能重新创建一个会议再开启了。

其他的字段就是发送过去的样子了。

### 删除一个会议

	https://api.zoom.us/v1/meeting/delete

删除一个已经创建的meeting。
对于立即的会议，当其被结束了之后，其就被删除了，这个api删除的是预约的和重复的meeting。

需要给出的是meeting的`id`和`host_id`。

### 列出所有的会议

	https://api.zoom.us/v1/meeting/list

列出所有的会议。
列出的是现在还存在的预约的和可重复的会议。

需要给出`host_id`，其只列出属于这个host的会议。

### 得到一个会议的信息

	https://api.zoom.us/v1/meeting/get
	
需要给出`id`和`host_id`。

其会给出这个会议的信息。

得到的数据和`/meeting/create`得到的是一样的。

在我们使用的场景中，会用来得到一个会议的状态。

### 更新一个会议的状态

	https://api.zoom.us/v1/meeting/update
	
需要给出`id`和`host_id`。

可以被更新的状态为在创建时的那些信息。

我们可能会用到这个API来操作正在进行的会议，比如说增加其时长。

这儿要仔细的考察一下预约的会议的情况，有可能我们更适合使用的是预约的会议。

经过我的测试，发现对于一个预约的会议，在预约时间没有到的时候，也可以开始，感觉这个地方有一些奇怪的。
这个地方的实际情况是这样的，对于一个预约的会议，如果时间没有到，但是host开启了这个会议，那么其他人也是可以加入的。但是主持人没有开启，那么加入在加入的时候就会被提醒说会议还没有开始。

总之，主持人的权限是很大的，他可以选择提前开始，准时开始，或者是根本就不开启。不管什么时候，参与者只有在主持人开启了会议的时候才能加入，不然都会被提示说等待主持人开启会议。

所以，预约的会议对我们来说也是不合适的。

### 结束一个会议

也就是删除一个会议的意思。

需要给出`id`和`host_id`。

这个是用来删除一个会议的，当然，如果一个会议正在进行，那么其肯定会被强制结束了。

基于上面给出的接口，我们可以实现自己的会议的逻辑，然后在提供出去一个REST API接口。