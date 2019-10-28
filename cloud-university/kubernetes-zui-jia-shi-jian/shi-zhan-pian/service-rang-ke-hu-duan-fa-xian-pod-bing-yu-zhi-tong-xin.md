# Service：让客户端发现pod并与之通信

## 配置Ingress处理TLS传输

```text
$ openssl genrsa -out tls.key 2048
Generating RSA private key, 2048 bit long modulus
..................+++
...................................................+++
```

```text
$ openssl req -new -x509 -key tls.key -out tls.cert -days  360 -subj /CN=cat.example.com 
```

```text
$ kubectl create secret tls tls-secret --cert=tls.cert --key=tls.key               
secret/tls-secret created

```

