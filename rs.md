# Recursos de Kubernetes: ReplicaSet

`ReplicaSet` es un recurso de Kubernetes que asegura que siempre se ejecute un número de replicas de un pod determinado. Por lo tanto, nos asegura que un conjunto de pods siempre estñan funcionando y disponibles. Nos proporciona las siguientes características:
  * Que no haya caída del servicio
  * Tolerancia a errores
  * Escabilidad dinámica

![rs](img/rs.png)

## Definición yaml de un ReplicaSet

Vamos a ver un ejemplo de definición de ReplicaSet en el fichero [`nginx-rs.yaml`](ejemplo/nginx/nginx-rs.yaml):

    apiVersion: extensions/v1beta1
    kind: ReplicaSet
    metadata:
      name: nginx
      namespace: default
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
            - image:  nginx
              name:  nginx

