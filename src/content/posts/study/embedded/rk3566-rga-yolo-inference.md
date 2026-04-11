---
title: 基于 RK3566 的 RGA 加速 YOLO 推理
published: 2025-12-13 22:15:20
description: 在 RK3566 开发板上利用 RGA 硬件加速器优化 YOLO 目标检测模型的推理性能，实现高效、低功耗的嵌入式 AI 应用
tags: [嵌入式开发]
category: 学习
draft: false
---

# 1. 前情提要

针对 8G 以上内存的开发板，如果按照官方给的 RGA 参考代码中的一些 resize cvtcolor 函数会报错。官方给出的答案是[申请4G以内的内存调用librga](https://github.com/airockchip/librga/blob/main/docs/Rockchip_FAQ_RGA_CN.md)。

# 2. 零拷贝 YOLO 推理

设备是 orangepi 3b 8G 的开发板，调用 usb 摄像头实现实时推理，因为希望使用到零拷贝，而使用 opencv 的话难免需要将数据拷贝到用户态进行处理，所以选择使用 RGA 加速图像处理，减少了 CPU 的占用。

1. 摄像头采集的数据直接写入连续的 DMA 缓冲区

核心代码如下，通过向视频设备驱动请求分配一个 DMA 缓冲区，摄像头采集的数据可以直接写入 DMA 缓冲区，实现高效的零拷贝数据传输，避免 CPU 拷贝开销。

```c++
struct v4l2_requestbuffers reqbuffer_dmabuf;
memset(&reqbuffer_dmabuf, 0, sizeof(reqbuffer_dmabuf));
reqbuffer_dmabuf.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;
reqbuffer_dmabuf.count = 4;
reqbuffer_dmabuf.memory = V4L2_MEMORY_DMABUF;

ret = ioctl(camera->fd, VIDIOC_REQBUFS, &reqbuffer_dmabuf);
if (ret == 0 && dmabuf_supported) {
    camera->use_dmabuf = true;
    // ... 继续DMABUF设置
} else {
    // 回退到MMAP模式
}
```

:::important
这里使用 dmabuf 的原因是 RGA 对 4g 以上内存的不兼容性，不然应该可以使用 mmap 。
:::

2. RGA 直接对这块 DMA 缓冲区的数据进行处理，同时将处理后的数据分配到一块新的 DMA 缓冲区

3. yolo 直接对新的 DMA 缓冲区上的数据进行处理

详细的代码实现参考：

::github{repo="Tz-slayer/rga_yolov11"}

使用 opencv 正常推理一次，完成整个流程，从摄像头采集数据到完成一次 yolo 推理到保存图像所耗费的时间：
<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251214111255-1765710775225.png"/>

使用 RGA 零拷贝所耗费的时间，由于最后有保存图像的操作，也就是最后图像数据还是要从内核态：

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251214111501-1765710901066.png"/>

总的时间虽然相差不大，但是实际执行颜色转换操作的时间是相差了 3 倍左右，并且使用 opencv 进行转换操作的时候，每次耗费的时间波动比较大，使用 RGA 相对稳定：

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251214111734-1765711054628.png"/>