# 如何升级版本
打开GitHub地址 https://github.com/argoproj/argo-cd/releases ,
示例：

打开此文件
```
https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.12/manifests/install.yaml
```
找到相应的版本，因为国外网络问题，需要替换 argocd/base/install.yaml 。

提交后，argocd 自动部署。
# ArgoCD 配置

## 添加账号
账号目录文件， argocd/overlays/production/argocd-cm.yaml, 添加此配置
```
apiVersion: v1
data:
  accounts.developer: login
  accounts.developer.enabled: "true"
```
使用argocd-cli工具登录，并输入账号密码
```
 argocd login argocd.arnoo.com --grpc-web
```
使用命令修改密码
```
rgocd account update-password --account testuser --new-password ******** --current-password *******

```
## 权限分配
除应用程序特定权限外的所有资源（请参阅下一个项目符号）
```
p, <role/user/group>, <resource>, <action>, <object>
```
应用程序、日志和 exec（属于 AppProject）：
```
p, <role/user/group>, <resource>, <action>, <appproject>/<object>
```
找到目录文件： argocd/kustomization.yaml
```
patches:
  - patch: |-
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: argocd-rbac-cm
        labels:
          app.kubernetes.io/part-of: argocd
      data:
        policy.default: role:readonly
        policy.csv: |
          p, role:pre-admin, applications, create, pre-admin/*, allow
          p, role:pre-admin, applications, delete, pre-admin/*, allow
          p, role:pre-admin, applications, get, pre-admin/*, allow
          p, role:pre-admin, applications, override, pre-admin/*, allow
          p, role:pre-admin, applications, sync, pre-admin/*, allow
          p, role:pre-admin, applications, update, pre-admin/*, allow
          p, role:pre-admin, logs, get, pre-admin/*, allow
          p, role:pre-admin, exec, create, pre-admin/*, allow
          p, role:pre-admin, projects, get, pre-admin, allow
          g, db-admins, role:pre-admin
    target:
      labelSelector: "app.kubernetes.io/name=argocd-rbac-cm"
```
此示例定义了一个名为的角色pre-admin，该角色具有八个权限，允许该角色对Argo CD AppProject中的（所有）对象执行操作（create/ delete/ get/ override/ sync/update应用程序、get日志、create，exec 和get appprojects）
