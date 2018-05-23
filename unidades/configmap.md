# Configurando nuestras aplicaciones: ConfigMaps

[`ConfigMap`](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) te permite definir un diccionario (clave,valor) para guardar información que puedes utilizar para configurar una aplicación.

Al crear un `ConfigMap` los valores se pueden indicar desde un directorio, un fichero o un literal.

    kubectl create cm mariadb --from-literal=root_password=my-password \
                              --from-literal=mysql_usuario=usuario     \
                              --from-literal=mysql_password=password-user \
                              --from-literal=basededatos=test
    configmap "mariadb" created
    
    kubectl get cm
    NAME      DATA      AGE
    mariadb   4         15s
    
    kubectl describe cm mariadb
    Name:         mariadb
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Data
    ====
    mysql_usuario:
    ----
    usuario
    root_password:
    ----
    my-password
    basededatos:
    ----
    test
    mysql_password:
    ----
    password-user
    Events:  <none>

Ahora podemos configurar el fichero yaml que define el despliegue, [`mariadb-deployment-configmap.yaml`](https://github.com/josedom24/kubernetes/blob/master/ejemplos/mariadb/mariadb-deployment-configmap.yaml):

    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: mariadb-deploy-cm
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
                  valueFrom:
                    configMapKeyRef:
                      name: mariadb
                      key: root_password
                - name: MYSQL_USER
                  valueFrom:
                    configMapKeyRef:
                      name: mariadb
                      key: mysql_usuario
                - name: MYSQL_PASSWORD
                  valueFrom:
                    configMapKeyRef:
                      name: mariadb
                      key: mysql_password
                - name: MYSQL_DATABASE
                  valueFrom:
                    configMapKeyRef:
                      name: mariadb
                      key: basededatos

Creamos el despliegue y probamos el acceso:

    kubectl create -f mariadb-deployment-configmap.yaml
    deployment.apps "mariadb-deploy-cm" created
    
    kubectl get pods -l app=mariadb
    NAME                                 READY     STATUS    RESTARTS   AGE
    mariadb-deploy-cm-57f7b9c7d7-ll6pv   1/1       Running   0          15s
    
    kubectl exec -it mariadb-deploy-cm-57f7b9c7d7-ll6pv -- mysql -u usuario -p
    Enter password: 
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 8
    Server version: 10.2.15-MariaDB-10.2.15+maria~jessie mariadb.org binary distribution

    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   
    MariaDB [(none)]> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | test               |
    +--------------------+
    2 rows in set (0.00 sec)


