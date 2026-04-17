---
sys: RuyiSDK
sys_ver: 0.46.0
sys_var: Buildroot
provider: milkv
status: PASS
last_update: 2026-04-09
model: Milk-V Duo S
profile: blink
---

# RuyiSDK 系统通信示例

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

```

## LED 闪烁控制测试

本文介绍如何使用 RuyiSDK 在 Milk-V Duo S 开发板上快速部署编译环境，并构建 LED 闪烁控制程序，验证板载 LED 的控制功能。

### 1. 准备工作

* **开发板**：Milk-V Duo S (512M, SG2000)

* **其他**：microSD 卡、USB Type-C 数据线

#### 操作系统安装与启动验证

确保您的开发板已刷入 RuyiSDK 支持的 Buildroot 系统镜像。

参考文档：https://milkv.io/zh/docs/duo/getting-started/boot

### 2. 获取源码并配置环境

#### 克隆源码

```bash

git clone https://github.com/milkv-duo/duo-examples.git

cd duo-examples

```

#### 配置编译环境

```bash

source envsetup.sh

```

### 3. 编译应用与验证

#### 编译构建

```bash

cd blink

make

```

#### 验证结果

检查生成的二进制文件：

```bash

file blink

```

### 4.传输并运行

默认用户名：`root`，默认密码：`milkv`

```bash

# 传输到开发板

scp blink root@192.168.42.1:/root/

# SSH 登录开发板

ssh root@192.168.42.1

```

#### 禁用系统自带 LED 脚本

运行 blink 程序前，需要先禁用系统自带的 LED 闪烁脚本，避免冲突：

```

mv /mnt/system/blink.sh /mnt/system/blink.sh_backup && sync

# 执行完后，需要重启开发板，使禁用生效：

reboot

```

重启后重新 SSH 登录

```bash

ssh root@192.168.42.1

# 运行测试

cd /root

chmod +x blink

./blink

```

运行后终端持续输出 LED 状态：

```bash

[root@milkv-duo]~# ./blink

Duo LED GPIO (wiringX) 25: High

Duo LED GPIO (wiringX) 25: Low

Duo LED GPIO (wiringX) 25: High

Duo LED GPIO (wiringX) 25: Low

...

```

蓝色 LED 会同步闪烁：`High` 亮起，` Low` 熄灭。

按 ` Ctrl+C ` 可终止程序。

### 5.恢复系统原有 LED 功能

测试完成后，如需恢复系统默认 LED 闪烁：

```

ssh root@192.168.42.1

# 恢复 LED 脚本

mv /mnt/system/blink.sh_backup /mnt/system/blink.sh && sync

# 重启开发板使恢复生效

reboot

```
