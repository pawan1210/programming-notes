# 30-oct-2018

### 7 - Why GIL

- It’s not that too technically difficult to remove the GIL. But the ramifications will be bad for single-threaded scripts. 
- The main problem with CPython is that it uses Reference Counting as its main process to keep track of Object’s life. Ref Counting main problem is that addition (Py_INCREF in C API) and subtraction (Py_DECREF in C API) in not atomic in nature in some platforms and is very slow in the supported ones. 
- Larry Hasting has removed GIL from Python3 by using atomic increment and atomic decrement in x86 platforms. But this slows Python3 by atleast 20x times. Also if you would peek inside a running interpreter you will see Py_INCREF and Py_DECEF is called millions of time even before your script will start to execute. Add this to slow the atomic functions and you will have slow down everything. 
- Also if you were to remove GIL then you have to make every C code in the VM thread safe which will degrade single threaded performance severely because of the overhead of unnecessary slow down single threaded performance because of the new locks. And if it’s not done then everything will be thread-unsafe and locks and unlocks have to be done manually which is not preferable considering Python is scripting language. 
- Also removing GIL means end of Refernce Counting and including GC and this thing will severely break C APIs.
- https://stackoverflow.com/questions/36479159/why-are-numpy-calculations-not-affected-by-the-global-interpreter-lock/36480941
- https://stackoverflow.com/questions/991904/why-is-there-no-gil-in-the-java-virtual-machine-why-does-python-need-one-so-bad?rq=1
- https://jeffknupp.com/blog/2012/03/31/pythons-hardest-problem/
- https://dev.to/docoprusta/explain-python-global-interpreter-lock-gil-like-im-five-25k8
- https://opensource.com/article/17/4/grok-gil


### 6 - asyncio call at a specific time

```python
import asyncio
import time


def callback(n, loop):
    print('callback {} invoked at {}'.format(n, loop.time()))



async def main(loop):
    now = loop.time()
    print('clock time: {}'.format(time.time()))
    print('loop time: {}'.format(now))

    print('registering callbacks')

    loop.call_at(now + 0.2, callback, 1, loop)
    loop.call_at(now + 0.1, callback, 2, loop)

    loop.call_soon(callback, 3 , loop)


    await asyncio.sleep(1)


event_loop = asyncio.get_event_loop()

try:
    print('entering event loop')
    event_loop.run_until_complete(main(event_loop))
finally:
    print('closing event loop')
    event_loop.close()

```


### 5 - asyncio call later 

```python
import asyncio


def callback(n):
    print('callback {} invoked'.format(n))



async def main(loop):
    print('registering callbacks')
    loop.call_later(0.2, callback, 1)
    loop.call_later(0.1, callback, 2)
    loop.call_soon(callback, 3)


    await asyncio.sleep(0.4)



event_loop = asyncio.get_event_loop()


try:
    print('entering event loop')
    event_loop.run_until_complete(main(event_loop))
finally:
    print('closing event loopp')
    event_loop.close()
```


### 4 - asyncio call soon and functools.partial

```python
import asyncio
import functools

def callback( arg, * , kwargs='default'):
        print( 'callback invoked with {} and {}'.format(arg, kwargs))


async def main(loop):
        print('registering callbacks')
        loop.call_soon(callback,1)
        wrapped = functools.partial(callback, kwargs='not default')
        loop.call_soon(wrapped,2)

        await asyncio.sleep(5)




event_loop = asyncio.get_event_loop()
try:
    print('entering event loop')
    event_loop.run_until_complete(main(event_loop))
finally:
    print('closing event loop')
    event_loop.close()
```

### 3 - Using a class as decorator

```python
def decorator1(f):
    def helper():
        print("Decorating", f.__name__)
        f()
    return helper

@decorator1
def foo():
    print("inside foo()")

foo()
```

The following will do the same job as above

```python
class decorator2:
    
    def __init__(self, f):
        self.f = f
        
    def __call__(self):
        print("Decorating", self.f.__name__)
        self.f()

@decorator2
def foo():
    print("inside foo()")

foo()
```

### 2 - timeit function without ipython

```python
#fibonnaci.py
def fib(n):
    if n == 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fib(n-1) + fib(n-2)

def fibi(n):
    old, new = 0, 1
    if n == 0:
        return 0
    for i in range(n-1):
        old, new = new, old + new
    return new
```

testing timeit
```python
from timeit import Timer

t1 = Timer("fib(10)","from fibonacci import fib")

for i in range(1,41):
	s = "fib(" + str(i) + ")"
	t1 = Timer(s,"from fibonacci import fib")
	time1 = t1.timeit(3)
	s = "fibi(" + str(i) + ")"
	t2 = Timer(s,"from fibonacci import fibi")
	time2 = t2.timeit(3)
	print("n=%2d, fib: %8.6f, fibi:  %7.6f, percent: %10.2f" % (i, time1, time2, time1/time2))
```
side note: implementation of fibonacci where you call yourself without name of function using self and with memorization

```python
class Fibonacci:

    def __init__(self, i1=0, i2=1):
        self.memo = {0:i1, 1:i2}

    def __call__(self, n):
        if n not in self.memo: 
            self.memo[n] = self.__call__(n-1) + self.__call__(n-2)  
        return self.memo[n]
    
fib = Fibonacci()
lucas = Fibonacci(2, 1)

for i in range(1, 16):
    print(i, fib(i), lucas(i)) 
```


### 1 - Checking if a object is iterable

```python
def iterable(obj):
     try:
         iter(obj)
         return True
     except TypeError:
         return False 
```

can check it over a simple class like this:

```python
class Reverse:
    """
    Creates Iterators for looping over a sequence backwards.
    """
    
    def __init__(self, data):
        self.data = data
        self.index = len(data)
    def __iter__(self):
        return self
    def __next__(self):
        if self.index == 0:
            raise StopIteration
        self.index = self.index - 1
        return self.data[self.index]
```