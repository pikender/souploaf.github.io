---
layout: post
title: Status Store As Strings vs Ints 
---

Synopsis
========

No Application exists without some status tracking.
*E-commerce* tracks order payment / shipping status.
*CRM* tracks page publishibility status etc...

There has been an internal debate on Status Representation as Strings vs
Ints

- Payment Status
  - Incomplete vs 0
  - Complete vs 1
- Shipping Status
  - Collected vs 0
  - Wrapped vs 1
  - Picked vs 2
  - Delivered vs 3
- Publishability Status
  - Unpublished vs 0
  - Published vs 1

Dilemna can easily be figured in above representations, if not, let me
help out :)

Payment Status with 0 value does not convey anything about Incomplete,
unless documented or shared by any means.

Above subtle confusion about meaning let us believe that its better to
store status as strings as no more context need to be shared / passed
when its seen

*Incomplete* would never mean *Complete* :P

Debate ended and lets get to work with strings at all places.

But wait, lets hear the reason on using ints too.

- First, and foremost, supporting point is low memory print on disc / ram
with ints
- Second, not the best point but can outweigh in specific situations, is
using *MATH* with integers like SUM, MULTIPLY (with strings its doable
but not useful in real terms)

Scale
=====

Performance outweighs everything when you compare data storage / memory
footprint at scale.

Every penny saved is penny earned.

Looks like suddenly, ints are winning the battle and strings are feeling
cornered.

But documentation and its meaning has its own significance which can't
be neglected.

One approach with ints for documentation could be to create a *Legends
Table* just for meaning sharing.

We can extend it further by imposing foreign_id constraints on status
column with documentation Master Table.

It gives us the benefit of preventing not allowed values to be stored.

BINGO !! What could be better than preventing misuse of system :)

Some Challenges still remain like documentation table should be created
for all contexts like Payments / Shipping Status etc or one table to
rule all.

*Note*: Documentation table exists only for *Documentation*, its use
 should not leak into Application like getting string representations
from here.That's an overhead on Dev Team to cope up :(

Ruby / Rails Context
====================

More than one status transitions are typically implemented using one-of
-the state_machine gems

state_machine gem, gives you nice methods like status?, set status and
everything is taken care magically

Out-of-the-box integrations with Orms like ActiveRecord, Mongoid etc.

With no in-depth insight, we rely on status names and they get stored as
strings unless otherwise stated, *value* option with status declaration
gives us BEST-OF-BOTH-WORLD, nice/meaningful methods and integer
representaion in DB 

```ruby
state_machine :status, :initial => :new do
  status :new, value: 1
end
```
