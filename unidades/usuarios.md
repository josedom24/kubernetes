# Introducción a la autentificación de usuarios en Kubernetes

Hasta ahora hemos accedido a nuestro cluster de Kubernetes usando el fichero de credenciales `~/.kube/mycluster.conf`:

    export KUBECONFIG=~/.kube/mycluster.conf 

En realidad el fichero por defecto de configuración es `~/.kube/config`, por lo que podemos guardar nuestro fichero de credenciales en este fichero:

    cp mycluster.conf config

Ya no hace falta exportar las credenciales, podemos obtener el cluster con el que estamos trabajando:

    kubectl config get-clusters

    NAME
    kubernetes

Y podemos obtener el contexto (un contexto determina el cluter, el usuario y el espacio de nombres que podemos utilizar):

    kubectl config get-contexts
    CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
    *         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   
    
