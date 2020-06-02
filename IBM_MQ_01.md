---
title: IBM MQ 使用 01篇
---

###### MQ的部分属性
队列管理器名称、队列、进程、名称列表、群集、认证信息对象   -》 最长48个字符
通道名称 -》 最长20个字符

队列名称区分大小写
所有控制命令区分大小写

> DESCR:
MAXMSGL:消息大小
队列管理器 -》 默认：4M 可调整范围 -》 32K ~ 100M
通道 -》 默认：4M 可以调整范围 -》 0 ~ 队列管理器的MAXMSGL
队列 -》 默认：4M 可调范围 -》 0 ~ 队列管理器的MAXMSGL

##### 动作01 使用MQ来完成单通道 发布 - 订阅 



##### 动作02 使用MQ来完成远端消息投递 - 进程间消息通讯(非集群)

当使用MQ来完成多方应用的进程间的沟通时，需要参与沟通应用方具备远程沟通的能力 - 即拥有远程沟通工具MQ，因为应用方无法直接连接消息服务器，需要借助MQ来将需要发送的消息传递至目标消息服务器，完成应用间进程的消息对话。

> 消息服务器配置方式

IBM-MQ消息服务配置指令(MQSC指令,指令均在mqm用户下执行)

1. 创建消息管理器
crtmqm WSMQ1

2. 启动消息管理器
strmqm WSMQ2

3. 通过MQSC指令创建队列、通道以及监听
    a. 创建配置文件  vim WSMQ1.conf
    b. 定义本地队列
        DEFINE QLOCAL( ***'Queue1'*** ) DEFPSIST(YES) MAXDEPTH(100) REPLACE
    c. 定义远程队列
        DEFINE QREMOTE( ***'RQueue1'*** ) RNAME( ***'Queue2'*** ) RQMNAME( ***'WSQM2'*** ) XMITQ( ***'TransQ1'*** )
    d. 定义传输队列
        DEFINE QLOCAL( ***'TransQ1'*** ) USAGE(XMITQ) DEFPSIST(YES) INITQ(SYSTEM.CHANNEL.INITQ) TRAGDATA( ***'CHAN_QMGR1_TO_QMGR2'*** ) TRIGTYPE(FIRST) TRIGGER REPLACE
    e. 定义发送通道
        DEFINE CHANNEL( ***'CHAN_QMGR1_TO_QMGR2'*** ) CHLTYPE(SDR) CONNAME( ***'192.168.43.98(1418)'*** ) XMITQ( ***'TransQ1'*** )
    f. 定义接收通道
        DEFINE CHANNEL( ***'CHAN_QMGR2_TO_QMGR1'*** ) CHLTYPE(RCVR)
    g. 定义服务器连接通道
        DEFINE CHANNEL(CHAN_SERVER_CON) CHLTYPE(SVRCON) REPLACE
    h. 定义死信队列
        DEFINE QLOCAL( ***'QDEAD1'*** ) DEFPSIST(YES) MAXDEPTH(100) REPLACE
    i. 变更消息管理其的死信队列
        ALTER QMGR DEADQ( ***'QDEAD1'*** )
    j. 定义消息监听器
        DEFINE LISTENER( ***'LISTENER.TCP'*** ) TRPTYPE(TCP) CONTROL(QMGR) PORT( ***1416*** ) REPLACE
    k. 启用消息监听器
        START LISTENER( ***'LISTENER.TCP'*** )
4. 命令WSMQ1读入配置文件并记录执行日志
runmqsc WSMQ1 < WSMQ1.conf > qcreate_WSMQ1.log

5. 需要通信的应用在其消息服务器上执行以上步骤，替换加粗内容

[说明]
针对以上步骤中的指令做如下说明：

* 远程队列定义中，RNAME指明目标端对应的队列，RQMNAME指明目标端对应的队列管理器，XMITQ指明远程队列所使用的传输队列
* 发送通道定义中，CONNAME指明目标端队列管理器的地址以及监听端口
* 使用死信队列用以处理消息无法被传送至目标端时的情况。

针对以上，其消息传递过程可描述如下：
当APP1需要与APP2对话时，双方对话内容通过自己的MQ来完成。
APP1 -> APP2:
APP1端远程发送队列，指定了需要和APP2端的哪个消息管理器下的哪个队列进行沟通，并明确自己需要采用哪个传输队列来进行信息交流。

沟通过程：
APP1通过连接CHAN_SERVER_CON服务器通道将消息放入远程队列，远程队列收到消息后将消息放入关联的传输队列，传输队列收到消息后，触发器会产生触发消息，通过发送通道将消息发送至目标端，即APP2端。APP2端的接收通道中收到消息后，通过识别消息属性中的目标队列名称，将其投递至对应的本地队列中去。

APP2通过连接CHAN_SERVER_CON服务器通道来获取指定本地队列上APP1发送来的消息。

以上，完成APP1与APP2的进程消息沟通。

[注] ***在发送通道中所指定的目标端ip、port以及发送通过名称需要和目标端的ip、port以及接收通道名称一致***

###### 部分属性注释

> 队列管理器

1. MAXMSGL - 最大消息长度： 输入队列管理器上的队列所允许的最大消息长度，范围是32kb至100mb，默认为4mb。若减小队列管理器的最大消息长度，那么还必须减小SYSTEM.DEFAULT.LOCAL.QUEUE的定义以及连接至此队列管理器的所有其他队列的最大消息长度。

> 队列

1. DEFPRTY - 缺省优先级： 输入放入队列的消息的缺省优先级，范围是0~9，0是最低优先级
2. USAGE - 用法： 若定义队列为本地队列，应选择值为正常；若定义队列为传输队列，应选择值为传输



ALTER CHL(SYSTEM.DEF.SVRCONN) CHLTYPE(SVRCONN) MCAUSER('mqm')

START CHL(SYSTEM.DEF.SVRCONN)

ALTER AUTHINFO(SYSTEM.DEFAULT.AUTHINFO.IDPWOS) AUTHTYPE(IDPWOS) CHCKCLNT(OPTIONAL)

ALTER QMGR CONNAUTH(' ') 

REFRESH SECURITY TYPE(CONNAUTH)

##### 关于IBM MQ V7.0以后通道访问权限问题

在开发环境中可以通过临时关闭通道权限认证功能以保证连接测试，进入MQSC下执行如下指令：
ALTER QMGR CHLAUTH(DISABLED)

正式生产环境中不建议采用此操作。

##### 正式环境下配置IBM MQ服务通道

***若要在客户端和IBM MQ之间进行通信，必须配置服务器连接通道。***

新的通道具备如下优势：
* 可以在IBM MQ队列管理器端可以轻松的识别客户端的活跃通道
* 客户端与IBM MQ之间的连接变得更加安全
* 创建CHLAUTH RECORDS将客户端的用户标识映射到IBM MQ系统上相应的用户标识

###### 创建服务器连接通道

1. 使用MQSC命令为队列管理器创建服务连接通道

* 为不受保护的客户端连接创建服务连接通道
> DEFINE CHANNEL(channelName) CHLTYPE(SVRCONN) TRPTYPE(TCP)

* 为受保护的客户端连接创建服务连接通道
> DEFINE CHANNEL(channelName) CHLTYPE(SVRCONN) TRPTYPE(TCP) SSLCIPH(cipherSpec)

2. 使用MQSC命令为队列管理器创建侦听器并启用(未使用现有侦听器)

> DEFINE LISTENER(listenerName) TRPTYPE(TCP) CONTROL(QMGR) PORT(portNumber)
> START LISTENER(listenerName)

3. 如果使用了SSL连接，则必须通过创建密钥库和证书来确保客户端的连接安全。

***以下内容使用CHLAUTH规则来保护服务连接通道的安全***

1. 在运行队列管理器的系统上获取一个用户标识(即创建一个普通用户)，该用户不能是特权用户。

Linux上管理员用户执行：useradd xiaoyu，用户创建结束使用passwd命令为其创建密码

2. 为每一个客户端IP地址创建通道认证记录，每条通道认证记录必须允许只有客户端IP才能使用已创建的服务连接通道来连接客户端和IBM MQ

> SET CHLAUTH(channelName) TYPE(ADDRESSMAP) ADDRESS(IPAddress) MCAUSER('xiaoyu')

3. 将常规IBM MQ接入权限授予创建用户
> SET AUTHREC OBJTYPE(QMGR) PRINCIPAL('xiaoyu') AUTHADD(CONNECT, INQ, DSP)

4. 授予创建用户可以请求IBM MQ的权限
> SET AUTHREC PROFILE('SYSTEM.DEFAULT.MODEL.QUEUE') OBJTYPE(QUEUE) PRINCIPAL('xiaoyu') AUTHADD(DSP, GET)

> SET AUTHREC PROFILE('SYSTEM.ADMIN.COMMAND.QUEUE') OBJTYPE(QUEUE) PRINCIPAL('xiaoyu') AUTHADD(DSP, PUT)

5. 授予创建用户对于客户端创建IBM MQ队列同步记录的权限
> SET AUTHREC PROFILE('SYSTEM.IMA.*') OBJTYPE(QUEUE) PRINCIPAL('xiaoyu') AUTHADD(CRT, PUT, GET, BROWSE)

> SET AUTHREC PROFILE('SYSTEM.DEFAULT.LOCAL.QUEUE') OBJTYPE(QUEUE) PRINCIPAL('xiaoyu') AUTHADD(DSP)

[注] 为了同步客户端与IBM MQ之间的信息传输，将创建一个名为以SYSTEM.IMA起始的队列，这个队列用于存储客户端与IBM MQ之间传输的信息。

6. 对于每条映射到或者来自IBM MQ的主题的映射规则，必须创建主题对象并且为其授予特定的权限。对于每条映射到或者来自IBM MQ的队列的映射规则，必须创建队列并为其授予特定的权限。

> DEFINE QLOCAL(queueName)

> SET AUTHREC PROFILE(queueName) OBJTYPE(QUEUE) PRINCIPAL('xiaoyu') AUTHADD(authorization)

> DEFINE TOPIC(topicObjectName) TOPICSTR(topicString)

> SET AUTHREC PROFILE(topicObjectName) OBJTYPE(TOPIC) PRINCIPAL('xiaoyu') AUTHADD(authorization)

###### USERLIST 用户清单

最多一百个用户ID的列表，这些ID禁止使用此通道。使用特殊值*MQADMIN标识特权或管理用户。该值的定义取决于操作系统，如下所示：
* 在Windows上，mqm组，Administrator组和SYSTEM的所有成员
* 在UNIX和Linux上mqm组的所有成员
* 在IBMi上，概要文件(用户)qmqm和qmqmadm以及qmqmadm组的所有成员，以及使用* ALLOBJ特殊设置定义的任何用户
* 在z/OS上，运行通道启动器，队列管理器和消息高级安全地址空间的用户ID

IBM MQ支持传输层安全性(TLS)和安全套接字层(SSL)协议，为消息通道和MQI通道提供链路级安全性。

###### 通道身份验证

创建通道认证记录可以执行以下功能：

* 阻止来自特定IP地址的连接
* 阻止来自特定用户的ID
* 设置MACUSER值以用于从特定IP地址连接的任何通道
* 设置MACUSER值以用于断言特定用户ID的任何通道
* 设置MACUSER值以用于允许特定的SSL或TLS专有名称
* 设置MACUSER值以用于来自特定队列管理器的任何通道
* 阻止来自确定的队列管理器声明的连接，除非连接来自指定的IP地址
* 阻止已存在的确定的SSL或TSL证书的连接，除非连接来自指定的IP地址

> 通道身份验证记录之间的交互

* 通道名称显示匹配的通道身份验证记录优先于使用通配符匹配的通道名称的身份验证记录
* 使用SSL或TLS DN的通道身份验证记录优先于使用用户ID、队列管理器名称或IP地址的记录
* 使用用户ID或队列管理器名称的通道身份验证记录优先于使用IP地址的记录
* 如果找到匹配的通道身份验证记录，并且指定了MCAUSER，则会将该MCAUSER分配给该通道
* 如果找到匹配的通道身份验证记录，并且该记录指定该通道无权访问，则通道的MCAUSER值将被标记为*NOACCESS，这个值稍后可以通过安全出口程序修改
* 如果找不到匹配的通道身份验证记录，或者找到了匹配的通道身份验证记录，并且指定要使用通道的用户ID，则检查MCAUSER字段
    1. 如果MCAUSER字段为空，则将客户端用户ID分配给该通道
    2. 如果MCAUSER字段不为空，则将其分配给通道
* 所有的安全出口程序都会运行。该退出程序可能会设置通道的用户ID或者决定拒绝访问
* 如果连接被阻止或MCAUSER设置为* NOACCESS，则通道结束
* 如果未阻止连接，则对于除客户端通道以外的任何通道，都将根据已阻止用户的列表检查在先前步骤中确定的通道用户ID
    1. 如果用户ID在被阻止的用户列表中，则通道结束
    2. 如果用户ID不在阻止的用户列表中，则通道运行

> 识别和认证用户

对于每个队列管理器，可以选择不同类型的身份验证信息对象以对用户ID和密码进行身份验证。认证信息对象有两种：

***DEFINE AUTHINFO(USE.OS) AUTHTYPE(IDPWOS)***
* IDPWOS用于指示队列管理器使用本地操作信息来认证用户ID和密码，如果选择使用本地操作系统，则只需设置公共属性

***DEFINE AUTHINFO(USE.LDAP) AUTHTYPE(IDPWLDAP) CONNAME('ldap1（389）， ldap(389)') LDAPUSER('CN=QMGR1') LDAPPWD('passwOrd') SECCOMM(YES)***
* IDPWLDAP用于指示队列管理器使用LDAP服务器来验证用户名和密码。




