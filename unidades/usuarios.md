# Introducción a la autentificación de usuarios en Kubernetes

Hasta ahora hemos accedido a nuestro cluster de Kubernetes con el usuario `admin` usando el fichero de credenciales `~/.kube/mycluster.conf`:

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

* Common Name (CN): Kubernetes interpretará el valor de este campo como el **nombre del usuario**.
* Organization (O): Kubernetes interpretará el valor de este campo como el **grupo del usuario**.

## Creación de usuarios

Vamos a crear un usuario que tenga de nombre **usuario1** y pertenezca al grupo **desarrollo**. Este usuario va a trabajar en el namespace **proyecto1**, que es lo primero que vamos a crear:

    kubectl create ns proyecto1

### Pasos que debe dar el usuario

1. El usuario debe crearse una clave privada, la va a guardar en el directorio `~/.certs`:

        cd ~/.certs
        openssl genrsa -out usuario1.key 2048

2. Va a crear una petición de certificado (CSR):

        openssl req -new -key usuario1.key -out usuario1.csr -subj "/CN=usuario1/O=desarrollo"

3. Va a mandar la el fichero CSR al administrador del cluster para que lo firme.

### Pasos que debe dar el administrados del cluster

1. Crea el certificado a partir del CSR firmándolo con el CA:

        openssl x509 -req -in usuario1.csr \
        -CA /etc/kuberenetes/ca.crt \
        -CAkey /etc/kuberenetes//ca.key \
        -CAcreateserial -out usuario1.crt -days 500

2. Devuelve el certificado (fichero .crt) al usuario (que lo guardará en `~/.certs`).

##### Creación del fichero de configuración

Si el usuario esta trabajando con el fichero de configuración por defecto de kubernetes: `~./kube/config`:

1. Añadimos las credenciales del nuevo usuario:

        kubectl config set-credentials usuario1 \
        --client-certificate=/home/debian/.certs/usuario1.crt  \
        --client-key=/home/debian/.certs/usuario1.key        

2. Añadimos el nuevo contexto:

        kubectl config set-context usuario1-context --cluster=kubernetes \
        --namespace=proyecto1 --user=usuario1

Si el usuario quiere guardar las credenciales en otro fichero:
    
    export KUBECONFIG=~/.kube/access_usuario1.conf 

1. Añadimos el nuevo cluster, indicando la ip del nodo master del cluster:

        kubectl config set-cluster mi_cluster \
        --certificate-authority=/home/debian/.minikube/ca.crt \
        --embed-certs=true --server=https://<<IP DEL MASTER>>:6443

2. Añadimos las credenciales del nuevo usuario y el nuevo contexto como hemos visto anteriormente.

## Acceso al cluster con el nuevo usuario

Suponemos que seguimos usando nuestro fichero de configuración `~/.kube/config`:

    export KUBECONFIG=~/.kube/config

Mostramos los contextos definidos en el fichero de configuración:

    kubectl config get-contexts
    CURRENT   NAME                          CLUSTER      AUTHINFO         NAMESPACE
              usuario1-context              minikube     usuario1         proyecto1
    *         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   

Elegimos el nuevo contexto:

    kubectl config use-context usuario1-context

    kubectl get pods
    Error from server (Forbidden): pods is forbidden: User "usuario1" cannot list resource "pods" in API group "" in the namespace "proyecto1"

Parece que el `usuario1` no tiene autorización en el namespace `proyecto1` para visualizar la lista de pods. Necesitamos gestionar las autorización con RBAC, pero ese tema lo veremos en la siguiente entrada del blog.

