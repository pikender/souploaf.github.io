---
layout: post
title: Is MySQL Master Slave Replication Working
---

Often, the bottleneck and single point of failure to many big systems is
Database.

There are **slaves** configured for Master to **share atleast the READ** load.

MySQL Master-Slave Replication is so robust at core, there are very less
mentions of its not working properly.

But when replication fails, and if left unnoticed, can cause serious
problems like wrong reporting, wrong results / interpetations in
applications connected to slaves.

E.g. E-Commerce selling T-shirts. All orders recorded in Master
(obviously :) ). An App connected to Slave reading delivered orders and
paying the merchant accordingly.

If replication fails, associated merchants will be unhappy to not get
any payments as none reported as released order since last payment.

Orders are sold but not replicated on slave to reflect :(

*Morale:* Replication Errors should not go unnoticed and reported almost
 realtime so they can be looked at the earliest.

Here are few links which can get you great idea on how to set-up basic
replication monitoring:

- [How to Monitor MySQL Replication?](http://blog.webyog.com/2012/11/20/how-to-monitor-mysql-replication/)
- [Replication monitoring with monit](http://www.elevatedcode.com/2007/12/05/replication-monitoring-with-monit.html)
- [More bits on monit](http://www.darkcoding.net/software/setting-up-monit-on-ubuntu/)
- [MySQL Master-Slave Replication – Bash Script Monitor](http://www.linuxtutorial.co.uk/master-slave-replication-bash-script-monitor/)

### Exceptional Replication Error due to mismatch in records created on master / slave when a bulk insert query replicated on slave form master

As mentioned <a href="/mysql-assign-value-on-collective-status/" target="_blank">here</a> 

```sql
INSERT INTO packages(`package_number`, `status`, `shipping_status`) (
  SELECT orders.order_number, IF(SUM(line_items.status) = COUNT(*) * 2,
"cancelled", "released"), orders.shipping_status 
  FROM line_items
  INNER JOIN orders ON orders.id = line_items.order_id
  WHERE line_items.shippable = 1 
  GROUP by line_items.order_id
)
```

Mass Insert Queries like above not dictating `ORDER BY` SQL clause can
result into rows in `packages table` with same package number having different
primary id column and disrupting the Replication Flow.

**Little fact about mysql replication** is that it works by replaying
commands over SLAVE as executed on MASTER using bin log being sent from
MASTER.

The bug was unusual and very-very hard to catch as no errors on
mysql-replication reported. 

It got surfaced by comparing same reports on MASTER and SLAVE and noticing different results and that too not because
of replication lag as stayed same throughout the day :(

Conclusion
==========

- Monitor your MySQL Replication as it can disrupt the system in unusual
  ways
- Data Migrations might result in different data-sets on Master / Slave,
  Keep a Watch !!
  - If possible, add a `ORDER BY` clause to force ordering and
    increasing chances of same data-set
  - After migration, compare the data-sets for exact replicas and set-up
    replication again if inconsistent
