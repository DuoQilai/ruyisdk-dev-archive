# GCC 和 LLVM 对 Sipeed Tang Mega 138K 的支持测试

| 项目 | 内容 |
|------|------|
| **测试用例名称** | GCC 和 LLVM 对 Sipeed Tang Mega 138K 的支持测试 |
| **测试说明** | 验证 GCC 和 LLVM 对 Sipeed Tang Mega 138K  的支持情况 |
| **前提与约束** | 开发板已通过 Type-C 连接至主机，FPGA 码流已烧录完成。 |
| **终止条件** | 测试用例全部通过 |

---

## 测试用例初始化

准备好开发板、Type-C 数据线、12V DC 电源适配器

### 1. 下载并安装高云云源软件： 

https://www.gowinsemi.com.cn/software/1

### 2. 下载高云 RiscV_AE350_SOC_RDS 软件开发套件：  
   
https://www.gowinsemi.com.cn/software/6

### 3. 克隆 Sipeed 官方 Tang Mega 138K 示例代码仓库：  

```bash

git clone --recursive https://github.com/sipeed/TangMega-138K-example.git

```

### 4. 硬件连接与驱动确认

### 5.烧录 FPGA 码流：

使用高云云源软件加载示例工程中的 .gprj 文件，编译后通过 Programmer 下载 .fs 位流文件

### 6.烧录完成后断开电源重新上电，等待开发板启动

### 7.确认串口通信：

```bash

minicom -b 115200 -D /dev/ttyUSB0

```

## 测试步骤

### 步骤 1：安装 ruyi 包管理器依赖包

```bash

sudo apt update; sudo apt install -y wget tar zstd xz-utils git build-essential

```

成功安装

### 步骤 2：安装 ruyi 包管理器

```bash

wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.47.0/ruyi.riscv64

chmod +x ruyi-0.47.0.riscv64

sudo cp -v ruyi-0.47.0.riscv64 /usr/local/bin/ruyi

```

成功安装

### 步骤 3：安装 GCC 和 LLVM 工具链

```bash

ruyi update

ruyi install gnu-plct llvm-plct

```

成功安装

### 步骤 4：创建并激活 ruyi 虚拟环境（GCC）

```bash

ruyi venv -t toolchain/gnu-plct manual venv-gnu-plct

. ~/venv-gnu-plct/bin/ruyi-activate

```

成功创建并激活虚拟环境

### 步骤 5：验证 GCC 版本

```bash

riscv64-plct-linux-gnu-gcc -v

```

输出版本号

### 步骤 6：编译并运行 Hello World（GCC）

```bash

cd ~/ae350_zephyr

source zephyr-env.sh

export ZEPHYR_TOOLCHAIN_VARIANT='cross-compile'

export CROSS_COMPILE=riscv64-plct-linux-gnu-

cd samples/hello_world

mkdir -p build && cd build

cmake -DBOARD=adp_xc7k_ae350 ../

make

/path/to/RiscV_AE350_SOC_RDS/flash/programmer.exe --mode 5AT --addr 0x600000 --file build/zephyr/zephyr.bin

```

输出 Hello, World!

### 步骤 7：编译并运行 coremark（GCC）

```bash

cd ~/ae350_zephyr/tests/benchmarks/coremark

mkdir -p build && cd build

cmake -DBOARD=adp_xc7k_ae350 ../

make

/path/to/RiscV_AE350_SOC_RDS/flash/programmer.exe --mode 5AT --addr 0x600000 --file build/zephyr/zephyr.bin

```

输出 coremark 结果

### 步骤 8：返回上级目录并退出 ruyi GCC 虚拟环境

```bash

cd ..; ruyi-deactivate

```

成功退出虚拟环境

### 步骤 9：创建并激活 ruyi 虚拟环境（LLVM）

```bash

ruyi venv -t toolchain/llvm-plct manual --sysroot-from gnu-plct venv-llvm-plct

. ~/venv-llvm-plct/bin/ruyi-activate

```

成功创建并激活虚拟环境

### 步骤 10：验证 LLVM 版本

```bash

clang -v

```

输出版本号

### 步骤 11：编译并运行 Hello World（LLVM）

```bash

cd ~/ae350_zephyr

source zephyr-env.sh

export ZEPHYR_TOOLCHAIN_VARIANT='cross-compile'

export CROSS_COMPILE=riscv64-plct-linux-gnu-

cd samples/hello_world

mkdir -p build && cd build

cmake -DBOARD=adp_xc7k_ae350 ../ -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++

make

/path/to/RiscV_AE350_SOC_RDS/flash/programmer.exe --mode 5AT --addr 0x600000 --file build/zephyr/zephyr.bin

```

输出 Hello, World!

### 步骤 12：编译 coremark（LLVM）

```bash

cd ~/ae350_zephyr/tests/benchmarks/coremark

mkdir -p build && cd build

cmake -DBOARD=adp_xc7k_ae350 ../ -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++

make

/path/to/RiscV_AE350_SOC_RDS/flash/programmer.exe --mode 5AT --addr 0x600000 --file build/zephyr/zephyr.bin

```

输出 coremark 结果

### 步骤 13：返回上级目录并退出 ruyi LLVM 虚拟环境

```bash

cd ..; ruyi-deactivate

```

成功退出虚拟环境

## 测试结论

针对 Sipeed Tang Mega 138K 开发板，RuyiSDK 无法独立完成从编译到烧录的完整测试闭环，其核心原因可归纳为以下两点：

**1. 开发流程的本质差异**
Tang Mega 138K 基于高云 GW5AST FPGA 芯片，其开发遵循“硬件编程 + 软件烧录”的双阶段模式。开发者必须首先使用高云半导体的**商业版云源软件**完成 FPGA 逻辑综合、布局布线及码流（.fs 文件）烧录，将 FPGA 配置为特定的 RISC-V 处理器系统后，才能进行后续的软件部署。这一硬件配置阶段是 RuyiSDK 作为软件工具链和包管理器所无法替代的。

**2. 烧录工具的专有依赖性**
即便在 RuyiSDK 创建的虚拟环境中使用其集成的工具链（如 `riscv64-plct-linux-gnu-gcc`）成功编译出可在 RISC-V 核上运行的 `hello` 程序，最终将其烧录至板载 Flash 的操作也必须依赖 RDS（RISC-V Development Suite）开发套件中的专用烧录工具 `programmer.exe`，并需指定特定的烧录模式（如 `--mode 5AT`）和起始地址（如 `--addr 0x600000`）。该工具为 Sipeed/RDS 提供的专有软件，RuyiSDK 无法接管或替代这一烧录环节。
