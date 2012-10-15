Protothread.h, a C++ "port" of Adam Dunkels' protothreads library
=================================================================

Adam Dunkels invented [protothreads](http://dunkels.com/adam/pt/), a nifty set
of C macros for helping write super-light, stackless threads. What
protothreads give you is the ability to write procedural, thread-style code,
but without the overhead of real threads. The kind of thing embedded
programmers normally use `switch` state machines for.

This is a port of Adam's protothread library to C++. If you're using C++, this
might be the way to go, as there are a few advantages over protothreads in C:

* You can make a Protothread class, so you don't need to pass the `struct pt*`
  around everywhere.
* You can use instance variables where you might have used statics, making
  your protothreads easy to multi-instance.
* You can write classes derived from Protothread that add helper variables and
  macros to read and wait for timers, specific I/O ports, etc.

*Okay, so show us an example.* Fair call.

Below is a C++-style protothread that implements a simple packet protocol.
Each packet has a sync byte, a length byte, *n* data bytes, and a checksum
byte. Packets are only processed if they’re good and complete:

```C++
bool UartThread::Run()
{
    PT_BEGIN();

    while (true) {
        // wait for sync byte
        PT_WAIT_UNTIL(ReadByte(ch));
        if (ch == Sync) {
            // read length byte, ensure packet not too big
            PT_WAIT_UNTIL(ReadByte(ch));
            len = ch;
            if (len <= MaxLength) {
                // read n data bytes
                for (i = 0; i < len; i++) {
                    PT_WAIT_UNTIL(ReadByte(ch));
                    data[i] = ch;
                }
                // read checksum, dispatch packet if valid
                PT_WAIT_UNTIL(ReadByte(ch));
                if (ValidChecksum(data, len, ch))
                    Dispatch(data, len);
            }
        }
    }

    PT_END();
}
```

Not bad, eh? Even with comments it’s much shorter and sweeter than the
equivalent state machine version (which, incidentally, is pretty much what the
protothread macros expand to):

```C++
bool UartThread::Run()
{
    while (true) {
        switch (state) {
        case StateSync:
            if (!ReadByte(ch))
                return true;
            if (ch != Sync)
                break;
            state = StateLength;

        case StateLength:
            if (!ReadByte(ch))
                return true;
            len = ch;
            if (len > MaxLength) {
                state = StateSync;
                break;
            }
            i = 0;
            state = StateData;

        case StateData:
            while (i < len) {
                if (!ReadByte(ch))
                    return true;
                data[i] = ch;
                i++;
            }
            state = StateChecksum;

        case StateChecksum:
            if (!ReadByte(ch))
                return true;
            if (ValidChecksum(data, len, ch))
                Dispatch(data, len);
            state = StateSync;
        }
    }
}
```

So there you go. I know which version I’d rather write and maintain.

The [Protothread.h](https://github.com/benhoyt/protothreads-cpp/blob/master/Protothread.h)
header file I put together from Adam Dunkels’ C version should have all you
need to get started. I’ve left writing a “protothread scheduler” as an
exerciser for the reader, as it would be both simple and application-
dependent.

Just a final word: I’m not suggesting protothreads are a replacement for
threads -- they’re not. But when you need the appearance of threads, or you’re
dealing with embedded micros and don’t have screeds of RAM, give them a try.

Read my [original blog entry](http://blog.brush.co.nz/2008/07/protothreads/)
or check out the [source](https://github.com/benhoyt/protothreads-cpp/blob/master/Protothread.h).
