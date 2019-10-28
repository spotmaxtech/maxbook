# Deployment: 声明式地升级应用

```text
$ k create -f cat-rc-and-service-v1.yaml
replicationcontroller/cat-v1 created
service/cat created
ingress.extensions/cat created

```

```text
$ curl cat.example.com
Hello World! I'm little cat!
Hostname: cat-v1-vqkdx
Ip:       10.1.0.85
Version:  v1
```

```text
$ k rolling-update cat-v1 cat-v2 --image=spotmax/cat:v2                                                    
Command "rolling-update" is deprecated, use "rollout" instead
Created cat-v2
Scaling up cat-v2 from 0 to 3, scaling down cat-v1 from 3 to 0 (keep 3 pods available, don't exceed 4 pods)
Scaling cat-v2 up to 1
Scaling cat-v1 down to 2
Scaling cat-v2 up to 2
Scaling cat-v1 down to 1
Scaling cat-v2 up to 3
Scaling cat-v1 down to 0
Update succeeded. Deleting cat-v1
replicationcontroller/cat-v2 rolling updated to "cat-v2"
```

```text
$ k create -f cat-deployment-v1.yaml --record
deployment.apps/cat created
```

```text
$ k rollout status deployment cat                      
deployment "cat" successfully rolled out

```

```text
$ k patch deployment cat -p '{"spec":{"minReadySeconds":10}}'
deployment.extensions/cat patched

```

```text
$ k set image deployment cat cat=spotmax/cat:v2                                                                                                                       130 ↵
deployment.extensions/cat image updated

```

```text
$ while True; do curl cat.example.com; echo; sleep 1;  done

```

```text
$ k set image deployment cat cat=spotmax/cat:v3 
```

```text
$ while True; do curl cat.example.com; echo; sleep 1;  done
```

```text
$ k rollout undo deployment cat                                                                                                                                       130 ↵
deployment.extensions/cat rolled back

```

```text
$ while True; do curl cat.example.com; echo; sleep 1;  done

```

```text
$ k apply -f cat-deployment-v3-with-readinesscheck.yaml                                                                                                               130 ↵
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/cat configured

```

```text
$ k rollout undo deployment cat                                                                                                                                       130 ↵
deployment.extensions/cat rolled back

```

