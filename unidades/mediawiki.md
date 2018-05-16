# Ejemplo: Desplegando la aplicación Mediawiki

[`mediawiki`](https://www.mediawiki.org/wiki/MediaWiki) es una aplicación escrita en PHP que nos permite gestionar una wiki. Vamos a hacer un despliegue en nuestro cluster de kubernetes.

En este ejemplo vamos a crear el `Deployment` sin utilizar su definición en Yaml, con la siguiente instrucción:

    kubectl run mediawiki --image=mediawiki --record
    deployment.apps "mediawiki" created
    
    kubectl get deploy
    NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    mediawiki   1         1         1            0           5s
    
    kubectl get rs
    NAME                   DESIRED   CURRENT   READY     AGE
    mediawiki-85bf9d6bd7   1         1         0         5s

    kubectl get pods
    NAME                         READY     STATUS    RESTARTS   AGE
    mediawiki-85bf9d6bd7-97bhg   1/1       Running   0          25s

Utilizando la instrucción `kubectl run` se crea un `Deployment` con un `RecordSet` asociado que arranca un pod. La opción `--record` nos va a permitir gaurdar las actualizaciones del `Deployment` como [`annotations`](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/) en la definición Yaml del mismo. Por lo tanto si vemos la definción del `Deployment`:

    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
        deployment.kubernetes.io/revision: "1"
        kubernetes.io/change-cause: kubectl run mediawiki --image=mediawiki     --record=true
    ...

