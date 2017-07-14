# my kubernetes yaml files

etcd v3 中取消了目录,因此没有了ls命令,不过可以通过下面的命令来查看所有的key<br>
ETCDCTL_API=3 ./etcdctl get / --prefix --keys-only
ETCDCTL_API=3 ./etcdctl --endpoints 127.0.0.1:2379 -w table endpoint status


To allow all server access proxy url
kubectl proxy --address='0.0.0.0' --accept-hosts='^*$' . 

By default the controller redirects (301) to HTTPS if TLS is enabled for that ingress . If you want to disable that behaviour globally, you can use ssl-redirect: "false" in the NGINX config map . 
To configure this feature for specific ingress resources, you can use the ingress.kubernetes.io/ssl-redirect: "false" annotation in the particular resource.

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=nginxsvc/O=nginxsvc" . 
kubectl create secret tls tls-secret --key tls.key --cert tls.crt . 


# use configmap for configuration . 
kubectl create configmap influxdb-config --from-file=config.toml=influxdb.config.toml  -n kube-system  
        volumeMounts:  
        - mountPath: /data   
          name: influxdb-storage 
        - mountPath: /etc/  
          name: influxdb-config  
      nodeSelector:  
          influxdb: "true"  
      volumes:  
      - name: influxdb-storage   
        hostPath:   
          path: /opt/influxdb   
      - name: influxdb-config   
        configMap:  
         name: influxdb-config   
         
# pull image from private registry
1. create login key . 
kubectl create secret docker-registry regsecret --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=testz@my.com  
2. create pod use the key . 
apiVersion: v1  
kind: Pod  
metadata:  
  name: private-reg  
spec:  
  containers:  
    - name: private-reg-container  
      image: <your-private-image>   
  imagePullSecrets:  
    - name: regsecret 
# use pv  
1.define pv(it can be in any namespace)  
apiVersion: v1  
kind: PersistentVolume  
metadata:  
  name: nfs-defalut  
  labels:
     release: "stable"  
spec:  
  storageClassName: slow
  capacity:  
    storage: 2Gi  
  accessModes:  
    - ReadWriteMany  
  nfs:  
    server: 192.168.168.8  
    path: "/volume1/k8s"  
 
 Class  
 A PV can have a class, which is specified by setting the storageClassName attribute to the name of a StorageClass. A PV of a particular class can only be bound to PVCs requesting that class. A PV with no storageClassName has no class and can only be bound to PVCs that request no particular class.  
 Mount Options  
 A Kubernetes administrator can specify additional mount options for when a Persistent Volume is being mounted on a node.  
 You can specify a mount option by using the annotation volume.beta.kubernetes.io/mount-options on your Persistent Volume.  
 
 2.Binding pv by pvc(create pnc in ns whihc your pod in)  
 PersistentVolumeClaim with a specific amount of storage requested and with certain access modes. A control loop in the master watches for new PVCs, finds a matching PV (if possible), and binds them together.Claims will remain unbound indefinitely if a matching volume does not exist. Claims will be bound as matching volumes become available. For example, a cluster provisioned with many 50Gi PVs would not match a PVC requesting 100Gi. The PVC can be bound when a 100Gi PV is added to the cluster.

 ---  
kind: PersistentVolumeClaim  
apiVersion: v1  
metadata:  
  name: nfs  
  namespace: kube-system  
spec:  
  storageClassName: slow
  accessModes:  
    - ReadWriteMany  
  resources:  
    requests:  
      storage: 1Gi  
  selector:  
    matchLabels:  
      release: "stable"   
    matchExpressions:  
      - {key: environment, operator: In, values: [dev]}  
Claims can specify a label selector to further filter the set of volumes. Only the volumes whose labels match the selector can be bound to the claim. The selector can consist of two fields:  
matchLabels - the volume must have a label with this value  
matchExpressions - a list of requirements made by specifying key, list of values, and operator that relates the key and values. Valid operators include In, NotIn, Exists, and DoesNotExist.  
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
 
A Note on Namespaces  
PersistentVolumes binds are exclusive, and since PersistentVolumeClaims are namespaced objects, mounting claims with “Many” modes (ROX, RWX) is only possible within one namespace.

StorageClasses  
Each StorageClass contains the fields provisioner and parameters, which are used when a PersistentVolume belonging to the class needs to be dynamically provisioned.  
kind: StorageClass  
apiVersion: storage.k8s.io/v1  
metadata:  
  name: standard  
provisioner: kubernetes.io/aws-ebs  
parameters:   
  type: gp2  

# export resources from kubernet cluster
kubectl get --export -o=json ns | \
jq '.items[] |
	select(.metadata.name!="kube-system") |
	select(.metadata.name!="default") |
	del(.status,
        .metadata.uid,
        .metadata.selfLink,
        .metadata.resourceVersion,
        .metadata.creationTimestamp,
        .metadata.generation
    )' >./cluster-dump/ns.json
    
for ns in $(jq -r '.metadata.name' < ./cluster-dump/ns.json);do
    echo "Namespace: $ns"
    kubectl --namespace="${ns}" get --export -o=json svc,rc,secrets,ds | \
    jq '.items[] |
        select(.type!="kubernetes.io/service-account-token") |
        del(
            .spec.clusterIP,
            .metadata.uid,
            .metadata.selfLink,
            .metadata.resourceVersion,
            .metadata.creationTimestamp,
            .metadata.generation,
            .status,
            .spec.template.spec.securityContext,
            .spec.template.spec.dnsPolicy,
            .spec.template.spec.terminationGracePeriodSeconds,
            .spec.template.spec.restartPolicy
        )' >> "./cluster-dump/cluster-dump.json"
done
Notice that pods and service tokens are explicitly omitted altogether, as they are inherently non-portable resources that are created and managed by other components.

kubectl create -f cluster-dump/ns.json
kubectl create -f cluster-dump/cluster-dump.json
