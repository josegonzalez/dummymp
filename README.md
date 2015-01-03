DummyMP - Multiprocessing for Dummies!
=======================================

Introduction
-------------
DummyMP is a Python module that simplifies parallelized computing for
those not familiar with Python's multiprocessing API. It includes
easy-to-use functions for deploying and managing computationally
expensive operations.

Simply said, it's a library that you can use to make your programs
faster!

Features
---------
Despite being simple, DummyMP packs in many useful features:

  * Run arbitrary functions in a parallized process
  * Serialized function returns - that means you can get the returns of
    the functions you ran with DummyMP, in the order you called the
    functions.
  * Isolated execution - that means changing a variable outside DummyMP
    does NOT affect any queued or running functions.
  * Log forwarding - logs can be forwarded from parallel processes/tasks
    to the central log handler. If your function logs, no worries - it
    will show up to the main log handler.
  * Comprehensive task manager - an automatic task manager starts
    processes/tasks based on CPU availability, and based on a configured
    "nice" value. This is important on shared servers, where CPU usage
    is shared across many users.
  * Configure process/task start aggression. DummyMP includes presets
    for users to pick from to configure how aggressive DummyMP should be
    when starting processes/tasks.
  * Configure maximum number of processes. By default, DummyMP considers
    the number of CPUs available to be the maximum number of processes
    possible.
  * Ability to kill all running tasks, as necessary. (Individual task
    support not available yet.)
  * Ability to completely reset DummyMP.
  * Callback support for starting and finishing processes/tasks, with
    completion, running, and total processes statistics.
  * And above all, a ridiculously simple API to do all of this!

Requirements
-------------
DummyMP requires the following:

  * Python >=2.6 (for 2.x version) or Python 3.x (all versions)
    * NOTE: Python 2.5 can be used, but it requires the backported
      multiprocessing module in order to work. We do not officially
      support Python 2.5 due to uncertainty about the multiprocessing
      module offered for older versions of Python.
  * psutil 2.x.x
    * Note that this is 2.x, NOT 1.x. Not many distributions carry the
      new 2.x version yet.

Examples
---------
It's really simple! Given a slow function myfunc():

    import dummymp
    import time
    
    def myfunc():
        print("Starting!")
        time.sleep(5)
        print("Done!")
    
    # Add functions to queue!
    dummymp.run(myfunc)
    dummymp.run(myfunc)
    dummymp.run(myfunc)
    
    # Process queue and run the functions until all are finished.
    dummymp.process_until_done()

What if your function needs arguments? No worries, you just have to
specify it within run():

    import dummymp
    import time
    
    def myfunc(a, **kwargs):
        print("Starting! %i %s!" % (a, kwargs["fruit"]))
        time.sleep(3)
        print("Done! %i %s!" % (a, kwargs["fruit"]))
    
    # Add functions to queue! Now with arguments and keyword arguments!
    dummymp.run(myfunc, 4, fruit="apples")
    dummymp.run(myfunc, 3, fruit="bananas")
    dummymp.run(myfunc, 2, fruit="oranges")
    
    # Process queue and run the functions until all are finished.
    # Note that because of the nature of parallelism, the numbers
    # printed may be out of order.
    dummymp.process_until_done()

Of course, you might want some returns with that. Here's how to do
returns:

    import dummymp
    import time
    
    def myfunc(num):
        print("Starting! My favorite number is: %i" % num)
        time.sleep(num)
        print("Done! My favorite number is: %i" % num)
        return num
    
    # Add functions to queue!
    dummymp.run(myfunc, 6)
    dummymp.run(myfunc, 4)
    dummymp.run(myfunc, 2)
    
    # Process queue and run the functions until all are finished.
    # Note that because of the nature of parallelism, the numbers
    # printed may be out of order.
    dummymp.process_until_done()
    
    # Display the returns!
    # Despite the functions finishing at different times, they will
    # return in the order called.
    # 
    # Output should be:
    #     Favorite numbers: {0: 6, 1: 4, 2: 2}
    print("Favorite numbers: %s" % str(dummymp.get_returns()))

For a more sophisticated example on how to use DummyMP, see example.py.
If you prefer a tutorial, continue on!

How to use?
------------
Note: Although the whole API is covered in this guide, it is not a
      substitute for the API documentation. The API documentation can
      be found in all of the source files, and is formatted to be
      accessible with pydoc.

### Step 0: Figure out your functions!
The first thing to do is to figure out what functions you can
parallelize. Functions that can run independently from other functions
are strong candidates for parallelization. Functions that depend on
each other's output (sequential functions) are weak candidates for
parallelization.

Functions that tend to be good candidates for parallelization include
functions that are repeated often, usually in a loop.

Note that it's not necessarily the function itself that defines if it
can be parallelized or not, but the use of the function.

Here are some examples to help explain what works and what doesn't.

This is an example of a function that works great with parallelization:

    def incr(n):
        return n + 1
    for x in range(0, 5):
        print incr(x)

This is an example of a function that will not work well with
parallelization:

    def incr(n):
        return n + 1
    x = 0
    while x < 10:
        x = incr(x)
        print(x)

Notice how the functions are exactly the same, but the usage of those
functions determined whether it could be parallelized or not.

The latter example requires the previous result before the next result
could be computed. Because of that, you must run the code in order, and
therefore you can not split the operation up.

With the former, however, you are simply incrementing each number in
the range. There is no dependency on any result, and therefore it's
possible to split the operation up.

If you have trouble visualizing this, try evaluating the statements.
Let's make the function more complicated so that we can't guess
anything.

    def setn(n):
        n = random.randint(0, n)
        return n

For the first example with the for loop, it will evaluate to:

    print setn(0)
    print setn(1)
    print setn(2)
    print setn(3)
    print setn(4)

For the second one, it's very different. We have no idea what the output
will be, and the input to the function depends on the output after the
intial run. Here's how it will look like:

    x = setn(0)
    print(x)
    x = setn(?)
    print(x)
    x = setn(?)
    print(x)
    ...

(? indicates unknown input.)

Furthermore, since we have a condition set on the output, we don't even
know when this loop will stop!

In the end, the general rule for determining parallelization:

  * If inputs (both variable and state, such as files) are known all of
    the time, it is likely to work with parallelization. (This includes
    pre-determined and static inputs.)
  * If there is any uncertainty for the inputs (variables depends on
    previous state, files need to be created, etc.), it is unlikely to
    work with parallelization.

To conclude, here are some applications that can be parallelized:

  * Some math algorithms (matrix multiplication of a large matrix, for
    instance)
  * Data file processing
  * Computing statistics from multiple data sets
  * Rendering multiple parts of a grid (such as map image rendering)
  * Bulk distributed database queries

Optimization
-------------
To speed things up, it's not a bad idea to get the CPU availability
at the very beginning so that DummyMP doesn't have to:

    # Get the CPU availability immediately
    dummymp.getCPUAvail()
    
    # Get processing! Processing/DummyMP code goes below...

If you have a lot of function calls, it may take a while before you
queue everything. Why not queue functions AND run them at the same time?

    # Loop
    for i in xrange(0, 1000):
        # Queue function with argument!
        dummymp.run(myfunc, i)
        
        # Process queue and handle processes!
        dummymp.process_process()
    
    # Now wait until everything's done!
    dummymp.process_until_done()
