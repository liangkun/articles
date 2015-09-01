API定义
========
本章定义Arbiter服务的API。

Web Service API
---------------
Arbiter第一期提供的是基于Web服务的RESTful API。本节详细定义这一组API。Arbiter使用JSON作为数据的标准交换格式。
该文档会随着功能API的增加不断完善。

在设计具体的API之前，我们先来看两个适用于所有API的设计：API版本号和结果分页。

### API版本号 ###
所有的API都是随时间演进变化的，换句话说，API是有版本的。在Arbiter中，我们主要使用HTTP请求和响应中的Media Type来
指定API的版本。

对于请求，如果没有指定Media Type，则默认访问当前的最新API版本。如果一个客户端想要访问某个特定的API版本，它应该以
如下格式指定Media Type：

`Accept: application/vnd.arbiter[.version]`

其中，version本身也是一个可选的部分，没有的话就是当前最新版本。可选的版本包括

-   `current`, 当前最新版本。
-   `v1, v2, v3...`，某个特定的版本。版本号都是整数。

在响应消息头中，Arbiter始终会携带 **当前最新** 的版本信息，如下格式：

`X-Arbiter-Media-Type: vnd.arbiter.v1`

上面这种携带API版本号的方法是学习
[Github Web API Version] (https://developer.github.com/v3/media/#request-specific-version)的。

### 结果分页 ###
首先，我们应该尽量避免通过HTTP返回一个巨大的结果集合。但有时候这是无法绕过的，对于API逻辑而言，其实也是合理的。比如，
如果我们要返回一个Job的所有Task列表。此时，我们通过分页来返回结果。这个思路来自于一个简单的想法：多个快速的请求比一个
很慢的请求更能适应不同的情况。再一次，我们向
[Github Web API Pagination] (https://developer.github.com/guides/traversing-with-pagination/)学习。

通过响应头中的`LINK`字段即相关协议来提供分页信息。用户的请求中可以指定`per_page`参数，设置服务器响应使用的每页结果
数目。如果请求中没有这个参数，那么这个参数将由服务器决定。用户请求中可以指定`page`参数，告诉服务器当前请求第几页。如果
没有指定，默认请求第一个页面。页号从`1`开始编号。

如果当前响应的是第一个页面，则：

    LINK: <http://host/path/to/resource?otherparams&page=2>; rel="next",
          <http://host/path/to/resource?otherparams&page=34>; rel="last"

如果当前响应的是最后一个页面，则：

    LINK: <http://host/path/to/resource?otherparams&page=1>; rel="first",
          <http://host/path/to/resource?otherparams&page=33>; rel="prev"

如果当前响应的是中间某个页面，则：

    LINK: <http://host/path/to/resource?otherparams&page=1>; rel="first",
          <http://host/path/to/resource?otherparams&page=15>; rel="prev",
          <http://host/path/to/resource?otherparams&page=17>; rel="next",
          <http://host/path/to/resource?otherparams&page=34>; rel="last"

看上去有些冗余，而且这些信息其实是可以放到response的JSON数据结构中的。不过，既然Github v3版本的接口这样设计了，
估计是经过仔细考虑绕过了一些坑的。(隐约觉得这样做，包括冗余的信息和把这些信息放到响应头的LINK字段中而不是响应体的JSON
中，存在某些内在的一致性。比如，分页处理的代码不用考虑这是哪个API，只要一份逻辑就可以了。）

### 通用数据格式 ###
-   时间格式: YYYYMMDD-hhmmss
-   空字段表示：null，不会缺失。
-   所有的序列编号从1开始。

### 认证与权限 ###
这部分第二期再设计吧。[参考](http://kufli.blogspot.hk/2013/08/sprayio-rest-service-authentication.html)

### URIs ###
这里，我们先定义这个RESTful服务中的主要实体。

-   `/jobs`：包含了所有job的信息。
-   `/jobs/{job_name}`：这个路径指向了一个特定的job，当前版本。job_name是一个全限定的名字，如"a.b.c.job"。
-   `/jobs/{job_name}/tasks`：一个job当前版本的所有的task信息。
-   `/jobs/{job_name}/tasks/{time}/{seq_number}`：一个job的当前版本的指定时间的第seq_number次执行的task
     信息。
-   `/jobs/{job_name}/{seq_number}`：一个job的所有历史版本的信息都会被保存在一个版本号下，版本号是整数，从1
    开始。
-   `/tasks`：包含了所有task的信息，这是tasks在状态维度的视图。
-   `/tasks/blocking`：阻塞状态的任务。
-   `/tasks/running`：运行状态的任务。
-   `/tasks/delaying`：正在执行且延时的任务。
-   `/tasks/finished`：执行结束的任务。
-   `/tasks/succeeded`：成功的任务。
-   `/tasks/failed`：所有失败的任务。
-   `/tasks/delayed`：成功且延时的任务。
-   `/users`：保存所有用户的信息。
-   `/users/{path/to/a/user_name}`：user也是通过目录结果组织的，这个路径指向了一个特定的用户。用户不分版本。
    支持目录和用户的符号连接。但不能形成环。
-   `/search/jobs`：搜索job。
-   `/search/tasks`：搜索task。

好，准备工作差不多了，下面我们来定义每个具体的API。稍补充一下，我们的数据结构都是JSON。不过，为了清晰起见，后面对于
数据结构的定义都采用了一种类似thrift的语法进行。

### 公共数据结构定义 ###
本节定义公共的数据结构，以便后续引用。如下：

    // 触发方式
    enum Trigger {
        MANUAL,
        FIXED_RATE,
        FIXED_DELAY
    }

    // 调度时间信息
    struct Schedule {
        optional long startTime;  // 开始触发时间
        optional long startDelay;  // 提交后延后触发延时
        optional long period;  // 触发周期
        optional long stopTime;  // 停止触发时间
        optional long stopDelay;  // 提交后超过该延时后停止触发
        optional long maxTimes;  // 最多触发多少次
    }

    // 执行资源需求
    struct Resource {
        optional string cluster;  // 需要的集群URI。
        optional string queue;  // 集群队列信息
    }

    // 作业信息。其中，作业id、版本号由服务自动产生，如果请求中携带，会被忽略。
    struct Job {
        optional long id;  // 作业id.
        optional string name;  // 全限定的Job名字。
        optional int version;  // Job的当前版本号。
        optional string owner;  // Job所有者。
        optional int priority;  // 静态优先级，-40 ~ 40，越小优先级越高。
        optional Trigger trigger;  // 触发方式
        optional Schedule schedule;  // 调度时间信息。
        optional Resource resource;  // 执行资源需求
        optional long expDuration;  // 期望的完成时间 = 触发时间 + expDuration。
        optional long maxDuration;  // 最长执行时间 = 触发时间 + maxDuration。
        optional int retries;  // 失败后最大重试次数。
        optional string code;  // 代码地址，支持svn和git。
        optional string buildCmdline;  // cmdline used to build the code.
        optional list<string> emails;  // 通知邮件。
        optional map<string, string> environments;  // 执行需要的环境变量。
        optional string cmdline;  // 执行使用的命令行。
        optional list<string> relyFiles;  // 执行需要的依赖文件。
        optional map<string, string> extraInfo;  // 额外的用户自定义信息。
    }

    // 任务状态。
    enum TaskStatus {
        CREATING,  // Task刚刚创建的中间状态。
        READY，  // 就绪状态，所有数据依赖都已满足，等待执行资源
        BLOCKING,  // 阻塞状态，等待依赖就绪
        PREPARING, // 准备状态
        RUNNING,  // 运行状态
        FINISHING,  // 正在结束

        SUCCEEDED,  // 正常结束
        FAILED,  // 失败结束
        DELAYED  // 正常结束，但结束时间延时
    }

    // Task信息
    struct Task {
        required string job;  // Task关联的全限定的Job名字。
        required int version;  // Task关联的Job的版本号。
        required long time;  // Task的触发时间。
        required long repeat;  // Task的第几次执行，从1开始数。
        required TaskStatus status;  // Task的状态。
    }

    // 错误响应，如果一个请求的响应是错误，那么响应的消息体总会是这个消息。
    struct ErrorResponse {
        required string exception;  // 导致错误的异常全名
        required string message;  // 错误的描述性消息
    }

### 文本宏 ###
由于在我们的Job信息中，可能存在动态的内容，比如，命令行的输入输出可能是时间的函数。这里，我们简单定义一个宏语法，来
支持这些“动态”内容的指定。第一期我们仅仅支持in，out，date三个宏。

下面，先给出最小功能集合定义，这也是0.1版的功能集合。

### 创建Job ###
-   `POST /jobs`，请求体中携带`Job`描述信息。响应中返回job的版本号与id等信息。

### 更新Job ###
-   `PATCH /jobs/{job_name}`, 更新指定的Job。

### 获取Job信息 ###
-   `GET /jobs/{job_name}`，获取指定的job信息，返回`Job`结构体。

### 删除Job信息 ###
-   `DELETE /jobs/{job_name}`， Job并不会被真的删除，而是被放置在了某个历史版本中。

### 手动触发一个Task ###
-   `POST /jobs/{job_name}/tasks/{time}`，重新触发指定时间的任务。返回任务信息`Task`。

### 获取指定Job的Task信息 ###
-   `GET /jobs/{job_name}/tasks/{time}/{seq_number}`，获取指定的task信息。返回`Task`。

