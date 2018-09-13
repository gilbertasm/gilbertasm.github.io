---
layout: post
title:  "Tunning and observing UDP buffers"
---
UDP buffers are controlled by 7 sysctl parameters. Individual buffer sizes are controlled by:

* `net.core.wmem_default, net.core.wmem_max` -- default and max socket send buffer size in bytes. Each socket gets `wmem_default` send buffer size by default, and can request up to wmem_max with [setsockopt][setsockopt] option `SO_SNDBUF`. 

* `net.core.rmem_default, net.core.rmem_max` -- default and max socket receive buffer size in bytes. Each socket gets `rmem_default` reveive buffer size by default, and can request up to rmem_max with [setsockopt][setsockopt] option `SO_RCVBUF`. 

Global system parameters are:

* `net.ipv4.udp_mem = "min pressure max"` -- these are numbers of **PAGES** (4KB) available for all UDP sockets in the system. min, pressure and max controls how memory is managed, but main point is that max is maximum size in PAGES for all UDP bufers in system. These values are set on boot time (if sysctls are not explicitly set) according to available RAM size.
* `net.ipv4.udp_rmem_min, net.ipv4.udp_wmem` -- minimal size for receive/send buffers (in bytes), guaranteed for each socket, even if if buffer size of all UDP sockets exceeds pressure parameter in `net.ipv4.udp_mem`.

Observing statistics with  `/proc/net/udp` and `/proc/net/snmp`.
 
These are best described [in this excellent blog post][packagecloud].


[setsockopt]: http://man7.org/linux/man-pages/man2/setsockopt.2.html
[packagecloud]: https://blog.packagecloud.io/eng/2017/02/06/monitoring-tuning-linux-networking-stack-sending-data/#monitoring-udp-protocol-layer-statistics