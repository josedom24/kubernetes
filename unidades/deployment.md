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
              containerPort: 80

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

Como ocurría con los `replicaSets` los `Deployment` también se pueden escalar, aumentando o disminuyendo el número de pods asociados:

    kubectl scale deployment nginx --replicas=4
    deployment.extensions "nginx" scaled
    
    kubectl get rs
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-84f79f5cdd   4         4         4         2m
    
    kubectl get pods
    NAME                     READY     STATUS    RESTARTS   AGE
    nginx-84f79f5cdd-7kzls   1/1       Running   0          2m
    nginx-84f79f5cdd-8zrcb   1/1       Running   0          2m
    nginx-84f79f5cdd-b2bn5   1/1       Running   0          2m
    nginx-84f79f5cdd-cz474   1/1       Running   0          2m

## Actualización del deployment

Podemos cambiar en cualquier momentos las propiedades del `deployment`, por ejemplo para hacer una actualización de la aplicación:

    kubectl set image deployment nginx nginx=nginx:1.13 --all
    deployment.apps "nginx" image updated

También podríamos haber cambiado la versión de la imagen modificando la definición Yaml del `Deployment` con la siguiente instrucción:

    kubectl edit deployment nginx

Y comprobamos que se ha creado un nuevo `RecordSet`, y unos nuevos pods con la nueva versión de la imagen.

    kubectl get rs
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-75f9c7b459   4         4         3         5s
    nginx-84f79f5cdd   0         0         0         6m
    
    kubectl get pods
    NAME                     READY     STATUS    RESTARTS   AGE
    nginx-75f9c7b459-87l5j   1/1       Running   0          5s
    nginx-75f9c7b459-92x8s   1/1       Running   0          5s
    nginx-75f9c7b459-fm6g5   1/1       Running   0          5s
    nginx-75f9c7b459-pz78j   1/1       Running   0          5s

La opción `--all` fuerza a actualizar todos los pods aunque no estén inicializados. 

## Rollback de nuestra aplicación

Si queremos volver a la versión anterior de nuestro despliegue, tenemos que ejecutar:

    kubectl rollout undo deployment/nginx
    deployment.apps "nginx" 

T comprobamos como se activa el antiguo `RecordSet` y se crean nuevos pods con la versión anterior de nuestra aplicación:
    
    kubectl get rs
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-75f9c7b459   0         0         0         1h
    nginx-84f79f5cdd   4         4         4         22h
  
    kubectl get pods
    NAME                     READY     STATUS    RESTARTS   AGE
    nginx-84f79f5cdd-cwgzx   1/1       Running   0          5s
    nginx-84f79f5cdd-hmd2l   1/1       Running   0          5s
    nginx-84f79f5cdd-rhnkg   1/1       Running   0          5s
    nginx-84f79f5cdd-vn7kd   1/1       Running   0          5s

## Eliminando el despliegue

Si eliminamos el `Deployment` se eliminarán el `RecordSet` asociado y los pods que se estaban gestionando.

    kubectl delete deployment nginx
    deployment.extensions "nginx" deleted
    
    kubectl get deploy
    No resources found.
    
    kubectl get rs
    No resources found.
    
    kubectl get pods
    No resources found.


