---
sys: RuyiSDK
sys_ver: 0.46.0
sys_var: Buildroot
provider: milkv
status: PASS
last_update: 2026-04-23
model: Milk-V Duo S
profile: adc
---

# RuyiSDK 芯片功能示例

### 安装 ruyi

#### 安装依赖包

```

sudo apt update; sudo apt install -y wget tar zstd xz-utils git build-essential

```

#### 安装 ruyi 包管理器

```

wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.46.0/ruyi-0.46.0.riscv64

chmod +x ruyi-0.46.0.riscv64

sudo cp -v ruyi-0.46.0.riscv64 /usr/local/bin/ruyi

```

#### 安装工具链

```

ruyi update

ruyi install gnu-plct llvm-plct

ruyi install gnu-milkv-milkv-duo-musl-bin

```

## ADC 芯片功能测试

本文介绍如何使用 RuyiSDK 在 Milk-V Duo S 开发板上快速部署编译环境，并构建 ADC 测试程序，验证芯片内部模数转换器通道的读取功能。

### 1. 准备工作

* **开发板**：Milk-V Duo S (512M, SG2000)

* **其他**：microSD 卡、USB Type-C 数据线

#### 操作系统安装与启动验证

确保您的开发板已刷入 RuyiSDK 支持的 Buildroot 系统镜像。

参考文档：https://milkv.io/zh/docs/duo/getting-started/boot

### 2. 获取源码

#### 克隆源码

```bash

git clone https://github.com/milkv-duo/duo-examples.git

cd duo-examples

```

### 3. 编译应用与验证

#### 创建虚拟环境

```bash

ruyi venv -t gnu-milkv-milkv-duo-musl-bin generic ./ruyi_venv

source ruyi_venv/bin/ruyi-activate

```

#### 编译 ADC 程序

```bash

cd adc

riscv64-unknown-linux-musl-gcc -o adcRead adcRead.c

```

#### 验证结果

检查生成的二进制文件：

```bash

file adcRead

```

#### 退出虚拟环境

```bash

ruyi-deactivate

```

### 4.传输并运行

默认用户名：`root`，默认密码：`milkv`

```bash

# 传输到开发板

scp adcRead root@192.168.42.1:/root/

# SSH 登录开发板

ssh root@192.168.42.1

# 运行测试

./adcRead

```

运行后，终端持续输出数据：
#### 选择通道 4 (VDDC_RTC)：

```bash

ADC4 value is 0
ADC4 value is 0
ADC4 value is 0
...

```

#### 选择通道 5 (PWR_GPIO1)：

```bash

ADC5 value is 4095
ADC5 value is 4095
ADC5 value is 4095
...

```

#### 选择通道 6 (PWR_VBAT_V)：

```bash

ADC6 value is 3712
ADC6 value is 3688
ADC6 value is 3696
...

```
