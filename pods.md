# Recursos de Kubernetes: pod

La unidad más pequeña de kuberentes son los **pod**, con los que podemos correr contenedores. Un **pod** represnta un conjunto de contenedores que comparten almacenamiento y una única IP. **Los pod son efímeros**, cuando se destruyen se pierde toda la información que contenía. Si queremos desarrollar aplicaciones persistentes tenemos que utilizar volúmenes.

|[pod](img/pod.png)