---
title: Target = GPU - Numba
layout: post
icon : fa-code 
---

Apart from compiling the code to run efficiently on the CPU, [Numba](http://numba.pydata.org/) can also compile python code so that it is executed on the GPU. Just as with the just-in-time compilation API, the CUDA interface is also straightforward. Incredible work has been done to make abstractions for commonly used tasks making the user code compact and effective.

## Writing a Kernel  

Writing a CUDA kernel using Numba is as simple as adding a decorator, `numba.cuda.jit` to a compatible function. Apart from that the indices of the threads inside the blocks and the blocks inside the threads has to be calculated to access the appropriate element of the array.

```python
@cuda.jit
def doubled(arr):
    x_idx = cuda.threadIdx.x
    y_idx = cuda.threadIdx.y
    arr[x_idx, y_idx] *= 2
```

A CUDA 'jitted' function cannot return a value rather it writes the result to an array. Now this function can be called after setting the block and grid sizes

```python
arr = np.random.randn(3, 3).astype(np.float32)
doubled[1, (3, 3)](arr)
```

Here the block size is `1` and thread dimension is `(3, 3)` defined as `<function_name>[<blocks_per_grid>, <threads_per_grid>]`.

## Auto Calculation of Indices  

Numba also provides `numba.cuda.grid()` function to automatically calculate the current index of the thread inside the block. Using this we can rewrite the previous `doubled` function as

```python
@cuda.jit
def doubled_auto_grid(arr):
    x_idx, y_idx = cuda.grid(2)
    arr[x_idx, y_idx] *= 2
```

Now the `x_idx` and `y_idx` is calculated by Numba for each of the threads. It can be called same as the previous function.

```python
arr = np.random.randn(3, 3).astype(np.float32)
doubled_auto_grid[1, (3, 3)](arr)
```

## Manual Transfer of Objects  

Although Numba automatically copies all the argument objects to and from the device, we can transfer memory explicitly to save overhead if read-only arrays are involved. The function definition is the same

```python
@cuda.jit
def doubled_man_copy(arr, out):
    x_idx, y_idx = cuda.grid(2)
    out[x_idx, y_idx] = arr[x_idx, y_idx] * 2
```

We introduce an argument `out` that will contain the output. The `arr` here is read only.

```python
arr = np.random.randn(3, 3).astype(np.float32)
out = cuda.device_array_like(arr)
arr_gpu = cuda.to_device(arr)
doubled_man_copy[1, (3, 3)](arr_gpu, out)
out = out.copy_to_host()
```

We create a global memory object with the `cuda.device_array_like` which takes the properties of `arr`. After that the `arr` is transferred to the GPU manually with `cuda.to_device()`. After the execution of the kernel we only transfer `out` back to the host with `out.copy_to_host()`, the `arr_gpu` is not transferred to the host.

## Array Reduction  

We can perform reduction operations on arrays using the GPU for calculations with Numba. We use the `cuda.reduce` decorator on a function that takes two arguments only, which represent two consecutive elements of the array.

```python
@cuda.reduce
def reduce_gpu(a, b):
    return a + b
```

One thing to note here is that `cuda.reduce` only takes one-dimensional array. So we need to manually flatten the input array if it has more than one dimensions.

```python
arr = np.random.randn(3, 3).astype(np.float32)
flattened_arr = arr.flatten()
reduced_arr = reduce_gpu(flattened_arr)
```

We do not pass elements as arguments to the function call rather the input array.

## Vectorized UFuncs on GPU  

The `vectorize` decorator can me targeted for the GPU by passing `target="cuda"` to the decorator.

```python
@vectorize(
    [float32(float32), ],
    target="cuda"
)
def doubled_ufunc(arr):
    arr *= 2
    return arr
```

Now this will be executed on the GPU. The call to it is the same as for the CPU.

```python
arr = np.random.randn(3, 3).astype(np.float32)
out = doubled_ufunc(arr)
```

This uFunc can return values, memory transfer is done by Numba.

## Generalized UFuncs on GPU  

Same as with uFuncs, generalized uFuncs can also be executed on the GPU by using `target="cuda"`.

```python
@guvectorize(
    [(float32[:, :], float32[:, :])],
    "(m,n)->(m,n)",
    target="cuda"
)
def doubled_gufunc(arr, out):
    for i in range(arr.shape[0]):
        for j in range(arr.shape[1]):
            out[i, j] = arr[i, j] * 2
```

Again, it is called same as for the CPU.

```python
arr = np.random.randn(3, 3).astype(np.float32)
out = cuda.device_array_like(arr)
doubled_gufunc(arr, out)
out = out.copy_to_host()
```