龙渊 Elasticsearch cluster
---

# 集群部署架构说明

采用 master、data、client 分离部署的模式

master 会部署在拥有 `es-master=true` 主机标签的节点上, 用作集群信息维护,自身不存放数据，建议部署3个以上的奇数master节点

data 会部署在拥有 `es-data=true` 主机标签的节点上, 用作数据存放，其数据卷通过es-storge的从容器进行挂载，节点数量根据需要进行配置

client 会部署在拥有 `es-client=true` 主机标签的节点上， 用于客户端连接(前面可接负载均衡器,或者直接在主机上暴露端口)，节点数量根据需要进行配置

如果主机节点没有相关标签， 对应模块部署会失败

master和client会尽量避免被部署在同一个节点主机， 但启动的容器数大于满足要求的节点主机数的话，还是允许部署的，data容器则一定不会被部署在同一台节点主机上，如果data容器数大于满足要求的节点主机数, 部署会失败

默认情况下，各es部件的容器会启动一个`es-sysctl`的从容器，以调整 `vm.max_map_count` 的值至262144， 如果节点主机已经进行了设置，可以关闭这个功能

# 调优选项说明

`master_heap_size`: 暂无生产环境运行配置经验，官方推荐512M-4G， 目前默认配置为1G，可根据实际压力情况进行调整

`data_heap_size`: 这个要根据节点主机的内存进行配置，龙渊32G的机器目前配置为22G，不一定合理，可根据实际压力情况进行调整

`client_heap_size`: 暂无生产环境运行配置经验，根据实际压力情况进行调整， 目前默认配置为1G

# 常用API

集群健康状态查看 `/_cluster/health?pretty`

集群状态详情查看 `/_cluster/state?pretty`

# 集群维护

TODO

# 周边工具安装

kibana 和 elasticsql 为可选安装项，默认会安装，如果不需要可以去掉

# 服务暴露

默认cluster搭建好后，是没有暴露在容器网络之外的，请根据需要自行通过负载均衡器进行相关服务端口的暴露

`es-client`: `http/https: 9200->9200`

`kibana`: `http/https: 80->5601`

`elastic-sql`: `http/https:  80->8080`

# 常见问题:

### Elasticsearch is still initializing the kibana index... 

执行 `curl -XDELETE http://localhost:9200/.kibana`

###  Elasticsearch is still initializing the Monitoring indices

执行 `curl -XDELETE 'http://localhost:9200/.monitoring*'`

