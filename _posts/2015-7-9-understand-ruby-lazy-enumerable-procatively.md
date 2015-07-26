---
layout: post
title: Understanding Enumerable#lazy Proactively 
---

Ruby 2.0.0 introduced `Enumerator::Lazy` to make infinite lists possible
in finite way.

E.g. If we need first n integers

As a first attempt, we will generate integers starting from 1 to a very
big number, and pick first n integers

- Very Big Number can put us in trouble, if not chosen wisely.
  - If its really big, it will take time / memory to store long list of
integers.
  - If its not sufficiently big, we will loose some / many integers

### Ruby 2.0.0 and after

```ruby
2.0.0-p451 :003 > (1..Float::INFINITY).lazy.map {|x| x}.first(10)
 => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] 
```

Above version without `.lazy` becomes unresponsive as it enters into
infinite loop

### Prior to Ruby 2.0.0, it would have been like

```ruby
def first_elements(n)
  a = []
  (1..Float::INFINITY).map do |x|
    a << x
    break if a.size > (n - 1)
  end
  a
end

first_elements(10)
 => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] 
```

Lot of hoops taken and doesn't look very declarative / expressive too

No doubt, `#lazy` is more expressive and appeals to single-line lovers :)

Though more expressive, `lazy` does provide a barrier to use effectively
and it's adoption in production ready code is rare / sparse.

**Analysing the simple example above, a *simple rule to overcome the
barrier of `Enumerator::Lazy` use* can be evaluating the condition to
break from infinite loop first and working backwards from it.** 

Another example to work out above rule is a blog by thoughtbot [Lazy Refactoring](https://robots.thoughtbot.com/lazy-refactoring)

```ruby
def self.search(query)
  matching_column = [:name, :city, :title].detect do |column|
    Event.where(column => query).exists?
  end

  if matching_column
    Event.where(matching_column => query)
  else
    Event.fuzzy_search(query)
  end
end
```

A simple walkthrough of above code

1. Check, if any column from [:name, :city, :title] has result matching
to query
2. Store the matching column if any
3. If there is any matching column, find the result on matched column
4. If no matching column found, return result of fuzzy search from query

Here, 

- *Infinite loop* translates to available options which is query result on any [:name, :city, :title]

```ruby
results = [:name, :city, :title].map {|column| Event.where(column => query)}
```

- *Breaking condition* is match found for query from avilable query results

```ruby
first_found = results.detect { |x| x.exists? }
```

### Sample Code to test lazy execution 

```ruby
class Event
  def self.where(o = {})
    o.has_key?(:name)
    self.new(o)
  end

  def initialize(details)
    @details = details
  end

  def exists?
    true
  end

end


query = 'lazy_works'

puts "Without Lazy"
first_result = [:name, :city, :title].map {|column| p Event.where(column
=> query)}.detect { |x| p x.exists? }
p first_result
puts

puts "With Lazy"
lazy_first_result = [:name, :city, :title].lazy.map {|column| p
Event.where(column => query)}.detect { |x| p x.exists? }
p lazy_first_result
puts
```

### Output

```ruby
Without Lazy
#<Event:0x000001020a25c8 @details={:name=>"lazy_works"}>
#<Event:0x000001020a2438 @details={:city=>"lazy_works"}>
#<Event:0x000001020a22d0 @details={:title=>"lazy_works"}>
true
#<Event:0x000001020a25c8 @details={:name=>"lazy_works"}>

With Lazy
#<Event:0x000001020a1b50 @details={:name=>"lazy_works"}>
true
#<Event:0x000001020a1b50 @details={:name=>"lazy_works"}>
```

## Observations

- Without lazy version evaluates all possible results and then filters
the desired result
- With lazy, it evaluates first result and checks whether it satisfies
filtering condition
  - As it does, it breaks without evalluating other results
- Ruby lazy Enumerable can come handy at places where early return is desired if first match is found


#### Please go through references, preferably as ordered, for in-depth, end-to-end know-how on Ruby lazy enumerable

References
==========

- [Enumerator::Lazy](http://ruby-doc.org/core-2.0.0/Enumerator/Lazy.html)
- [Ruby 2.0 Enumerable::Lazy](http://railsware.com/blog/2012/03/13/ruby-2-0-enumerablelazy/)
- [Ruby 2.0 Works Hard So You Can Be Lazy](http://patshaughnessy.net/2013/4/3/ruby-2-0-works-hard-so-you-can-be-lazy)
- [Stop including Enumerable, return Enumerator instead](http://blog.arkency.com/2014/01/ruby-to-enum-for-enumerator/)
- [Lazy Refactoring](https://robots.thoughtbot.com/lazy-refactoring)
- [Implementing Lazy Enumerables in Ruby](http://www.sitepoint.com/implementing-lazy-enumerables-in-ruby/?utm_source=rubyweekly&utm_medium=email)
