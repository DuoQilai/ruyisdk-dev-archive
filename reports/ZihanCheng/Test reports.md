## 测试用例 1-3：GCC和LLVM对Sipeed Tang Mega 138K的支持测试

| 项目 | 内容 |
|------|------|
| **测试用例标识** | 1-3 |
| **测试用例名称** | GCC和LLVM对Sipeed Tang Mega 138K的支持测试 |
| **测试说明** | 验证GCC和LLVM对Sipeed Tang Mega 138K的支持情况 |
| **前提与约束** | 开发板已通过 Type-C 连接至主机，FPGA 码流已烧录完成。 |
| **终止条件** | 测试用例全部通过 |

---

### 测试用例初始化

准备好开发板、Type-C数据线、12V DC电源适配器

1. 下载并安装高云云源软件：  
   https://www.gowinsemi.com.cn/software/1

2. 下载高云 RiscV_AE350_SOC_RDS 软件开发套件：  
   https://www.gowinsemi.com.cn/software/6

3. 克隆 Sipeed 官方 Tang Mega 138K 示例代码仓库：  
   ```bash
   git clone --recursive https://github.com/sipeed/TangMega-138K-example.git
硬件连接与驱动确认

烧录 FPGA 码流：
使用高云云源软件加载示例工程中的 .gprj 文件，编译后通过 Programmer 下载 .fs 位流文件

烧录完成后断开电源重新上电，等待开发板启动

确认串口通信：

bash
minicom -b 115200 -D /dev/ttyUSB0
测试步骤
步骤 1：安装 ruyi 包管理器依赖包

bash
sudo apt update; sudo apt install -y wget tar zstd xz-utils git build-essential
期望结果：成功安装

步骤 2：安装 ruyi 包管理器

bash
wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.47.0/ruyi.riscv64
chmod +x ruyi-0.47.0.riscv64
sudo cp -v ruyi-0.47.0.riscv64 /usr/local/bin/ruyi
期望结果：成功安装

步骤 3：安装 GCC 和 LLVM 工具链

bash
ruyi update
ruyi install gnu-plct llvm-plct
期望结果：成功安装

步骤 4：创建并激活 ruyi 虚拟环境（GCC）

bash
ruyi venv -t toolchain/gnu-plct manual venv-gnu-plct
. ~/venv-gnu-plct/bin/ruyi-activate
期望结果：成功创建并激活虚拟环境

步骤 5：验证 GCC 版本

bash
riscv64-plct-linux-gnu-gcc -v
期望结果：输出版本号

步骤 6：编译并运行 Hello World（GCC）

bash
cd ~/ae350_zephyr
source zephyr-env.sh
export ZEPHYR_TOOLCHAIN_VARIANT='cross-compile'
export CROSS_COMPILE=riscv64-plct-linux-gnu-
cd samples/hello_world
mkdir -p build && cd build
cmake -DBOARD=adp_xc7k_ae350 ../
make
/path/to/RiscV_AE350_SOC_RDS/flash/programmer.exe --mode 5AT --addr 0x600000 --file build/zephyr/zephyr.bin
期望结果：输出 Hello, World!

步骤 7：编译并运行 coremark（GCC）

bash
cd ~/ae350_zephyr/tests/benchmarks/coremark
mkdir -p build && cd build
cmake -DBOARD=adp_xc7k_ae350 ../
make
/path/to/RiscV_AE350_SOC_RDS/flash/programmer.exe --mode 5AT --addr 0x600000 --file build/zephyr/zephyr.bin
期望结果：输出 coremark 结果

步骤 8：返回上级目录并退出 ruyi GCC 虚拟环境

bash
cd ..; ruyi-deactivate
期望结果：成功退出虚拟环境

步骤 9：创建并激活 ruyi 虚拟环境（LLVM）

bash
ruyi venv -t toolchain/llvm-plct manual --sysroot-from gnu-plct venv-llvm-plct
. ~/venv-llvm-plct/bin/ruyi-activate
期望结果：成功创建并激活虚拟环境

步骤 10：验证 LLVM 版本

bash
clang -v
期望结果：输出版本号

步骤 11：编译并运行 Hello World（LLVM）

bash
cd ~/ae350_zephyr
source zephyr-env.sh
export ZEPHYR_TOOLCHAIN_VARIANT='cross-compile'
export CROSS_COMPILE=riscv64-plct-linux-gnu-
cd samples/hello_world
mkdir -p build && cd build
cmake -DBOARD=adp_xc7k_ae350 ../ -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
make
/path/to/RiscV_AE350_SOC_RDS/flash/programmer.exe --mode 5AT --addr 0x600000 --file build/zephyr/zephyr.bin
期望结果：输出 Hello, World!

步骤 12：编译 coremark（LLVM）

bash
cd ~/ae350_zephyr/tests/benchmarks/coremark
mkdir -p build && cd build
cmake -DBOARD=adp_xc7k_ae350 ../ -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
make
/path/to/RiscV_AE350_SOC_RDS/flash/programmer.exe --mode 5AT --addr 0x600000 --file build/zephyr/zephyr.bin
期望结果：输出 coremark 结果

步骤 13：返回上级目录并退出 ruyi LLVM 虚拟环境

bash
cd ..; ruyi-deactivate
期望结果：成功退出虚拟环境
