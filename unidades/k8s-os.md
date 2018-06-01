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

Tenemos que modificar la configuración del pod *kube-controller-manager* indicado el proveedor cloud que vamos autilzar y el fichero de configuración que debe utilizar (`cloud.conf`). Además debemos asegurarnos que el pod tiene acceso al fichero de configuración `cloud.conf` y al certificado de la CA. 
Para realizar la configuración debemos modificar el fichero `/etc/kubernetes/manifests/kube-controller-manager.yaml` de la siguiente forma:

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

