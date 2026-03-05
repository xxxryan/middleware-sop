# install

## 前置准备清单

1. K8s 集群可用，kubectl 已配置

2. Helm 3 可用（Chart 要求）

3. 集群内已有 MySQL Service（例如 mysql.mysql.svc.cluster.local:3306）

4. 准备一个 namespace（建议 apollo）

## 安装 Helm Chart（官方仓库）

添加官方 chart repo：

```bash
helm repo add apollo https://charts.apolloconfig.com
helm repo update
helm search repo apollo
```

## 部署 apollo-service（ConfigService + AdminService）

### 准备 values.yaml

[values-apollo-service.yaml](./values-apollo-service.yaml)

### 安装

```bash
helm upgrade --install apollo-service \
  -n apollo \
  -f values-apollo-service.yaml \
  apollo/apollo-service
```

## 部署 apollo-portal（Web UI）

Portal chart 需要配置：

- portaldb.*（PortalDB 连接）

- config.envs（至少一个 env 名称）

- config.metaServers.<env> 指向你刚部署的 ConfigService 地址

### 准备 values-apollo-portal.yaml

[values-apollo-portal.yaml](./values-apollo-portal.yaml)

### 安装

```bash
helm upgrade --install apollo-portal \
  -n apollo \
  -f values-apollo-portal.yaml \
  apollo/apollo-portal
```
