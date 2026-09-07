
root@controlplane:~$ kubectl get pods -n kube-system -o wide
NAME                                   READY   STATUS    RESTARTS      AGE   IP              NODE           NOMINATED NODE   READINESS GATES

etcd-controlplane                      1/1     Running   2 (32m ago)   18d   172.30.1.2      controlplane   <none>           <none>


ENDpoint health
###

https://127.0.0.1:2379 means localhost

root@controlplane:~$ kubectl exec -n kube-system etcd-controlplane -- \
  etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health
https://127.0.0.1:2379 is healthy: successfully committed proposal: took = 23.942472ms
root@controlplane:~$ 

endpointstatus
####
kubectl exec -n kube-system etcd-controlplane -- \
  etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --write-out=table \
  endpoint status
