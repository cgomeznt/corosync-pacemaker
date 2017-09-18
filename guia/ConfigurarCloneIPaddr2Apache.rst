Configurar Clone IPaddr2 y  Apache Resource
===========================================


Antes de configurar el Clone debe tener la guia de Resource Clones.

https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Configuring_the_Red_Hat_High_Availability_Add-On_with_Pacemaker/ch-advancedresource-HAAR.html


clone-node-max=1 Siempre debe estar en uno por si existe una falla siempre este balanceado el cluster.

# pcs resource clone WebSite globally-unique=true clone-max=2 clone-node-max=1

# pcs resource clone ClusterIP globally-unique=true clone-max=2 clone-node-max=1

# pcs resource unclone ClusterIP-clone

# pcs resource restart WebSite-clone
