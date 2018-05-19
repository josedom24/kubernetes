# El servicio DNS en Kubernetes

Existe un componente de Kubernetes llamado *KubeDNS*, que ofrece un servidor DNS para que los pods puedan resolver diferentes nombres de recursos (servicios, pods, ...) a direcciones IP.

El servicio *KubeDNS* se comounica con el servidor de API y comprueba los servicios y pods creado para gestionar los diferentes registros de sus zonas de DNS.

## ¿Qué se puede resolver?

* Cada vez que se crea un nuevo servicio se crea un registro de tipo A con el nombre `servicio.namespace.svc.cluster.local`.
* Para cada puerto nombrado se crea un registro SRV del tipo `_nombre-puerto._nombre-protocolo.my-svc.my-namespace.svc.cluster.local` que resuelve el número del puerto y al CNAME: `servicio.namespace.svc.cluster.local`.
* Para cada pod creado con dirección IP 1.2.3.4, se crea un registro A de la forma `1-2-3-4.default.pod.cluster.local`.

## Comprobamos el DNS

Creamos un pod con la imagen [`busybox`](https://www.busybox.net/) a partir del fichero [ppbusybox.yaml`](../ejemplos/busybox/busybox.yaml):

    kubectl create -f busybox.yaml

Si tenemos los siguientes servicios creados:

    kubectl get service
    NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        5d
    nginx        NodePort    10.111.102.186   <none>        80:30305/TCP   2d

La consulta para resolver la IP de un servicio sería:

    kubectl exec -it busybox -- nslookup nginx
    Server:    10.96.0.10
    Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

    Name:      nginx
    Address 1: 10.111.102.186 nginx.default.svc.cluster.local

Como podemos observar el servidor DNS se llama `kube-dns.kube-system.svc.cluster.local` y tiene la IP `10.96.0.10`, es un servicio que representa un conjunto de pods que se está ejecutando en el espacio de nombres `kube-config`:

    kubectl get pods --namespace=kube-system -o wide
    NAME                                       READY     STATUS    RESTARTS   AGE
    ...
    kube-dns-86f4d74b45-c4zxz                  3/3       Running   0          6d
    ...

     kubectl get services --namespace=kube-system 
    NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
    ...
    kube-dns      ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP   6d
    ...

Y podemos comprobar el fichero de resolución de los pods de la siguiente forma:

    kubectl exec -it busybox -- cat /etc/resolv.conf
    nameserver 10.96.0.10
    search default.svc.cluster.local svc.cluster.local cluster.local openstacklocal
    options ndots:5

Para terminar veamos la resolución del nombre de un pod:

    kubectl exec -it busybox -- nslookup 192-168-72-178.default.pod.cluster.local
    Server:    10.96.0.10
    Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

    Name:      192-168-72-178.default.pod.cluster.local
    Address 1: 192.168.72.178

