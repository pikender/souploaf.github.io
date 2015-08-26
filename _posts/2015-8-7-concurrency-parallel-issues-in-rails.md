---
layout: post
title: Concurrency Parallel Testing / Issues Rails
---

While looking for ways to test concurrency issues and reliability of
pessimistic and optimistic locking to solve them, came across 

- [Testing multiprocess behaviour is difficult and requires a way to
  synchronize processes at specific execution
points](https://github.com/forkbreak/fork_break) and
- [Blog using fork_break gem](http://www.hairoftheyak.com/testing-concurrency-in-rails/)

When trying to extend above solution to Rails App and putting
breakpoints, realised that [fork_break](https://github.com/forkbreak/fork_break) helps with syncronization of
block of ruby code and not code executing in some other process :(

While extrapolating the learning of breakpoints to Rails Debugger, came
to know about interesting feature in debugger about

- [Remote Debugging with ruby-debug](http://bashdb.sourceforge.net/ruby-debug.html#Remote-Debugging)
and
- [Hooking Rails to start remote debugger with an initializer only when --debugger is used](http://stackoverflow.com/questions/14032313/rails-start-remote-debugger-with-an-initializer-only-when-debugger-is-used)

On the journey to find way to put breakpoints from a parent process and
controlling their pause / resume to achieve behavior similar to
[fork_break gem](https://github.com/forkbreak/fork_break)

_Will not promise to add more details once back but honest intention lies
there to add someday :)_


Not Related Reference:

- [Rails 3.2.7 SSL Localhost](https://gist.github.com/trcarden/3295935)


