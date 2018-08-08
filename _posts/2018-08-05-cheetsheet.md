---
title: Cheat Sheet - concurrent.futures and AsyncIO
layout: post
icon : fa-code 
---

## concurrent.futures, Pooling threads/processes

### Executer

Base methods are

- submit(function, args)    Returns a future object
- map(function, iterable, timeout, chunksize)   Returns generator of result values
- shutdown(wait)

Can be used with a context manager

### Thread Pool

`ThreadPoolExecuter(workers, thread_name_prefix, initializer, initargs)`

```python
#! usr/bin/python3

import time
import concurrent.futures

def callback_fn(future):
    print("From callback of future 3: ", future.result())

def my_func(value):
    time.sleep(value/10)
    return value ** 2
# Submit
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executer:
    future1 = executer.submit(my_func, 10)
    future2 = executer.submit(my_func, 20)
    future3 = executer.submit(my_func, 30)
    print("From future 1: ", future1.result())
    print("From future 2: ", future2.result())
    # Optional callback
    future3.add_done_callback(callback_fn)
# Map
list_of_vals = [1, 2, 3] * 3
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executer:
    results = executer.map(my_func, list_of_vals)
    results = list(result for result in results)
    print("Mapped results: ", results)
```

### Process Pool

`ProcessPoolExecutor(max_workers, mp_context, initializer, initargs)`

```python
#! usr/bin/python3

import concurrent.futures

def callback_fn(future):
    print("From callback of future 3: ", future.result())

def my_func(value):
    value = value ** 5
    value = value ** 4
    value = value ** 3
    value = value / 100000
    return value ** 2
# Submit
with concurrent.futures.ProcessPoolExecutor(max_workers=3) as executer:
    future1 = executer.submit(my_func, 10)
    future2 = executer.submit(my_func, 20)
    future3 = executer.submit(my_func, 30)
    print("From future 1: ", future1.result())
    print("From future 2: ", future2.result())
    # Optional callback
    future3.add_done_callback(callback_fn)
# Map
list_of_vals = [1, 2, 3] * 10
with concurrent.futures.ProcessPoolExecutor(max_workers=3) as executer:
    results = executer.map(my_func, list_of_vals)
    results = list(result for result in results)
    print("Mapped results: ", results)
```

### Future object

Represents a future operation, i.e. result set after calling it in the future.  
Methods available are

- cancel()
- cancelled()
- running()
- done()
- result(Timeout)
- exception(Timeout)
- add_done_callback(function)   function takes one argument, the future.
- set_result()
- set_exception()
- set_running_or_notify_cancel()

Executed by

- concurrent.futures.as_completed(iterable_of_futures, timeout), returns a tuple (completed_futures, not_completed_futures)
- concurrent.futures.wait(iterable_of_futures, timeout, return_when), returns a tuple (completed_futures, not_completed_futures), here `return_when` can be
  - FIRST_COMPLETED
  - FIRST_EXCEPTION
  - ALL_COMPLETED

```python
#! usr/bin/python3

import time
import concurrent.futures

def my_func(value):
    time.sleep(value/10)
    return value ** 2
# With future module functions
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executer:
    future1 = executer.submit(my_func, 10)
    future2 = executer.submit(my_func, 20)
    future3 = executer.submit(my_func, 30)
    completed, not_completed = concurrent.futures.wait([future1, future2, future3])
    for result in completed:
        print("From futures: ", result.result())
```

## AsyncIO

### Event loop

`asyncio.get_event_loop` returns the event loop

Methods are  
Control and status

- run_forever(),then manually call stop() and close()
- run_until_complete(future)
- is_running()
- stop()
- is_closed()
- close()
- time()

Run coroutines

- call_soon(coroutine, arguments, context)
- call_soon_threadsafe(coroutine, arguments, context)
- call_later(delay, coroutine, arguments, context)  delay is an `int` or `float`
- call_at(when, coroutine, arguments, context)  when is an `int` or `float`

### Coroutines

A function that can transfer control while it waits for another operation.

- A coroutine function is created by `async def`. Checked by `iscoroutinefunction()`
- Coroutine function returns a Coroutine object on execution. Check by `iscoroutine()`
- Can transfer control by `await <command>` to another coroutine.
- Can `return` values to caller, either loop or another coroutine.
- Can `raise` exceptions to caller, either loop or another coroutine.

Running a coroutine

- asyncio.run(coroutine, debug), takes care of the event loop by creating and ending a new loop.

### Future objects

- Similar to `concurrent.futures.Future`. Represents a computation that is done at a future time.
- Can created by `asyncio.Future(loop)`.
- Can be added to a coroutine by

```python
'''
This stores the result of a coroutine in a future object.
By using a future we can await the result of the future or perform any other operations on the future.
'''
import asyncio
# The coroutine taking the future as argument
async def my_coroutine(future):
    await asyncio.sleep(1)
    future.set_result('Future is done!')
# The event loop
loop = asyncio.get_event_loop()
# The future object
future = asyncio.Future()
# Schedule execution of future object at a future time
asyncio.ensure_future(my_coroutine(future))
# Run until the future is completed
loop.run_until_complete(future)
# Close the loop
loop.close()
```

Available methods

- cancel()
- cancelled()
- done()
- result()
- exception()
- add_done_callback()
- remove_done_callback()
- set_result()
- set_exception()
- get_loop()

### Tasks

- A coroutine is converted into a Task object which is then scheduled in an event loop.
- Created by `loop.create_task(coroutine)` or `asyncio.Task(coroutine, loop)`.

Methods available are

- all_tasks(loop), same as `asyncio.all_tasks(loop)`
- current_task(loop), same as `asyncio.current_task(loop)`
- cancel()
- get_stack(limit)
- print_stack(limit, file_name)

Executing tasks

- asyncio.as_completed(iterable_of_futures), returns a tuple (completed_futures, not_completed_futures)
- asyncio.gather(coroutines_or_futures, loop, return_exceptions)
- asyncio.run_coroutine_threadsafe(coroutine, loop)
- asyncio.wait(iterable_of_futures, loop, timeout, return_when)
- asyncio.wait_for(future, timeout), here `return_when` can be
  - FIRST_COMPLETED
  - FIRST_EXCEPTION
  - ALL_COMPLETED

Creation and checks

- asyncio.ensure_future(coroutine_or_future, loop)
- asyncio.wrap_future(concurrent.future.Future(), loop)
- asyncio.iscoroutine(object)
- asyncio.iscoroutinefunction(object)

Control and status

- asyncio.current_task(loop)
- asyncio.all_tasks(loop)
- asyncio.sleep(delay, result, loop)
- asyncio.shield(argument_to_await, loop)
