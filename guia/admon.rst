Administracion de un Nodo Cluster
=================================

Manejo de los Nodos.
++++++++++++++++++++

Listar los nodos.::

	# crm node list
	nodo1: normal
	nodo2: normal

Colocar un nodo en modo de mantenimiento.::

	# crm node maintenance nodo1

	# crm node list
	nodo1: normal
		standby=off maintenance=on
	nodo2: normal

	# crm node status
	<nodes>
	  <node id="nodo1" uname="nodo1">
		<instance_attributes id="nodes-nodo1">
		  <nvpair id="nodes-nodo1-maintenance" name="maintenance" value="on"/>
		</instance_attributes>
	  </node>
	  <node id="nodo2" uname="nodo2"/>
	</nodes>

Una vez colocado el nodo en modo mantenimiento para activarlo nuevamente hay que correr "crm node ready nodename".::

	# crm node ready nodo1

	# crm node status
	<nodes>
	  <node id="nodo1" uname="nodo1">
		<instance_attributes id="nodes-nodo1">
		  <nvpair id="nodes-nodo1-standby" name="standby" value="off"/>
		  <nvpair id="nodes-nodo1-maintenance" name="maintenance" value="off"/>
		</instance_attributes>
	  </node>
	  <node id="nodo2" uname="nodo2"/>
	</nodes>

Mostrar, Borrar configurar atributos del nodo.::

	# crm node status
	<nodes>
	  <node id="nodo1" uname="nodo1">
		<instance_attributes id="nodes-nodo1">
		  <nvpair id="nodes-nodo1-standby" name="standby" value="off"/>
		  <nvpair id="nodes-nodo1-maintenance" name="maintenance" value="off"/>
		</instance_attributes>
	  </node>
	  <node id="nodo
::

	# crm node attribute nodo1 delete maintenance
	Deleted nodes attribute: id=nodes-nodo1-maintenance name=maintenance

	# crm node attribute nodo1 delete standby
	Deleted nodes attribute: id=nodes-nodo1-standby name=standby

	# crm node status
	<nodes>
	  <node id="nodo1" uname="nodo1">
		<instance_attributes id="nodes-nodo1"/>
	  </node>
	  <node id="nodo2" uname="nodo2"/>
	</nodes>

Colocar en nodo en Standby.::

	# crm node standby nodo2

	# crm status
	Last updated: Tue Sep 13 20:45:58 2016		Last change: Tue Sep 13 20:45:55 2016 by root via crm_attribute on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo2 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 2 resources configured, 2 expected votes

	Node nodo2: standby
	Online: [ nodo1 ]

	Full list of resources:

	 ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo1
	 Apache	(ocf::heartbeat:apache):	Started nodo1

Retornar el nodo a Online.::

	# crm node online nodo2

	# crm status
	Last updated: Tue Sep 13 20:47:26 2016		Last change: Tue Sep 13 20:47:23 2016 by root via crm_attribute on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo2 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 2 resources configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:

	 ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo2
	 Apache	(ocf::heartbeat:apache):	Started nodo2

Manejo de los Recursos.
++++++++++++++++++++++++

Ver configuracion actual de los recursos del cluster.::

	# crm configure show
	node nodo1 \
		attributes
	node nodo2 \
		attributes standby=off
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
		stonith-enabled=false \
		no-quorum-policy=ignore \
		last-lrm-refresh=1473733528

Group Resources. nos permite iniciar, detener y administrar un solo resource.::

	# crm configure group HTTP-GROUP ClusterIP Apache
	INFO: modified location:cli-prefer-ClusterIP from ClusterIP to HTTP-GROUP
	INFO: modified location:cli-prefer-Apache from Apache to HTTP-GROUP

	# crm configure show
	node nodo1 \
		attributes
	node nodo2 \
		attributes standby=off
	primitive Apache apache \
		params configfile="/etc/httpd/conf/httpd.conf" \
		op monitor interval=30s \
		op start timeout=40s interval=0 \
		op stop timeout=60s interval=0
	primitive ClusterIP IPaddr2 \
		params ip=192.168.1.150 cidr_netmask=24 \
		op monitor interval=30s
	group HTTP-GROUP ClusterIP Apache
	location cli-prefer-Apache HTTP-GROUP role=Started inf: nodo2
	location cli-prefer-ClusterIP HTTP-GROUP role=Started inf: nodo2
	property cib-bootstrap-options: \
		have-watchdog=false \
		dc-version=1.1.14-8.el6_8.1-70404b0 \
		cluster-infrastructure="classic openais (with plugin)" \
		expected-quorum-votes=2 \
		stonith-enabled=false \
		no-quorum-policy=ignore \
		last-lrm-refresh=1473733528

	# crm status
	Last updated: Tue Sep 13 20:52:45 2016		Last change: Tue Sep 13 20:52:29 2016 by root via cibadmin on nodo2
	Stack: classic openais (with plugin)
	Current DC: nodo2 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 2 resources configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:

	 Resource Group: HTTP-GROUP
		 ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo2
		 Apache	(ocf::heartbeat:apache):	Started nodo2

Order Start/Stop un Resources. En este ejemplo se configura para inicia y detener ClusterIP y Apache en un orden. ClusterIP se iniciara primero y luego de estar iniciado sera cuando se inicie Apache, cuando se detenga Apache sera primero y luego ClusterIP.::

	# crm configure order ClusterIP-before-Apache inf: ClusterIP Apache

	# crm configure show
	node nodo1 \
		attributes
	node nodo2 \
		attributes standby=off
	primitive Apache apache \
		params configfile="/etc/httpd/conf/httpd.conf" \
		op monitor interval=30s \
		op start timeout=40s interval=0 \
		op stop timeout=60s interval=0
	primitive ClusterIP IPaddr2 \
		params ip=192.168.1.150 cidr_netmask=24 \
		op monitor interval=30s
	group HTTP-GROUP ClusterIP Apache
	order ClusterIP-before-Apache inf: ClusterIP Apache
	location cli-prefer-Apache HTTP-GROUP role=Started inf: nodo2
	location cli-prefer-ClusterIP HTTP-GROUP role=Started inf: nodo2
	property cib-bootstrap-options: \
		have-watchdog=false \
		dc-version=1.1.14-8.el6_8.1-70404b0 \
		cluster-infrastructure="classic openais (with plugin)" \
		expected-quorum-votes=2 \
		stonith-enabled=false \
		no-quorum-policy=ignore \
		last-lrm-refresh=1473733528

Colocation Resources. Configurar o forzar los recursos esten en un nodo al mismo tiempo. Aunque en este caso es mejor configurar el grupo de Resource.::

	# crm configure colocation IP-with-Apache inf: ClusterIP Apache
	WARNING: IP-with-Apache: resource ClusterIP is grouped, constraints should apply to the group
	WARNING: IP-with-Apache: resource Apache is grouped, constraints should apply to the group

	# crm configure show
	node nodo1 \
		attributes
	node nodo2 \
		attributes standby=off
	primitive Apache apache \
		params configfile="/etc/httpd/conf/httpd.conf" \
		op monitor interval=30s \
		op start timeout=40s interval=0 \
		op stop timeout=60s interval=0
	primitive ClusterIP IPaddr2 \
		params ip=192.168.1.150 cidr_netmask=24 \
		op monitor interval=30s
	group HTTP-GROUP ClusterIP Apache
	order ClusterIP-before-Apache inf: ClusterIP Apache
	colocation IP-with-Apache inf: ClusterIP Apache
	location cli-prefer-Apache HTTP-GROUP role=Started inf: nodo2
	location cli-prefer-ClusterIP HTTP-GROUP role=Started inf: nodo2
	property cib-bootstrap-options: \
		have-watchdog=false \
		dc-version=1.1.14-8.el6_8.1-70404b0 \
		cluster-infrastructure="classic openais (with plugin)" \
		expected-quorum-votes=2 \
		stonith-enabled=false \
		no-quorum-policy=ignore \
		last-lrm-refresh=1473733528

Resource Prefered Location. Podemos configurar la localidad preferida de un resource o grupo segun el score de la localidad. Mas positivo un nodo los recursos correran ahi, menos negativo indica que el resource no correra en ese nodo.::

	# crm configure location HTTP-GROUP-prefer-NODO1 HTTP-GROUP 50: nodo1
	
	# crm configure show
	node nodo1 \
		attributes
	node nodo2 \
		attributes standby=off
	primitive Apache apache \
		params configfile="/etc/httpd/conf/httpd.conf" \
		op monitor interval=30s \
		op start timeout=40s interval=0 \
		op stop timeout=60s interval=0
	primitive ClusterIP IPaddr2 \
		params ip=192.168.1.150 cidr_netmask=24 \
		op monitor interval=30s
	group HTTP-GROUP ClusterIP Apache
	order ClusterIP-before-Apache inf: ClusterIP Apache
	location HTTP-GROUP-prefer-NODO1 HTTP-GROUP 50: nodo1
	colocation IP-with-Apache inf: ClusterIP Apache
	location cli-prefer-Apache HTTP-GROUP role=Started inf: nodo2
	location cli-prefer-ClusterIP HTTP-GROUP role=Started inf: nodo2
	property cib-bootstrap-options: \
		have-watchdog=false \
		dc-version=1.1.14-8.el6_8.1-70404b0 \
		cluster-infrastructure="classic openais (with plugin)" \
		expected-quorum-votes=2 \
		stonith-enabled=false \
		no-quorum-policy=ignore \
		last-lrm-refresh=1473733528

Verificar el status de los Nodes y Resources.::

	# crm status
	Last updated: Tue Sep 13 21:12:27 2016		Last change: Tue Sep 13 21:10:37 2016 by root via cibadmin on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo2 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 2 resources configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:

	 Resource Group: HTTP-GROUP
		 ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo2
		 Apache	(ocf::heartbeat:apache):	Started nodo2

Migrar un Resources o Grupo.::

	# crm resource migrate HTTP-GROUP nodo2
	INFO: Move constraint created for HTTP-GROUP to nodo2

	# crm status
	Last updated: Tue Sep 13 21:24:05 2016		Last change: Tue Sep 13 21:24:00 2016 by root via crm_resource on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo2 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 2 resources configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:

	 Resource Group: HTTP-GROUP
		 ClusterIP	(ocf::heartbeat:IPaddr2):	Started nodo2
		 Apache	(ocf::heartbeat:apache):	Started nodo2
