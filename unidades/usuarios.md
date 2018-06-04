# Introducción a la autentificación de usuarios en Kubernetes

Hasta ahora hemos accedido a nuestro cluster de Kubernetes con el usaurio `admon` usando el fichero de credenciales `~/.kube/mycluster.conf`:

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

En un cluster real vamos a tener diferentes usuarios, grupos y privilegios. En el caso de los usuarios no existe un objeto en la API que nos permita gestionarlos. La creación de usuarios la debe gestionar el administrador del cluster y tenemos varias formas de autentificación: 

* Autentificación basada en certificadas
* Autetificación basada en tokens
* Autentficación básica HTTP
* OAuth2

Vamos a ver como crear nuevos usuarios y autentficarlos por medio de certificados.

## Auntentificación basada en certificados

Cuando instalamos Kubernetes se crea una Autoridad Certificadora (CA):

* `/etc/kubernetes/pki/ca.crt`: Clave pública del CA.
* `/etc/kubernetes/pki/ca.key`: Clave privada del CA.

Todos los certificados firmados por dicha CA van a ser aceptados por el servidor Kubernetes API. Por lo tanto vamos a crear un certificado con OpenSSL que posteriormente el administrador del cluster firmará con la CA. En el certificado que vamos a crear hay que tener en cuenta dos campos:

* Common Name (CN): Kuberntes interpretará el valor de este campo como el **nombre del usuario*.
* Organization (O): Kuberntes interpretará el valor de este campo como el **grupo del usuario*.