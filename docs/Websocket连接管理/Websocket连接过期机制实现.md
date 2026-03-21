# Websocket连接过期机制实现

## 实现计划

1. 用户连接内容context 中增加“exp“和"jti"字段，每次用户连接到websocket路由时，检验解析token后写入字段

2. 每次用户发送message到websocket 路由时，解析connection的context内容，获取"jti"和“exp”字段，先比较jti后再与当前时间now做比较，过期则断开连接

3. 添加兜底保障，每次用户连接后添加一个定时器，过期时间到达时自动断开连接（如果连接未断开）



## 具体实现

#### 首先我们要改造connection存储的信息

这个信息也就是context。原来只存储用户的uid信息，现在我们加入token过期时间点expiry，以及conn唯一的timer对应的timer_id，在用户提前断连时回收计时器

```cpp
struct ConnectionContext
{
    std::string uid;
    trantor::TimerId timer_id;
    std::chrono::time_point<std::chrono::system_clock> expiry;
};
```

#### 第一个改造：

我们在发送消息的路由入口增加检查，检查很简单，只要把conn里存储的expiry和当前时间点做比较，过期就断连即可

```cpp
if (conn_info->expiry > std::chrono::system_clock::now())
	if (!Container::GetInstance().GetConnectionService()->RemoveConnection(conn));

```

#### 第二步

我们在接入新连接时启动定时器；这里我们把定时器启动的逻辑加在connection service层，因为timer是给conn断连专用的，这样处理更加统一，逻辑不会分散
因为新的连接接入时，要检验token，这个时候我们可以顺便把expiry提取出来，然后作为timer的计时参数
我选择在AddConnection的加锁范围内启动定时器，原因是我希望定时器启动前对conn的其他操作不会干涉到这个环节，因为在没启动之前context的timer_id是没被初始化的，这时候如果用户断连，会调用到removeConnection，回收timer的话因为没有启动定时器，显然这么做是不合理的
另外在启动定时器之前我增加了一个0~3秒之间的随机小抖动，防止大量连接同时断连给服务器负载过大，增加了一个工具函数 Utils::GetRandomJitter()

```cpp
auto delay = static_cast<double>(std::chrono::duration_cast<std::chrono::seconds>
	(expiry - std::chrono::system_clock::now()).count())+ Utils::GetRandomJitter();if (conn&& conn->connected())
{
	timer_id = drogon::app().getLoop()->runAfter(delay, [weak_conn = std::weak_ptr(conn), self = shared_from_this()]()
		{
			if (weak_conn.lock() && weak_conn.lock()->connected())
			{
				self->RemoveConnection(weak_conn.lock());
			}
			LOG_ERROR << "Timer not reclaimed in time: "<<weak_conn.lock()->getContext<ConnectionContext>()->timer_id;
		});
}

```

启动前我还是检查了一下，如果断连就不启动timer，避免资源浪费。**<u>这里最重要的一点在于，捕获变量时我使用了weak_ptr，这样不会增加引用计数，因为用户可能会提前断连，这样的话，如果直接使用shared_ptr的话会增加引用计数，导致必须等到定时器结束时才会释放conn</u>**

#### 第三步

这里我们增加removeConnection的回调函数的逻辑，在断连时要执行回收的操作

```cpp
drogon::Task<> ConnectionService::OnUserDisconnected(std::string uid, trantor::TimerId timer_id) const
{
	drogon::app().getLoop()->invalidateTimer(timer_id);
	co_await _redis_service->SetOffline(uid);
}
```

虽然框架会帮我们自动回收，但是在用户主动断连的情况下，我们也要及时把定时器回收，避免资源的浪费










