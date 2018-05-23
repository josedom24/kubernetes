# Configurando nuestras aplicaciones con variables de entorno

Para configurar las aplicaciones que vamos a desplegar usamos variables de entorno, por ejemplo podemos ver las variables de entorno que podemos definir para configurar la imagen docker de [MariaDB](https://hub.docker.com/_/mariadb/).

Podemos definir un `Deployment` que defina un contenedor configurado por medio de variables de entorno, [`mariadb-deployment.yaml`](https://github.com/josedom24/kubernetes/blob/master/ejemplos/mariadb/mariadb-deployment.yaml):

    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: mariadb-deployment
      labels:
        app: mariadb
        type: database
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: mariadb
            type: database
        spec:
          containers:
            - name: mariadb
              image: mariadb
              ports:
                - containerPort: 3306
                  name: db-port
              env:
                - name: MYSQL_ROOT_PASSWORD
                  value: my-password

Y creamos el despliegue:

    kubectl create -f mariadb-deployment.yaml
    deployment.apps "mariadb-deployment" created

O directamente ejecutando:

    kubectl run mariadb --image=mariadb --env MYSQL_ROOT_PASSWORD=my-password

Veamos el pod creado:

    kubectl get pods -l app=mariadb
    NAME                                READY     STATUS    RESTARTS   AGE
    mariadb-deployment-fc75f956-f5zlt   1/1       Running   0          15s

Y probamos si podemos acceder, introduciendo la contraseña configurada:

    kubectl exec -it mariadb-deployment-fc75f956-f5zlt -- mysql -u root -p
    Enter password: 
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 8
    Server version: 10.2.15-MariaDB-10.2.15+maria~jessie mariadb.org binary distribution

    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    MariaDB [(none)]> 

