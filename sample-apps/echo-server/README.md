# Echo Server(Nginx Ingress版)

Echo Server是一个用来观察HTTP请求的简易程序，当你访问它的时候，它会回显你的请求。这个小程序很有用，特别当你通过Service或Ingress暴露Http应用的时候，可以用来观察到你应用的Request header（大部分反向代理的问题都可以通过这个小程序看出端倪）。

安装方法，使用本项目的提供的yaml文件：

```bash
kubectl apply -f echo-server.yaml
kubectl apply -f echo-server-ingress.yaml
```

访问`http://<node-ip>:<ingress-node-port>/echo-server`，会得到类似下面的内容：

```
Hostname: echo-server-6fbbbbd9fb-dffx7

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=10.233.102.133
	method=GET
	real path=/
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://172.50.10.16:8080/

Request Headers:
	accept=text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
	accept-encoding=gzip, deflate
	accept-language=zh-cn
	connection=close
	host=172.50.10.16:30080
	upgrade-insecure-requests=1
	user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_5) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/11.1.1 Safari/605.1.15
	x-forwarded-for=192.168.100.11
	x-forwarded-host=172.50.10.16:30080
	x-forwarded-port=80
	x-forwarded-proto=http
	x-original-uri=/echo-server
	x-real-ip=192.168.100.11
	x-request-id=45be093d74fe030bc5ca20e407b8b721
	x-scheme=http

Request Body:
	-no body in request-
```
