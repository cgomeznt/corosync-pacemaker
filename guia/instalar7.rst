Instalar Corosync - Pacemake en CentOS 7
========================================

La instalacion y configuracion es de este link http://clusterlabs.org/quickstart-redhat.html



All examples assume two nodes that are reachable by their short name and IP address::

	node1 - 192.168.1.111
	node2 - 192.168.1.112

The convention followed is that [ALL] # denotes a command that needs to be run on all cluster machines, and [ONE] # indicates a command that only needs to be run on one cluster host.

RHEL 7
+++++++

Install
++++++++

Pacemaker ships as part of the Red Hat High Availability Add-on. The easiest way to try it out on RHEL is to install it from the Scientific Linux or CentOS repositories.

If you are already running CentOS or Scientific Linux, you can skip this step. Otherwise, to teach the machine where to find the CentOS packages, run::

	[ALL] # cat < /etc/yum.repos.d/centos.repo 
	[centos-7-base] 
	name=CentOS-$releasever - Base 
	mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os 
	#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/ 
	enabled=1 
	EOF

Next we use yum to install pacemaker and some other necessary packages we will need::

	[ALL] # yum install pacemaker pcs resource-agents

Create the Cluster
+++++++++++++++++++

The supported stack on RHEL7 is based on Corosync 2, so thats what Pacemaker uses too.

Need sure the services are started::

	[ALL] # systemctl start corosync
	[ALL] # systemctl start pacemaker
	[ALL] # systemctl status pcsd
	[ALL] # systemctl status corosync
	[ALL] # systemctl status pacemaker
	[ALL] # systemctl status pcsd

::

	[ALL] # systemctl enable corosync
	[ALL] # systemctl enable pacemaker
	[ALL] # systemctl enable pcsd


Remembrer disabled the firewall an selinux.::

	# systemctl stop firewalld
	# systemctl status firewalld

First we set up the authentication needed for pcs.::

	[ALL] # echo CHANGEME | passwd --stdin hacluster 
	[ONE] # pcs cluster auth node1 node2 -u hacluster -p CHANGEME --force 


We now create a cluster and populate it with some nodes. Note that the name cannot exceed 15 characters (we'll use 'pacemaker1').::

	[ONE] # pcs cluster setup --force --name pacemaker1 node1 node2

Start the Cluster
++++++++++++++++++
::

	[ONE] # pcs cluster start --all

Show Status ::

	# pcs status

Set Cluster Options
++++++++++++++++++++++

With so many devices and possible topologies, it is nearly impossible to include Fencing in a document like this. For now we will disable it.::

	[ONE] # pcs property set stonith-enabled=false 

One of the most common ways to deploy Pacemaker is in a 2-node configuration. However quorum as a concept makes no sense in this scenario (because you only have it when more than half the nodes are available), so we'll disable it too.::

	[ONE] # pcs property set no-quorum-policy=ignore --force

For demonstration purposes, we will force the cluster to move services after a single failure::

	[ONE] # pcs resource defaults migration-threshold=1

Add a Resource
+++++++++++++++

Lets add a cluster service, we'll choose one doesn't require any configuration and works everywhere to make things easy. Here's the command::

	[ONE] # pcs resource create my_first_svc Dummy op monitor interval=120s

my_first_svc" is the name the service will be known as.

"ocf:pacemaker:Dummy" tells Pacemaker which script to use (Dummy - an agent that's useful as a template and for guides like this one), which namespace it is in (pacemaker) and what standard it conforms to (OCF).

"op monitor interval=120s" tells Pacemaker to check the health of this service every 2 minutes by calling the agent's monitor action.

You should now be able to see the service running using::

	[ONE] # pcs status 

or.::

	[ONE] # crm_mon -1

Simulate a Service Failure
+++++++++++++++++++++++++++

We can simulate an error by telling the service to stop directly (without telling the cluster)::

	[ONE] # crm_resource --resource my_first_svc --force-stop 

If you now run crm_mon in interactive mode (the default), you should see (within the monitor interval of 2 minutes) the cluster notice that my_first_svc failed and move it to another node.

Next Steps
++++++++++++

* Configure Fencing
* Add more services - see Clusters from Scratch for examples of how to add IP address, Apache and DRBD to a cluster
* Learn how to make services prefer a specific host
* Learn how to make services run on the same host
* Learn how to make services start and stop in a specific order
* Find out what else Pacemaker can do - see Pacemaker Explained for an comprehensive list of concepts and options

Los archivos de configuracion son:
++++++++++++++++++++++++++++++++++++
::

	/etc/corosync/corosync.conf
	/var/lib/pacemaker/cib/cib.xml

El archivo cib.xml no se deberia editar directamente, preferiblemente con el comando pcs.


