---
title: STM32CubeMX+VS Code+EIDE 环境配置
published: 2025-10-10 09:55:32
description: 在 VS Code 中通过 EIDE 插件与 STM32CubeMX 集成，快速搭建基于 CMake 的 STM32 编译开发环境
tags: [嵌入式开发]
category: 学习
draft: false
---

:::warning

前排提醒，本教程不涉及任何工具的下载安装，环境变量的配置，善用百度，这些工具包括但不限于 交叉编译工具 arm-none-eabi-gcc，芯片调试工具 OpenOCD 等，其实 EIDE 插件提供了这些工具的下载，当你使用的时候留意右下角进行 install 即可。

:::

# 1. STM32CubeMX 配置

在 `Project Manager -> Project` 设置中，将自动生成的工程模板切换为 CMake。

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010020102-1760061662200.png"/>

# 2. VS Code 配置

插件商店安装 EIDE 插件，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010020411-1760061851020.png"/>

新建一个空项目，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010021137-1760062297979.png"/>

由于所用芯片为 STM32F103C8T6，属于 Cortex-M 系列，因此在新建项目时选择 Cortex-M 作为项目类型，名称我这里使用 eide 作为名称，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010021325-1760062405278.png"/>

文件路径选择和 STM32CubeMX 中的 `Toolchain Folder Location` 一样的路径即可，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010021942-1760062782593.png"/>

最后在 STM32CubeMX 中生成的项目根文件夹目录应该是这样的，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010022136-1760062896420.png"/>

# 3. EIDE 插件配置

首先来配置一下编译过程中应该包含的头文件，链接的库以及宏定义，如何配置可以参考 `项目根文件夹->cmake->stm32cubemx->CMakeLists.txt`，这个 CMake 配置文件说明了应该怎么正确编译，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010022351-1760063031390.png"/>

其中主要关注这些，MX_Include_Dirs 的值就是应该包含的头文件路径，MX_Defines_Syms 的值就是宏定义，

```cmake
# STM32CubeMX generated symbols (macros)
set(MX_Defines_Syms 
	USE_HAL_DRIVER 
	STM32F103xB
    $<$<CONFIG:Debug>:DEBUG>
)

# STM32CubeMX generated include paths
set(MX_Include_Dirs
    ${CMAKE_CURRENT_SOURCE_DIR}/../../Core/Inc
    ${CMAKE_CURRENT_SOURCE_DIR}/../../Drivers/STM32F1xx_HAL_Driver/Inc
    ${CMAKE_CURRENT_SOURCE_DIR}/../../Drivers/STM32F1xx_HAL_Driver/Inc/Legacy
    ${CMAKE_CURRENT_SOURCE_DIR}/../../Drivers/CMSIS/Device/ST/STM32F1xx/Include
    ${CMAKE_CURRENT_SOURCE_DIR}/../../Drivers/CMSIS/Include
)
```

可以手动点击 + 添加，也可以直接 modify 这个配置文件添加，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010023129-1760063489627.png"/>

链接脚本文件修改如下，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010023352-1760063632518.png"/>

添加项目源文件，这里注意同时需要添加 .s 来负责硬件初始化，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010033631-1760067391622.png"/>

最后进行编译即可。

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010024545-1760064345455.png"/>

编译成功之后切换调试器为 OpenOCD，

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010024740-1760064460788.png"/>

这里的 Chip Config 和 Interface Config 根据使用的芯片以及下载器进行选择即可。

<img src="https://raw.githubusercontent.com/Tz-slayer/image-bed/master/markdown/20251010025317-1760064797226.png"/>