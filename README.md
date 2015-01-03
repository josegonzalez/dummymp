DummyMP - A multiprocessing library for dummies!
=================================================

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
  * [psutil 2.x.x][1]
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
If you prefer a tutorial, continue on to the documentation!

Documentation
--------------
See [docs/README.md][2] for a more thorough guide on how to use DummyMP.

Online HTML documentation can be found [here][3]

API documentation can be found in all of the DummyMP source files, and
is formatted to be accessible with pydoc.

Bugs/Issues
------------
DummyMP is currently in beta, so there's bound to be a few bugs. Please
report them to our [issues tracker][4].

License
--------
    # DummyMP - Multiprocessing Library for Dummies!
    # Copyright 2014 Albert Huang.
    # 
    # Licensed under the Apache License, Version 2.0 (the "License");
    # you may not use this file except in compliance with the License.
    # You may obtain a copy of the License at
    # 
    #   http://www.apache.org/licenses/LICENSE-2.0
    # 
    # Unless required by applicable law or agreed to in writing, software
    # distributed under the License is distributed on an "AS IS" BASIS,
    # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
    # implied. See the License for the specific language governing
    # permissions and limitations under the License.
    # 

[1]: https://github.com/giampaolo/psutil
[2]: docs/README.md
[3]: https://pythonhosted.org/dummymp/
[4]: https://github.com/alberthdev/dummymp/issues
