
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



root@controlplane:~$ etcdctl endpoint status --cluster --write-out=table
{"level":"warn","ts":"2026-09-07T19:12:28.513283Z","caller":"flags/flag.go:94","msg":"unrecognized environment variable","environment-variable":"ETCDCTL_API=3"}
+-------------------------+------------------+---------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|        ENDPOINT         |        ID        | VERSION | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA  | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-------------------------+------------------+---------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
| https://172.30.1.2:2379 | 264d7b068180479b |   3.6.8 |           3.6.0 |  7.4 MB | 3.0 MB |                   59% | 2.1 GB |      true |      false |         5 |      13447 |              13447 |        |                          |             false |
+-------------------------+------------------+---------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
root@controlplane:~$ 


If the interviewer asks:

"What happens if etcd loses quorum?"

Say:

"The etcd cluster cannot commit new writes because Raft requires a majority. As a result, the Kubernetes API server cannot persist new cluster-state changes. Existing workloads may continue running on nodes, but Kubernetes control-plane operations such as creating or updating resources can fail."

