.. Pebble documentation master file, created by
   sphinx-quickstart on Thu Oct 17 23:52:22 2013.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Pebble
======

.. only:: html

   :Release: |release|
   :Date: |today|

Modern languages should natively support concurrency, threading and synchronization primitives. Their usage should be the most intuitive possible, yet allowing all the required flexibility.

Pebble aims to help managing threads and processes in an easier way. It wraps Python's standard library threading and multiprocessing objects.


`Concurrent Module`
-------------------

.. decorator:: concurrent.process(timeout=None, name=None, daemon=True, context=None, pool=None)

   Runs the decorated function in a concurrent process, taking care of the results and error management.

   The decorated function will return a pebble.ProcessFuture_ object.

   If *timeout* is set, the process will be stopped once expired and the future object will raise a *concurrent.futures.TimeoutError* exception.

   The *name* parameter let you define the process name.

   The *daemon* parameter switches between daemon and non-daemon threads.

   The *context* parameter can be used to specify the multiprocessing.context_ object used for starting the process.

   The *pool* parameter accepts a pebble.ProcessPool_ object. If provided, the pool will be used to run the function instead of a dedicated process. The *name*, *daemon* and *context* parameters will be ignored.

.. decorator:: concurrent.thread(name=None, daemon=True, pool=None)

   Runs the decorated function in a concurrent thread, taking care of the results and error management.

   The decorated function will return a concurrent.futures.Future_ object.

   The *name* parameter let you define the thread name.

   The *daemon* parameter switches between daemon and non-daemon threads.

   The *pool* parameter accepts a pebble.ThreadPool_ object. If provided, the pool will be used to run the function instead of a dedicated thread. The *name* and *daemon* parameters will be ignored.

`Asynchronous Module`
---------------------

.. decorator:: asynchronous.process(timeout=None, name=None, daemon=True, context=None, pool=None)

   Runs the decorated function in a concurrent process, taking care of the results and error management.

   The decorated function will return an asyncio.Future_ object.

   If *timeout* is set, the process will be stopped once expired and the future object will raise a *asyncio.TimeoutError* exception.

   The *name* parameter let you define the process name.

   The *daemon* parameter switches between daemon and non-daemon threads.

   The *context* parameter can be used to specify the multiprocessing.context_ object used for starting the process.

   The *pool* parameter accepts a pebble.ProcessPool_ object. If provided, the pool will be used to run the function instead of a dedicated process. The *name*, *daemon* and *context* parameters will be ignored.

.. decorator:: asynchronous.thread(name=None, daemon=True, pool=None)

   Runs the decorated function in a concurrent thread, taking care of the results and error management.

   The decorated function will return a an asyncio.Future_ object.

   The *name* parameter let you define the thread name.

   The *daemon* parameter switches between daemon and non-daemon threads.

   The *pool* parameter accepts a pebble.ThreadPool_ object. If provided, the pool will be used to run the function instead of a dedicated thread. The *name* and *daemon* parameters will be ignored.


`Pebble Module`
---------------

.. _pebble.ProcessPool:
.. class:: pebble.ProcessPool(max_workers=multiprocessing.cpu_count(), max_tasks=0, initializer=None, initargs=None, context=None)

   A Pool allows to schedule jobs into a Pool of Processes which will perform them concurrently.
   Process pools work as well as a *context manager*.

   *max_workers* is an integer representing the amount of desired process workers managed by the pool. If *max_tasks* is a number greater than zero, each worker will be restarted after performing an equal amount of tasks.
   *initializer* must be callable, if passed, it will be called every time a worker is started, receiving *initargs* as arguments.

   The *context* parameter can be used to specify the multiprocessing.context_ object used for starting the worker processes.

   .. data:: active

      True if the Pool is running, false otherwise.

   .. function:: schedule(function, args=(), kwargs={}, timeout=None)

      Schedule the function to be executed within the Pool.

      Returns a pebble.ProcessFuture_ object representing the execution of the callable.

      *function* is the function which is about to be scheduled.

      *args* and *kwargs* will be passed to the function respectively as its arguments and keyword arguments.

      *timeout* can be None, an integer or a float. Once expired, it will force the timed out task to be interrupted and the worker will be restarted. *Future.result()* will raise *TimeoutError*, callbacks will be executed.

   .. function:: map(function, *iterables, chunksize=1, timeout=None)

      Concurrently compute the *function* using arguments from each of the iterables.
      Stop when the shortest iterable is exhausted.

      *chunksize* controls the size of the chunks the iterables will be broken into before being passed to the function.

      *timeout* is an integer or a float. If given, it will be assigned to each chunk of the iterables.
      If the computation of the given chunk will last longer than *timeout*, its execution will be terminated. Iterating over its result will raise *TimeoutError*, all computations over the chunk will be lost.

      A pebble.ProcessMapFuture_ object is returned. Its *result* method will return an iterable containing the results of the computation in the same order as they were given.

   .. function:: close()

      No more job will be allowed into the Pool, queued jobs will be consumed.
      To ensure all the jobs are performed call *ProcessPool.join()* just after closing the Pool.

   .. function:: stop()

      The Pool will be stopped abruptly. All enqueued and running jobs will be lost.
      To ensure the Pool to be released call *ProcessPool.join()* after stopping the Pool.

   .. function:: join(timeout=None)

      Waits for all workers to exit, must not be called before calling either *close()* or *stop()*.
      If *timeout* is set and some worker is still running after it expired, a TimeoutError will be raised.

      The *join* function must be called only in the main loop. Calling it in a pebble.ProcessFuture_ callback will result in a deadlock.

.. _pebble.ThreadPool:
.. class:: pebble.ThreadPool(max_workers=multiprocessing.cpu_count(), max_tasks=0, initializer=None, initargs=None)

   A ThreadPool allows to schedule jobs into a Pool of Threads which will perform them concurrently.
   Thread pools work as well as a *context manager*.

   *max_workers* is an integer representing the amount of desired process workers managed by the pool. If *max_tasks* is a number greater than zero, each worker will be restarted after performing an equal amount of tasks.
   *initializer* must be callable, if passed, it will be called every time a worker is started, receiving *initargs* as arguments.

   .. data:: active

      True if the Pool is running, false otherwise.

   .. function:: schedule(function, args=(), kwargs={})

      Schedule the function to be executed within the Pool.

      Returns a concurrent.futures.Future_ object representing the execution of the callable.

      *function* is the function which is about to be scheduled.

      *args* and *kwargs* will be passed to the function respectively as its arguments and keyword arguments.

   .. function:: map(function, *iterables, chunksize=1)

      Concurrently compute the *function* using arguments from each of the iterables.
      Stop when the shortest iterable is exhausted.

      *chunksize* controls the size of the chunks the iterables will be broken into before being passed to the function.

      *timeout* is an integer or a float. If given, it will be assigned to every chunk of the iterables.
      If the computation of the given chunk will last longer than *timeout*, iterating over its result will raise *TimeoutError*.

      A pebble.MapFuture_ object is returned. Its *result* method will return an iterable containing the results of the computation in the same order as they were given.

   .. function:: close()

      No more job will be allowed into the Pool, queued jobs will be consumed.
      To ensure all the jobs are performed call *ThreadPool.join()* just after closing the Pool.

   .. function:: stop()

      The ongoing jobs will be performed, all the enqueued ones dropped; this is a fast way to terminate the Pool.
      To ensure the Pool to be released call *ThreadPool.join()* after stopping the Pool.

   .. function:: join(timeout=None)

      Waits for all workers to exit, must not be called before calling either *close()* or *stop()*.
      If *timeout* is set and some worker is still running after it expired, a TimeoutError will be raised.

      The *join* function must be called only in the main loop. Calling it in a pebble.ProcessFuture_ callback will result in a deadlock.

.. decorator:: pebble.synchronized([lock])

   A synchronized function prevents two or more callers to interleave its execution preventing race conditions.

   The *synchronized* decorator accepts as optional parameter a *Lock*, *RLock* or *Semaphore* from *threading* and *multiprocessing* modules.

   If no synchronization object is given, a *threading.Lock* will be employed. This implies that between different decorated functions only one at a time will be executed.

.. decorator:: pebble.sighandler(signals)

   Convenience decorator for setting the decorated *function* as signal handler for the specified *signals*.

   *signals* can either be a single signal or a list/tuple of signals.

.. function:: pebble.waitforthreads(threads, timeout=None)

   Waits for one or more *Thread* to exit or until *timeout* expires.

   *threads* is a list containing one or more *threading.Thread* objects.
   If *timeout* is not None the function will block for the specified amount of seconds returning an empty list if no *Thread* is ready.

   The function returns a list containing the ready *Threads*.

   .. note::

      Expired *Threads* are not joined by *waitforthreads*.

.. function:: pebble.waitforqueues(queues, timeout=None)

   Waits for one or more *Queue* to be ready or until *timeout* expires.

   *queues* is a list containing one or more *Queue.Queue* objects.
   If *timeout* is not None the function will block for the specified amount of seconds returning an empty list if no *Queue* is ready.

   The function returns a list containing the ready *Queues*.

.. _pebble.ProcessFuture:
.. class:: pebble.ProcessFuture()

   This class inherits from concurrent.futures.Future_. The sole difference with the parent class is the possibility to cancel running executions.

   .. function:: cancel()

      Cancel a running or enqueued call. If the call has already completed then the method will return False, otherwise the call will be cancelled and the method will return True. If the call is running, the process executing it will be stopped and replaced with a new one picking the next task.

.. _pebble.MapFuture:
.. class:: pebble.MapFuture()

   This class inherits from concurrent.futures.Future_. It is returned by the *map* function of a *ThreadPool*.

   .. function:: result()

      Returns an iterator over the results of the *map* function. If a call raises an exception, then that exception will be raised when its value is retrieved from the iterator. The returned iterator raises a concurrent.futures.TimeoutError if __next__() is called and the result isn’t available after timeout seconds from the original call to Pool.map().

   .. function:: cancel()

      Cancel the computation of enqueued element of the iterables passed to the *map* function. If all the elements are already in progress or completed then the method will return False. True is returned otherwise.

.. _pebble.ProcessMapFuture:
.. class:: pebble.ProcessMapFuture()

   This class inherits from pebble.ProcessFuture_. It is returned by the *map* function of a *ProcessPool*.

   .. function:: result()

      Returns an iterator over the results of the *map* function. If a call raises an exception, then that exception will be raised when its value is retrieved from the iterator. The returned iterator raises a concurrent.futures.TimeoutError if __next__() is called and the result isn’t available after timeout seconds from the original call to Pool.map().

   .. function:: cancel()

      Cancel the computation of running or enqueued element of the iterables passed to the *map* function. If all the elements are already completed then the method will return False. True is returned otherwise.

.. exception:: pebble.ProcessExpired

   Raised by *Future.result()* functions if the related process died unexpectedly during its execution.

   .. data:: exitcode

      Integer representing the process' exit code.

   .. data:: pid

      Integer representing the process identifier.


Programming Guidelines
----------------------

The Python's multiprocessing `programming guidelines`_ apply as well for all functionalities within the *process* namespace.

Pool workers termination
++++++++++++++++++++++++

When a `Future` is cancelled or the underlying task times out or the `ProcessPool` is stopped, the affected worker processes are terminated. As a consequence, scheduled functions which allocate resources such as temporary files or child processes are to be handled carefully. If a worker process is terminated abruptly due to the above reason, such resources will not be relinquished.

Concurrent process decorator
++++++++++++++++++++++++++++

When the start method is either `spawn` or `forkserver`, the recommendation is to limit the decoration via `concurrent.process` to top level functions only. More specifically, inner scope functions decoration is not supported.

The following will not work with the mentioned start methods:

::

   def outer():

       @concurrent.process
       def inner():
           return

       future = inner()

       return future.result()


Examples
--------

Concurrent decorators
+++++++++++++++++++++

Run a function in a separate process and wait for its results.

::

    from pebble import concurrent

    @concurrent.process
    def function(arg, kwarg=0):
        return arg + kwarg

    future = function(1, kwarg=1)

    print(future.result())

Quite often developers need to integrate in their projects third party code which appears to be unstable, to leak memory or to hang. The concurrent function allows to easily take advantage of the isolation offered by processes without the need of handling any multiprocessing primitive.

::

    from pebble import concurrent
    from concurrent.futures import TimeoutError
    from third_party_lib import unstable_function

    @concurrent.process(timeout=10)
    def function(arg, kwarg=0):
        unstable_function(arg, kwarg=kwarg)

    future = function(1, kwarg=1)

    try:
        results = future.result()
    except TimeoutError as error:
        print("unstable_function took longer than %d seconds" % error.args[1])
    except ProcessExpired as error:
        print("%s. Exit code: %d" % (error, error.exitcode))
    except Exception as error:
        print("unstable_function raised %s" % error)
        print(error.traceback)  # Python's traceback of remote process


Asynchronous decorators
+++++++++++++++++++++++

Asynchronous decorators expose the same functionalities as for the `concurrent` namespace but are `asyncio` compatible instead.

::

    import asyncio

    from pebble import asynchronous

    @asynchronous.process
    def function(arg, kwarg=0):
        return arg + kwarg

    async def asynchronous_function():
        result = await function(1, kwarg=1)
        print(result)

    asyncio.run(asynchronous_function())


More Pickles
++++++++++++

All process APIs accept `multiprocessing.context` compatible objects. Therefore, it is possible to use alternative implementations such as `multiprocess`_ with `dill`_ in order to serialize more object types.

::

    import threading
    import multiprocess

    from pebble import concurrent

    @concurrent.process(context=multiprocess.get_context('spawn'))
    def function(unpicklable_object):
        """Pickle an unpicklable object."""
        unpicklable_object.pickle_this()

    class Unpicklable:
        """This class cannot be serialized by standard Python Pickle."""
        def __init__(self):
            self.lock = threading.Lock()

        def pickle_this(self):
            with self.lock:
                print('Just pickled!')

    def main():
        unpicklable = Unpicklable()
        future = function(unpicklable)
        future.result()

    if __name__ == '__main__':
        main()


Pools
+++++

The *ProcessPool* has been designed to support task timeouts and critical errors. If a task reaches its timeout, the worker will be interrupted immediately. Abrupt interruptions of the workers are dealt transparently.

The *map* function returns a *Future* object to better control its execution. When the first result is ready, the *result* function will return an iterator. The iterator can be used to retrieve the results no matter their outcome.

::

    from concurrent.futures import TimeoutError
    from pebble import ProcessPool, ProcessExpired

    def function(n):
        return n

    with ProcessPool() as pool:
        future = pool.map(function, range(100), timeout=10)

        iterator = future.result()

        while True:
            try:
                result = next(iterator)
            except StopIteration:
                break
            except TimeoutError as error:
                print("function took longer than %d seconds" % error.args[1])
            except ProcessExpired as error:
                print("%s. Exit code: %d" % (error, error.exitcode))
            except Exception as error:
                print("function raised %s" % error)
                print(error.traceback)  # Python's traceback of remote process

The following example shows how to compute the Fibonacci sequence up to a certain duration after which, all the remaining computations will be cancelled as they would timeout anyway.

::

    from pebble import ProcessPool
    from concurrent.futures import TimeoutError

    def fibonacci(n):
        if n == 0: return 0
        elif n == 1: return 1
        else: return fibonacci(n - 1) + fibonacci(n - 2)

    with ProcessPool() as pool:
        future = pool.map(fibonacci, range(50), timeout=10)

        try:
            for n in future.result():
                print(n)
        except TimeoutError:
            print("TimeoutError: aborting remaining computations")
            future.cancel()

To compute large collections of elements without incurring in IPC performance limitations, it is possible to use the *chunksize* parameter of the *map* function.
If a `timeout` is provided, it will be applied to the whole chunk and not to the single element. If the computation of a chunk times out, the results for the whole chunk will be lost and a `TimeoutError` exception will be raised in place.

::

    from pebble import ProcessPool
    from multiprocessing import cpu_count
    from concurrent.futures import TimeoutError

    def function(n):
        return n

    elements = list(range(1000000))

    cpus = cpu_count()
    size = len(elements)
    chunksize = size / cpus
    # the timeout will be assigned to each chunk
    # therefore, we need to consider its size
    timeout = 10 * chunksize

    with ProcessPool(max_workers=cpus) as pool:
        future = pool.map(function, elements, chunksize=chunksize, timeout=timeout)

        assert list(future.result()) == elements

The `concurrent` and `asynchronous` decorators accept a *pool* parameter. This is useful to control how many instances of decorated functions can be run at the same time.

::

    from concurrent.futures import wait
    from pebble import concurrent, ProcessPool

    pool = ProcessPool(max_workers=4)

    @concurrent.process(pool=pool)
    def function(arg, kwarg=0):
        return arg + kwarg

    futures = []

    # Maximum 4 executions of `function` will be executed in parallel
    for _ in range(100):
        futures.append(function(1, kwarg=1))

    wait(futures)


Pools and AsyncIO
+++++++++++++++++

Pebble pool implemetations are `asyncio` compatible. Parameters such as timeouts can be passed via the `loop.run_in_executor` function.

::

    import time
    import asyncio

    from pebble import ProcessPool

    SLEEP = 10
    TIMEOUT = 3

    def function(seconds):
        print(f"Going to sleep {seconds}s..")
        time.sleep(seconds)
        print(f"Slept {seconds}s.")

    async def main():
        loop = asyncio.get_running_loop()
        pool = ProcessPool()

        await loop.run_in_executor(pool, function, TIMEOUT, SLEEP)

    asyncio.run(main())


Control process resources usage
*******************************

By combining the *resource* module and the *ProcessPool* initializer function, it is possible to control the amount of resources each process can consume.

In the following example, the memory consumption of each worker process is limited to 1 Kb.

::

    import resource
    from pebble import ProcessPool

    MAX_MEM = 1024

    def initializer(limit):
        """Set maximum amount of memory each worker process can allocate."""
        soft, hard = resource.getrlimit(resource.RLIMIT_AS)
        resource.setrlimit(resource.RLIMIT_AS, (limit, hard))

    def function():
        """This function tries to allocate 1Mb worth of string."""
        string = ''

        for _ in range(1024):
            string += 1024 * 'A'

    pool = ProcessPool(initializer=initializer, initargs=(MAX_MEM,))
    future = pool.schedule(function)

    assert isinstance(future.exception(), MemoryError)


Sighandler decorator
++++++++++++++++++++

The syntax ::

    import signal
    from pebble import sighandler

    @sighandler((signal.SIGINT, signal.SIGTERM))
    def signal_handler(signum, frame):
        print("Termination request received!")

Is equivalent to ::

    import signal

    def signal_handler(signum, frame):
        print("Termination request received!")

    signal.signal(signal.SIGINT, signal_handler)
    signal.signal(signal.SIGTERM, signal_handler)

Running the tests
-----------------

Please refer to the `.travis.yml` to see how to run the tests.

.. _concurrent.futures.Future: https://docs.python.org/3/library/concurrent.futures.html#future-objects
.. _asyncio.Future: https://docs.python.org/3/library/asyncio-future.html#asyncio.Future
.. _multiprocessing.context: https://docs.python.org/3/library/multiprocessing.html#contexts-and-start-methods
.. _`programming guidelines`: https://docs.python.org/3/library/multiprocessing.html#programming-guidelines
.. _`dill`: https://pypi.org/project/dill/
.. _`multiprocess`: https://pypi.org/project/multiprocess/

.. toctree::
   :maxdepth: 2
