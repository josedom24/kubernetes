# Recursos de Kubernetes: Ingress

Hasta ahora tenemos dos opciones principales para acceder a nuestras aplicaciones desde el exterior:

1. Utilizando servicios del tipo *NodePort*: Esta opción no es muy viable para entornos de producción ya que tenemos que utilizar puertos aleatorios desde 30000-40000.
2. Utilizando servicios del tipo *LoadBalancer*: Esta opción sólo es válida si trabajamos en un proveedor Cloud que nos cree un balanceador de carga para cada una de las aplicaciones, en cloud público puede ser una opción muy cara.

La solución puede ser utilizar un [`Ingress controller`](https://kubernetes.io/docs/concepts/services-networking/ingress/) que nos permite utilizar un proxy inverso (HAprozy, nginx, traefik,...) que por medio de reglas de encaminamiento que obtiene de la API de Kubernetes nos permite el acceso a nuestras aplicaciones por medio de nombres.

![ingress](img/ingress.png)

* Vamos a desplegar una pod que implementa un proxy inverso (podemos utilizar varias opciones: nginx, HAproxy, traefik,...) que esperara peticiones HTTP y HTTPS.
* Por lo tanto el `Ingress Controller` será el nodo del cluster donde se ha instalado el pod. En nuestro caso, para realizar el despliegue vamos utilizar un recurso de Kubernetes llamado [`DaemontSet`](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) que asegura que el despliegue se hace en todos los nodos del cluster y que los puertos que expone los pods (80 y 443) están mapeado y son accesible en los nodos. Por lo que cada uno de los nodos del cluster va a poseer una copia del `Ingress Controller`.

## 
