## Введение
[cuda_intrro](https://enccs.github.io/cuda/1.01_GPUIntroduction/)
![](https://i.imgur.com/hxR5s99.png)
GPU не прощает халатного обращения с памятью

## API
[cuda api](https://enccs.github.io/cuda/2.01_DeviceQuery/)
```
__host__ ​__device__​ cudaError_t cudaGetDeviceCount(int* numDevices)
```
`__host__ ​__device__​` - означает, что может выполняться и на host, и на device
`__host__` - CPU
`__device__` - GPU
`cudaError_t` - возвращаемый тип
передаем указатель numDevices, который говорит, куда нам надо записать 
>[!info]
>Запуск на удаленно сервере
>1) git clone https://github.com/zhmurov/pcp-2024.git
>2) nvcc hello.cu (реализация ниже, просто выводит устройства)

```cpp
#include <stdio.h>

int main()
{
    int driverVersion = 0;
    cudaDriverGetVersion(&driverVersion);
    printf("CUDA driver: %d\n", driverVersion);

    int runtimeVersion = 0;
    cudaRuntimeGetVersion(&runtimeVersion);
    printf("CUDA runtime: %d\n", runtimeVersion);

    int         numDevices;
    cudaError_t stat = cudaGetDeviceCount(&numDevices);

    for (int i = 0; i < numDevices; i++)
    {
        cudaDeviceProp prop;
        stat = cudaGetDeviceProperties(&prop, i);

        printf("%d: %s, CC %d.%d, %d SMs running at %dMHz, %luMB\n", i, prop.name,
            prop.major, prop.minor,
            prop.multiProcessorCount,
            prop.clockRate/1000,
            prop.totalGlobalMem/1024/1024);
    }

    return 0;
}
```

[launch gpu](https://enccs.github.io/cuda/2.02_HelloGPU/)
```cpp
#include <stdio.h>

__global__ void gpu_kernel()
{
    int i = threadIdx.x + blockIdx.x*blockDim.x;
    printf("%d\n", i);
}

int main()
{
    gpu_kernel<<<4, 16>>>();
    cudaDeviceSynchronize();
}
~    
```
`__global__` - запуск на CPU, выполнение на GPU
gpu_kernel<<<numBlocks, threadsPerBlock>>>(..)
numBlocks - количество блоков
threadsPerBlock - количество потоков в блоке
## Memory
[education](https://enccs.github.io/cuda/2.03_VectorAdd/)
Чтобы не путать где храним переменные:
```cpp
float* h_x;
float* d_x;
```
h_ - переменная на памяти хоста
d_ - переменная на памяти девайса

Перенос кода с CPU на GPU:
Код на CPU
```cpp
#include <iostream>

int main()
{
    int N = 256;
    float* h_x = (float*)calloc(N, sizeof(float));
    float* h_y = (float*)calloc(N, sizeof(float));
    float* h_z = (float*)calloc(N, sizeof(float));
    
    for (int i = 0; i < N; i++)
    {
        h_x[i] = i*0.1;
        h_y[i] = i*0.01;
    }
    
    ///// HPC ZONE
    for (int i = 0; i < N; i++)
    {
	    h_z[i] = h_x[i] + h_y[i];
    }
    ///// HPC ZONE
    
    for(int i = 0; i < N; i ++)
    {
        std::cout << h_z[i] << std::endl;
    }
    free(h_x);
    free(h_y);
    free(h_z);
}
```

Код на GPU:
```cpp
#include <iostream>

__global__ void vec_add_kernel(float* d_x, float* d_y, float* d_z, int N)
{
    int i = threadIdx.x + blockIdx.x*blockDim.x;
    if (i < N)
    {
        d_z[i] = d_x[i] + d_y[i];
    }
}

int main()
{
    int N = 256;
    float* h_x = (float*)calloc(N, sizeof(float));
    float* h_y = (float*)calloc(N, sizeof(float));
    float* h_z = (float*)calloc(N, sizeof(float));

    float* d_x;
    float* d_y;
    float* d_z;

    cudaMalloc((void**)&d_x, N*sizeof(float));
    cudaMalloc((void**)&d_y, N*sizeof(float));
    cudaMalloc((void**)&d_z, N*sizeof(float));

    for (int i = 0; i < N; i++)
    {
        h_x[i] = i*0.1;
        h_y[i] = i*0.01;
    }

    cudaMemcpy(d_x, h_x, N*sizeof(float), cudaMemcpyHostToDevice);
    cudaMemcpy(d_y, h_y, N*sizeof(float), cudaMemcpyHostToDevice);

///// HPC ZONE
    vec_add_kernel<<<N/16 + 1, 16>>>(d_x, d_y, d_z, N);
///// HPC ZONE

    cudaMemcpy(h_z, d_z, N*sizeof(float), cudaMemcpyDeviceToHost);

    for(int i = 0; i < N; i ++)
    {
        std::cout << h_z[i] << std::endl;
    }

    cudaFree(d_x);
    cudaFree(d_y);
    cudaFree(d_z);

    free(h_x);
    free(h_y);
    free(h_z);
}

```

[Интересная физическая задача с применением cuda](https://enccs.github.io/cuda/2.04_HeatEquation/)
