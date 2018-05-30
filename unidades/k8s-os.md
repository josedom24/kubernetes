# Integración de Kubernetes con OpenStack

Vamos a realizar una instalación de Kubernetes y los vamos a configurar para que utilice los recursos ofrecidos por OpenStack: en concreto, vamos a poder crear de forma dinámica volúmenes (*PersistentVolumen*) ofrecidos por el componente de OpenStack encargado de la gestión de volúmenes: **Cinder**, y vamos a poder crear *Services* del tipo *LoadBalancer* creando un balanceador en el componente de redes de OpenStack: **Neutron**.

Tenemos dos opciones de configuración para conseguir comunicarnos con un proveedor de cloud:

* Configurar el componente [Kube Controller Manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/), que entre otras funciones, se encargará de conectar con el proveedor cloud (en nuestro caso OpenStack).
* Desde la versión 1.6 de Kubernetes se ha introducido un nuevo componente [Cloud Controller Manager](https://kubernetes.io/docs/tasks/administer-cluster/running-cloud-controller/) que específicamente es el encargado de gestionar el proveedor cloud, de tal manera que la arquitectura de componentes de Kuberentes se hace más modular.

En esta unidad vamos a configurar el *Kube Controller Manager* para comunicarse con OpenSatck. Para la segunda opción utilizando *Cloud Controller Manager* para comunicarse con OpenStack puede ser muy interesante el repositorio de [`openstack-cloud-controller-manager`](https://github.com/dims/openstack-cloud-controller-manager).

## Integración de Kubernetes y OpenStack con Kubeadm

Vamos a partir de una instalación de kubernetes con kubeadm (puedes seguir el apartado [Instalación de kubernetes con kubeadm](kubeadm.md)).

> Si necesitas resinstalar tu instalación con kubeadm puedes ejecutar `kubeadm reset` en todos los nodos del cluster.

