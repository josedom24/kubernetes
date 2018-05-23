# Recursos de Kubernetes: Namespaces

Los [`Namespaces`](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) nos permiten aislar recursos para el uso por los distintos usuarios del cluster, para trabajar en distintos proyectos. A cada `namespace` se le puede asignar una cuota y definirle reglas y políticas de acceso.

## Trabajando con Namespaces

Para obtener la lista de `Namespaces` ejecutamos:

    kubectl get namespaces
    NAME          STATUS    AGE
    default       Active    1d
    kube-public   Active    1d
    kube-system   Active    1d

* `default`: Espacio de nombres por defecto.
* `kube-system`: Espacio de nombres creado y gestionado por Kubernetes.
* `kube-public`: Espacio de nombres accesible por todos los usuarios, reservado para uso interno del cluster.

Para crear un nuevo `Namespace`:

    kubectl create ns proyecto1
    namespace "proyecto1" created

Otra forma de crear un `Namespace` es a partir de un fichero yaml con su definición:

    apiVersion: v1
    kind: Namespace
    metadata:
      name: proyecto1

Podemos ver las características del nuevo espacio de nombres:

    kubectl describe ns proyecto1
    Name:         proyecto1
    Labels:       <none>
    Annotations:  <none>
    Status:       Active

    No resource quota.

    No resource limits.

Y su definición yaml:

    kubectl get ns proyecto1 -o yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      creationTimestamp: 2018-05-23T16:19:58Z
      name: proyecto1
      resourceVersion: "152566"
      selfLink: /api/v1/namespaces/proyecto1
      uid: 2306825c-5ea5-11e8-ab66-fa163e99cb75
    spec:
      finalizers:
      - kubernetes
    status:
      phase: Active

## Crear recursos en un namespace

Para crear un recurso en un `namespace` debemos indicar el nombre del espacio de nombres en la etiqueta `namespace` en su definición:

    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: nginx
      namespace: proyecto1
      ...

También podemos crearlos sin el fichero yaml:

    kubectl run nginx --image=nginx -n proyecto1
    deployment.apps "nginx" created

    kubectl get deploy -n proyecto1
    NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx     1         1         1            1           15s

Y creamos el servicio asociado:

    kubectl expose deployment/nginx --port=80 --type=NodePort -n proyecto1
    service "nginx" exposed
    
    kubectl get services -n proyecto1
    NAME      TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    nginx     NodePort   10.107.121.169   <none>        80:30352/TCP   10s

## Eliminando un namespace

Al eliminar un `namespace` se borran todos los recursos que hemos creado en él. 

    kubectl delete ns proyecto1
    namespace "proyecto1" deleted
