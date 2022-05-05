# 创建自己的chart

前面我们快速使用了helm，包括安装开源的chart、练习了基础指令。我们的目标也包括让自己的应用使用chart模式管理起来。

这里体验一下

## helm create创建chart模版

```
$ helm create <myfirstchart可以是自己应用的名字>
Creating myfirstchart

$ tree myfirstchart
myfirstchart
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

可以看到指令生成了好多模版文件，比较熟悉的是deployment、ingress、service、account这些，包含了应用常用的编排文件。

```bash
# 我们什么也不改动，直接安装试试
$ helm install firstrelase ./myfirstchart
NAME: firstrelase
LAST DEPLOYED: Mon Feb 10 21:54:01 2020
NAMESPACE: liuzongxian
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace liuzongxian -l "app.kubernetes.io/name=myfirstchart,app.kubernetes.io/instance=firstrelase" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace liuzongxian port-forward $POD_NAME 8080:80
```

看到可以安装成功，查看一下

```bash
$ k get all
NAME                                            READY   STATUS    RESTARTS   AGE
pod/firstrelase-myfirstchart-77577bc9c5-kdksg   1/1     Running   0          47s

NAME                               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/firstrelase-myfirstchart   ClusterIP   172.22.7.136   <none>        80/TCP    48s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/firstrelase-myfirstchart   1/1     1            1           48s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/firstrelase-myfirstchart-77577bc9c5   1         1         1       48s
```

deployment/replicaset/pod/service，都创建好了，所以一般应用来讲，只需要我们修改模版中的参数就好了。

常见的修改是直接修改values.yaml，参数不复杂，例如可以修改image、nodeSelector、亲和性等信息。

```yaml
# cat values.yaml
# Default values for myfirstchart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name:

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths: []
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}
```

{% hint style="info" %}
我们给自己的应用改造成chart，一般都会从helm create开始，并根据实际情况做些修改。
{% endhint %}

我们在根据实际应用编写chart时，可以经常做些语法检查，查看是否正确。

```yaml
$ helm lint ./myfirstchart
==> Linting ./myfirstchart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

到这里，我们创建并使用了自己的第一个chart！
