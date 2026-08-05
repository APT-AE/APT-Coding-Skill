# DMA 工作模式和reload说明

注意，该文档适用于芯片103x，110x和173x

# 1\.DMA工作模式

DMA工作模式分为连续服务模式（DMA\_CRx\[SMODE\] = 1）和单次服务模式（DMA\_CRx\[SMODE\] = 0）。

- 单次服务模式，是指DMA每收到一次操作请求，只进行一次原子传输。

- 连续服务模式，是指DMA每收到一次操作请求，会连续进行LTC\[11:0\]次原子传输。

其中，原子传输的大小由DMA\_CRx\[DSIZE\]和DMA\_CRx\[TSIZE\]共同决定，具体可参见103x DMA寄存器。



# 2\.Reload说明

DMA Reload功能，可以通过设置DMA\_CRx\[DSIZE\]=0开启。

- Relaod功能开启时，DMA数据传输结束之后，硬件会重新使能DMA通道（DMA\_MTRx\[CHEN\]=1）。

- Relaod功能关闭时，DMA数据传输结束之后，硬件会关闭DMA通道（DMA\_MTRx\[CHEN\]=0）。

DMA Reload功能，只能作用于源和目标地址相同，且传输数据长度相同的情况。



