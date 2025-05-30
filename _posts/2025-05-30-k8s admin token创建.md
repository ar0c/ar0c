创建admin账户

```yaml
# admin.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
```

```bash
kubectl apply -f admin.yaml
```

```bash
# 查看账户有无密码
k get serviceaccounts admin -n kube-system 
# 创建临时token（1小时）
k create token admin -n kube-system
```

创建secret的token

```yaml
# token.yaml
apiVersion: v1
kind: Secret
metadata:
  name: admin-secret
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: admin
type: kubernetes.io/service-account-token
```

```bash
kubectl apply -f token.yaml
# 获取
kubectl get secret admin-secret -n kube-system -o jsonpath='{.data.token}' | base64 -d
```

