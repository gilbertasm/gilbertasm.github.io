---
layout: post
title:  "Bundler and IPv6"
---

During routine system update on client machine, `bundle update` just hanged, nothing was happening. Process table did not show anything interesting:

{% highlight bash %}
root     14766  0.0  0.0 110736  6280 pts/5    S+   17:22   0:00      \_ /bin/bash /usr/src/mor/x12/update.sh
root     14771  0.0  0.0 106076   664 pts/5    S+   17:22   0:00          \_ /bin/bash /usr/src/mor/x12/update.sh
root     14773  0.0  0.0 100924   708 pts/5    S+   17:22   0:00          |   \_ tee -a update.log
root     14772  0.0  0.0 106076   684 pts/5    S+   17:22   0:00          \_ /bin/bash /usr/src/mor/x12/update.sh
root     14774  0.0  0.0 100924   708 pts/5    S+   17:22   0:00          |   \_ tee -a update.log
root     16835  0.2  0.2 278244 84780 pts/5    Sl+  17:23   0:02          \_ ruby /usr/local/rvm/gems/ruby-2.1.2@global/bin/bundle update
{% endhighlight %}

Good old strace revealed that bundler is trying to [connect][connect] using IPv6 and nothing is happening:

{% highlight bash %}
[root@CentOS-69-64-minimal ~]# strace -p 16835
Process 16835 attached
connect(12, {sa_family=AF_INET6, sin6_port=htons(80), inet_pton(AF_INET6, "2a04:4e42::70", &sin6_addr), sin6_flowinfo=0, sin6_scope_id=0}, 28
{% endhighlight %}

"Easy" and blunt way to solve is temporary [disable IPv6][disableIPv6], but it was interesting to find the root for such behavior. Googling bundler IPv6 lead to this [stack overflow answer][answer], there more elegant solution was proposed.
Still, it is strange that such important peace of Ruby infrastructure is misbehaving for a long time (I experienced this issue about month ago and today again), and there is not much fuss about this. Maybe this happens only in some random corner cases, who knows.

[disableIPv6]: https://wiki.centos.org/FAQ/CentOS7#head-8984faf811faccca74c7bcdd74de7467f2fcd8ee
[connect]: https://linux.die.net/man/2/connect
[answer]: https://stackoverflow.com/questions/49800432/gem-cannot-access-rubygems-org/50349235#50349235