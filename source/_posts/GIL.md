---
title: GIL, the magic lock inside the CPython
date: 2021-11-24 10:42:53
tags:
---

# GIL, the magic lock inside the CPython

What makes Python different from other languages, the GIL(**Global Interpreter Lock**) must be the cherry on top of the cake.

Simply put, the GIL is a lock inside the CPython interpreter to prevent the reference counting of an object from becoming a race condition at the bytecode level.

The reference counting would look like this:

```python=
import sys
a = [1,2]
b = a

print(sys.getrefcount(a)) # 3
```

Since Python garbage collection uses reference counting to check unused objects at runtime, each object in the runtime will use a number count to check whether it is used by other objects. If not, then CPython will delete the object to free the memory to avoid memory leaks.

## Disadvantages

The purpose of using the GIL is to deal with the issue of reference counting. However, due to this lock, there are some disadvantages related to it. For example:

1. Threads in Python become useless, especially when dealing with CPU-bound tasks. Moreover, it slows down even lower than a single thread.
2. When using multi-threads or multi-process, even if there is a GIL, the locking mechanism is still needed.

## Comparison of single-process, multi-thread, multi-process, and process pool.

If we conduct a simple test to write our program using the above four methods to compare the speed, then we can see the result by order will like below:

Multi-process-pool > Multi-process > Single process > Multi-thread

The reason why multithreading is the last one is because of the GIL, even if there are many threads, tasks in Python can still only run on a single thread. In addition, context switching will also slow down the program and cause more waste of resources.

## Solution

There are many ways to overcome this obstacle. Here are some solutions to bypass the GIL:

1. Use another interpreter to execute Python, such as PyPy, Jython.
2. Use multi-processes with pools to perform CPU-bound tasks.
3. Use C extensions to write the key block that you really care about.

So now, let's deep dive into the third solution since the first two are easy to figure out.

## C extensions

If we want to write the C extension, we can use the `ctypes` module to load our c program dynamically. For example, if we want to call a function `say_hello` look like below:

```c=
#include<stdio.h>

void say_hello() {

  printf("This is a function call from C Language\n");

}
```

Then we use the below command to compile it to shared library

```bash=
gcc share_method.c -shared -o share_method.so
```

Then we can use the `ctypes` module in Python to invoke the method from the C program in the Python program.

```python=
from ctypes import cdll

if __name__ == '__main__':
    lib = cdll.LoadLibrary("share_method.so")

    lib.say_hello()
```

The output of this program is below:


> This is a function call from C Language


If we want to do CPU-bound analysis, we can modify our C code to the below:

```c=
void loop() {
//   create a dead loop here
    printf("This is a function call from infinite loop \n");
  while (1) {
    ;
  }
}
```
It will be similar to CPU-bound calculations.

Then we can use Python Thread to call this method via `ctypes`

```python=
from ctypes import cdll
from threading import Thread

if __name__ == '__main__':
    lib = cdll.LoadLibrary("share_method.so")

    t = Thread(target=lib.loop)

    t.start()
```
Finally, we can see two of our CPUs got high usage with the `top` command.

## Conclusion

GIL seems to be an inevitable problem when dealing with multi-threaded programming, but it is still useful in IO-bound tasks such as web crawlers and file operations, etc. So it depends on the situation you have to solve.

The conclusion is below:

1. If you want to deal with IO-bound tasks, then use the multi-thread with pool.
2. If you want to deal with CPU-bound tasks, then use the multi-process with pool.
3. Or you can also use some techniques above to bypass the GIL.

