Protothread.h, a C++ "port" of Adam Dunkels' protothreads library
=================================================================

Adam Dunkels invented [protothreads](http://dunkels.com/adam/pt/), a nifty set
of C macros for helping write super-light, stackless threads. This is a port
of his protothread library to C++. If you're using C++, this might be the way
to go, as there are a few advantages over protothreads in C:

* You can make a Protothread class, so you don't need to pass the `struct pt*` around everywhere.
* You can use instance variables where you might have used statics, making your protothreads easy to multi-instance.
* You can write classes derived from Protothread that add helper variables and macros to read and wait for timers, specific I/O ports, etc.

Read my [original blog entry](http://blog.brush.co.nz/2008/07/protothreads/) or check
out the [source](https://github.com/benhoyt/protothreads-cpp/blob/master/Protothread.h).
