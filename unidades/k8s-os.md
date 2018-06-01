# Integración de Kubernetes con OpenStack

Vamos a realizar una instalación de Kubernetes y los vamos a configurar para que utilice los recursos ofrecidos por OpenStack: en concreto, vamos a poder crear de forma dinámica volúmenes (*PersistentVolumen*) ofrecidos por el componente de OpenStack encargado de la gestión de volúmenes: **Cinder**, y vamos a poder crear *Services* del tipo *LoadBalancer* creando un balanceador en el componente de redes de OpenStack: **Neutron**.

Tenemos dos opciones de configuración para conseguir comunicarnos con un proveedor de cloud:

* Configurar el componente [Kube Controller Manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/), que entre otras funciones, se encargará de conectar con el proveedor cloud (en nuestro caso OpenStack).
* Desde la versión 1.6 de Kubernetes se ha introducido un nuevo componente [Cloud Controller Manager](https://kubernetes.io/docs/tasks/administer-cluster/running-cloud-controller/) que específicamente es el encargado de gestionar el proveedor cloud, de tal manera que la arquitectura de componentes de Kuberentes se hace más modular.

En esta unidad vamos a configurar el *Kube Controller Manager* para comunicarse con OpenSatck. Para la segunda opción utilizando *Cloud Controller Manager* para comunicarse con OpenStack puede ser muy interesante el repositorio de [`openstack-cloud-controller-manager`](https://github.com/dims/openstack-cloud-controller-manager).

## Integración de Kubernetes y OpenStack con Kubeadm

Vamos a partir de una instalación de kubernetes con kubeadm (puedes seguir el apartado [Instalación de kubernetes con kubeadm](kubeadm.md)).

> Si necesitas reinstalar kubeadm debes ejecutar `kubeadm reset` en todos los nodos del cluster.

## Configuración del acceso a OpenStack

Lo primero, vamos a crear un fichero `cloud.conf` donde vamos a guardar las credenciales de acceso a OpenStack y los recursos que vamos a utilizar:

    [Global]
    auth-url=https://<openstack_endpoint>:5000/v3
    domain-name=Nombre del dominio
    tenant-name=Nombre del proyecto
    username=usuario
    password=contraseña
    ca-file=/etc/kubernetes/ca.crt

    [LoadBalancer]
    subnet-id=bf7be908-51a4-45d1-8403-391cfe1a73aa
    floating-network-id=49812d85-8e7a-4c31-baa2-d427692f6568

Se pueden configurar más opciones que puedes encontrar en este [enlace](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/#cloud-conf).

En mi caso el acceso a OpenStack se hace de forma cifrada (con https) por lo que necesito el certificado de la Autoridad Certificadora. Los ficheros `cloud.conf` y `ca.crt` los guardo en el directorio `/etc/kubernetes` del master y los nodos del cluster.

## Configurando kube-controller-manager

Tenemos que modificar la configuración del pod `kube-controller-manager` indicado el proveedor cloud que vamos autilizar y el fichero de configuración que debe utilizar (`cloud.conf`). Además debemos asegurarnos que el pod tiene acceso al fichero de configuración `cloud.conf` y al certificado de la CA. 
Para realizar la configuración debemos modificar el fichero `/etc/kubernetes/manifests/kube-controller-manager.yaml` en el nodo master de la siguiente forma:

<pre>
... 
spec:
  containers:
  - command:
    - kube-controller-manager
    ...
    <strong>- --cloud-provider=openstack</strong>
    <strong>- --cloud-config=/etc/kubernetes/cloud.conf</strong>
...
    volumeMounts:
    ...
    <strong>- mountPath: /etc/kubernetes/cloud.conf
      name: cloud-config
      readOnly: true 
    - mountPath: /etc/kubernetes/ca.crt
      name: cloud-ca
      readOnly: true</strong>
...
volumes:
  ...
  <strong>- hostPath:
      path: /etc/kubernetes/cloud.conf
      type: FileOrCreate
    name: cloud-config
  - hostPath:
      path: /etc/kubernetes/ca.crt
      type: FileOrCreate
    name: cloud-ca</strong>
</pre>

Una vez modificado correctamente el fichero, el pod `kube-controller-manager` se reiniciará con la nueva configuración, podemos comprobar que la modificación ha sido realizada con la siguiente instrucción:

    kubectl describe pod kube-controller-manager -n kube-system | grep '/etc/kubernetes/cloud.conf'

          --cloud-config=/etc/kubernetes/cloud.conf
          /etc/kubernetes/cloud.conf from cloud-config (ro)
        Path:          /etc/kubernetes/cloud.conf

## Configurando kubelet

A continuación debemos **reiniciar el componente `kubelet` en el master y en los nodos del cluster** modificando su configuración para indicarles el proveedor cloud que vamos a utilizar y el fichero de configuración que debe utilizar (`cloud.conf`). Para ello modificamos el fichero `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` añadiendo la siguiente línea:

    Environment="KUBELET_EXTRA_ARGS=--cloud-provider=openstack --cloud-config=/etc/kubernetes/cloud.conf"

Y reiniciamos el servicio:

    systemctl daemon-reload
    systemctl restart kubelet

Y comprobamos que el servicio está ejecutándose:

    ps xau | grep /usr/bin/kubelet

    root      5323 14.0  3.6 354508 74652 ?        Ssl  17:52   0:01 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true --network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin --cluster-dns=10.96.0.10 --cluster-domain=cluster.local --authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt --cadvisor-port=0 --rotate-certificates=true --cert-dir=/var/lib/kubelet/pki --cloud-provider=openstack --cloud-config=/etc/kubernetes/cloud.conf

## Probando la creación dinámica volúmenes por Cinder

Lo primero es crear un recurso del tipo [`StorageClass`](https://kubernetes.io/docs/concepts/storage/storage-classes/) Que nos permite configurar un origen o clase de almacenamiento disponible en el cluster. Para ello creamos el fichero `cinder-sc.yaml`:

    apiVersion: storage.k8s.io/v1beta1
    kind: StorageClass
    metadata:
      name: standard
      annotations:
        storageclass.beta.kubernetes.io/is-default-class: "true"
      labels:
        kubernetes.io/cluster-service: "true"
        addonmanager.kubernetes.io/mode: EnsureExists
    provisioner: kubernetes.io/cinder

Y creamos el recurso:

    kubectl create -f cinder-sc.yaml 
    storageclass.storage.k8s.io "standard" created

    kubectl get sc
    NAME                 PROVISIONER            AGE
    standard (default)   kubernetes.io/cinder   40s

A continuación vamos a crear un recurso `PersistentVolumeClaim` con el fichero `demo-cinder-pvc.yaml`:

    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: cinder-claim
      annotations:
        volume.beta.kubernetes.io/storage-class: "standard"
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi

Y lo creamos:

    kubectl create -f demo-cinder-pvc.yaml 
    persistentvolumeclaim "cinder-claim" created

Y podemos comprobar como de forma dinámica se ha creado un recurso `PersistentVolumen`:

    kubectl get pv,pvc
    
    NAME                                                         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS       CLAIM                  STORAGECLASS   REASON    AGE
    persistentvolume/pvc-577fe8a8-65c7-11e8-a509-fa163e99cb75    1Gi        RWO            Delete           Bound        default/cinder-claim   standard                 4s

    NAME                                 STATUS    VOLUME                                      CAPACITY   ACCESS MODES       STORAGECLASS   AGE
    persistentvolumeclaim/cinder-claim   Bound      pvc-577fe8a8-65c7-11e8-a509-fa163e99cb75   1Gi        RWO            standard       17s

Además podemos comprobar como realmente se ha creado un volumen en OpenStack:

    openstack volume list                   
    +--------------------------------------+-------------------------------------------------------------+-----------+------+-------------+
    | ID                                   | Name                                                        | Status    |  Size |Attached to |
    +--------------------------------------+-------------------------------------------------------------+-----------+------+-------------+
    | a6f6e873-5c1d-40d0-be26-c8307bf4edf3 | kubernetes-dynamic-pvc-577fe8a8-65c7-11e8-a509-fa163e99cb75 | available |    1 |             |
    +--------------------------------------+-------------------------------------------------------------+-----------+------+-------------+

