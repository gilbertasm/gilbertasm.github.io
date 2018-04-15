---
layout: post
title:  "Understanding MySQL replication coordinates"
date:   2018-04-15 16:15:26 +0300
categories: mysql
---

Basic MySQL replication (as implemented in MySQL 5.5) conceptually is very simple - master server use binlog dump thread to write data changes (row based, statement based or mixed depending on settings) into binary log and slave use I/O thread to copy statements from binay log into relay log. Another slave thread - SQL thread - reads data from relay log and applies them (so that changes made in master server would appear in slave server too). This basic MySQL replication design is shown in picture bellow.

![Replication setup]({{ "/assets/images/mysql1.png" | absolute_url }})



[SHOW SLAVE STATUS][show-slave-status] is main command in order to check state of MySQL replication as it shows information about slave threads and other parameters. Here is example from actual running slave server (information not related with slave threads is not shown) with explanation:

{% highlight html %}
Master_Log_File: mysql-bin.001363  - master binlog filename from which the I/O thread is currently reading.
Read_Master_Log_Pos: 867649780     - position in master binlog file up to which the I/O thread has read.
Relay_Log_File: slave-relay.000453 - relay log filename from which the SQL thread is currently reading and executing.
Relay_Log_Pos: 867649926 -  position in relay log file up to which the SQL thread has read and executed.
Relay_Master_Log_File: mysql-bin.001363 - binary log filename containing the most recent event executed by the SQL thread.
Exec_Master_Log_Pos: 867649780 - position in master binlog file to which the SQL thread has read and executed
{% endhighlight %}

Picture below shows graphical representation how these parameters fits into basic MySQL replication setup.

![Replication coordinates]({{ "/assets/images/mysql2.png" | absolute_url }})
            
These 6 parameters logically can be grouped into three tuples (filename, position) and these 3 pairs gives us nice overview of slave threads progress:

- (Master_Log_File, Read_Master_Log_Pos) - this pair of coordinates show information about slave I/O thread state. Slave I/O thread is reading from binlog mysql-bin.001363 and it has read up to 867649780 position in that file.

- (Relay_Log_File, Relay_Log_Pos) - this pair of coordinates show information about slave SQL thread state from relay log perspective. Slave SQL thread is reading from relay file slave-relay.000453 and has read and executed statements up to 867649926 position in that file.

- (Relay_Master_Log_File, Exec_Master_Log_Pos) - this pair of coordinates show information about slave SQL thread state from Master binlog perspective. Slave SQL thread is reading from relay file slave-relay.000453 and has read and executed statements up to 867649926 position in that file. This correspond to mysql-bin.001363 binlog file and position 867649780  in master server.  That is if we start reading from master binlog file pointed by Relay_Master_Log_File in position starting from Exec_Master_Log_Pos and if we start reading from slave relay log file pointed by Relay_Log_File starting from position Relay_Log_Pos we should get the same information.

[show-slave-status]: https://dev.mysql.com/doc/refman/5.5/en/show-slave-status.html