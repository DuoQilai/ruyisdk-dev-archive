
# 使用 RuyiSDK 在 Milk-V Duo256m 上编译blink教程
### LED 闪烁的脚本禁用
```bash
ssh root@192.168.42.1
mv /mnt/system/blink.sh /mnt/system/blink.sh_backup && sync
```
### 搭建编译环境


**安装基础依赖**

```bash
sudo apt update
sudo apt install git build-essential gcc make libncurses5-dev flex bison libssl-dev rsync
```

**下载 RISC-V 交叉编译工具链**
```bash
# 创建一个工作目录
mkdir -p ~/milkv
cd ~/milkv

# 下载官方预编译的工具链 (可能需要几分钟，取决于网速)
wget https://sophon-file.sophon.cn/sophon-prod-s3/drive/23/03/07/16/host-tools.tar.gz

# 解压
tar -xf host-tools.tar.gz
```

**获取官方 Blink 裸机示例代码**

```bash
git clone https://github.com/milkv-duo/duo-examples.git
cd duo-examples
```


**先运行环境配置脚本**

```bash
cd ~/milkv/duo-examples
source envsetup.sh
```

这个脚本会帮你**自动拉取交叉编译工具链**（不用你手动 wget 了），并配置好环境变量。观察输出，等它跑完。

**进入 blink 目录编译**

```bash
cd blink
make
```

### 将二进制文件传输至开发板并运行验证

**将编译好的二进制传输至开发板**

```bash
scp blink root@192.168.42.1:~
```

**SSH连接到开发板**

```bash
ssh root@192.168.42.1
```

**运行官方逻辑验证程序**

```bash
./blink
```
