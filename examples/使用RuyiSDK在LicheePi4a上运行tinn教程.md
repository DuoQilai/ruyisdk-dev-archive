# RuyiSDK Tinn 示例移植报告

## 总目标

用 **RuyiSDK 工具链**把一个「应用实战类」C 项目在 **RISC-V** 上跑通，并沉淀为可复现示例（Linux 服务器 → QEMU → RevyOS/LicheePi 4A）。

## Tinn 是什么

Tinn（Tiny Neural Network）是一个 **约 200 行、无依赖的 C99 神经网络库**。本示例用 `semeion.data`（16×16 像素手写数字数据集）进行训练与推理，终端输出训练误差下降和预测结果。

## 对用户有什么用

- **给一个“真实负载”的可复现验证用例**：包含数据读取、训练迭代、推理输出、模型落盘（`saved.tinn`），能验证工具链、libm、文件 I/O、性能与稳定性。
- **提供可迁移模板**：把 256 维输入替换为自己的特征（传感器/日志/信号处理结果），即可复用同一流程做本地分类。
- **体现 RuyiSDK 的价值**：用统一的包管理/版本/路径，把“从装环境到在 QEMU/实机跑通”的流程标准化。

---

## 三阶段验证
 
### 1) x86 Linux 原生验证（先证明项目本身能跑）

- 环境：x86_64 Linux；依赖 `gcc`/`make`/`wget`
- 源码：`git clone https://github.com/glouw/tinn.git`
- 数据：`wget http://archive.ics.uci.edu/ml/machine-learning-databases/semeion/semeion.data`
- 编译运行：`make` → `./test`
- 结果：训练过程正常输出，error 明显下降，最终输出预测向量

### 2) Ruyi 工具链交叉编译 + QEMU RISC-V 验证（证明 Ruyi 编译产物可运行）

- 环境：x86_64 Linux；依赖 `ruyi`（0.46.0）+ `gnu-plct` + `qemu-riscv64`（8.2+）
- 工具链：`gnu-plct`（通用 ISA；QEMU 不支持 `xthead*` 扩展）

**2.1 安装/启用 ruyi**

```bash
mkdir -p ~/.local/bin
cd /tmp
wget -O ruyi https://mirror.iscas.ac.cn/ruyisdk/ruyi/releases/0.46.0/ruyi-0.46.0.amd64
chmod +x ruyi
mv ruyi ~/.local/bin/ruyi
export PATH="$HOME/.local/bin:$PATH"
ruyi version
```

**2.2 安装 QEMU 与通用工具链（gnu-plct）**

```bash
ruyi install gnu-plct
ruyi install qemu-user-riscv-upstream
```

安装后的工具默认位于：
`~/.local/share/ruyi/binaries/x86_64/<package-and-version>/bin/`

- 编译命令：
  ```bash
  ~/.local/share/ruyi/binaries/x86_64/gnu-plct-0.20250912.0/bin/riscv64-plct-linux-gnu-gcc \
    -O2 -static Tinn.c test.c -o tinn_riscv_qemu -lm
  ```
- 验证：`qemu-riscv64 ./tinn_riscv_qemu`
- 结果：在 QEMU 上训练过程正常输出，error 下降

**输出示例（节选）：**
```
error 0.214713940138 :: learning rate 1.000000
error 0.083898224624 :: learning rate 0.990000
error 0.050066582152 :: learning rate 0.980100
error 0.033050256934 :: learning rate 0.970299
...
error 0.001376556288 :: learning rate 0.279042
0.000000 0.000000 1.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000
0.000061 0.000871 0.999866 0.000018 0.000000 0.000001 0.000012 0.000000 0.000025 0.000000
```

---

### 3) RevyOS（LicheePi 4A）实机验证（证明在真实板子上可运行）

- 硬件/系统：LicheePi 4A（TH1520）+ RevyOS（Debian 13 riscv64，kernel `6.6.119-th1520`）
- 连接：SSH（示例：`ssh debian@192.168.31.179`）

**3.1 从主机拷贝到板子并运行（最小复现）**

在主机（x86_64 Linux）执行：

```bash
cd /home/fengde/Projects/RuyiSDK/tinn
scp ./semeion.data ./tinn_lpi4a debian@192.168.31.179:/home/debian/
```

在板子（RevyOS）执行：

```bash
cd /home/debian
chmod +x ./tinn_lpi4a
./tinn_lpi4a
```

**结果**：训练过程正常输出，error 下降，最终输出预测向量（节选）：

```
error 0.204766627310 :: learning rate 1.000000
error 0.076982875328 :: learning rate 0.990000
error 0.050114389193 :: learning rate 0.980100
error 0.035771862636 :: learning rate 0.970299
...
error 0.001372004783 :: learning rate 0.279042
0.000000 0.000000 1.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000 0.000000
0.000102 0.000336 0.997616 0.005362 0.000000 0.000048 0.001111 0.000056 0.000254 0.000000
```

---