背景介绍

Django是一个开放源代码的Web应用框架，由Python写成。采用了MTV的框架模式，即模型M，视图V和模版T。它最初是被开发来用于管理劳伦斯出版集团旗下的一些以新闻内容为主的网站的，即是CMS（内容管理系统）软件。并于2005年7月在BSD许可证下发布。这套框架是以比利时的吉普赛爵士吉他手Django Reinhardt来命名的。2019年12月2日，Django 3. 0发布 。目前的django版本是4.2.3

官网地址
MTV 模型Django 的 MTV 模式本质上和 MVC 是一样的，也是为了各组件间保持松耦合关系，只是定义上有些许不同，Django 的 MTV 分别是指：
● M 表示模型（Model）：编写程序应有的功能，负责业务对象与数据库的映射(ORM)。
● T 表示模板 (Template)：负责如何把页面(html)展示给用户。
● V 表示视图（View）：负责业务逻辑，并在适当时候调用 Model和 Template。除了以上三层之外，还需要一个 URL 分发器，它的作用是将一个个 URL 的页面请求分发给不同的 View 处理，View 再调用相应的 Model 和 Template，MTV 的响应模式如下所示：简易图：


用户操作流程图：


解析：用户通过浏览器向我们的服务器发起一个请求(request)，这个请求会去访问视图函数：

● a.如果不涉及到数据调用，那么这个时候视图函数直接返回一个模板也就是一个网页给用户。
● b.如果涉及到数据调用，那么视图函数调用模型，模型去数据库查找数据，然后逐级返回。视图函数把返回的数据填充到模板中空格，最后返回网页给用户。
Django 的设计思想

• DRY（Don’t repeat yourself）：不重复造轮子• MVT• 快速开发• 灵活易于扩展• 松耦合• 显式优于隐式

django适合做什么

● 内容管理系统• 博客• CMS• Wiki
● 企业内部系统• 会议室预定• 招聘管理• ERP & CRM• 报表系统
● 运维管理系统• CMDB• 发布管理• 作业管理• 脚本管理• 变更管理• 故障管理
有哪些产品在使用django



本课程的大纲



Django 开发环境准备

Python 开发环境准备 – PyCharm 介绍• PyCharm IDE，出自 JetBrains 公司，IDEA 系列产品为 JetBrains 产出的产品• 最好用的免费 Python IDE• PyCharm 有 Community 版本，有 Enterprise 版本• PyCharm 社区版不支持 Django 开发， 但可以安装 Django 类库，能够实现 Django 代码的自动提示
安装 PyCharm

下载地址
新建一个django项目

选择一个目录，并在命令行输入以下内容

django startproject recruitment
cd recruitment
python3 manage.py startapp job
python3 manage.py runserver 0.0.0.0:8000
# 修改allow host
ALLOWED_HOSTS = ['0.0.0.0']
# 创建一个超级管理管，并按照提示输入用户名、密码
python3 createsuperuser

django项目的目录结构



登录django后台





Django 的自定义模板

• Django 模板包含了输出的 HTML 页面的静态部分的内容• 模板里面的动态内容在运行时被替换• 在 views 里面指定每个 URL 使用哪个模板来渲染页面• 模版继承与块（Template Inheritance & Block） • 模板继承允许定义一个骨架模板，骨架包含站点上的公共元素（如头部导航，尾部链接）• 骨架模板里面可以定义 Block 块，每一个 Block 块都可以在继承的页面上重新定义/覆盖• 一个页面可以继承自另一个页面• 定义一个匿名访问页面的基础页面，基础页面中定义页头• 添加页面 job/templates/base.html






添加 URL 路径映射

• 让添加的页面，能够通过 URL 访问到• /joblist/ 的路径访问到 views 里面定义的 joblist 视图• 这个视图时一个 Method View，方法表示一个视图



列表展示

● 模板添加定义，View 页面添加完，URL 中也定义路由之后，再访问页面：
● http://127.0.0.1:8000/joblist/

配置日志记录

● Django 里面使用 dictConfig 格式来配置日志
● Dictionary 对象，至少包含如下内容:
    ○ version, 目前唯一有效的值是 1
    ○ Handler, logger 是可选内容，通常需要自己定义
    ○ Filter, formatter 是可选内容，可以不用定义
● 定义日志输出格式， 分别定义 全局日志记错， 错误日志处理， 自定义的 日志处理器

发送通知：钉钉群消息集成

• 安装钉钉聊天机器人： pip install DingtalkChatbot• 测试群消息• python ./manage.py shell --settings=settings.local• from interview import dingtalk• dingtalk.send("秋季招聘面试启动通知, 自 2020/09/01 开始秋季招聘")• 定义 通知面试官的方法def notify_interviewer(modeladmin, request, queryset)• 注册到 modeladmin中actions = (export_model_as_csv, notify_interviewer, )

django的中间件

● 什么是中间件 Middleware ?
● 注入在 Django 请求/响应 处理流程中的钩子框架，能对 request/response 作处理
● 广泛的使用场景
    ○ 登录认证，安全拦截
    ○ 日志记录，性能上报
    ○ 缓存处理，监控告警….
● 自定义中间件的 2 种方法
● 使用函数实现
● 使用类实现
示例创建一个请求日志，性能日志记录中间件

输出到 Response Header 里面的 X-Page-Ducration-ms

Sentry 集成

解决的问题： 应用的错误，异常监控统计，报警通知；性能监控统计，对问题进行跟踪Sentry 架构之美• 开放的架构：可与 AD 域账号集成，与 Google/Stackoverflow 等账号集成• 有完善的插件支持：Webhook/Gitlab/Jira/Slack/PushOver/....• 支持不同环境（开发、测试、预发、线上）；可以配置灵活的告警• 跨平台，跨端的支持：Python/Java/JavaScript/Ruby/Go/..., Android/iOS/Web/...• API 简单、易用，自动集成；安装简单：架构依赖多，但使用 Docker 可以一个命令安装• 自动对错误，异常进行统计聚合，按照上下文的Tag进行聚合• 可以对性能进行统计分析，可抽样;可视化的趋势分析• 多租户，支持双因素认证，敏感内容自动脱敏

* 两种方法安装 Sentry ： 
    * 使用 Docker 官方服务（量大需要付费，使用方便）；
    * 自己搭建 服务（从源码安装，或者使用docker 搭建服务）；
* 使用 Docker 来安装 sentry, 使用 release 版本
    * https://github.com/getsentry/onpremise/releases
    * ./install.sh
    * docker-compose up -d 
* Django 配置集成 sentry ， 自动上报未捕获异常， 错误日志

集成效果

告警趋势可视化: Prometheus & Grafana

Prometheus 数据类型• Counter：计数器，总是增长的整数值；请求数，订单量，错误数等；• Gauge：可以上下波动的计量值，比如温度，内存使用量，处理中的请求；• Summary：提供观测样本的摘要，包含样本数量，样本值的和；滑动窗口计算:请求耗时，响应数据大小• Histogram：把观测值放到配置好的桶中做统计，请求耗时，响应数据大小等；



Django 中使用缓存

● Django 缓存的存储方式
  * Memcached 缓存
  * Redis 缓存 （需要安装 django-redis 包）
  * 数据库缓存
  * 文件系统缓存
  * 本地内存缓存    
  * 伪缓存( Dummy Cache， 用于开发、测试) • 自定义缓存
    

Celery 集成：Celery 的使用

● 什么是 Celery?
● 一个分布式的任务队列
    ○ 简单： 几行代码可以创建一个简单的 Celery 任务
    ○ 高可用：工作机会自动重试
    ○ 快速：可以执行一分钟上百万的任务
    ○ 灵活：每一块都可以扩展


Celery 中的核心概念

• Task: 一个需要执行的任务，任务通常异步执行• Period Task: 需要定时执行的任务，定时一定间隔执行，也可以使用 crontab 表达式设定执行周期和时间点• Message Broker: 消息代理，临时存储，传输任务到工作节点的消息队列。可以用 Redis,RabbitMQ, Amazon SQS 作为消息代理。消息代理可以有多个，以保障系统的高可用。• Worker：工作节点，执行任务的进程，worker可以有多个，保障系统的高可用和扩展性。

Celery 使用场景

大量需要使用异步任务的场景• 发送电子邮件，发送 IM 消息通知• 爬取网页， 数据分析• 图像、视频处理• 生成报告，深度学习



Flower: 一个实时的 Celery 任务监控系统



Django之美：Signals信号及其使用场景

什么是 Signals?• Django 的信号• Django 框架内置的信号发送器，这个信号发送器在框架里面• 有动作发生的时候，帮助解耦的应用接收到消息通知• 当动作发生时，允许特定的信号发送者发送消息到一系列的消息接收者• Signals 是同步调用


信号的应用场景• 系统解耦；代码复用：实现统一处理逻辑的框架中间件； -> 可维护性提升• 记录操作日志，增加/清除缓存，数据变化接入审批流程；评论通知；• 关联业务变化通知，• 例：通讯录变化的异步事件处理，比如员工入职时发送消息通知团队新人入职，员工离职时异步清理员工的权限等等；

生产环境中的安全：应用安全

访问限流 – 防恶意攻击• Rest Framework API 限流• 应用限流: 对页面的访问频次进行限流

可以对匿名用户，具名用户进行限流可以设置峰值流量（如每分钟60次请求）也可以设置连续一段时间的流量限制（比如每天3000次）



云环境中的部署：容器的基础用法 – Docker 容器介绍

● Docker:
    ○ 码头工人，轻量级的，可移植，自包含的容器，来自动化、版本化应用的发布。
    ○ Docker上跑的容器是一个个的集装箱；
● Docker的基础是LXC
    ○ LXC用于应用程序的隔离，每个应用程序分配独立的命名空间，隔离的CPU, 内存，磁盘，网络资源
    ○ 每个应用内部可以单跑一套容器系统，功
    能上相当于传统的虚拟机，但本质上是内核层面对资源的隔离
● Docker 容器的分层和版本管理
    ○ Docker把应用和系统打包到一起（image镜像），进行版本化管理。
    ○ 应用之于Docker，如同代码之于Git/SVN，一个命令可以把应用部署到docker上。
容器的基础用法






持续集成：CICD的工作流程






