# rabbitmq 安装教程
* `特别说明`：切换单机或集群需要删除data和logs下的目录数据，不然来回切换的时候可能会有遗留的旧数据
* cp example.env .env
* cp docker-compose-example.yml docker-compose.yml

## 单机
* env文件打开：`CLUSTER_DIR=single`, 注释：`CLUSTER_DIR=cluster`
* docker-compose.yml 打开：rabbitmq01,注释：rabbitmq02、rabbitmq03
* 执行`docker-compose up -d`

## 集群
* env文件打开：`CLUSTER_DIR=cluster`, 注释：`CLUSTER_DIR=single`
* docker-compose.yml 打开：rabbitmq01、rabbitmq02、rabbitmq03 
* 执行`docker-compose up -d`
```shell
#集群其他操作命令
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@mq1
rabbitmqctl start_app
```


## 开启管理界面
* 进入容器启动后台管理插件 `rabbitmq-plugins enable rabbitmq_management`
* 浏览器登陆`http://本地ip:15672/` 输入登陆账号密码 myuser mypass

## tcp连接方式
* 本地ip + 5672端口

## 安装delay插件
* [官网查看有哪些插件](https://www.rabbitmq.com/community-plugins.html)
* [github查看有哪些插件](https://github.com/rabbitmq)
### 下载插件
* 需要下载对应版本的插件的`.ez`文件
```shell
# 进入容器
# 复制插件
cp /opt/rabbitmq/plugins-my/rabbitmq_delayed_message_exchange-3.12.0.ez /opt/rabbitmq/plugins

# 开启插件
rabbitmq-plugins enable rabbitmq_delayed_message_exchange

# 查看启动的插件列表
rabbitmq-plugins list

# 关闭插件
rabbitmq-plugins disable rabbitmq_delayed_message_exchange
```

## 集群常用的功能
### 开启镜像队列(配置镜像策略)
* 管理后台直接配置
* 命令行配置
  ```text
    ##命令
    rabbitmqctl set_policy [-p Vhost] Name Pattern Definition [Priority]
  
    ##参数
    -p Vhost： 可选参数，针对指定vhost下的queue进行设置
    Name: policy的名称
    Pattern: queue的匹配模式(正则表达式)
    Definition：镜像定义，包括三个部分ha-mode, ha-params, ha-sync-mode
        ha-mode:指明镜像队列的模式，有效值为 all/exactly/nodes
            all：表示在集群中所有的节点上进行镜像
            exactly：表示在指定个数的节点上进行镜像，节点的个数由ha-params指定
            nodes：表示在指定的节点上进行镜像，节点名称通过ha-params指定
        ha-params：ha-mode模式需要用到的参数
        ha-sync-mode：进行队列中消息的同步方式，有效值为automatic和manual
    priority：可选参数，policy的优先级
    
    ##示例（在管理后台先添加一个test_direct_002队列）
    rabbitmqctl set_policy test_direct_002 "test_direct_002" '{"ha-mode":"all","ha-sync-mode":"automatic"}'
  ```