## Helm

Kubernetes 包管理器，Helm 是查找、分享和使用软件构件 Kubernetes 的最优方式。Helm 管理名为 chart 的 Kubernetes 包的工具。Helm 可以做以下的事情：

* 从头开始创建新的 chart
* 将 chart 打包成归档(tgz)文件
* 与存储 chart 的仓库进行交互
* 在现有的 Kubernetes 集群中安装和卸载 chart
* 管理与 Helm 一起安装的 chart 的发布周期

对于Helm，有三个重要的概念：

* chart 创建Kubernetes应用程序所必需的一组信息。
* config 包含了可以合并到打包的chart中的配置信息，用于创建一个可发布的对象。
* release 是一个与特定配置相结合的chart的运行实例。

Helm 客户端 是终端用户的命令行客户端。负责以下内容：

* 本地 chart 开发
* 管理仓库
* 管理发布
* 与 Helm 库建立接口
* 发送安装的 chart
* 发送升级或卸载现有发布的请求

Helm 库提供执行所有 Helm 操作的逻辑。与 Kubernetes API 服务交互并提供以下功能：

* 结合 chart 和配置来构建版本
* 将 chart 安装到 Kubernetes 中，并提供后续发布对象
* 与 Kubernetes 交互升级和卸载 chart
* 独立的 Helm 库封装了 Helm 逻辑以便不同的客户端可以使用它。

## Prometheus

Prometheus 是一套开源的监控系统、报警、时间序列的集合，最初由 SoundCloud 开发，后来随着越来越多公司的使用，于是便独立成开源项目。自此以后，许多公司和组织都采用了 Prometheus 作为监控告警工具。

## ELK

ElasticSearch：ES 作为一个搜索型文档数据库，拥有优秀的搜索能力，以及提供了丰富的 REST API 让我们可以轻松的调用接口。

Logstash：Filebeat 是一款轻量的数据收集工具.通过 Logstash 同样可以进行日志收集，但是若每一个节点都需要收集时，部署 Logstash 有点过重，因此这里主要用到 Logstash 的数据清洗能力，收集交给 Filebeat 去实现。

Kibana：Kibana 是一款基于 ES 的可视化操作界面工具，利用 Kibana 可以实现非常方便的 ES 可视化操作。

# DevOps

## DevOps 环境搭建

## 微服务 DevOps 实战

