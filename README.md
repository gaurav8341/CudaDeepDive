# Cuda 



## Cuda func call

These run parallely and asynchronously, each thread runs independantly in **thread**.

Also called as cuda kernels.

```c
func<<<num_blocks, block_size>>>(*args);
```

## Cuda Threads

- `num_blocks`:

These threads can be grouped together in one block. How many blocks to run in one single func call, is defines in fucn call along with block size.

- `block_size`:

The block size means how many threads will run in each thread.

The memory management is also affected by these thread hierarchy

There can be only 1024 threads in one block.

## Cuda Provided Thread Hierarachy Variables

- `gridDim.x`: number of blocks in the cuda kernel call

- `blockIdx.x`: The index of current block within the grid

- `blockDim.x`: The number of thread in the block, block size

- `threadIdx.x`: Index of the thread within the block.

## Coordinating Parallel Threads

Using above cuda threads variable, we can use this to access any perticular memory.

Be mindfull of the value calculated otherwise youll corrupt the data of other processes. Make sure calculated address is within the bound.

`data_index = threadIdx.x + blockIdx.x * blockDim.x`

The above case is for when you only require single grid for whatever data operation. What if your data is more larger than even your grid.

```cpp
__global__ void kernel(int *a, int N)
{
  int indexWithinTheGrid = threadIdx.x + blockIdx.x * blockDim.x;
  int gridStride = gridDim.x * blockDim.x;

  for (int i = indexWithinTheGrid; i < N; i += gridStride)
  {
    // do work on a[i];
  }
}
```

we jump by the `blockDim.x` elements, this way we can process data larger than the single grid can process.

## Accessing memory in Host and in GPU

We use `malloc` and `free` to allocate and free the memory in cpu programming. 

We can use `cudaMallocManaged` and `cudaFree` calls to manage the memory in both host and GPU.

- CPU Memory Mgmt:

```cpp
// CPU-only

int N = 2<<20;
size_t size = N * sizeof(int);

int *a;
a = (int *)malloc(size);

// Use `a` in CPU-only program.

free(a);
```

- GPU Cuda Memory Mgmt


```cpp
// Accelerated

int N = 2<<20;
size_t size = N * sizeof(int);

int *a;
// Note the address of `a` is passed as first argument.
cudaMallocManaged(&a, size);

// Use `a` on the CPU and/or on any GPU in the accelerated system.

cudaFree(a);
```
## Error Handling in Cuda

In cuda, the errors are handled using a type `cudaError_t`. These are very essential for any kind of memory operations.

## Streaming Multiprocessors.

Gpus have certain number of streaming multiprocessors depending on type of GPU. When the blocks of threads are scehduled to run on SM depening on the kernel configuration ie block size and number of blocks. One block will be on single SM. 

All the threads in block will run in single SM. 

SM Optimization can be done, by configuring the number of blocks to match the number of SM in GPU.

SM is larger block of cuda cores, so all the threads in the block will run parallely, depending on how many cuda cores are there in singls SM
This architecture is called SIMT(Single Instruction Multiple Threads) All cores in SM run same instruction but on different data. We can have same PC, stack pointer for all threads, in this architecture.(Till Volta generation this was the case after that each thread is managed independantly in nvidia.

## Unified Memory

Like in the CPU, in GPU as well the memory behaviour is same. We initialize and allocate the memory using `cudaMallocManaged`, but the data may not be present in the memory. 

In above cases, the page fault may occur, which brings that data in to the memory. 

To avoid the pag faults we can use `cudaMemPrefetchAsync`. This operation is async and requires the argument stating where the data should be transferred.

