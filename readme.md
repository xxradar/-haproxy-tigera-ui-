# How to setup HAProxy for Tigera UI
```
mkdir certs
...
```
```
mkdir haproxy
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
#   reqadd X-Forwarded-Proto:\ https
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
Run HAproxy
```
docker run -d --name my-running-haproxy3 \
       -v $PWD/haproxy:/usr/local/etc/haproxy:ro  -\
       -v $PWD/certs:/etc/ssl/private:ro \
       -p 7443:443 \
       --sysctl net.ipv4.ip_unprivileged_port_start=0 \
       haproxy:2.3
```
