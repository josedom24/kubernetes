# Ejemplo: Desplegando WordPress con MariaDB

Puedes encontrar todos los ficheros con los que vamos a trabajar en el directorio [`wordpress`](https://github.com/josedom24/kubernetes/tree/master/ejemplos/wordpress).

Vamos a trabajar en un `namespace` lamado *wordpress*:

    kubectl create -f wordpress-ns.yaml 
    namespace "wordpress" created

## Desplegando la base de datos MariaDB

A continuación vamos a crear los `secrets` necesarios para la configuración de la base de datos, vamos a guardarlo en el fichero `mariadb-secret.yaml`:

    kubectl create secret generic mariadb-secret --namespace=wordpress \
                                --from-literal=dbuser=user_wordpress \
                                --from-literal=dbname=wordpress \
                                --from-literal=dbpassword=password1234 \
                                --from-literal=dbrootpassword=root1234 \
                                -o yaml --dry-run > mariadb-secret.yaml


    kubectl create -f mariadb-secret.yaml 
    secret "mariadb-secret" created

Creamos el servicio, que será de tipo *ClusterIP*:

    kubectl create -f mariadb-srv.yaml 
    service "mariadb-service" created

Y desplegamos la aplicación:

    kubectl create -f mariadb-deployment.yaml 
    deployment.apps "mariadb-deployment" created

Comprobamos los recursos que hemos creado hasta ahora:

    kubectl get deploy,service,pods -n wordpress
    NAME                                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    deployment.extensions/mariadb-deployment   1         1         1            1           20s

    NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
    service/mariadb-service   ClusterIP   10.98.24.76   <none>        3306/TCP   20s

    NAME                                     READY     STATUS    RESTARTS   AGE
    pod/mariadb-deployment-844c98579-cgp84   1/1       Running   0          20s

## Desplegando la aplicación Wordpress

Lo primero creamos el servicio:

    kubectl create -f wordpress-srv.yaml 
    service "wordpress-service" created

Y realizamos el despliegue:

    kubectl create -f wordpress-deployment.yaml 

Y vemos los recursos creados:

    kubectl get deploy,service,pods -n wordpress
    NAME                                         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    deployment.extensions/mariadb-deployment     1         1         1            1           6m
    deployment.extensions/wordpress-deployment   1         1         1            1           25s

    NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    service/mariadb-service     ClusterIP   10.98.24.76      <none>        3306/TCP                     6m
    service/wordpress-service   NodePort    10.111.158.165   <none>        80:30331/TCP,443:30015/TCP   25s

    NAME                                        READY     STATUS    RESTARTS   AGE
    pod/mariadb-deployment-844c98579-cgp84      1/1       Running   0          6m
    pod/wordpress-deployment-866b7d9fd8-wf5t4   1/1       Running   0          25s

Por último creamos el recurso `ingress` que nos va a permitir el acceso a la aplicación utilizando un nombre:

    kubectl create -f wordpress-ingress.yaml 
    ingress.extensions "wordpress-ingress" created

    kubectl get ingress -n wordpress
    NAME                HOSTS                      ADDRESS   PORTS     AGE
    wordpress-ingress   wp.172.22.200.178.nip.io             80        20s

Y accedemos:

![wp](img/wp1.png)