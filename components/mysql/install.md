# Install

## 前置准备清单

1. K8s 集群可用，`kubectl` 已配置

2. Helm 3 可用

3. 准备一个 namespace（示例使用 `xdev`）

## 安装 Helm Chart（Bitnami 仓库）

添加 chart repo：

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami/mysql
```

## 准备 values.yaml

请先按环境修改以下配置（至少密码）：

- `auth.rootPassword`
- `auth.password`
- `primary.persistence.storageClass`（如集群有默认 StorageClass 可留空）

配置文件：

[values.yaml](./values.yaml)

## 安装 MySQL

```bash
helm upgrade --install mysql \
  -n xdev \
  --create-namespace \
  -f values.yaml \
  bitnami/mysql
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
  --image=bitnami/mysql:8.0.36-debian-12-r0 \
  --command -- mysql -hmysql.xdev.svc.cluster.local -P3306 -uapollo -papollo -e "SELECT 1;"
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
