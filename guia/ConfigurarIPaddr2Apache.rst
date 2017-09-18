Configurar IPaddr2 y Apache
=============================


List Resource Agent (RA) classes.
+++++++++++++++++++++++++++++++++
::

	# pcs resource standards
	# pcs resource agents

List available RAs
++++++++++++++++++++
::

	pcs   # pcs resource agents ocf
	pcs   # pcs resource agents lsb
	pcs   # pcs resource agents service
	pcs   # pcs resource agents stonith
	pcs   # pcs resource agents

List resource describe RA
++++++++++++++++++++++++++
::

	# pcs resource describe IPaddr2
	# pcs resource describe apache

::

	# pcs resource create ClusterIP ocf:heartbeat:IPaddr2 ip=192.168.1.110 cidr_netmask=32 op monitor interval=30s

Pacemaker controla Apache por lo que no deben tener el servicio disabled y no iniciarlo.

Adicionalmente deben tener configurado un VHost para el status del servidor, si esto no le funciona no intente seguir configurando, el status del servidor debe estar al 100%.::

	# vi server_status.conf 
	 <Location /server-status>
		SetHandler server-status
		Require local
	 </Location>

Verifiquen este VHost.::

	# curl http://localhost/server-status
	<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
	<html><head>
	<title>Apache Status</title>
	</head><body>
	<h1>Apache Server Status for localhost (via 127.0.0.1)</h1>

	<dl><dt>Server Version: Apache/2.4.6 (CentOS) OpenSSL/1.0.1e-fips PHP/5.4.16</dt>
	<dt>Server MPM: prefork</dt>
	<dt>Server Built: Apr 12 2017 21:03:28
	</dt></dl><hr /><dl>
	<dt>Current Time: Sunday, 17-Sep-2017 21:59:54 EDT</dt>
	<dt>Restart Time: Sunday, 17-Sep-2017 21:50:40 EDT</dt>
	<dt>Parent Server Config. Generation: 1</dt>
	<dt>Parent Server MPM Generation: 0</dt>
	<dt>Server uptime:  9 minutes 14 seconds</dt>
	<dt>Server load: 0.05 0.03 0.05</dt>
	<dt>Total accesses: 42 - Total Traffic: 196 kB</dt>
	<dt>CPU Usage: u.01 s.01 cu0 cs0 - .00361% CPU load</dt>
	<dt>.0758 requests/sec - 362 B/second - 4778 B/request</dt>
	<dt>1 requests currently being processed, 5 idle workers</dt>
	</dl><pre>__W___..........................................................
	................................................................
	................................................................
	................................................................
	</pre>
	<p>Scoreboard Key:<br />
	"<b><code>_</code></b>" Waiting for Connection, 
	"<b><code>S</code></b>" Starting up, 
	"<b><code>R</code></b>" Reading Request,<br />
	"<b><code>W</code></b>" Sending Reply, 
	"<b><code>K</code></b>" Keepalive (read), 
	"<b><code>D</code></b>" DNS Lookup,<br />
	"<b><code>C</code></b>" Closing connection, 
	"<b><code>L</code></b>" Logging, 
	"<b><code>G</code></b>" Gracefully finishing,<br /> 
	"<b><code>I</code></b>" Idle cleanup of worker, 
	"<b><code>.</code></b>" Open slot with no current process<br />
	<p />


	<table border="0"><tr><th>Srv</th><th>PID</th><th>Acc</th><th>M</th><th>CPU
	</th><th>SS</th><th>Req</th><th>Conn</th><th>Child</th><th>Slot</th><th>Client</th><th>VHost</th><th>Request</th></tr>

	<tr><td><b>0-0</b></td><td>22290</td><td>0/7/7</td><td>_
	</td><td>0.00</td><td>56</td><td>0</td><td>0.0</td><td>0.03</td><td>0.03
	</td><td>127.0.0.1</td><td nowrap>srv-vccs-haproxywaf01:80</td><td nowrap>GET /server-status HTTP/1.1</td></tr>

	<tr><td><b>1-0</b></td><td>22291</td><td>0/8/8</td><td>_
	</td><td>0.01</td><td>26</td><td>0</td><td>0.0</td><td>0.04</td><td>0.04
	</td><td>127.0.0.1</td><td nowrap>srv-vccs-haproxywaf01:80</td><td nowrap>GET /server-status HTTP/1.1</td></tr>

	<tr><td><b>2-0</b></td><td>22292</td><td>0/7/7</td><td><b>W</b>
	</td><td>0.01</td><td>0</td><td>0</td><td>0.0</td><td>0.03</td><td>0.03
	</td><td>127.0.0.1</td><td nowrap>srv-vccs-haproxywaf01:80</td><td nowrap>GET /server-status HTTP/1.1</td></tr>

	<tr><td><b>3-0</b></td><td>22293</td><td>0/7/7</td><td>_
	</td><td>0.00</td><td>50</td><td>0</td><td>0.0</td><td>0.03</td><td>0.03
	</td><td>127.0.0.1</td><td nowrap>srv-vccs-haproxywaf01:80</td><td nowrap>GET /server_status HTTP/1.1</td></tr>

	<tr><td><b>4-0</b></td><td>22294</td><td>0/7/7</td><td>_
	</td><td>0.00</td><td>11</td><td>0</td><td>0.0</td><td>0.03</td><td>0.03
	</td><td>127.0.0.1</td><td nowrap>srv-vccs-haproxywaf01:80</td><td nowrap>GET /server-status HTTP/1.1</td></tr>

	<tr><td><b>5-0</b></td><td>22587</td><td>0/6/6</td><td>_
	</td><td>0.00</td><td>41</td><td>0</td><td>0.0</td><td>0.03</td><td>0.03
	</td><td>127.0.0.1</td><td nowrap>srv-vccs-haproxywaf01:80</td><td nowrap>GET /server-status HTTP/1.1</td></tr>

	</table>
	 <hr /> <table>
	 <tr><th>Srv</th><td>Child Server number - generation</td></tr>
	 <tr><th>PID</th><td>OS process ID</td></tr>
	 <tr><th>Acc</th><td>Number of accesses this connection / this child / this slot</td></tr>
	 <tr><th>M</th><td>Mode of operation</td></tr>
	<tr><th>CPU</th><td>CPU usage, number of seconds</td></tr>
	<tr><th>SS</th><td>Seconds since beginning of most recent request</td></tr>
	 <tr><th>Req</th><td>Milliseconds required to process most recent request</td></tr>
	 <tr><th>Conn</th><td>Kilobytes transferred this connection</td></tr>
	 <tr><th>Child</th><td>Megabytes transferred this child</td></tr>
	 <tr><th>Slot</th><td>Total megabytes transferred this slot</td></tr>
	 </table>
	<hr>
	<table cellspacing=0 cellpadding=0>
	<tr><td bgcolor="#000000">
	</td></tr>olor="#ffffff" face="Arial,Helvetica">SSL/TLS Session Cache Status:</font></b>
	<tr><td bgcolor="#ffffff">
	cache type: <b>SHMCB</b>, shared memory: <b>512000</b> bytes, current entries: <b>0</b><br>subcaches: <b>32</b>, indexes per subcache: <b>88</b><br>index usage: <b>0%</b>, cache usage: <b>0%</b><br>total entries stored since starting: <b>1</b><br>total entries replaced since starting: <b>0</b><br>total entries expired since starting: <b>1</b><br>total (pre-expiry) entries scrolled out of the cache: <b>0</b><br>total retrieves since starting: <b>0</b> hit, <b>0</b> miss<br>total removes since starting: <b>0</b> hit, <b>0</b> miss<br></td></tr>
	</table>
	</body></html>

Nos aseguramos de que el servicio de apache este disable y en stop.::

	# systemctl disable httpd
	# systemctl stop httpd
	# systemctl status httpd
	● httpd.service - The Apache HTTP Server
	   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
	   Active: inactive (dead)
		 Docs: man:httpd(8)
		       man:apachectl(8)

	sep 17 20:55:22 srv-vccs-haproxywaf01 systemd[1]: Starting The Apache HTTP Server...
	sep 17 20:55:31 srv-vccs-haproxywaf01 systemd[1]: Started The Apache HTTP Server.
	sep 17 20:56:21 srv-vccs-haproxywaf01 systemd[1]: Stopping The Apache HTTP Server...
	sep 17 20:56:22 srv-vccs-haproxywaf01 systemd[1]: Stopped The Apache HTTP Server.
	sep 17 21:17:01 srv-vccs-haproxywaf01 systemd[1]: Unit httpd.service cannot be reloaded because it is inactive.


	# pcs resource create WebSite ocf:heartbeat:apache configfile="/etc/httpd/conf/httpd.conf" statusurl="http://127.0.0.1/server-status" op monitor interval=15s



En todos los servidores, download el HAProxy OCF resource agent.::

	# cd /usr/lib/ocf/resource.d/heartbeat
	# curl -O https://raw.githubusercontent.com/thisismitch/cluster-agents/master/haproxy

En todos los servidores, hacer que sea executable.::

	# sudo chmod +x haproxy

Feel free to review the contents of the resource before continuing. It is a shell script that can be used to manage the HAProxy service.

Now we can use the HAProxy OCF resource agent to define our haproxy cluster resource.

Add haproxy Resource
With our HAProxy OCF resource agent installed, we can now configure an haproxy resource that will allow the cluster to manage HAProxy.

On either load balancer server, create the haproxy primitive resource with this command:

	# pcs configure primitive haproxy ocf:heartbeat:haproxy op monitor interval=15s
The specified resource tells the cluster to monitor HAProxy every 15 seconds, and to restart it if it becomes unavailable.

Check the status of your cluster resources by using sudo crm_mon or sudo crm status:
