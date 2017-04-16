# my kubernetes yaml files

etcd v3 中取消了目录,因此没有了ls命令,不过可以通过下面的命令来查看所有的key
 ETCDCTL_API=3 ./etcdctl get / --prefix --keys-only
