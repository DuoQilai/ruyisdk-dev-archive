# RuyiSDK 系统通信示例

## Mailbox 双核通信测试

本文介绍如何使用 RuyiSDK 在 Milk-V Duo S 开发板上快速部署编译环境，并构建 mailbox 双核通信程序，验证大核与小核之间的通信功能。

### 1. 准备工作

#### 硬件环境

* **开发板**：Milk-V Duo S (512M, SG2000)

* **其他**：microSD 卡、USB Type-C 数据线

#### 操作系统安装与启动验证

确保您的开发板已刷入 RuyiSDK 支持的 Buildroot 系统镜像。

参考文档：https://ruyisdk.org/docs/Package-Manager/cases/case3/

#### 重启系统

将 microSD 卡插入 Milk-V Duo S，重启。使用 SSH 登录系统（用户名：`root`，密码：`milkv`）。



### 2. 部署 RuyiSDK 环境

#### 安装 ruyi 包管理器

```bash

wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.41.0/ruyi-0.41.0.riscv64

chmod +x ruyi-0.41.0.riscv64

sudo cp -v ruyi-0.41.0.riscv64 /usr/local/bin/ruyi

ruyi update

```
#### 烧录系统镜像

```bash

ruyi device provision

```

按提示选择开发板型号（Milk-V Duo S）和系统配置，输入 SD 卡设备路径完成烧录。

### 3. 编译应用与验证

#### 获取源码

```bash

# 克隆源码

git clone https://github.com/milkv-duo/duo-examples.git

cd duo-examples

```

#### 编译构建

激活虚拟环境并配置环境变量进行编译：

```bash

#配置编译环境

source envsetup.sh

```

按提示选择开发板型号（Milk-V Duo S）和架构（riscv64）。

```bash

#编译 mailbox-test

cd mailbox-test

make

```

#### 验证结果

检查生成的二进制文件：

```bash

file mailbox_test

```

### 4.传输并运行

```bash

# 传输到开发板

scp mailbox_test root@192.168.42.1:/root/

# SSH 登录开发板后运行

ssh root@192.168.42.1

cd /root

./mailbox_test

```

### 5.预期现象

* **终端输出：**

```bash

C906B: cmd.param_ptr = 0x2

C906B: cmd.param_ptr = 0x3

```

* **板载蓝色 LED 短暂亮起后熄灭**
