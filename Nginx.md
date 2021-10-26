* Windows下安装启动等流程
  * 启动：C:\server\nginx-1.0.2>start nginx或C:\server\nginx-1.0.2>nginx.exe；建议使用第一种，第二种会使你的cmd窗口一直处于执行中，不能进行其他命令操作。
  * 停止：C:\server\nginx-1.0.2>nginx.exe -s stop或C:\server\nginx-1.0.2>nginx.exe -s quit；stop是快速停止nginx，可能并不保存相关信息；quit是完整有序的停止nginx，并保存相关信息。
  * 重新载入Nginx：C:\server\nginx-1.0.2>nginx.exe -s reload；当配置信息修改，需要重新载入这些配置时使用此命令。
  * 重新打开日志文件：C:\server\nginx-1.0.2>nginx.exe -s reopen
  * 查看Nginx版本：C:\server\nginx-1.0.2>nginx -v

* nginx.conf 配置文件分为三部分：

  * 第一部分：全局块

    * 从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

    * Nginx 同 redis 类似都采用了 io 多路复用机制，每个 worker 都是一个独立的进程，但每个进程里只有一个主线程，通过异步非阻塞的方式来处理请求， 即使是千上万个请求也不在话下。每个 worker 的线程可以把一个 cpu 的性能发挥到极致。所以 worker 数和服务器的 cpu数相等是最为适宜的。设少了会浪费 cpu，设多了会造成 cpu 频繁切换上下文带来的损耗。

    * 设置 worker 数量：

      worker_processes 4

      #work 绑定 cpu(4 work 绑定 4cpu)。

      worker_cpu_affinity 0001 0010 0100 1000

      #work 绑定 cpu (4 work 绑定 8cpu 中的 4 个) 。

      worker_cpu_affinity 0000001 00000010 00000100 00001000
    
  * 第二部分：events 块

    * events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多work process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。
    * 连接数 worker_connection；这个值是表示每个worker进程所能建立连接的最大值，所以，一个nginx能建立的最大连接数，应该是worker_connections * worker_processes。当然，这里说的是最大连接数，对于HTTP请求本地资源来说，能够支持的最大并发数量是 worker_connections * worker_processes，如果是支持http1.1的浏览器每次访问要占两个连接，所以普通的静态访问最大并发数是：worker_connections * worker_processes /2，而如果是HTTP作为反向代理来说，最大并发数量应该是 worker_connections * worker_processes/4。因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接。
    
  * 第三部分：http块

    * 这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。需要注意的是：http 块也可以包括 http 全局块、server 块。
    * http 全局块：http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等
      * upstream：负载均衡
        * 轮询（默认）：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除。
        * weight：weight 代表权,重默认为 1,权重越高被分配的客户端越多；指定轮询几率，weight 和访问比率成正比，用于后端服务器性能不均的情况。
        * ip_hash：每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。
        * fair（第三方）：按后端服务器的响应时间来分配请求，响应时间短的优先分配。
    * server 块：这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。
      * 全局 server 块：最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。
      * location 块：一个 server 块可以配置多个 location 块。这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。
        * = ：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。
        * ~：用于表示 uri 包含正则表达式，并且区分大小写。
        * ~*：用于表示 uri 包含正则表达式，并且不区分大小写。
        * ^~：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location块中的正则 uri 和请求字符串做匹配。
        * 注意：如果 uri 包含正则表达式，则必须要有~或者~*标识。
      * proxy_pass：反向代理
      * 动静分离从目前实现角度来讲大致分为两种：一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案；另外一种方法就是动态跟静态文件混合在一起发布，通过 nginx 来分开。
      * 通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。具体 Expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304，如果有修改，则直接从服务器重新下载，返回状态码 200。
