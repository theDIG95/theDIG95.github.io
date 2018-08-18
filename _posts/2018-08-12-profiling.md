---
title: Profiling the performance - Python
layout: post
icon : fa-code 
---

To make a program efficient we must make it utilize the available resources efficiently. In order to do that we must first identify the portions of the code that are inefficient. Efficient of the code is with respect to the resources it uses with the most obvious ones being the CPU and the Memory usage. However the program may also be using other resources such as the network or the permanent storage devices or even sensors in an embedded application. Here we will look how to identify the CPU and memory usage of a python program.

## The example problem  

For the example problem we will use an implementation of the [Julia Set](https://en.wikipedia.org/wiki/Julia_set). Here we do not worry about if it it is the fastest implementation because we want to see how fast it runs.  
We define the code to profile as  

```python
#! usr/bin/python3

def create_inputs():
    # Set bounds
    x_start, x_end = -1.8, 1.8
    y_start, y_end = -1.8, 1.8
    # Set width of data
    width = 100
    # Empty holder for values
    x_vals = list()
    y_vals = list()
    # Calculate difference between points
    x_step = (x_end - x_start) / width
    y_step = (y_end - y_start) / width
    # Calculate values
    x_val = x_start
    while x_val <= x_end:
        x_vals.append(x_val)
        x_val += x_step
    y_val = y_start
    while y_val <= y_end:
        y_vals.append(y_val)
        y_val += y_step
    return x_vals, y_vals


def jset():
    # Get input values
    x_vals, y_vals = create_inputs()
    # Set parameters
    threshold = 2
    max_iters = 254
    x_center = -0.62772
    y_center = -0.42193
    # Empty holder for output
    output_vals = list()
    # Calculate output for each point
    for x_val in x_vals:
        for y_val in y_vals:
            output_vals.append(
                calc_point_val(x_val, y_val, threshold, max_iters, x_center, y_center))

def calc_point_val(x_val, y_val, thresh, max_iters, x_center, y_center):
    # Iteration number
    no_iter = 0
    # Start iterations
    while True:
        # Calculate point value
        val = ((x_val ** 2 + y_val ** 2) ** (1/2))
        # Check threshold
        if val < thresh:
            no_iter += 1
            val = val + ((x_center ** 2 + y_center ** 2) ** (1/2))
        # Break if greater than threshold
        else:
            break
        # Break if iterations complete
        if no_iter >= max_iters:
            break
    return no_iter

def main():
    # Call main function
    jset()

if __name__ == '__main__':
    main()
```

As you can see here it does not use any external libraries such as numpy or PIL to accelerate the code as we are just using it as an example for profiling.

## When and Why  

Profiling is usually done after an implementation of the code or at least a module or sub-module is complete because premature optimization can lead to complexities. That does not mean we do not design the code to be optimized when we begin writing, rather we do not go into the complexities such as comparing the performance of different approaches or libraries at the initial stage. The first objective is to get the code to work, then we get the code to work fast.  
Also it is highly beneficial to unit test the code before working on optimization as we want to retain the functionality of the code. Unit tests become even more helpful when the code base is large and we might break the code during optimizations.  
We profile a portion of the code at a time, usually the one which we think might be causing slow down. Then with profiling we can narrow down to exact portion of the code which slows it down and then focus on it.

## Before we profile  

A couple of things to keep in mind while profiling are

- Reduce the number of background applications to get more accurate results.
- If possible drop down to run level 1 on unix systems to decrease background apps.
- Profile multiple times and use the minimum number as this is the lowest the program can go.
- Disable automatic system throttling such as Intel's TurboBoost.

## Finding the overall runtime  

The most simple method of calculating the runtime of a piece of code is to use the `time` module of python. using this we can create time stamps before and after the code block and use the difference to find the total runtime.

```python
import time

start_time = time.time()
# CODE
# GOES
# HERE
end_time = time.time()
print("Total execution time is: ", end_time - start_time)
```

This gives us the wall clock time of execution of the code.  
Besides this we can use `%time` of the `IPython` terminal to measure the execution time.

```python
In [1]: import jset_profile

In [2]: %time jset_profile.main()
CPU times: user 1.41 s, sys: 0 ns, total: 1.41 s
Wall time: 1.41 s
```

This works the same as manually taking time stamps. Also, we can use `%timeit` to measure the time for a number of loops.

```python
In [1]: import jset_profile

In [2]: %timeit jset_profile.main()
1.45 s ± 56 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

This appropriates choses the number of times to run the code, but we can manually specify the execution loop number by using `-n` and `-r` flag.

## The cProfile module  

Python's standard library provides a profiling tool called [cProfile and profile](https://docs.python.org/3/library/profile.html) both of which provide the same functionality, but as the `cProfile` is a C extension the overhead is reduced and the results are returned faster.  
It can be used as a command line tool

```bash
$ python -m cProfile jset_profile.py 
        20206 function calls in 1.441 seconds

    Ordered by: standard name

    ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.005    0.005    1.441    1.441 jset_profile.py:27(jset)
        1    0.000    0.000    1.441    1.441 jset_profile.py:3(<module>)
        1    0.000    0.000    0.000    0.000 jset_profile.py:3(create_inputs)
    10000    1.435    0.000    1.435    0.000 jset_profile.py:43(calc_point_val)
        1    0.000    0.000    1.441    1.441 jset_profile.py:62(main)
        1    0.000    0.000    1.441    1.441 {built-in method builtins.exec}
    10200    0.001    0.000    0.001    0.000 {method 'append' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

Here it sidplays the total number of calls it made to methods or functions along with their execution times. As we can see here the `jset_profile.py:43(calc_point_val)` i.e. the `calc_point_val` function in the file is called 10000 times with time of 0 seconds per call and total execution time of 1.435 seconds. This is the function that takes up most of the time and indicates that this particular method needs attention.

## cProfile output to file for more analysis  

We can also save the output to the file via the `-o <file_name>` flag. This helps us keep record and also for more in-depth analysis using the [pstats](https://docs.python.org/3/library/profile.html#module-pstats) module included in the standard library. This file is not human readabl so we need the `pstats` module.

```python
In [1]: import pstats

In [2]: jstats = pstats.Stats("jstats.stats")

In [3]: jstats.strip_dirs()
Out[3]: <pstats.Stats at 0x7fcfede36978>

In [4]: jstats.sort_stats("calls")
Out[4]: <pstats.Stats at 0x7fcfede36978>

In [5]: jstats.print_stats()
Sat Aug 18 20:02:06 2018    jstats.stats

        20206 function calls in 1.425 seconds

    Ordered by: call count

    ncalls  tottime  percall  cumtime  percall filename:lineno(function)
    10200    0.001    0.000    0.001    0.000 {method 'append' of 'list' objects}
    10000    1.419    0.000    1.419    0.000 jset_profile.py:43(calc_point_val)
        1    0.000    0.000    1.425    1.425 {built-in method builtins.exec}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        1    0.000    0.000    0.000    0.000 jset_profile.py:3(create_inputs)
        1    0.005    0.005    1.425    1.425 jset_profile.py:27(jset)
        1    0.000    0.000    1.425    1.425 jset_profile.py:3(<module>)
        1    0.000    0.000    1.425    1.425 jset_profile.py:62(main)
```

To visualize this data we can use the [SnakeViz](https://jiffyclub.github.io/snakeviz/) library. It used the output file and then displays a GUI in the browser through which we can analyze the call stack.
![SnakeViz example](https://jiffyclub.github.io/snakeviz/img/func_info.png)

## Line by line execution time  

If we need even more in-depth execution time of the program we can use the [line_profiler](https://github.com/rkern/line_profiler) library. This outputs information for each line of the function or method we want to profile. We have to specify the method we have to profile using the `@profile` decorator. We decorate the `calc_point_val` method as we identified it to be using the most time. The decorator doesn't need to be defined or imported.

```python
@profile
def calc_point_val(x_val, y_val, thresh, max_iters, x_center, y_center):
```

Then we use it as a command line program using `kernprof`.

```bash
$ kernprof -l -v jset_profile.py
Wrote profile results to jset_profile.py.lprof
Timer unit: 1e-06 s

Total time: 7.8453 s
File: jset_profile.py
Function: calc_point_val at line 42

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    42                                           @profile
    43                                           def calc_point_val(x_val, y_val, thresh, max_iters, x_center, y_center):
    44                                               # Iteration number
    45     10000       5700.0      0.6      0.1      no_iter = 0
    46                                               # Start iterations
    47     10000       5026.0      0.5      0.1      while True:
    48                                                   # Calculate point value
    49   2279663    1954604.0      0.9     24.9          val = ((x_val ** 2 + y_val ** 2) ** (1/2))
    50                                                   # Check threshold
    51   2279663    1307634.0      0.6     16.7          if val < thresh:
    52   2278634    1269174.0      0.6     16.2              no_iter += 1
    53   2278634    2075002.0      0.9     26.4              val = val + ((x_center ** 2 + y_center ** 2) ** (1/2))
    54                                                   # Break if greater than threshold
    55                                                   else:
    56      1029        536.0      0.5      0.0              break
    57                                                   # Break if iterations complete
    58   2278634    1216777.0      0.5     15.5          if no_iter >= max_iters:
    59      8971       5331.0      0.6      0.1              break
    60     10000       5513.0      0.6      0.1      return no_iter

```

This gives us the line-by-line execution times. An important thing to note here is the `Timer unit: 1e-06 s`, as the times in the columns are not in seconds.  
Here we can see that the calculation on line 49 and 53 take a lot of time which can be made faster.

## Finding the memory usage  

After we have made the application run faster we can also profile it to save on the memory consumption too. For this we can use [memory_profiler](https://pypi.org/project/memory_profiler/). This will give us the memory incremeent after each line of code. This can help us see if memory consumption is reduced after we optimize our code.  
As with `line_profiler` we need to add a decorator `@profile` to the method or function to profile. The decorator doesn't need to be defined or imported.

```python
@profile
def jset():
```

Now we can view the memory changes by using the module.

```bash
$ python -m memory_profiler jset_profile.py
Filename: jset_profile.py

Line #    Mem usage    Increment   Line Contents
================================================
    26   31.461 MiB   31.461 MiB   @profile
    27                             def jset():
    28                                 # Get input values
    29   31.461 MiB    0.000 MiB       x_vals, y_vals = create_inputs()
    30                                 # Set parameters
    31   31.461 MiB    0.000 MiB       threshold = 2
    32   31.461 MiB    0.000 MiB       max_iters = 254
    33   31.461 MiB    0.000 MiB       x_center = -0.62772
    34   31.461 MiB    0.000 MiB       y_center = -0.42193
    35                                 # Empty holder for output
    36   31.461 MiB    0.000 MiB       output_vals = list()
    37                                 # Calculate output for each point
    38   31.461 MiB    0.000 MiB       for x_val in x_vals:
    39   31.461 MiB    0.000 MiB           for y_val in y_vals:
    40   31.461 MiB    0.000 MiB               output_vals.append(
    41   31.461 MiB    0.000 MiB                   calc_point_val(x_val, y_val, threshold, max_iters, x_center, y_center))
```

As you can see here the memory increment is zero megabytes so we can safely assume that memory consumption is low byt that is dependent on the operations and all the variables we are storing in it.  
We can also create a live graph of the memory consumption of the process by using `mprof` provided by the same library.

```bash
$ mprof run jset_profile.py
mprof: Sampling memory every 0.1s
running as a Python program.
$ mprof plot
Using last profile data.
$ mprof clean
```

This will first run the code and then plot the results using matplotlib.

## Viewing objects on the heap  

Sometimes it is also valuable to see the objects in the memory to try and find if there are any unintended objects that are causing a memory leak. We can use [Python Object Graphs](https://mg.pov.lt/objgraph/) which give the ability to inspect objects.  
We can do it in an interactive console or in script. However I've found that the IPython console doesn't give such accurate results because it has activitu going on in the background.  
To do it in script we import the library and then view growth before and after the function call.  

```python
import objgraph

# The main function
def main():
    # Call main function
    print("========Before=======")
    objgraph.show_growth()
    jset()
    print("=========After=========")
    objgraph.show_growth()
```

It gives the following output.  

```bash
========Before=======
function                       2814     +2814
dict                           1589     +1589
tuple                          1305     +1305
wrapper_descriptor             1131     +1131
weakref                        1030     +1030
method_descriptor               878      +878
builtin_function_or_method      811      +811
getset_descriptor               585      +585
set                             471      +471
list                            400      +400
=========After=========
```

As you can see the growth after the function call is not present as the function doesn't return anything. The growth before is due to the import of the library itself.