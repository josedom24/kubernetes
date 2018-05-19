 kubectl create -f letschat/
deployment.extensions "letschat" created
service "letschat" created
deployment.extensions "mongo" created
service "mongo" created
jose@pandora:~/github/kubernetes/ejemplos$ kubectl get deploy,rs,service,pods
NAME                             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/letschat   3         3         3            3           <invalid>
deployment.extensions/mongo      1         1         1            1           <invalid>
deployment.extensions/nginx      2         2         2            2           2d

NAME                                        DESIRED   CURRENT   READY     AGE
replicaset.extensions/letschat-57cb7f589f   3         3         3         <invalid>
replicaset.extensions/mongo-769fdf6975      1         1         1         <invalid>
replicaset.extensions/nginx-6f596bfb6d      2         2         2         2d
replicaset.extensions/nginx-84f79f5cdd      0         0         0         2d

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          6d
service/letschat     NodePort    10.100.157.57    <none>        8080:30283/TCP   <invalid>
service/mongo        ClusterIP   10.97.213.5      <none>        27017/TCP        <invalid>
service/nginx        NodePort    10.111.102.186   <none>        80:30305/TCP     2d

NAME                            READY     STATUS    RESTARTS   AGE
pod/busybox                     1/1       Running   79         3d
pod/letschat-57cb7f589f-czl8d   1/1       Running   0          <invalid>
pod/letschat-57cb7f589f-jfnwb   1/1       Running   0          <invalid>
pod/letschat-57cb7f589f-zzwnv   1/1       Running   0          <invalid>
pod/mongo-769fdf6975-nlt4r      1/1       Running   0          <invalid>
pod/nginx-6f596bfb6d-5wnl7      1/1       Running   0          2d
pod/nginx-6f596bfb6d-6nzfn      1/1       Running   0          2d
