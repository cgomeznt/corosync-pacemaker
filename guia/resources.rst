Agregar y Borrar Recursos del Cluster (Cluster Resources)
===========================================================

Al tener instalado los crmsh podemos.::

	# crm help
	Help overview for crmsh
	Available topics:

		    Overview       Help overview for crmsh

		    Topics         Available topics

		    Description    Program description
		    CommandLine    Command line options
		    Introduction   Introduction
		    Interface      User interface
		    Completion     Tab completion
		    Shorthand      Shorthand syntax
		    Features       Features
		    Shadows        Shadow CIB usage
		    Checks         Configuration semantic checks
		    Templates      Configuration templates
		    Testing        Resource testing
		    Security       Access Control Lists (ACL)
		    Resourcesets   Syntax: Resource sets
		    AttributeListReferences Syntax: Attribute list references
		    AttributeReferences Syntax: Attribute references
		    RuleExpressions Syntax: Rule expressions
		    Reference      Command reference

Ver status del Cluster.::

	# crm status
	Last updated: Mon Sep 12 20:55:52 2016		Last change: Mon Sep 12 20:48:20 2016 by hacluster via crmd on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo1 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 0 resources configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:

Ver la configuracion del cluster.::

	# crm configure show
	node nodo1
	node nodo2
	property cib-bootstrap-options: \
		have-watchdog=false \
		dc-version=1.1.14-8.el6_8.1-70404b0 \
		cluster-infrastructure="classic openais (with plugin)" \
		expected-quorum-votes=2

Agregar Cluster Resources. Cada recurso de clúster está definido por un agente de recursos. Agentes de recursos deben proporcionar Cluster Linux con un estado de recursos completa y disponibilidad en cualquier momento ! Las clases de agente de recursos más importantes y más utilizados son :
* LSB (Linux Standard Base) - Estos son los agentes de recursos de clúster comunes que se encuentran en el directorio /etc/init.d (scripts init).
* OCF (Open Cluster Framework) - En realidad extendieron LSB agentes de recursos de clúster y por lo general apoyan parámetros adicionales.

De esto podemos suponer que siempre es mejor utilizar OCF (si está disponible) sobre LSB agentes de recursos de apoyo desde OCF parámetros de configuración adicionales y están optimizados para los recursos de clúster

Ver los Agents Resources.::

	# crm ra list lsb
	auditd                   blk-availability         bmc-snmp-proxy           cman                     corosync
	corosync-notifyd         crond                    exchange-bmc-os-info     haldaemon                halt
	ip6tables                ipmievd                  iptables                 iscsi                    iscsid
	killall                  libvirt-guests           lvm2-lvmetad             lvm2-monitor             mdmonitor
	messagebus               modclusterd              multipathd               netconsole               netfs
	network                  nfs                      nfs-rdma                 nfslock                  oddjobd
	pacemaker                postfix                  quota_nld                rdisc                    rdma
	restorecond              ricci                    rpcbind                  rpcgssd                  rpcidmapd
	rpcsvcgssd               rsyslog                  sandbox                  saslauthd                single
	sshd                     udev-post                winbind  

::

	# crm ra list ocf
	CTDB                ClusterMon          Delay               Dummy               Filesystem          HealthCPU
	HealthSMART         IPaddr              IPaddr2             IPsrcaddr           LVM                 MailTo
	Route               SendArp             Squid               Stateful            SysInfo             SystemHealth
	VirtualDomain       Xinetd              apache              conntrackd          controld            db2
	dhcpd               ethmonitor          exportfs            iSCSILogicalUnit    mysql               named
	nfsnotify           nfsserver           nginx               oracle              oralsnr             pgsql
	ping                pingd               portblock           postfix             remote              rsyncd
	symlink             tomcat    


Para ver mas detalle de un agente.::

	# crm ra meta IPaddr2


Antes de empezar la adición de recursos a nuestro Cluster tenemos que desactivar STONITH (Shoot The Other Node In The Head) - ya que no estamos utilizando en nuestra configuración.::

	# crm configure property stonith-enabled=false

Tambien quitamos el quorum.::

	# crm configure property no-quorum-policy=ignore

Verificamos que se haya apagado.::

	# crm configure show
	node nodo1
	node nodo2
	property cib-bootstrap-options: \
		have-watchdog=false \
		dc-version=1.1.14-8.el6_8.1-70404b0 \
		cluster-infrastructure="classic openais (with plugin)" \
		expected-quorum-votes=2 \
		stonith-enabled=false
		no-quorum-policy=ignore \
		last-lrm-refresh=1473733528

Agregamos un IP Resources.::
La informacion que necesitamos configurar es:
Cluster Resource Name: ClusterIP
Resource Agent: ocf:heartbeat:IPaddr2 (se obtiene con “crm ra meta IPaddr2”)
IP address: 192.168.1.100
Netmask: 24
Monitor interval: 30 seconds (se obtiene con  “crm ra meta IPaddr2”)

Corremos el siguiente comando.::

	# crm configure primitive ClusterIP ocf:heartbeat:IPaddr2 params ip=192.168.1.150 cidr_netmask="24" op monitor interval="30s"

Verificamos.::

	# crm configure show
	node nodo1
	node nodo2
	primitive ClusterIP IPaddr2 \
		params ip=192.168.1.150 cidr_netmask=24 \
		op monitor interval=30s
	property cib-bootstrap-options: \
		have-watchdog=false \
		dc-version=1.1.14-8.el6_8.1-70404b0 \
		cluster-infrastructure="classic openais (with plugin)" \
		expected-quorum-votes=2 \
		stonith-enabled=false


Vemos el estatus.::

	# crm status
	Last updated: Mon Sep 12 21:16:19 2016		Last change: Mon Sep 12 21:15:21 2016 by root via cibadmin on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo1 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 1 resource configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:

	 ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo1


Como se puede ver el Agente esta configurada en el nodo1.::

	ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo1

Lo podemos migrar para el otro nodo y verificamos.::

	# crm resource migrate ClusterIP nodo2
	INFO: Move constraint created for ClusterIP to nodo2

	# crm status
	Last updated: Tue Sep 13 20:14:03 2016		Last change: Tue Sep 13 20:13:59 2016 by root via crm_resource on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo2 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 1 resource configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:

	 ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo2

Agregamos el Httpd Resource. Siguiente recursos es un servidor Web Apache . Antes de la configuración de recursos de clúster de Apache, el paquete httpd debe estar instalado y configurado en ambos nodos ! La información que necesita para configurar el servidor Web Apache es :

Cluster Resource Name: Apache
Resource Agent: ocf:heartbeat:apache (get this info with “crm ra meta apache”)
Configuration file location: /etc/httpd/conf/httpd.conf
Monitor interval: 30 seconds (se obtiene con “crm ra meta apache”)
Start timeout: 40 seconds (se obtiene con “crm ra meta apache”)
Stop timeout: 60 seconds (se obtiene con “crm ra meta apache”)

::

	# crm configure primitive Apache ocf:heartbeat:apache params configfile=/etc/httpd/conf/httpd.conf op monitor interval="30s" op start timeout="40s" op stop timeout="60s"

Verificamos.::

	# crm configure show
	node nodo1
	node nodo2
	primitive Apache apache \
		params configfile="/etc/httpd/conf/httpd.conf" \
		op monitor interval=30s \
		op start timeout=40s interval=0 \
		op stop timeout=60s interval=0
	primitive ClusterIP IPaddr2 \
		params ip=192.168.1.150 cidr_netmask=24 \
		op monitor interval=30s
	property cib-bootstrap-options: \
		have-watchdog=false \
		dc-version=1.1.14-8.el6_8.1-70404b0 \
		cluster-infrastructure="classic openais (with plugin)" \
		expected-quorum-votes=2 \
		stonith-enabled=false

Vemos el estatus y vemos como lo tomo el otro nodo.::

	# crm status
	Last updated: Mon Sep 12 21:23:09 2016		Last change: Mon Sep 12 21:22:27 2016 by root via cibadmin on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo1 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 2 resources configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:

	 ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo1
	 Apache	(ocf::heartbeat:apache):	Started nodo2


Apache y ClusterIP en este momento se ejecuta en diferentes nodos del clúster , pero vamos a arreglar esto más tarde, el establecimiento de restricciones de recursos como : colocación (colocating resources), orden (order in which resources start and stop)

Borrar Cluster Resources
Delete una configuracion de Cluster Resources con “crm configure delete” comando seguido de un Resource Name que se quiera borrar ejemplo.::

	crm configure delete resourcename

Vamos a detener el recurso de apache y luego borrarlo.::

	# crm resource stop Apache

	# crm status
	Last updated: Mon Sep 12 21:28:04 2016		Last change: Mon Sep 12 21:28:02 2016 by root via cibadmin on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo1 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 2 resources configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:

	 ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo1
	 Apache	(ocf::heartbeat:apache):	(target-role:Stopped) Started nodo2

::

	# crm configure delete Apache

	# crm status
	Last updated: Mon Sep 12 21:28:33 2016		Last change: Mon Sep 12 21:28:31 2016 by root via cibadmin on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo1 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 1 resource configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:

	 ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo1

::

	# crm configure show
	node nodo1
	node nodo2
	primitive ClusterIP IPaddr2 \
		params ip=192.168.1.150 cidr_netmask=24 \
		op monitor interval=30s
	property cib-bootstrap-options: \
		have-watchdog=false \
		dc-version=1.1.14-8.el6_8.1-70404b0 \
		cluster-infrastructure="classic openais (with plugin)" \
		expected-quorum-votes=2 \
		stonith-enabled=false
