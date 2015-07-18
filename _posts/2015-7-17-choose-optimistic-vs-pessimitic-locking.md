---
layout: post
title: Choose Optimstic vs Pessimitic Locking 
---

We got hit by lot of bugs, all because of concurrency.

We had a case where returns are created for cancellations.

It was structured to tell cancelled_qty and a return would be created for that much quantity.

Return Creation will then trigger setting of cancelled_qty for
concerned line_item, it will also take responsibility of bumping
product inventory and cancel line_item if everything cancelled.

Above is not enough so Return will also decide to refund amount and shipping
charge if needed.

Everything was going great and suddenly we started getting lot of
reports of ambiguous data from Operations Team.

Reports of

- Multiple Refunds
- Less Inventory as Expected
- More Returns Count than cancelled


On detailed analysis of audits, logs and code flow, it turned out as
concurrency issue, validated by very little difference in creation time
of returns

As in some cases, returns / refunds are automatically processed and
resulted in loss of money both as hard cash and less user's able to buy
due to less calculated inventory :(

Cocurrency issues can be solved:

- adding extra checks to ensure that invalid returns get a failed signal
 and abort / revert the execution keeping system intact
- ensuring concurrent updates are blocked using locking

Extra checks took us a little ahead but turned out not the ultimate
savior and we decided to explore options of locking.

As soon as you enter the **LOCKS LAND**, you are presented with a
difficult task of opting out of **OPTIMISTIC / PESSIMISTIC LOCKING**

Both have their Pros / Cons :)

Our Use Case was more of preventing creation of unwanted returns and if
started it should kill itself when diagnosed as unhealthy ( Self
Sacrifice for Other's Wellness )

As we need to Abort the process and all the updates done till diagnosis
of unwanted creation / updates, Optimistic Locking came as obvious choice :)

Rule to Decide b/w Pessismitic and Optimistic Locking:

- Revert the process and all execution till now [^See Note Below]
  - Choose Optimistic Locking
- Just want to prevent corruption of already present record
- And, succesive execution of record will keep the record intact
  - Go for Pessimitic Locking

Being in Rails Ecosystem and so much logic / updates implemented using
callback pattern, **OPTIMISTIC LOCKING** seems to be an obvious choice in
most of the cases :P

**Note** : *Pessimistic Locking doesn't abort the execution, it just
 waits for transaction to finish having the lock and continues with
execution when lock released*
