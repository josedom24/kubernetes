# Almacenamiento disponible en Kubernetes: PersistentVolumen

Ya hemos visto que podemos añadir almacenamiento a un pod, sin embargo habría que distinguir dos conceptos:

* El desarrollador de aplicaciones no debería conocer con profundidad las características de almacenamiento que le ofrece el cluster. Desde este punto de vista, al desarrollador le puede dar igual que tipo de volumen puede utilizar (aunque en algún caso puede ser interesante indicarlo), lo que le interesa es, por ejemplo, el tamaño y las operaciones (lectura, lectura y escritura) del almacenamiento que necesita, y obtener del cluster un almacenamiento que se ajuste a esas características. La solicitud de almacenamiento se realiza con un elemento del cluster llamado *PersistentVolumenCliams*.
* El administrador será el responsable de dar de alta en el cluster los distintos tipos de almacenamientos que hay disponibles, y que se representa con un recurso llamado *PersistentVolumen*.

## Definiendo un PersistentVolumen

Un *PersistentVolumen* es un objeto que representa los volúmenes disponibles en el cluster. En él se van a definir los detalles del [backend](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes) de almacenamiento que vamos a utilizar, el tamaño disponible, los [modos de acceso](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes), las [políticas de reciclaje](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaim-policy), etc.

Tenemos tres modos de acceso, que depende del backend que vamos a utilizar:

* ReadWriteOnce: read-write solo para un nodo (RWO)
* ReadOnlyMany: read-only para muchos nodos (ROX)
* ReadWriteMany: read-write para muchos nodos (RWX)

Las políticas de reciclaje de volúmenes también depende del backend y son:

* Retain: Reclamación manual
* Recycle: Reutilizar contenido
* Delete: Borrar contenido

## Creando un PersistentVolumen con NFS

Vamos a instalar en el master del cluster (lo podríamos tener en cualquier otro servidor) un servidor NFS para compartir directorios en los nodos del cluster.

### Configuración en el master

En el master como root, ejecutamos:

    apt install nfs-kernel-server
    mkdir -p /shared/kubernetes/www

Y en el fichero `/etc/export` declaramos el directorio que vamos a exportar:

    /shared 10.0.0.0/24(rw,sync,no_root_squash,no_all_squash)

>Nota: La red 10.0.0.0/24 es la red interna donde se encuentra el master y los nodos del cluster.

Por último reiniciamos el servicio:

    systemctl restart nfs-kernel-server.service

Y comprobamos los directorios exportados:

    showmount -e 127.0.0.1
    Export list for 127.0.0.1:
    /shared 10.0.0.0/24

### Configuración en los nodos

En cada uno de los nodos del cluster vamos a montar el directorio compartido, para ello:

    apt install nfs-common
    
Y comprobamos los directorios exportados en el master:

    showmount -e 10.0.0.4
    Export list for 10.0.0.4:
    /shared 10.0.0.0/24

Y ya podemos montarlo:

    mount -t nfs4 10.0.0.4:/shared /data
