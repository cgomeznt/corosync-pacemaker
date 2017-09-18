Resource Clones
==================

https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Configuring_the_Red_Hat_High_Availability_Add-On_with_Pacemaker/ch-advancedresource-HAAR.html

8.1. Resource Clones
+++++++++++++++++++++

You can clone a resource so that the resource can be active on multiple nodes. For example, you can use cloned resources to configure multiple instances of an IP resource to distribute throughout a cluster for node balancing. You can clone any resource provided the resource agent supports it. A clone consists of one resource or one resource group.

Note
Only resources that can be active on multiple nodes at the same time are suitable for cloning. For example, a Filesystem resource mounting a non-clustered file system such as ext4 from a shared memory device should not be cloned. Since the ext4 partition is not cluster aware, this file system is not suitable for read/write operations occurring from multiple nodes at the same time.

⁠8.1.1. Creating and Removing a Cloned Resource
+++++++++++++++++++++++++++++++++++++++++++++++

You can create a resource and a clone of that resource at the same time with the following command.
::
	pcs resource create resource_id standard:provider:type|type [resource options] clone  [meta clone_options]

The name of the clone will be resource_id-clone.
You cannot create a resource group and a clone of that resource group in a single command.
Alternately, you can create a clone of a previously-created resource or resource group with the following command.::

	pcs resource clone resource_id | group_name [clone_options]...

The name of the clone will be resource_id-clone or group_name-clone.

Note
You need to configure resource configuration changes on one node only.

Note
When configuring constraints, always use the name of the group or clone.

When you create a clone of a resource, the clone takes on the name of the resource with -clone appended to the name. The following commands creates a resource of type apache named webfarm and a clone of that resource named webfarm-clone.::

	# pcs resource create webfarm apache clone

Use the following command to remove a clone of a resource or a resource group. This does not remove the resource or resource group itself.::

	pcs resource unclone resource_id | group_name

For information on resource options, see Section 5.1, “Resource Creation”.
Table 8.1, “Resource Clone Options” describes the options you can specify for a cloned resource.
⁠
::

	Table 8.1. Resource Clone Options

	Field	Description
	 priority, target-role, is-managed	Options inherited from resource that is being cloned, as described in Table 5.3, “Resource Meta Options”.
	 clone-max	How many copies of the resource to start. Defaults to the number of nodes in the cluster.
	 clone-node-max	How many copies of the resource can be started on a single node; the default value is 1.
	 clone-min	The number of instances that must be running before any dependent resources can run. This parameter can be of particular use for services behind a virtual IP and HAProxy, such as is often required for an OpenStack platform.
	 notify	When stopping or starting a copy of the clone, tell all the other copies beforehand and when the action was successful. Allowed values: false, true. The default value is false.
	 globally-unique	Does each copy of the clone perform a different function? Allowed values: false, true
	If the value of this option is false, these resources behave identically everywhere they are running and thus there can be only one copy of the clone active per machine.
	If the value of this option is true, a copy of the clone running on one machine is not equivalent to another instance, whether that instance is running on another node or on the same node. The default value is true if the value of clone-node-max is greater than one; otherwise the default value is false.
	 ordered	Should the copies be started in series (instead of in parallel). Allowed values: false, true. The default value is false.
	 interleave	Changes the behavior of ordering constraints (between clones/masters) so that copies of the first clone can start or stop as soon as the copy on the same node of the second clone has started or stopped (rather than waiting until every instance of the second clone has started or stopped). Allowed values: false, true. The default value is false.

⁠8.1.2. Clone Constraints
+++++++++++++++++++++++++

In most cases, a clone will have a single copy on each active cluster node. You can, however, set clone-max for the resource clone to a value that is less than the total number of nodes in the cluster. If this is the case, you can indicate which nodes the cluster should preferentially assign copies to with resource location constraints. These constraints are written no differently to those for regular resources except that the clone's id must be used.
The following command creates a location constraint for the cluster to preferentially assign resource clone webfarm-clone to node1.::

	# pcs constraint location webfarm-clone prefers node1

Ordering constraints behave slightly differently for clones. In the example below, webfarm-stats will wait until all copies of webfarm-clone that need to be started have done so before starting itself. Only if no copies of webfarm-clone can be started then webfarm-stats will be prevented from being active. Additionally, webfarm-clone will wait for webfarm-stats to be stopped before stopping itself.::

	# pcs constraint order start webfarm-clone then webfarm-stats

Colocation of a regular (or group) resource with a clone means that the resource can run on any machine with an active copy of the clone. The cluster will choose a copy based on where the clone is running and the resource's own location preferences.
Colocation between clones is also possible. In such cases, the set of allowed locations for the clone is limited to nodes on which the clone is (or will be) active. Allocation is then performed as normally.
The following command creates a colocation constraint to ensure that the resource webfarm-stats runs on the same node as an active copy of webfarm-clone.::

	# pcs constraint colocation add webfarm-stats with webfarm-clone

⁠8.1.3. Clone Stickiness
++++++++++++++++++++++++++++
To achieve a stable allocation pattern, clones are slightly sticky by default. If no value for resource-stickiness is provided, the clone will use a value of 1. Being a small value, it causes minimal disturbance to the score calculations of other resources but is enough to prevent Pacemaker from needlessly moving copies around the cluster.
