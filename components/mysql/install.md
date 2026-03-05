# Install

## 前置准备清单

1. K8s 集群可用，`kubectl` 已配置

2. Helm 3 可用

3. 准备一个 namespace（示例使用 `xdev`）

## Chart 说明

当前目录是一个本地 Helm Chart，使用 MySQL 官方镜像 `docker.io/mysql`。

## 准备 values.yaml

请先按环境修改以下配置（至少密码）：

- `auth.rootPassword`
- `auth.password`
- `persistence.storageClass`（如集群有默认 StorageClass 可留空）

配置文件：

`components/mysql/values.yaml`

## 安装 MySQL

```bash
helm upgrade --install mysql \
  -n xdev \
  --create-namespace \
  -f components/mysql/values.yaml \
  ./components/mysql
```

## 验证部署状态

```bash
kubectl get pods -n xdev
kubectl get svc -n xdev
```

等待 `mysql-0` 为 `Running` 且 `READY 1/1`。

## 集群内连接测试

```bash
kubectl run mysql-client --rm -it --restart=Never \
  -n xdev \
  --image=mysql:8.4.0 \
  --command -- mysql -hmysql.xdev.svc.cluster.local -P3306 -uapp_user -pChangeMe_App_123! -e "SELECT 1;"
```

如果返回 `1` 则表示连接成功。

## 卸载

```bash
helm uninstall mysql -n xdev
```

如需同时删除 PVC（会丢失数据）：

```bash
kubectl delete pvc -n xdev -l app.kubernetes.io/instance=mysql
```
