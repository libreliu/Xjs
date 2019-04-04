# Unikernel 调研报告
## "痛点"

Unikernel 是可以在裸机上直接运行的，但是对调试的支持非常差，很难调试。所以需要一种解决方案来增强对 Unikernel 的调试的支持。为了在裸机上开发，与不同的设备进行通信调试。

## 现有的 Unikernel 实现
http://unikernel.org/projects/



### ClickOS

ClickOS 是基于开源虚拟化技术的高性能虚拟化软件平台。ClickOS 的虚拟机很小（5MB），可以快速启动（20ms），延迟很小（45us）。

（可以考虑，但是没有明确的 ARM Portibility，并且没有 Bare-metal）。

### Clive 
用 Go 语言编写的在分布式和云计算环境中工作的操作系统；裸机支持还在进行，且开发不是非常活跃。

### DrawBridge
于应用程序沙盒的新型虚拟化形式的研究原型，结合了两个核心技术：Picoprocess 和 Library OS。

主要用于 Windows 程序的 Unikernel 化；没有计划要加入裸机的支持。

https://www.microsoft.com/en-us/research/project/drawbridge/?from=http%3A%2F%2Fresearch.microsoft.com%2Fen-us%2Fprojects%2Fdrawbridge%2F

### HALVM
全程是 The Haskell Lightweight Virtual Machine。

Glasgow Haskell 编译器工具套件的一个移植，使开发人员能够编写可直接在 Xen 虚拟机管理程序上运行的高级、轻量 VMs。

### includeOS
> In the summer of 2018 Horizon 2020 awarded the IncludeOS project funds to port IncludeOS to the ARM architecture. During 2019 we expect IncludeOS to boot on the Raspberry Pi M3 B+. Our goal is to provide your IoT project with a secure and real-time capable operating system on CPU platforms.
> https://www.includeos.org/blog/2018/port-to-arm.html
> 待交付，应该可行。
- LING
> https://github.com/cloudozer/ling/tree/raspberry-pi
- MigrageOS
> https://mirage.io/wiki/arm64
> Thanks to Solo5 and hvt MirageOS can run on ARM CPUs which support the ARM virtualization extensions. As the layer for Mirage currently only supports the 64bit architecture a 64bit CPU is required.

> So far this has been tested on the following SOCs.

> Broadcom BCM2837 on Raspberry Pi 3/3+
> Allwinner A64 on A64-OLinuXino / Pine A64
> Amlogic S905 on Odroid-C2
> It should be possible on all A53 based SOCs as long as a recent Kernel is available.

> In the following the process to build yourself a debian based firmware image for the raspberry Pi 3/3+ is described. For other targets the process is very similar and usually only differs in the bootloader related part.

> If you are not into builing your own image, you can try to use Arch Linux as they seem to ship KVM enabled 64bit Kernel for the Raspberry Pi.
- OSv
> https://github.com/cloudius-systems/osv/wiki/AArch64

- HermitCore only on x86

Together, I believe unikernels have the potential to change the cloud-computing ecosystem as well as to dominate the emerging IoT market.

includeOS - 不支持 ARM


