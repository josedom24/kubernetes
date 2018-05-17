# Recursos de Kubernetes: Services

Los servicios ([`services`](https://kubernetes.io/docs/concepts/services-networking/service/)) nos permiten acceder a nuestra aplicaciones.

* Un servicio es una abstracción que define un conjunto de pods que implementan un micro-servicio. (Por ejemplo el *servicio frontend*).
* Ofrecen una dirección virtual (ip y puerto) y un nombre que identifica al conjunto de pods que representa, al cual nos podemos conectar.
* La conexión al servicio se puede realizar desde otros pods o desde el exterior.

![services](img/services.png)

Los servicios se implementan con *iptables*. El componente *kube-proxy* de Kubernetes se comunica con el servidor de API para comprobar si se han creado nuevos servicios. 

Cuando se crea un nuevo servicio se abre un puerto aleatorio que permite que accediendo a la IP del cluster y a ese puerto se acceda al conjunto de pods. Si tenemos más de un pod el acceso se hará siguiento una política *round-robin*.
