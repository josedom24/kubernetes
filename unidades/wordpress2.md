# Ejemplo: Desplegando WordPress con MariaDB con almacenamiento persistente

Puedes encontrar todos los ficheros con los que vamos a trabajar en el directorio [`wordpress2`](https://github.com/josedom24/kubernetes/tree/master/ejemplos/wordpress2).

## Configuración del servidor NFS

Para este ejercicio hemos creado dos directorio que hemos exportado en el servidor NFS, en el fichero `/etc/exports` encontramos:

    /var/shared/vol1 10.0.0.0/24(rw,sync,no_root_squash,no_all_squash)
    /var/shared/vol2 10.0.0.0/24(rw,sync,no_root_squash,no_all_squash)

Y en los clientes montamos dichos directorios:

    mount -t nfs4 10.0.0.4:/var/shared/vol1 /var/data/vol1
    mount -t nfs4 10.0.0.4:/var/shared/vol2 /var/data/vol2

## Gestión del almacenamiento para nuestro despliegue

Lo primero que hacemos es crear los dos *pv* que podemos encontrar definidos en el fichero [`wordpress-pv.yaml`](https://github.com/josedom24/kubernetes/tree/master/ejemplos/wordpress2/wordpress-pv.yaml):

    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: volumen1
    spec:
      capacity:
        storage: 5Gi
      accessModes:
        - ReadWriteMany
      persistentVolumeReclaimPolicy: Recycle
      nfs:
        path: /var/shared/vol1
        server: 10.0.0.4
    ---
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: volumen2
    spec:
      capacity:
        storage: 5Gi
      accessModes:
        - ReadWriteMany
      persistentVolumeReclaimPolicy: Recycle
      nfs:
        path: /var/shared/vol2
        server: 10.0.0.4

Lo creamos:

    kubectl create -f wordpress-pv.yaml
    persistentvolume "volumen1" created
    persistentvolume "volumen2" created

A continuación vamos a trabajar con un *namespace*, por lo tanto lo creamos:

    kubectl create -f wordpress-ns.yaml 
    namespace "wordpress" created

Vamos arealizar la solicitud de almacenamiento para la base de datos, que tenemos definido en el fichero [`mariadb-pvc.yaml`](https://github.com/josedom24/kubernetes/tree/master/ejemplos/wordpress2/mariadb-pvc.yaml):

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mariadb-pvc
      namespace: wordpress
    spec:
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 5Gi

De forma similar solicitamos el almacenamiento para nuestra aplicación. Esta solicitud la tenemos definida en el fichero [`wordpress-pvc.yaml`](https://github.com/josedom24/kubernetes/tree/master/ejemplos/wordpress2/wordpress-pvc.yaml).

Creamos las solicitudes:

    kubectl create -f wordpress-pvc.yaml 
    persistentvolumeclaim "wordpress-pvc" created
    
    kubectl create -f mariadb-pvc.yaml  
    persistentvolumeclaim "mariadb-pvc" created

Y lo comprobamos:

    kubectl get pv,pvc -n wordpress     
    NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                     STORAGECLASS   REASON    AGE
    persistentvolume/volumen1   5Gi        RWX            Recycle          Bound     wordpress/wordpress-pvc                            50s
    persistentvolume/volumen2   5Gi        RWX            Recycle          Bound     wordpress/mariadb-pvc                              49s

    NAME                                  STATUS    VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    persistentvolumeclaim/mariadb-pvc     Bound     volumen2   5Gi        RWX                           10s
    persistentvolumeclaim/wordpress-pvc   Bound     volumen1   5Gi        RWX                           23s




kubectl create -f mariadb-srv.yaml 
service "mariadb-service" created


kubectl create -f wordpress-srv.yaml 
service "wordpress-service" created

kubectl create -f wordpress-ingress.yaml 
ingress.extensions "wordpress-ingress" created


kubectl create -f mariadb-secret.yaml 
secret "mariadb-secret" created


kubectl create -f mariadb-deployment.yaml                      
deployment.apps "mariadb-deployment" created

kubectl create -f wordpress-deployment.yaml                        
deployment.apps "wordpress-deployment" created

kubectl get deploy -n wordpress            
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
mariadb-deployment     1         1         1            0           0s
wordpress-deployment   1         1         1            0           <invalid>






Puedes encontrar todos los ficheros con los que vamos a trabajar en el directorio [`wordpress2`](https://github.com/josedom24/kubernetes/tree/master/ejemplos/wordpress2).

## Definiendo el almacenamiento

Creamos dos *pv* para la base de datos y la aplicación:

En el fichero `mariadb-pv.yaml`:

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mariadb-pvc
      namespace: wordpress
      labels:
        app: wordpress
        type: database
    spec:
      capacity:
        storage: 5Gi
      accessModes:
        - ReadWriteMany
      persistentVolumeReclaimPolicy: Recycle
      nfs:
        path: /var/shared
        server: 10.0.0.4

Vamos a trabajar en un `namespace` llamado *wordpress*:

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

## Problemas que nos encontramos

En realidad no son problemas, son la consecuencia de que los **pods son efimeros**, cuando se elimina un pod su información se pierde. Por lo tanto nos podemos encontrar con algunas circunstancias:

1. ¿Qué pasa si eliminamos el despliegue de mariadb?, o, ¿se elimina el pod de mariadb y se crea uno nuevo?. En estas circunstancias **se pierde la información de la base de datos** y el proceso de instalación comenzará de nuevo.
2. ¿Qué pasa si escalamos el despliegue de la base de datos y tenemos dos pods ofreciendo la base de datos?. En cada acceso a la aplicación se va a balancear la consulta a la base de datos entre los dos pods (**uno que tiene la información de la instalación y otro que que no tiene información**), por lo que en los accesos consecutivos nos va a ir mostrando la aplicación y en el siguiente acceso nos va a decir que hay que instalar el wordpress.
3. Si escribimos un post en el wordpress y subimos una imagen, ese fichero se va a guardar en el pod que está corriendo la aplicación, por lo tanto si se borra, **se perderá el contenido estático**.
4. En el caso que tengamos un pods con contenido estático (por ejemplo imágenes) y escalamos el despliegue de wordpress a dos pods, **en uno se encontrará la imagen pero en el otro no**, por lo tanto en los distintos accesos consecutivos que se hagan a la aplicación se ira mostrando o no la imagen según el pod que este respondiendo.

Para solucionar estos problemas veremos en los siguientes apartados la utilización de volúmenes en Kubernetes.