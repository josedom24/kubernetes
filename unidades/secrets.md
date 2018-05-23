# Configurando nuestras aplicaciones: Secrets

Los [`Secrets`](https://kubernetes.io/docs/concepts/configuration/secret/) nos permiten guardar información sensible que será codificada. Por ejemplo,nos permite guarda contraseñas, clasves shh, ...

Al crear un `Secret` los valores se pueden indicar desde un directorio, un fichero o un literal.

    kubectl create secret generic mariadb --from-literal=password=root
    secret "mariadb" created
    
    kubectl get secret
    NAME                  TYPE                                  DATA      AGE
    ...
    mariadb               Opaque                                1         15s
    
    kubectl describe secret mariadb
    Name:         mariadb
    Namespace:    default
    Labels:       <none>
    Annotations:  <none>

    Type:  Opaque

    Data
    ====
    password:  4 bytes

Los `Secrets` no son seguro, no están encriptados.

    kubectl get secret mariadb -o yaml
    apiVersion: v1
    data:
      password: cm9vdA==
    kind: Secret
    metadata:
      creationTimestamp: 2018-05-23T18:22:27Z
      name: mariadb
      namespace: default
      resourceVersion: "162405"
      selfLink: /api/v1/namespaces/default/secrets/mariadb
      uid: 3fa5e1ad-5eb6-11e8-ab66-fa163e99cb75
    type: Opaque
    
    echo 'cm9vdA==' | base64 --decode
    root

Podemos definir un `Deployment` que defina un contenedor configurado por medio de variables de entorno, [`mariadb-deployment-secret.yaml`](https://github.com/josedom24/kubernetes/blob/master/ejemplos/mariadb/mariadb-deployment-secret.yaml):

    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: mariadb-deploy-secret
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
                    secretKeyRef:
                      name: mariadb
                      key: password

Creamos el despliegue y probamos el acceso:

    kubectl create -f mariadb-deployment-secret.yaml
    deployment.apps "mariadb-deploy-secret" created
    
    kubectl get pods -l app=mariadb
    NAME                                    READY     STATUS    RESTARTS   AGE
    mariadb-deploy-secret-f946dddfd-kkmlb   1/1       Running   0          15s
        
    kubectl exec -it mariadb-deploy-secret-f946dddfd-kkmlb -- mysql -u root -p
    Enter password: 
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 8
    Server version: 10.2.15-MariaDB-10.2.15+maria~jessie mariadb.org binary distribution

    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    MariaDB [(none)]> 