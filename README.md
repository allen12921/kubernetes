# my kubernetes yaml files

etcd v3 中取消了目录,因此没有了ls命令,不过可以通过下面的命令来查看所有的key<br>
ETCDCTL_API=3 ./etcdctl get / --prefix --keys-only


To allow all server access proxy url
kubectl proxy --address='0.0.0.0' --accept-hosts='^*$' . 

By default the controller redirects (301) to HTTPS if TLS is enabled for that ingress . If you want to disable that behaviour globally, you can use ssl-redirect: "false" in the NGINX config map . 
To configure this feature for specific ingress resources, you can use the ingress.kubernetes.io/ssl-redirect: "false" annotation in the particular resource.

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=nginxsvc/O=nginxsvc" . 
kubectl create secret tls tls-secret --key tls.key --cert tls.crt . 


use configmap for configuration . 
kubectl create configmap influxdb-config --from-file=config.toml=influxdb.config.toml  -n kube-system  
        volumeMounts:  
        - mountPath: /data   
          name: influxdb-storage . 
        - mountPath: /etc/  
          name: influxdb-config . 
      nodeSelector:  
          influxdb: "true" . 
      volumes:  
      - name: influxdb-storage . 
        hostPath:   
          path: /opt/influxdb . 
      - name: influxdb-config . 
        configMap:  
         name: influxdb-config . 
         
use pv  
1.define pv(it can be in any namespace)  
apiVersion: v1  
kind: PersistentVolume  
metadata:  
  name: nfs-defalut  
spec:  
  capacity:  
    storage: 2Gi  
  accessModes:  
    - ReadWriteMany  
  nfs:  
    server: 192.168.168.8  
    path: "/volume1/k8s"  
    
 2.Binding pv by pvc  
 PersistentVolumeClaim with a specific amount of storage requested and with certain access modes. A control loop in the master watches for new PVCs, finds a matching PV (if possible), and binds them together.Claims will remain unbound indefinitely if a matching volume does not exist. Claims will be bound as matching volumes become available. For example, a cluster provisioned with many 50Gi PVs would not match a PVC requesting 100Gi. The PVC can be bound when a 100Gi PV is added to the cluster.

 ---  
kind: PersistentVolumeClaim  
apiVersion: v1  
metadata:  
  name: nfs  
  namespace: kube-system  
spec:  
  accessModes:  
    - ReadWriteMany  
  resources:  
    requests:  
      storage: 1Gi  
 3.Using  
 Pods use claims as volumes.  
 Pods use claims as volumes. The cluster inspects the claim to find the bound volume and mounts that volume for a pod. For volumes which support multiple access modes, the user specifies which mode desired when using their claim as a volume in a pod.  
       volumes:  
       - name: influxdb-storage  
         persistentVolumeClaim:  
          claimName: nfs  
 4.Releasing  
 When a user is done with their volume, they can delete the PVC objects from the API  
 5.Reclaiming  
 The reclaim policy for a PersistentVolume tells the cluster what to do with the volume after it has been released of its claim. Currently, volumes can either be Retained, Recycled or Deleted.Retention allows for manual reclamation of the resource,deletion removes both the PersistentVolume object from Kubernetes, as well as deleting the associated storage asset in external infrastructure. Volumes that were dynamically provisioned are always deleted.
 6.Recycling  
 If supported by appropriate volume plugin, recycling performs a basic scrub (rm -rf /thevolume/*) on the volume and makes it available again for a new claim.  
 

