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

安装好后可以运行`./test.sh`（位于 repo 根目录下）来测试编译 Example Service 能否通过，并且能否在 QEMU 中运行。

### includeOS Demo
includeOS 的开发体验非常便捷。


### includeOS 源码概览


## 技术路线

## 参考文献