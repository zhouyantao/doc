# RabbitMQ 命令行操作
```
1. 通用参数
查看 list
清空 purge
删除 delete
帮助 --help

2. 状态查看
# 查看状态
rabbitmqctl status
# 查看绑定
rabbitmqctl list_bindings
# 查看 channel
rabbitmqctl list_channels
# 查看 exchange
rabbitmqctl list_exchanges
# 查看 channel
rabbitmqctl list_channels
# 查看 connection
rabbitmqctl list_connections
3. 队列相关
# 查看 queue
rabbitmqctl list_queues
# 删除 queue
rabbitmqctl delete_queue
# 清空 queue
rabbitmqctl purge_queues
4. 用户相关
# 新建用户
rabbitmqctl add_user
# 修改用户密码
rabbitmqctl change_password
# 删除用户
rabbitmqctl delete_user
# 设置用户角色
rabbitmqctl set_user_tags
5. 应用启停
# 启动应用
rabbitmqctl start_app
# 关闭应用, 保留 erlang 虚拟机
rabbitmqctl stop_app
# 关闭应用, 退出 erlang 虚拟机
rabbitmqctl stop
6. 集群相关
# 加入
rabbitmqctl join_cluster
# 离开
rabbitmqctl reset
7. 镜像队列
# 设置镜像队列
rabbitmqctl sync_queue
# 取消
rabbitmqctl cancel_sync_queue