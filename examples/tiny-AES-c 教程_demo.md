
## 1.准备工作

### 下载系统镜像

```Plain Text
https://github.com/milkv-duo/duo-buildroot-sdk-v2/releases/download/v2.0.1/milkv-duo256m-musl-riscv64-sd_v2.0.1.img.zip
```

### 解压系统镜像

```Plain Text
unzip milkv-duo256m-musl-riscv64-sd_v2.0.1.img.zip
```

### 系统镜像烧录进存储卡

```Plain Text
sudo dd if=milkv-duo256m-musl-riscv64-sd_v2.0.1.img of=/dev/sdX bs=1M; sync
```

### 安装依赖包

```Plain Text
sudo apt update; sudo apt install -y wget tar zstd xz-utils git build-essential
```

### 安装ruyi包管理器

```Plain Text
wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.45.0/ruyi-0.45.0.amd64
chmod +x ruyi-0.45.0.amd64
sudo cp -v ruyi-0.45.0.amd64 /usr/local/bin/ruyi
```

### 安装GCC工具链

```Plain Text
ruyi update
ruyi install gnu-plct 
```

## 2.获取 tiny-AES-c 源码并编译（基于官方原生文件）

### 创建并激活ruyi虚拟环境（GCC）

```Plain Text
ruyi venv -t toolchain/gnu-plct milkv-duo venv-aes
. ~/venv-aes/bin/ruyi-activate
```

### 验证GCC版本

```Plain Text
riscv64-plct-linux-gnu-gcc -v
```

执行后应显示RISC-V架构的GCC工具链版本信息，确认工具链安装成功且激活生效。

### 克隆 tiny-AES-c 项目源码

```Plain Text
git clone https://github.com/kokke/tiny-AES-c.git
cd tiny-AES-c
```

### 编译官方原生测试程序（验证逻辑运算）

```Plain Text
riscv64-plct-linux-gnu-gcc aes.c test.c -static -o aes-official-test -O2 -lm
```

编译算力测试程序：

```Plain Text
riscv64-plct-linux-gnu-gcc aes.c aes_official_test.c -static -o aes-perf-test -O2 -lm
```

## 3.将二进制文件传输至开发板并运行验证

### 将编译好的二进制传输至开发板

```Plain Text
scp aes_official_test root@192.168.42.1:~
```

### SSH连接到开发板

```Plain Text
ssh root@192.168.42.1
```

### 运行官方逻辑验证程序

```Plain Text
./aes-official-test
```
