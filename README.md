# 使用Linux内核和树莓派学习操作系统开发

这个仓库包含一步步的指南，教你如何从零创造一个简单的操作系统(OS)内核。我把这个OS叫做Raspberry Pi OS 或者只是 RPI OS。大量的 RPi OS 源码基于 [Linux kernel](https://github.com/torvalds/linux), 但是 OS 的功能非常有限且只支持 [Raspberry PI 3](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/). 

每个课程按照这样的方式来设计，它首先解释一些内核功能在RPi OS中是如何被实现的，然后它尝试证明在Linux内核中同样的功能是如何工作的。每节课在 [src](https://github.com/s-matyukevich/raspberry-pi-os/tree/master/src) 目录中有相关的文件夹，which contains a snapshot of the OS source code at the time when the lesson had just been completed. This allows the introduction of new concepts gracefully and helps readers to follow the evolution of the RPi OS. 理解这个指南不要求任何特定的OS开发技能。

关于这个项目的目的和历史的更多信息，请阅读 [导言](docs/Introduction.md). 项目仍然在开发中，如果你想要参与 - 请阅读 [投稿指南](docs/Contributions.md).

<p>
  <a href="https://twitter.com/RPi_OS" target="_blank">
    <img src="https://raw.githubusercontent.com/s-matyukevich/raspberry-pi-os/master/images/twitter.png" alt="Follow @RPi_OS on twitter" height="34" >
  </a>

  <a href="https://www.facebook.com/groups/251043708976964/" target="_blank">
    <img src="https://raw.githubusercontent.com/s-matyukevich/raspberry-pi-os/master/images/facebook.png" alt="Follow Raspberry Pi OS on facebook" height="34" >
  </a>

  <a href="https://join.slack.com/t/rpi-os/shared_invite/enQtNDQ1NTg2ODc1MDEwLWVjMTZlZmMyZDE4OGEyYmMzNTY1YjljZjU5YWI1NDllOWEwMjI5YzVkM2RiMzliYjEzN2RlYmUzNzBiYmQyMjY" target="_blank">
    <img src="https://raw.githubusercontent.com/s-matyukevich/raspberry-pi-os/master/images/slack.png" alt="Join Raspberry Pi OS in slack" height="34" >
  </a>

  <a href="https://www.producthunt.com/upcoming/raspberry-pi-os" target="_blank">
    <img src="https://raw.githubusercontent.com/s-matyukevich/raspberry-pi-os/master/images/subscribe.png" alt="Subscribe for updates" height="34" >
  </a>
</p>

## Table of Contents

* **[导言](docs/Introduction.md)**
* **[投稿指南](docs/Contributions.md)**
* **[预备课](docs/Prerequisites.md)**
* **课程 1: 内核初始化** 
  * 1.1 [Introducing RPi OS, or bare metal "Hello, world!"](docs/lesson01/rpi-os.md)
  * Linux
    * 1.2 [项目结构](docs/lesson01/linux/project-structure.md)
    * 1.3 [内核编译系统](docs/lesson01/linux/build-system.md) 
    * 1.4 [启动序列](docs/lesson01/linux/kernel-startup.md)
  * 1.5 [练习](docs/lesson01/exercises.md)
* **课程 2: 处理器初始化**
  * 2.1 [RPi OS](docs/lesson02/rpi-os.md)
  * 2.2 [Linux](docs/lesson02/linux.md)
  * 2.3 [练习](docs/lesson02/exercises.md)
* **课程 3: 中断处理**
  * 3.1 [RPi OS](docs/lesson03/rpi-os.md)
  * Linux
    * 3.2 [底层异常处理](docs/lesson03/linux/low_level-exception_handling.md) 
    * 3.3 [中断控制器](docs/lesson03/linux/interrupt_controllers.md)
    * 3.4 [计时器](docs/lesson03/linux/timer.md)
  * 3.5 [练习](docs/lesson03/exercises.md)
* **课程 4: 进程调度器**
  * 4.1 [RPi OS](docs/lesson04/rpi-os.md) 
  * Linux
    * 4.2 [调度器基本结构](docs/lesson04/linux/basic_structures.md)
    * 4.3 [Forking一个任务](docs/lesson04/linux/fork.md)
    * 4.4 [调度器](docs/lesson04/linux/scheduler.md)
  * 4.5 [练习](docs/lesson04/exercises.md)
* **课程 5: 用户进程和系统调用** 
  * 5.1 [RPi OS](docs/lesson05/rpi-os.md) 
  * 5.2 [Linux](docs/lesson05/linux.md)
  * 5.3 [练习](docs/lesson05/exercises.md)
* **课程 6: 虚拟内存管理**
  * 6.1 [RPi OS](docs/lesson06/rpi-os.md) 
  * 6.2 Linux (In progress)
  * 6.3 [Exercises](docs/lesson06/exercises.md)
* **课程 7: 信号和中断等待** (To be done)
* **课程 8: 文件系统** (To be done)
* **课程 9: 可执行文件 (ELF)** (To be done)
* **课程 10: 驱动** (To be done)
* **课程 11: 网络** (To be done)

