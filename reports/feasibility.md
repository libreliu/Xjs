# includeOS on ARM 可行性报告

## 项目介绍
IncludeOS 是一个 C++ 的 Unikernel 实现，并可以在 bare-metal 上运行。IncludeOS 提供了丰富的用于网络编程的库，但是目前还不支持在 ARM 上运行。裸机运行的 IncludeOS 相较于 Linux 发行版拥有更快的启动速度，并且减少了进程切换等的无谓开销。现有的树莓派的 Unikernel 主要是对一些开关 GPIO 等相关的实现，但是对网络的支持很弱。在 IoT 领域中，有许多应用场景对延迟的要求十分苛刻，而本项目意在将 IncludeOS 移植到 ARM 上，这样对延迟敏感的 IoT 应用场景会有很大帮助。

## 理论依据


## 技术依据

在「技术依据」章节，我们将对 includeOS 的开发环境配置，启动过程和相关技术进行简要的介绍。

### includeOS 环境配置
Ref: [GetStarted @ IncludeOS](http://www.includeos.org/get-started.html)

includeOS 会将操作系统相关的代码，以及链接和测试工具安装到系统中。默认的安装目录是`/usr/local/includeos`。

对于一般开发者来说，可能更希望使用一个自定义的目录，这可以通过设置环境变量解决。此处是我们写的一个比较方便的脚本：

```shell
#!/bin/bash

# IncludeOS tools & static labs will be located here
export INCLUDEOS_PREFIX=`pwd`/IncludeOS_Install/
export PATH=$PATH:$INCLUDEOS_PREFIX/bin
export CC="clang"
export CXX="clang++"

echo "New Env Var: $PATH"
echo "In IncludeOS_Install Env Now"
echo 'export PS1="[\u@ios-env(\h) \W]\$ "' > bash_start_arm.rc
bash --init-file bash_start_arm.rc -i
echo "Exited from IncludeOS_Install Env"
```
在配置好环境变量之后，就可以进入安装：
```
git clone "https://github.com/includeos/IncludeOS"
cd IncludeOS
./install.sh
```
安装以 Fedora, Arch, Ubuntu 和 Debian 为宜。安装过程中注意安装依赖。`vmrunner`等脚本都是 Python 2 的，安装时请提前装好 Python 2 对应的包，否则按 ImportError 的指示来安装也是可以的。

*Notice:* 在创建网桥时默认采用的是 `ifconfig` 命令（参`./etc/scripts/create_bridge.sh`）。现在的发行版较少默认安装；Arch 用户可以使用`sudo pacman -Sy net-tools`进行安装。

安装好后可以运行`./test.sh`（位于 repo 根目录下）来测试编译 Example Service 能否通过，并且能否在 QEMU 中运行。

### includeOS Demo
includeOS 的开发体验非常便捷。只需要复制`seed/service`文件夹到任意位置，并且对`Service::start`函数进行更改即可。

`CMakeLists.txt` 中的 `SERVICE_NAME` 和 `BINARY` 字段也可更改，会生成不同的 binary 文件名。

示例的 service 如下所示：
```c
#include <service>
#include <cstdio>
#include <isotime>
#include <kernel/cpuid.hpp>

void Service::start(const std::string& args)
{
#ifdef __GNUG__
  printf("Built by g++ " __VERSION__ "\n");
#endif
  printf("Hello world! Time is now %s\n", isotime::now().c_str());
  printf("Args = %s\n", args.c_str());
  printf("Try giving the service less memory, eg. 5MB in vm.json\n");
  printf("CPU has RDRAND: %d\n", CPUID::has_feature(CPUID::Feature::RDRAND));
  printf("CPU has RDSEED: %d\n", CPUID::has_feature(CPUID::Feature::RDSEED));
}
```

完成后运行以下命令：
```bash
mkdir build && cd build
cmake ..
make
boot my_service
```
即可打开 QEMU，并且以`my_service`这个 binary 启动。

### includeOS 构建过程
Installing IncludeOS means building all the OS components, such as IRQ manager, PCI manager, the OS class etc., combining them into a static library os.a using GNU ar, and putting it in an architecture specific directory under $INCLUDEOS_PREFIX along with all the public os-headers (the “IncludeOS API”). This is what you’ll be including parts of, into the service. Device drivers are built as their own libraries, and must be explicitly added in the CMakeLists.txt of your service. This makes it possible to only include the drivers you want, while still not having to explicitly mention a particular driver in your code.

安装 IncludeOS 意味着构建所有 OS 组件，比如 IRQ 管理器，PCI 管理器，OS 类——把他们用 GNU `ar` 整合到一个静态库 `os.a` 中，然后

When the service gets built it will turn into object files, which eventually gets statically linked with the os-library, drivers, plugins etc. It will also get linked with the pre-built standard libraries (libc.a, libc++.a etc.) which we provide as a downloadable bundle, pre-built using this script. Only the objects actually needed by the service will be linked, turning it all into one minimal elf-binary, your_service, with OS included.

This binary contains a multiboot header, which has all the information the bootloader needs to boot it. This gives you a few options for booting, all available through the simple boot tool that comes with IncludeOS:

Qemu kernel option: For 32-bit ELF binaries qemu can load it directly without a bootloader, provided a correct multiboot header. This is what boot <service path> will do by default. The boot tool will generate something like $ qemu_system_x86_64 -kernel your_service ..., which will boot your service directly. Adding -nographic will make the serial port output appear in your terminal. For 64-bit ELF binaries Qemu has a paranoid check that prevents this, so we’re using a 32-bit IncludeOS as chainloader for that. If boot <service path> detects a 64-bit ELF it will use the 32-bit chainloader as -kernel, and add the 64 bit binary as a “kernel module”, e.g. -initrd <my_64_bit_kernel>. The chainloader will copy the 64-bit binary to the appropriate location in memory, modify the multiboot info provided by the bootloader to the kernel, and jump to the new kernel, which boots as if loaded directly by e.g. GRUB.
Legacy: Attach our own minimal bootloader, using the utility vmbuild. It combines our minimal bootloader and your_service binary into a disk image called your_service.img. At this point the bootloader gets the size and location of the service hardcoded into it. The major drawback of using this bootloader is that it doesn’t fetch information about system memory from the BIOS so you can’t know exactly how much memory you have, above 65Mb. (Which CMOS can provide)
Grub: Embed the binary into a GRUB filesystem, and have the Grub chainloader boot it for you. This is what we’re doing when booting on Google Compute Engine. You can do this on Linux using boot -g <service path>, which will produce a bootable your_service.grub.img. Note that GRUB is larger than IncludeOS itself, so expect a few megabytes added to the image size.
To run with vmware or virtualbox, the image has to be converted into a supported format, such as vdi or vmdk. This is easily done in one command with the qemu-img-tool, that comes with Qemu. We have a script for that too. Detailed information about booting in vmware, which is as easy as boot, is provided here.

Inspect the main CMakeLists.txt and then follow the trail of cmake scripts in the added subfolders for information about how the OS build happens. For more information about building individual services, check out the CMakeLists.txt of one of the example services, plus the linker script, linker.ld for the layout of the final binary. Note that most of the CMake magic for link- and include paths, adding drivers, plugins etc. is tucked away in the post.service.cmake.

### includeOS 启动过程（x86）

### includeOS 源码概览


## 技术路线

## 参考文献