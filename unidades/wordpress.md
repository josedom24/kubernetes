# Wordpress

1. Creo el namespace

2.

kubectl create secret generic mariadb-secret --namespace=wordpress --from-literal=dbuser=user_wordpress --from-literal=dbname=wordpress --from-literal=dbpassword=password1234 --from-literal=dbrootpassword=root1234 -o yaml --dry-run > mariadb-secret.yaml


kubectl create -f wordpress-ns.yaml 
namespace "wordpress" created
jose@debian:~/github/kubernetes/ejemplos/wordpress$ kubectl create -f mariadb-secret.yaml 
secret "mariadb-secret" created
jose@debian:~/github/kubernetes/ejemplos/wordpress$ kubectl create -f mariadb-srv.yaml 
service "mariadb-service" created
jose@debian:~/github/kubernetes/ejemplos/wordpress$ kubectl create -f mariadb-deployment.yaml 
deployment.apps "mariadb-deployment" created
jose@debian:~/github/kubernetes/ejemplos/wordpress$ kubectl get deploy,service,pods -n wordpress
NAME                                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/mariadb-deployment   1         1         1            1           <invalid>

NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/mariadb-service   ClusterIP   10.98.24.76   <none>        3306/TCP   <invalid>

NAME                                     READY     STATUS    RESTARTS   AGE
pod/mariadb-deployment-844c98579-cgp84   1/1       Running   0          <invalid>


wordpress

 kubectl create -f wordpress-srv.yaml 
service "wordpress-service" created
jose@debian:~/github/kubernetes/ejemplos/wordpress$ kubectl create -f wordpress-deployment.yaml 


 kubectl get deploy,service,pods -n wordpress
NAME                                         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/mariadb-deployment     1         1         1            1           6m
deployment.extensions/wordpress-deployment   1         1         1            1           <invalid>

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/mariadb-service     ClusterIP   10.98.24.76      <none>        3306/TCP                     7m
service/wordpress-service   NodePort    10.111.158.165   <none>        80:30331/TCP,443:30015/TCP   <invalid>

NAME                                        READY     STATUS    RESTARTS   AGE
pod/mariadb-deployment-844c98579-cgp84      1/1       Running   0          6m
pod/wordpress-deployment-866b7d9fd8-wf5t4   1/1       Running   0          <invalid>


ingress
