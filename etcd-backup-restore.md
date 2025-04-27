### ETCD BACKUP & RESTORE ###

For Testing purpose we will follow the below flow, 

- Create a sample configmap in development namespace
- Take the snapshot of etcd.
- Delete the configmap
- Restore the snapshot which should restore the configmap.

```bash
kubectl create namespace development

kubectl create configmap test-configmap --from-literal=foo=bar --from-literal=you=me -n development

kubectl get configmap -n development
NAME               DATA   AGE
kube-root-ca.crt   1      5h46m
test-configmap     2      86m

# ETCD snapshot save
sudo ETCDCTL_API=3 ./etcdctl snapshot save /etc/backups/etcd/backup.db  && \ 
                    --endpoints=https://127.0.0.1:2379  && \ 
                    --cacert=/etc/kubernetes/pki/etcd/ca.crt && \
                    --cert=/etc/kubernetes/pki/etcd/server.crt && \  
                    --key=/etc/kubernetes/pki/etcd/server.key
2025-04-27 15:32:35.936330 I | clientv3: opened snapshot stream; downloading
2025-04-27 15:32:36.231133 I | clientv3: completed snapshot read; closing
Snapshot saved at /etc/backups/etcd/backup.db

# ETCD snapshot status
sudo ETCDCTL_API=3 ./etcdctl snapshot status /etc/backups/etcd/backup.db && \
                    --cacert=/etc/kubernetes/pki/etcd/ca.crt && \
                    --cert=/etc/kubernetes/pki/etcd/server.crt && \
                    --key=/etc/kubernetes/pki/etcd/server.key
22f7442f, 21668, 1064, 4.3 MB


kubectl delete configmap test-configmap -n development
configmap "test-configmap" deleted


kubectl get configmap -n development
NAME               DATA   AGE
kube-root-ca.crt   1      5h50m

# Stopping kubelet. This is mandatory as we need to update the etcd datadir in etcd.yaml which is a static pod controlled/managed by kubelet
sudo systemctl stop kubelet

# Restore the snapshot
sudo ETCDCTL_API=3 ./etcdctl snapshot restore /etc/backups/etcd/backup.db   --data-dir /var/lib/etcd-restore
2025-04-27 15:36:13.268588 I | mvcc: restore compact to 20955
2025-04-27 15:36:13.278559 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32

sudo vi /etc/kubernetes/manifests/etcd.yaml
# update all references of datadir paths

# restart the kubelet
sudo systemctl restart kubelet

# Check for the controlplane pods
sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
6da856577fff2       499038711c081       27 seconds ago      Running             etcd                      1                   ec9b5c049f6ac       etcd-controlplane   
2338dc9107afc       8d72586a76469       44 seconds ago      Running             kube-scheduler            4                   564d7b6201225       kube-scheduler-controlplane
b29b40405bcc1       1d579cb6d6967       50 seconds ago      Running             kube-controller-manager   3                   677f65dd2acb8       kube-controller-manager-controlplane

# Now the we got back the configmap that was deleted
kubectl get configmap -n development
NAME               DATA   AGE
kube-root-ca.crt   1      5h55m
test-configmap     2      95m
```
