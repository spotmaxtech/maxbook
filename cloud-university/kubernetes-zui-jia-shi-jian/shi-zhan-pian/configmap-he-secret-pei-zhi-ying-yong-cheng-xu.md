# ConfigMap和Secret：配置应用程序

```text
$ kubectl create configmap myconfigmap --from-literal=foo=bar --from-literal=bar=baz --from-literal=one=two
configmap/myconfigmap created

```

```text
$ k create configmap fortune-config --from-literal=sleep-interval=12
configmap/fortune-config created

```

```text
$ k create configmap fortune-config --from-file=configmap-files
configmap/fortune-config created

```

```text
$ k create -f fortune-pod-configmap-volume.yaml                
pod/fortune-configmap-volume created

```

```text
k port-forward fortune-configmap-volume 8080:80 &

```

```text
$ curl -H "Accept-Encoding: gzip" -I localhost:8080                       
Handling connection for 8080
HTTP/1.1 200 OK
Server: nginx/1.17.5
Date: Fri, 25 Oct 2019 05:55:17 GMT
Content-Type: text/html
Last-Modified: Tue, 22 Oct 2019 16:16:19 GMT
Connection: keep-alive
ETag: W/"5daf2b53-264"
Content-Encoding: gzip

```

```text
$ k exec fortune-configmap-volume -c web-server ls /etc/nginx/conf.d 
```

```text

```

