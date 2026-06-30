# RuyiSDK兼容Linux开发板调研报告

## 调研目的

为了开展 **RuyiSDK GCC/LLVM 工具链验证**，需要选择能够运行 Linux、支持标准 GNU/LLVM 工具链、无需依赖厂商专属 IDE 或工具链的 RISC-V 开发平台。本报告对近期具有代表性的开发板进行了调研，重点关注其处理器、CPU IP 核、Linux 支持情况及其作为 RuyiSDK 测试平台的适用性。

## 开发板信息汇总

| 序号 | 开发板                | 处理器       | IP 核(主CPU)              | 发布时间          | 购买链接                                                     | 备注                                                         |
| ---- | --------------------- | ------------ | ------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | **Milk-V Titan**      | UR-DP1000    | UR-CP100 ×8               | **2026 年**       | [京东](http://item.jd.com/10165331347566.html?sdx=ehi-lLxFuZiE6JnJZ4Zfj8cjtDeUDg0rsmpKtqxHZNWLPe_RLJhe4nvmpU3iU2KT) | 新一代高性能 RISC-V Linux 工作站平台，支持 PCIe、DDR4，适合大型工程和 LLVM/GCC 性能测试。 |
| 2    | **BIT-BRICK K1**      | SpacemiT K1  | SpacemiT X60 ×8           | **2025 年**       | [淘宝](https://item.taobao.com/item.htm?abbucket=17&id=886103226653&loginBonus=1&mi_id=0000F2ufsW0DHC6rrbNcypBikAoeD0xrkEdAKPJZ373_XUc&ns=1&priceTId=213e033917824667415004184e11b1&skuId=5918874730117&spm=a21n57.sem.item.9.3d863903es7whV&utparam=%7B%22aplus_abtest%22%3A%22c830c9fd80fef7aed270b89c708c09e2%22%7D&xxc=taobaoSearch) | 新一代高性能 Linux RISC-V 平台，与 BPI-F3、Milk-V Jupiter 同生态。 |
| 3    | **MangoPi MQ-Pro**    | Allwinner D1 | T-Head C906               | **2024 年**       | [淘宝](https://item.taobao.com/item.htm?abbucket=17&id=586376508252&loginBonus=1&mi_id=0000iR9hkbArP41qmEqMZWpwIwYiFx3r7sV9CFNhRrI2d5Q&ns=1&priceTId=213e033917824666925698595e11b1&skuId=5134857298508&spm=a21n57.sem.item.3.3d863903es7whV&utparam=%7B%22aplus_abtest%22%3A%222b175e2f19e52fc7f2b2c3ed852bad96%22%7D&xxc=taobaoSearch) | 社区成熟，价格低，适合作为基础 Linux RISC-V 测试平台。([RISC-V International](https://riscv.org/blog/the-release-of-the-first-two-mass-produced-development-boards-aosp-powered-by-th1520-soc/)) |
| 4    | **BeagleV-Ahead**     | TH1520       | T-Head C910 ×4            | **2023 年 7 月**  | [淘宝](https://item.taobao.com/item.htm?abbucket=17&id=816639537334&loginBonus=1&mi_id=0000_FNCjiVwOhj8eSDo0jnewI61R0C4yZnWlKbZf9segJI&ns=1&priceTId=213e00ec17824658412438978e0eb2&skuId=5516854101696&spm=a21n57.sem.item.5.2a5b3903PLnAE1&utparam=%7B%22aplus_abtest%22%3A%2271e0a363e9c69f191a9f6351daac4ac2%22%7D&xxc=taobaoSearch) | 高性能 Linux SBC，开放生态完善，适合 GCC/LLVM 测试。([RISC-V International](https://riscv.org/blog/2023/07/the-release-of-the-first-two-mass-produced-development-boards-aosp-powered-by-th1520-soc/)) |
| 5    | **BeagleV-Fire**      | MPFS025T     | RV64GC ×4                 | **2024 年**       | [淘宝](https://item.taobao.com/item.htm?abbucket=17&id=846250586941&loginBonus=1&mi_id=0000Qa9kMZ__4pUKo7PlbAqkWLR2UOsdoQP_LW3dSwwnG14&ns=1&priceTId=213e033917824666578646248e11b1&spm=a21n57.sem.item.2.58883903ZL0Ght&utparam=%7B%22aplus_abtest%22%3A%2200cf069f90d281a1e9ec94cadc1e26dc%22%7D&xxc=taobaoSearch) | FPGA + Linux 平台，适合验证标准 RISC-V 工具链，不依赖专属编译器。([BeagleBoard](https://www.beagleboard.org/beaglev)) |
| 6    | **LicheeRV Dock**     | Allwinner D1 | T-Head C906               | **2024 年**       | [淘宝](https://item.taobao.com/item.htm?abbucket=17&id=665203616406&loginBonus=1&mi_id=00007OBE3i_9V4cJF9TQ8dGjfN0WvDKXkPVPfbJKZIyKAnM&ns=1&priceTId=213e033917824667855557774e11b1&skuId=5261683362375&spm=a21n57.sem.item.2.4af43903A858Fp&utparam=%7B%22aplus_abtest%22%3A%22bdaa116fa7f5dbd43f879e5ebd167f2f%22%7D&xxc=taobaoSearch) | 轻量级 Linux 开发板。                                        |
| 7    | **Lichee Cluster 4A** | TH1520 ×7    | T-Head C910 ×4 (per node) | **2023 年 10 月** | [淘宝](https://item.taobao.com/item.htm?abbucket=17&id=739945833283&loginBonus=1&mi_id=0000hrWeafmS4TieOQJPMLDxT-EGORgeUKOS769GrGWjOdc&ns=1&priceTId=213e033917824668405853934e11b1&skuId=5277828463984&spm=a21n57.sem.item.9.43703903QWBzkj&utparam=%7B%22aplus_abtest%22%3A%22ca699a70705b438e09f61098f17d03e0%22%7D&xxc=taobaoSearch) | 面向集群计算，可用于分布式编译和多节点 RISC-V 测试。([wiki.sipeed.com](https://wiki.sipeed.com/hardware/en/lichee/th1520/lc4a/lc4a.html)) |
| 8    | **Lichee Console 4A** | TH1520       | T-Head C910 ×4            | **2023 年 10 月** | [淘宝](https://item.taobao.com/item.htm?abbucket=17&id=888061897351&loginBonus=1&mi_id=0000qWRgK_OJQ8TSZq3_ryV370U4hrYH9Q9AN42HKjk__P0&ns=1&priceTId=213e033917824668739486436e11b1&skuId=5731163652248&spm=a21n57.sem.item.19.43703903QWBzkj&utparam=%7B%22aplus_abtest%22%3A%2291abfa9b448f67d2be743a2ce2352944%22%7D&xxc=taobaoSearch) | 便携式 Linux RISC-V 开发终端([wiki.sipeed.com](https://wiki.sipeed.com/hardware/en/lichee/th1520/lcon4a/lcon4a.html)) |

---

## CPU IP 核分类

| CPU IP 核                | 开发板                                               |
| ----------------------- | ------------------------------------------------- |
| **Xuantie C910**        | BeagleV-Ahead、Lichee Cluster 4A、Lichee Console 4A |
| **Xuantie C906**        | MangoPi MQ-Pro、LicheeRV Dock                      |
| **SiFive U54**          | BeagleV-Fire                                      |
| **SpacemiT X60**        | BIT-BRICK K1                                      |
| **UltraRISC UR-DP1000** | Milk-V TITAN                                      |

