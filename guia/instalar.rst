Instalar Corosync - Pacemake en CentOS
=======================================

::

	# yum install corosync pacemaker crmsh

Cada nodo deberá compartir la misma clave de autenticación para poder formar parte del clúster. Es una medida de seguridad básica. De no ser así, descubierto el servicio de clúster no sería demasiado difícil comenzar a añadir nodos al clúster con fines maliciosos.

Creamos este repo para instalar los script de admon de cluster.::

	# vi /etc/yum.repos.d/ha-clustering.repo

	[haclustering]
	name=HA Clustering
	baseurl=http://download.opensuse.org/repositories/network:/ha-clustering:/Stable/CentOS_CentOS-6/
	enabled=1
	gpgcheck=0

	# yum --enablerepo=haclustering install crmsh

En el nodo 1 creamos la clave, pero como el pide generar entropia y se vulve lento y molesto hacemos::

	cat /dev/urandom >/dev/null &
	# corosync-keygen 

Recuerda luego matar el procedo del cat en background

Lo copiamos al nodo 2::

	# scp /etc/corosync/authkey 192.168.1.105:/etc/corosync/

En ambos nodos, le damos permisos a la clave (no iniciará si no le damos estos permisos bastante estrictos)::

	# chmod 400 /etc/corosync/authkey

En ambos nodos creamos una copia del archivo de configuracion de corosync.::

	# cp -dp corosync.conf.example corosync.conf

En ambos nodos especificamos la red en la que se va a escuchar el heartbeat de Pacemaker. /etc/corosync/corosync.conf:

	# vi corosync.conf
	compatibility: whitetank

	aisexec {
		# Run as root - this is necessary to be able to manage resources with Pacemaker
		user: root
		group: root
	}

	service {
		# Load the Pacemaker Cluster Resource Manager
		ver: 1
		name: pacemaker
		use_mgmtd: no
		use_logd: no
	}

	totem {
		version: 2
		#How long before declaring a token lost (ms)
		    token: 5000
		# How many token retransmits before forming a new configuration
		    token_retransmits_before_loss_const: 10
		# How long to wait for join messages in the membership protocol (ms)
		    join: 1000
		# How long to wait for consensus to be achieved before starting a new
		# round of membership configuration (ms)
		    consensus: 7500
		# Turn off the virtual synchrony filter
		    vsftype: none
		# Number of messages that may be sent by one processor on receipt of the token
		    max_messages: 20
		# Stagger sending the node join messages by 1..send_join ms
		    send_join: 45
		# Limit generated nodeids to 31-bits (positive signed integers)
		    clear_node_high_bit: yes
		# Disable encryption
		    secauth: off
		# How many threads to use for encryption/decryption
		    threads: 0
		# Optionally assign a fixed node id (integer)
		# nodeid: 1234interface {
	interface {
		     ringnumber: 0  
		    member {
		        memberaddr: 10.0.0.1
		    }
		    member {
		        memberaddr: 10.0.0.2
		    }
	bindnetaddr: 10.0.0.0  (netwokip)
		            mcastaddr: 226.94.1.1
		            mcastport: 5405
		            ttl: 1
		    }
		}

	logging {
		fileline: off
		to_stderr: no
		to_logfile: yes
		to_syslog: yes
		logfile: /var/log/cluster/corosync.log
		debug: off
		timestamp: on

	logger_subsys {
		subsys: AMF
		debug: off
		}
	}

	amf {
		mode: disabled
	}


En ambos nodos iniciamos los servicios.::

	# service corosync restart
	# service pacemaker restart	

No olvides los iptables.:

	# service iptables stop

Probamos el cluster y esperamos un momento

	# crm_mon
	Last updated: Thu Sep  1 22:22:04 2016          Last change: Thu Sep  1 22:20:49 2016 by hacluster via crmd on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo1 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 0 resources configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

tambien con.::

	# crm status
	Last updated: Mon Sep 12 20:50:36 2016		Last change: Mon Sep 12 20:48:20 2016 by hacluster via crmd on nodo1
	Stack: classic openais (with plugin)
	Current DC: nodo1 (version 1.1.14-8.el6_8.1-70404b0) - partition with quorum
	2 nodes and 0 resources configured, 2 expected votes

	Online: [ nodo1 nodo2 ]

	Full list of resources:






