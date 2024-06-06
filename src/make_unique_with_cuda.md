date: 2022-08-23 11:10:23

tags: cuda cpp

# make_unique_with_cuda

```c++
#include "stdio.h"
#include <memory>
namespace cuda
{
    template <typename T>
    [[nodiscard]] static auto malloc(std::size_t const size)
    {
        //nodiscard implies must use it return value, or will encounter an error
        static T *d{nullptr};
        cudaMalloc(&d, sizeof(T) * size);
        return d;
    }
    template <typename T>
    static void free(T *ptr)
    {
        if (ptr)
        {
            cudaFree(ptr);
            ptr=nullptr;
        }
    }
    template <typename T>
    [[nodiscard]] static auto makeUnique(std::size_t size)
    {
        return std::unique_ptr<T[], decltype(&free<T>)> { malloc<T>(size), free<T> };
    }
} // namespace name
__global__ void kernel(float3 *d)
{
    int id = threadIdx.x;
    // d[id].x=d[id].y=d[id].z=id*1.1;
    printf("%g\t%g\t%g\n", d[id].x, d[id].y, d[id].z);
}
void showh(float3 *d)
{
    for (int id = 0; id < 5; id++)
    {
        printf("%g\t%g\t%g\n", d[id].x, d[id].y, d[id].z);
    }
}
using namespace cuda;
int main(void)
{
    auto const count = 5;
    auto hp_points{std::make_unique<float3[]>(count)};
    for (int i = 0; i < count; i++)
    {
        hp_points[i].x=i*1.1;
        hp_points[i].y=i*1.1;
        hp_points[i].z=i*1.1;
    }
    // showh(hp_points.get());
    auto dp_points{cuda::makeUnique<float3>(count)};
    cudaMemcpy(dp_points.get(),hp_points.get(), //get() will return unique ptr's address
    sizeof(float3)*count,cudaMemcpyHostToDevice);
    kernel<<<1,5>>>(dp_points.get());
    cudaDeviceSynchronize();
    dp_points.~unique_ptr();
    return 0;
}
```