# my kubernetes yaml files

etcd v3 中取消了目录,因此没有了ls命令,不过可以通过下面的命令来查看所有的key<br>
ETCDCTL_API=3 ./etcdctl get / --prefix --keys-only


To allow all server access proxy url
kubectl proxy --address='0.0.0.0' --accept-hosts='^*$' . 

By default the controller redirects (301) to HTTPS if TLS is enabled for that ingress . If you want to disable that behaviour globally, you can use ssl-redirect: "false" in the NGINX config map . 
To configure this feature for specific ingress resources, you can use the ingress.kubernetes.io/ssl-redirect: "false" annotation in the particular resource.

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=nginxsvc/O=nginxsvc" . 
kubectl create secret tls tls-secret --key tls.key --cert tls.crt . 
