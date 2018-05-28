# Almacenamiento en Kubernetes

Como hemos comentado anteriormente los pods son efímero, la información guardada en ellos no es persistente, pero es evidentemente que necesitamos que nuestras aplicaciones tengan la posibilidad de que su información no se pierda. La solución es añadir [volúmenes](https://kubernetes.io/docs/concepts/storage/volumes/) (almacenamiento persistente) a los pods para que lo puedan utilizar los contenedores. Los volúmenes son considerados otro recurso de Kubernete.

## Definiendo volúmenes en un pod

En la definición de un pod, además de especificar los contenedores que lo van a formar, también podemos indicar los volúmenes que tendrá. Además la definición de cada contenedor tendremos que indicar los puntos de montajes de los diferentes volúmenes. Por ejemplo, el el fichero [`pod-nginx.yaml`](https://github.com/josedom24/kubernetes/blob/master/ejemplos/volumen/pod-nginx.yaml) podemos ver la definición de tres tipos de volúmenes en un pod:

    apiVersion: v1
    kind: Pod
    metadata:
      name: www
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - mountPath: /home
          name: home
        - mountPath: /git
          name: git
          readOnly: true
        - mountPath: /temp
          name: temp
      volumes:
      - name: home
        hostPath:
          path: /home/debian
      - name: git
        gitRepo:
          repository: https://github.com/josedom24/kubernetes.git
      - name: temp
        emptyDir: {}

En la sección `volumes` definimos los volúmenes disponibles y en la definción del contenedor, con la etiqueta `volumeMounts`, indicamos los puntos de montajes.

Hemos definido tres volúmenes de diferente tipo:

* `hostPath`: Este volumen corresponde a un directorio o fichero del nodo donde se crea el pod. Como vemos se monta en el directorio `/home` del contenedor. En cluster multinodos este tipo de volúmenes no son efectivos, ya que no tenemos duplicada la información en los distintos nodos y su contenido puede depender del nodo donde se cree el pod.
* `gitRepo`: El contenido del volumen corresponde a un repositorio de github, lo vamos a montar en el el directorio `/git` y como vemos lo hemos configurado de sólo lectura.
* `emptyDir`: El contenido de este volumen, que hemos montado en el directorio `/temp` se borrará al eliminar el pod. Lo utilizamos para compartir información entre los contenedores de un mismo pod.

En la documentación de Kubernetes puedes encontrar todos los [tipos de volúmenes que podemos utilizar](https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes).

Veamos los volúmenes que hemos creado:

    kubectl create -f pod-nginx.yaml 
    pod "www" created

    kubectl describe pod www
    ...
    Volumes:
      home:
        Type:          HostPath (bare host directory volume)
        Path:          /home/debian
        HostPathType:  
      git:
        Type:        GitRepo (a volume that is pulled from git when the pod is created)
        Repository:  https://github.com/josedom24/kubernetes.git
        Revision:    
      temp:
        Type:    EmptyDir (a temporary directory that shares a pod's lifetime)
        Medium:  

Accedemos al pod y vemos los contenidos de cada directorio:

    kubectl exec -it www -- bash
    
    root@www:/# cd /home
    root@www:/home# ls
    fichero.txt
    root@www:/home# cd /git
    root@www:/git# ls
    kubernetes
    root@www:/git# cd /temp
    root@www:/temp# ls
    root@www:/temp# ^C
    root@www:/temp# 

* El directorio `/home` tiene un fichero que esta en el directorio `/home/debian` del nodo donde se ha ejecutado el pod.
* El directorio `/git` tiene el contenido del repositorio github que hemos indicado.
* El directorio `/temp` corresponde a un directorio vacío que podemos utilizar para compartir información entre los contenedores del pod,la información de este directorio se perderá al eliminarlo.

## Compartiendo información en un pod

Veamos con un ejemplo la posibilidad de compartir información entre contenedores de un pod. En el fichero [`pod2-nginx.yaml`](https://github.com/josedom24/kubernetes/blob/master/ejemplos/volumen/pod2-nginx.yaml) creamos un pod con dos contenedores y un volumen:

    apiVersion: v1
    kind: Pod
    metadata:
      name: two-containers
    spec:

      volumes:
      - name: shared-data
        emptyDir: {}

      containers:
      - name: nginx-container
        image: nginx
        volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html

      - name: busybox-container
        image: busybox
        command:
          - sleep
          - "3600"
        volumeMounts:
        - name: shared-data
          mountPath: /pod-data

Vamos a crear el pod, vamos acceder al contenedor `busybox-cotainer` y vamos a escribir un `index.html`, que al estar compartido por el contenedor `nginx-container` podremos ver ala acceder a él.

    kubectl create -f pod2-nginx.yaml 
    pod "two-containers" created
        
    kubectl get pods 
    NAME             READY     STATUS    RESTARTS   AGE
    two-containers   2/2       Running   0          10s
    www              1/1       Running   0          1h

    kubectl exec -it two-containers -c busybox-container -- sh
    / # cd /pod-data/
    /pod-data # echo "Prueba de compartir información entre contenedores">index.html
    /pod-data # exit

    kubectl port-forward two-containers 8080:80
    Forwarding from 127.0.0.1:8080 -> 80
    Forwarding from [::1]:8080 -> 80
    Handling connection for 8080

![nginx](img/compartir.png)
