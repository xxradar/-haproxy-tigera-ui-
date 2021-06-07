# How to setup HAProxy for Tigera UI
### Create certificates
```
mkdir certs
cd certs
openssl req -x509 -newkey rsa:2048 -keyout ./tls.key -out ./tls.crt -days 365 -nodes -subj "/CN=test3.radarhack.com"
cat tls.crt >tls.pem
cat tls.key >>tls.pem
```

### Create HAproxy config file

```
mkdir haproxy
cd haproxy
```
Create a file haproxy.cfg
```
global
	daemon
	maxconn      25000
	maxcompcpuusage 20
    log          /var/lib/haproxy/dev/log local7 notice

    defaults
	timeout http-request 45s
	timeout connect 5s
	timeout client 45s
	timeout server 45s
	timeout queue 60s


frontend www-https
   bind *:443 ssl crt /etc/ssl/private/tls.pem alpn h2,http/1.1
   default_backend  CALENT3_TIGERA

backend CALENT3_TIGERA
    description BE Preprod Calico Enterprise UI
    mode http
    balance source
    server calent3-no1 10.11.2.113:30771 check maxconn 1024 ssl verify none


backend BLACKHOLE
	# empty backend to force 503 response
	mode http
	log global
```
Note: 10.11.2.113:30771 in this cfg file is the node port on a k8s node.
### Run HAproxy...
```
docker run -d --name my-running-haproxy3 \
       -v $PWD/haproxy:/usr/local/etc/haproxy:ro  -\
       -v $PWD/certs:/etc/ssl/private:ro \
       -p 7443:443 \
       --sysctl net.ipv4.ip_unprivileged_port_start=0 \
       haproxy:2.3
```
The UI is available in this case on `https://test3.radarhack.com:7443`
