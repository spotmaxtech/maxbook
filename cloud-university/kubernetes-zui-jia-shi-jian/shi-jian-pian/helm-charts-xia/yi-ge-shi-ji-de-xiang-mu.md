# 一个实际的项目

{% hint style="info" %}
选取了一个实际项目，分析下几方面

* 资源类型有哪些
* 提取了哪些变量
* 常用的语法
{% endhint %}

### 准备工作

接入一个实际项目，拉取并解压出来一个Helm Chart

```
helm repo add ml-platform http://harbor-v2.mobvista.com/chartrepo/ml-platform
helm pull ml-platform/predict --untar
```

看看里面有什么

```
$ tree predict
predict
├── Chart.yaml
├── README.md
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── autoscaling.yaml
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── pdb.yaml
│   ├── service-monitor.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

挨个看看模版内容，可以拿来参考改造其他项目

{% hint style="info" %}
helm template predict 可以查看模版渲染之后的yaml内容
{% endhint %}

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: {{ .Release.Namespace }}
  name: {{ include "predict.fullname" . }}
  labels:
    {{- include "predict.labels" . | nindent 4 }}
spec:
  progressDeadlineSeconds: 600
  #replicas: {{ .Values.autoscaling.minReplicas }}
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      {{- include "predict.selectorLabels" . | nindent 6 }}
  {{- with .Values.strategy }}
  strategy:
    {{- toYaml . | nindent 4 }}
  {{- end }}
  template:
    metadata:
      labels:
        {{- include "predict.selectorLabels" . | nindent 8 }}
      annotations:
        {{ tpl .Values.annotations . | nindent 8 | trim }}
    spec:
      {{- if .Values.hostNetwork }}
      hostNetwork: {{ .Values.hostNetwork }}
      {{- end }}
      serviceAccountName: {{ include "predict.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 6 }}
      {{- end }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          env:
            - name: NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: HOST_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.hostIP
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            {{- if not .Values.aws.enabled }}
            - name: AWS_ACCESS_KEY_ID
              valueFrom:
                secretKeyRef:
                  key: aws_access_key_id
                  name: aws-secret
                  optional: true
            - name: AWS_SECRET_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  key: aws_secret_access_key
                  name: aws-secret
                  optional: true
            {{- end }}
            - name: VERSION
              value: {{ .Values.image.tag | default .Chart.AppVersion }}
            {{- range $key, $value := .Values.consul }}
            - name: {{ printf "CONSUL_%s" $key | upper }}
              value: "{{ $value }}"
            {{- end }}
            {{- if .Values.gpu.enabled }}
            #- name: NVIDIA_VISIBLE_DEVICES
            #  value: "all"
            - name: NVIDIA_DRIVER_CAPABILITIES
              value: "compute,utility"
            {{- end}}
          ports:
            {{- range $key, $value := .Values.service.ports }}
            - name: {{ $key }}
              containerPort: {{ $value.port }}
              protocol: {{ $value.protocol }}
            {{- end }}
          volumeMounts:
          - name: config-dir
            mountPath: /etc/predict
          - name: data-vol
            mountPath: /data/mind_model
          {{- if .Values.aws.enabled }}
          - name: ec2-metadata
            mountPath: /opt/aws/bin/ec2-metadata
          {{- end }}
          {{- if .Values.oss.enabled }}
          - name: oss-config
            mountPath: /home/predict/.ossutilconfig
            subPath: .ossutilconfig
          - name: aws-config-volume
            mountPath: /home/predict/.aws/config
            subPath: config
          {{- end}}
          livenessProbe:
            failureThreshold: 15
            httpGet:
              path: /
              port: http
            initialDelaySeconds: {{ .Values.livenessProbe.initialDelaySeconds }}
            periodSeconds: 300
            timeoutSeconds: 30
          readinessProbe:
            failureThreshold: 15
            httpGet:
              path: /
              port: http
            initialDelaySeconds: {{ .Values.readinessProbe.initialDelaySeconds }}
            periodSeconds: 300
            timeoutSeconds: 30
          {{- if .Values.resources }}
          resources:
            limits:
              cpu: {{ .Values.resources.limits.cpu }}
              memory: {{ .Values.resources.limits.memory }}
              {{- if .Values.gpu.enabled }}
              {{ .Values.gpu.corporation }}/gpu: {{ .Values.gpu.count }}
              {{- end}}
            requests:
              cpu: {{ .Values.resources.requests.cpu }}
              memory: {{ .Values.resources.requests.memory }}
          {{- end }}
          lifecycle:
            postStart:
              exec:
                command:
                - /bin/bash
                - -c
                - |
                  python3 /usr/local/consul/consul.py config ${CONSUL_DC} > /etc/consul/consul.json
                  cp -f /etc/predict/service.json /data/recommend/predict_service/etc/predict_service.json
                  cp -f /etc/predict/watch.json /data/recommend/predict_service/etc/consul_watches.json
            preStop:
              exec:
                command:
                - /bin/bash
                - -c
                - |
                  python3 /data/recommend/predict_service/script/consul_tool.py deregister --conf /data/recommend/predict_service/etc/predict_service.json
                  /usr/local/consul/consul leave
      initContainers:
      - image: {{ .Values.initImage | default "busybox" }}
        command:
        - sh
        - -c
        - |
          echo 10240 > /proc/sys/net/core/somaxconn
          echo "core-%e-%p-%t" > /proc/sys/kernel/core_pattern
          echo 1048576 > /proc/sys/fs/aio-max-nr
        imagePullPolicy: Always
        name: sysinit
        securityContext:
          privileged: true
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
    {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
    {{- end }}
      volumes:
      - name: config-dir
        configMap:
          name: {{ include "predict.fullname" . }}-config
          items:
          - key: service.json
            path: service.json
          - key: watch.json
            path: watch.json 
          - key: factor.json
            path: factor.json 
          - key: ms.json
            path: ms.json 
      - name: data-vol
        emptyDir:
        {{- with .Values.emptyDir }}
          {{- toYaml . | nindent 10 }}
        {{- end }}
      {{- if .Values.oss.enabled }}
      - name: oss-config
        configMap:
          name: {{ include "predict.fullname" . }}-config
          items:
          - key: .ossutilconfig
            path: .ossutilconfig
      - name: aws-config-volume
        configMap:
          name: {{ include "predict.fullname" . }}-config
          optional: true
          items:
          - key: config
            path: config
      {{- end }}
      {{- if .Values.aws.enabled }}
      - name: ec2-metadata
        hostPath:
          path: /usr/bin/ec2-metadata
      {{- end }}

```

### configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: {{ .Release.Namespace }}
  name: {{ include "predict.fullname" . }}-config
data:
  service.json: |
    {
        "keyprefix": "{{ .Values.consul.keyprefix }}",
        "service": "{{ .Values.consul.service }}",
        "metrics_prefix": "{{ .Values.metrics.prefix }}",
        "path": "/data/mind_model/",
        "http": {{ .Values.service.ports.http.port }},
        "tcp": {{ .Values.service.ports.tcp.port }},
        "monitor_port": {{ .Values.service.ports.metrics.port }},
        {{- if .Values.server.omp }}
        "omp_num": {{ .Values.server.omp }},
        {{- end }}
        {{- if .Values.server.cpu }}
        "cpu_num": {{ .Values.server.cpu }},
        {{- end }}
        {{- if .Values.server.smp }}
        "smp_num": {{ .Values.server.smp }},
        {{- end }}
        {{- if .Values.server.thread }}
        "thread_num": {{ .Values.server.thread }},
        {{- end }}
        {{- if .Values.ms }}
        "ms_cfg": "{{ .Values.ms.config }}",
        {{- end }}
        "discard" : {{ .Values.server.discard }},
        "timeout": {{ .Values.server.timeout }}
    }
  watch.json: |
    {
        "watches": [
        {
            "type": "keyprefix",
            "prefix": "{{ .Values.consul.keyprefix }}",
            "args": [
                "python3", 
                "/data/recommend/predict_service/script/model_tool.py",
                "--key={{ .Values.consul.keyprefix }}",
                "--tcp={{ .Values.service.ports.tcp.port }}",
                "--http={{ .Values.service.ports.http.port }}",
                "--factor_map=consul/{{ .Values.metrics.prefix }}/factor_map.json",
                "--cluster={{ .Values.consul.service }}"
            ]
        },
        {
            "type": "key",
            "key": "consul/{{ .Values.metrics.prefix }}/factor_map.json",
            "args": [
                "python3",
                "/data/recommend/predict_service/script/consul_tool.py",
                "register",
                "--conf=/data/recommend/predict_service/etc/predict_service.json",
                "--factor_map=consul/{{ .Values.metrics.prefix }}/factor_map.json"
            ]
        }
      ]
    }
  .ossutilconfig: |
    [Credentials]
    language=EN
    {{- if .Values.oss }}
    endpoint={{ .Values.oss.endpoint }}
    accessKeyID={{ .Values.oss.id }}
    accessKeySecret={{ .Values.oss.secret }}
    {{- end }}
  factor.json: |
    {
      {{- range $key, $value := .Values.factor.instance }}
      "{{ $key }}": {{ $value }},
      {{- end }}
      "unknown": {{ .Values.factor.unknown }}
    }
  config: |
    [default]
    s3 =
        addressing_style = virtual
        max_bandwidth = 50MB/s
        endpoint_url = http://{{ .Values.oss.endpoint }}
    [plugins]
    endpoint = awscli_plugin_endpoint
  ms.json: |
    {
      {{- with .Values.ms }}
      "timeout_ms": "{{ .client.timeout_ms }}",
      "timeout_upper": {{ .client.timeout_upper }},
      "timer_period": {{ .client.timer_period }},
      "cluster_prefix": "{{ .client.cluster_prefix }}",
      "recover": {{ .client.recover }},
      "request_gap": {{ .client.request_gap }},
      {{- end }}
      "weight_balance": true,
      "smooth_weight": true,
      "stale": true
    }

```

### autoscaling.yaml

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  namespace: {{ .Release.Namespace }}
  name: {{ include "predict.fullname" . }}-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "predict.fullname" . }}
  {{- with .Values.autoscaling.behavior }}
  behavior:
    {{- toYaml . | nindent 4 }}
  {{- end }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
  {{- range $key, $value := .Values.autoscaling.metrics }} 
  {{- if $value.enabled }}
  - type: {{ $value.type }}
    {{ $value.type | lower }}:
      {{- if eq $value.type "Pods" }}
      metric:
        name: {{ $value.name }}
      {{- else }}
      name: {{ $value.name }}
      {{- end }}
      target:
        type: {{ $value.target.type }}
        {{- if eq $value.target.type "Utilization" }}
        averageUtilization: {{ $value.target.value }}
        {{- else }}
        averageValue: {{ $value.target.value }}
        {{- end }}
  {{- end }}
  {{- end }}

```

pdb.yaml

```yaml
{{- if .Values.podDisruptionBudget -}}
{{- if semverCompare ">=1.21-0" .Capabilities.KubeVersion.GitVersion -}}
apiVersion: policy/v1
{{- else if semverCompare ">=1.10-0" .Capabilities.KubeVersion.GitVersion -}}
apiVersion: policy/v1beta1
{{- end }}
kind: PodDisruptionBudget
metadata:
  name: {{ include "predict.fullname" . }}-pdb
  namespace: {{ .Release.Namespace }}
spec:
{{ toYaml .Values.podDisruptionBudget | indent 2 }}
  selector:
    matchLabels:
      {{- include "predict.labels" . | nindent 6 }}
{{- end -}}

```

### service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "predict.fullname" . }}
  labels:
    {{- include "predict.labels" . | nindent 4 }}
  annotations:
    {{ tpl .Values.annotations . | nindent 4 | trim }}
spec:
  type: {{ .Values.service.type }}
  ports:
    {{- range $key, $value := .Values.service.ports }}
    - port: {{ $value.port }}
      targetPort: {{ $key }}
      protocol: {{ $value.protocol }}
      name: {{ $key }}
    {{- end }}
  selector:
    {{- include "predict.selectorLabels" . | nindent 4 }}

```

### ingress.yaml

```yaml
{{- if .Values.ingress.enabled -}}
{{- $fullName := include "predict.fullname" . -}}
{{- $svcPort := .Values.service.port -}}
{{- if semverCompare ">=1.14-0" .Capabilities.KubeVersion.GitVersion -}}
apiVersion: networking.k8s.io/v1beta1
{{- else -}}
apiVersion: extensions/v1beta1
{{- end }}
kind: Ingress
metadata:
  name: {{ $fullName }}
  labels:
    {{- include "predict.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
{{- if .Values.ingress.tls }}
  tls:
  {{- range .Values.ingress.tls }}
    - hosts:
      {{- range .hosts }}
        - {{ . | quote }}
      {{- end }}
      secretName: {{ .secretName }}
  {{- end }}
{{- end }}
  rules:
  {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
        {{- range .paths }}
          - path: {{ . }}
            backend:
              serviceName: {{ $fullName }}
              servicePort: {{ $svcPort }}
        {{- end }}
  {{- end }}
{{- end }}

```

### serviceaccount.yaml

```yaml
{{- if .Values.serviceAccount.create -}}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "predict.serviceAccountName" . }}
  labels:
    {{- include "predict.labels" . | nindent 4 }}
  {{- if and .Values.aws.enabled .Values.aws.role }}
  annotations:
    eks.amazonaws.com/role-arn: {{ .Values.aws.role }}
  {{- else }}
  {{- with .Values.serviceAccount.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
  {{- end }}
{{- end -}}

```

### service-monitor.yaml

```yaml
# Copyright (c) 2020, NVIDIA CORPORATION.  All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: {{ include "predict.fullname" . }}-servicemonitor
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "predict.labels" . | nindent 4 }}
    release: {{ .Values.serviceMonitor.release }}
spec:
  selector:
    matchLabels:
      {{- include "predict.selectorLabels" . | nindent 6 }}
  endpoints:
  - port: "metrics"
    path: "{{ .Values.service.ports.metrics.path }}"
    interval: {{ .Values.serviceMonitor.interval | default "120s" }}

```
