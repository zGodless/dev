# 容器编排
    它不直接替你写微服务业务逻辑，而是负责把很多个微服务实例部署起来、调度到机器上、暴露服务、做健康检查、扩缩容、滚动发布、服务发现、配置管理等。

## 自动恢复故障服务

## 快速伸缩
  自动扩容、弹性伸缩
  根据流量自适应 

# Kubernetes
├── Deployment  管 API 多副本
├── Service 管服务发现和负载均衡
├── ConfigMap / Secret 管配置和密钥
├── Ingress / Gateway 管外部入口
├── HPA 管自动扩缩容
├── Probe 管健康检查和自愈
├── RollingUpdate 管滚动发布
├── CronJob / Job 管定时任务和一次性任务
├── PV / PVC / StorageClass 管持久化存储
└── 监控、日志、链路追踪管可观测性