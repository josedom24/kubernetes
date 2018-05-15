# Recursos de Kubernetes: Deployment

[`Deployment`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) e la unidad de más alto nivel que podemos gestionar en Kubernetes. Nos permite definir diferentes funciones:

* Control de replicas
* Escabilidad de pods
* Actualizaciones continúas
* Despliegues automáticos
* *Rollback* a versiones anteriores

![deoployment](img/deployment.png)

## Definición yaml de un Deployment

Vamos a ver un ejemplo de definición de un `Deployment` en el fichero [`nginx-deployment.yaml`](ejemplo/nginx/nginx-deployment.yaml):

    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: nginx
      namespace: default
      labels:
        app: nginx
    spec:
      revisionHistoryLimit: 2
      strategy:
        type: RollingUpdate
      replicas: 2
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - image: nginx
            name: nginx
            ports:
            - name: http
              containerPort: 8080

El despliegue de un `Deployment` crea un ReplicaSet y los Pods correspondientes. Por lo tanto en la definición de un `Deployment` se define también el replicaSet asociado. En la práctica siempre vamos a trabajar con `Deployment`. Los atributos relacionados con el `Deployment` que hemos indicado en la definición son:

* `revisionHistoryLimit`: Indicamos cuántos `ReplicaSets` antiguos deseamos conservar, para poder realizar *rollback* a estados anteriores. Por defecto, es 10.
* `strategy`: Indica el modo en que se realiza una actualización del `Deployment`: `recreate`: elimina los Pods antiguos y crea los nuevos; `RollingUpdate`: actualiza los Pods a la nueva versión.

Puedes encontrar más parámetros en la documentación.

Cuando creamos un `Deployment`, se crea el `ReplicaSet` asociado y todos los pods que hayamos indicado.

    kubectl create -f nginx-deployment.yaml 
    deployment.extensions "nginx" created

    kubectl get pods
    NAME                     READY     STATUS    RESTARTS   AGE
    nginx-84f79f5cdd-b2bn5   1/1       Running   0          44s
    nginx-84f79f5cdd-cz474   1/1       Running   0          44s
    
    kubectl get deploy
    NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx     2         2         2            2           49s
    
    kubectl get rs
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-84f79f5cdd   2         2         2         53s
    
    kubectl get pods
    NAME                     READY     STATUS    RESTARTS   AGE
    nginx-84f79f5cdd-b2bn5   1/1       Running   0          56s
    nginx-84f79f5cdd-cz474   1/1       Running   0          56s


