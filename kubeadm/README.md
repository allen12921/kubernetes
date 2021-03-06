# Install 

kubeadm join --token 8fd088.79f9a3d7492c47b8 10.0.1.227:6443
kubectl --namespace=kube-system scale deployment kube-dns --replicas=2

By default, your cluster will not schedule pods on the master for security reasons, If you want to be able to schedule pods on the master,
kubectl taint nodes --all node-role.kubernetes.io/master-

# Tear down
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
Then, on the node being removed, reset all kubeadm installed state:
kubeadm reset


Force delete:
kubectl delete pod NAME --grace-period=0 --force


# to extend ectd to cluster:
change /etc/kubernetes/manifests/etcd.yaml then kubelet will restart etcd pods then restart kubelet
  - command:  
    - etcd 
    - --listen-client-urls=http://0.0.0.0:2379 
    - --advertise-client-urls=http://192.168.168.41:2379  
    - --listen-peer-urls=http://192.168.168.41:2380 
    - --initial-advertise-peer-urls=http://192.168.168.41:2380  
    - --initial-cluster="default=http://192.168.168.41:2380,ect2=http://192.168.168.42:2380"  
    - --initial-cluster-token=mycluster
    - --data-dir=/var/lib/etcd 
    - --initial-cluster-state=new
 ##- --force-new-cluster (if you etcd data was copied from other cluster then enable this at the first start time)

you can copy either the same etcd.yaml to other worker node same place and change ip also replace --initial-cluster-state and add --name
  - command:  
    - etcd  
    - --name=ect2 
    - --listen-client-urls=http://0.0.0.0:2379   
    - --advertise-client-urls=http://192.168.168.42:2379    
    - --listen-peer-urls=http://192.168.168.42:2380
    - --initial-cluster-token=mycluster
    - --initial-advertise-peer-urls=http://192.168.168.42:2380    
    - --initial-cluster="default=http://192.168.168.41:2380,ect2=http://192.168.168.42:2380"   
    - --initial-cluster-state=existing  
    - --data-dir=/var/lib/etcd  
 
 or directly start new ectd from other servers sometime you may want add -strict-reconfig-check option. If this option is passed to etcd, etcd rejects reconfiguration requests if the number of started members will be less than a quorum of the reconfigured cluster.


 
# for fit fluentd-elasticsearch installation
kubectl label node ozintel-docker01.lan beta.kubernetes.io/fluentd-ds-ready=true
and delete below option from /etc/sysconfig/docker so that write container logs to json.log file and the kubelet can creates symlinks that
capture the pod name, namespace, container name & Docker container ID to the docker logs for pods in the /var/log/containers
--log-driver=journald

Add below to  spec to allow assign pods on master   
      tolerations:  
            - effect: NoSchedule   
              operator: Exists  
              
                
Use nodeSelector   to assign pods on specified label nodes .       
 kubectl label nodes ozintel-docker03.lan influxdb=true  
 
 add below to  spec   
 nodeSelector:  
        influxdb: "true"  
