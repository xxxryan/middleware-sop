# Install

## 前置准备清单

1. K8s 集群可用，`kubectl` 已配置
2. Helm 3 可用
3. 集群内已有 MySQL（示例：`mysql.xdev.svc.cluster.local:3306`）
4. 命名空间统一使用 `xdev`

## 关键前提（官方文档必做）

Apollo 服务端依赖两套库，缺任意一套都会启动失败：

- `ApolloConfigDB`（给 config-service/admin-service 使用）
- `ApolloPortalDB`（给 portal 使用）

你需要确保两份官方 SQL 已导入：

- [apolloconfigdb.sql](https://github.com/apolloconfig/apollo/blob/master/scripts/sql/profiles/mysql-default/apolloconfigdb.sql)
- [apolloportaldb.sql](https://github.com/apolloconfig/apollo/blob/master/scripts/sql/profiles/mysql-default/apolloportaldb.sql)

## 初始化 Apollo 数据库（推荐执行一次）

### 1) 端口转发到本地

```bash
kubectl -n xdev port-forward svc/mysql 13306:3306
```

### 2) 下载官方 SQL

```bash
curl -L -o apolloconfigdb.sql \
  https://raw.githubusercontent.com/apolloconfig/apollo/master/scripts/sql/profiles/mysql-default/apolloconfigdb.sql
curl -L -o apolloportaldb.sql \
  https://raw.githubusercontent.com/apolloconfig/apollo/master/scripts/sql/profiles/mysql-default/apolloportaldb.sql
```

### 3) 创建 DB 并授权 apollo 用户

先执行仓库内脚本：`components/apollo/init-apollo-db.sql`

```bash
mysql -h127.0.0.1 -P13306 -uroot -proot < components/apollo/init-apollo-db.sql
```

### 4) 导入官方表结构和初始数据

```bash
mysql -h127.0.0.1 -P13306 -uroot -proot ApolloConfigDB < apolloconfigdb.sql
mysql -h127.0.0.1 -P13306 -uroot -proot ApolloPortalDB < apolloportaldb.sql
```

### 5) 验证导入结果

```bash
mysql -h127.0.0.1 -P13306 -uroot -proot -e \
"select Id,`Key`,`Value` from ApolloConfigDB.ServerConfig limit 3;"
mysql -h127.0.0.1 -P13306 -uroot -proot -e \
"select Id,`Key`,`Value` from ApolloPortalDB.ServerConfig limit 3;"
```

## 添加 Apollo Helm 仓库

```bash
helm repo add apollo https://charts.apolloconfig.com
helm repo update
helm search repo apollo
```

## 部署 apollo-service（ConfigService + AdminService）

配置文件：`values-apollo-service.yaml`

```bash
helm upgrade --install apollo-service \
  -n xdev \
  -f values-apollo-service.yaml \
  apollo/apollo-service
```

## 部署 apollo-portal（Web UI）

配置文件：`values-apollo-portal.yaml`

```bash
helm upgrade --install apollo-portal \
  -n xdev \
  -f values-apollo-portal.yaml \
  apollo/apollo-portal
```

## 部署后验证

```bash
kubectl get pods -n xdev
kubectl get svc -n xdev
kubectl logs -n xdev deploy/apollo-service-apollo-configservice --tail=100
kubectl logs -n xdev deploy/apollo-service-apollo-adminservice --tail=100
kubectl logs -n xdev deploy/apollo-portal --tail=100
```

Portal 默认 Service 为 `ClusterIP`，如需外部访问可自行配置 Ingress 或临时端口转发：

```bash
kubectl -n xdev port-forward svc/apollo-portal 8070:8070
```
