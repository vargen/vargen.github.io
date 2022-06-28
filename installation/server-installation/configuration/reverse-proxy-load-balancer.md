# Reverse Proxy / Load Balancer

If you want to terminate TLS yourself, comment out CIFUZZ\_CERT\_FILE and CIFUZZ\_CERT\_KEY. Change CIFUZZ\_SERVER\_PORT to some other port, but keep port 443 (or other port where users will be accessing CI Fuzz) in CIFUZZ\_SERVER\_ORIGIN.

Set your reverse proxy or load balancer to direct HTTP and HTTP2/GRPC traffic to CIFUZZ\_SERVER\_PORT. Nginx example:

```
server {
 listen 443 ssl;
  ssl_certificate      /home/azureuser/tls/cifuzz.onprem.lab_cert.pem;
  ssl_certificate_key  /home/azureuser/tls/cifuzz.onprem.lab_key.pem;
 location / {
  proxy_pass http://localhost:80/;
 }
}
server {
 listen 6443 http2 ssl;
  ssl_certificate      /home/azureuser/tls/cifuzz.onprem.lab_cert.pem;
  ssl_certificate_key  /home/azureuser/tls/cifuzz.onprem.lab_key.pem;
 client_max_body_size 500M;
 location / {
   grpc_pass grpc://localhost:80;
 }
}
```

****
