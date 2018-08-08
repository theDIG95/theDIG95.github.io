---
title: Executer pools - concurrent.futures
layout: post
icon : fa-code 
---

A pool of resources is a reusable set of resources that can perform some function multiple times on new data without having to be initialized and cleaned up after each run. By using such a reusable method a lot of overhead is reduced that comes with spawning and cleaning up resources. It is suitable when the same or even different operation has to be done on large amount of data and the existing resources are re allocated to the next input.

## The Executer  

The executer represents an abstract class of a pool of resources which can contain any number of workers. It will divide the input data among the available workers and reuse the free workers as they finish processing a piece of the input data.  
As this is an abstract class it is not used directly rather it's solid base classes are used , which are

- `import concurrent.futures.ThreadPoolExecutor` which uses threads as workers, suitable with I/O bound operations.
- `import concurrent.futures.ProcessPoolExecutor` which uses processes as workers, suitable with CPU bound operations.

The available methods of the abstract class and it's subclasses are.

- `submit(function, args)` returns a `Future` object discussed later. Submits a function to be executed in the pool.
- `map(function, iterable, timeout, chunksize)` returns generator over the result values.
- `shutdown(wait)` free the workers after the `wait` time delay

## The Future object  

The `Future` object represents a result or computation that will be done at a later time i.e. in the future. By using this object we can start the processes concurrently without having to wait for the result of each input. Then the future object can be checked for the availability of the result and executed after any other operations have been done without spending time waiting for the result.  
One important thing to note is that the `AsyncIO` module of python also has futures, they are mostly the same but with the important distinction that the ones here in `concurrent.futures` package are blocking.  
The `Future` is created by submitting a callable or function to the executer via `Executer.submit()` method. Not the result or the exception can be gotten by these methods.

- `result(timeout)` will block until the result or timeout.
- `exception(timeout)` will wait for the raised exception or till the timeout.

## Examples with the ThreadPoolExecuter  

For this example we use a function that spends some time waiting and then returns a result with small amount of computation.

```python
def my_func(value):
    time.sleep(value/10)
    return value + 2
```

Now we will manually create inputs for the executor's workers. The executer is used as a context manager to take care of the creation and cleanup of the workers.

```python
# Submit
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executer:
    future1 = executer.submit(my_func, 10)
    future2 = executer.submit(my_func, 20)
    future3 = executer.submit(my_func, 30)
    print("From future 1: ", future1.result())
    print("From future 2: ", future2.result())
    # Optional callback
    future3.add_done_callback(callback_fn)
```

Here we have added an optional callback to the third future to perform a different operation on it's result. The callback is defined as

```python
def callback_fn(future):
    print("From callback of future 3: ", future.result())
```

Also, as in this case the function on each of the input is the same so we can use `map` to easily pass the inputs to the workers and retrieve the values as a single list.

```python
# Map
list_of_vals = [1, 2, 3] * 3
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executer:
    results = executer.map(my_func, list_of_vals)
    results = list(result for result in results)
    print("Mapped results: ", results)
```

As these futures are blocking we can wait for the completion of all the results or process them one by one. Here we wait for the completion of the futures and retrieve the results.

```python
# With future module functions
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executer:
    future1 = executer.submit(my_func, 10)
    future2 = executer.submit(my_func, 20)
    future3 = executer.submit(my_func, 30)
    completed, not_completed = concurrent.futures.wait([future1, future2, future3])
    for result in completed:
        print("From futures: ", result.result())
```

## Examples with the ProcessPoolExecuter  

All the same methods can be applied to the `ProcessPoolExecuter` by just changing the name to Process instead of Thread. But we use a more CPU bound unction for this example

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
# With future module functions
with concurrent.futures.ProcessPoolExecutor(max_workers=3) as executer:
    future1 = executer.submit(my_func, 10)
    future2 = executer.submit(my_func, 20)
    future3 = executer.submit(my_func, 30)
    completed, not_completed = concurrent.futures.wait([future1, future2, future3])
    for result in completed:
        print("From futures: ", result.result())
```
