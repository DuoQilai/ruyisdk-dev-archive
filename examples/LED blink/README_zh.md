---
sys: revyos
sys_ver: 20250930
sys_var: RISC-V 64
provider: PLCT/Sipeed
status: Supported
last_update: 2026-04-17

model: LicheePi 4A
profile: sipeed-lpi4a

---

#  RuyiSDK 外设控制示例

- **安装依赖包**

  ```bash
  sudo apt update
  sudo apt install -y libgpiod-dev nano
  ```

- **安装 RuyiSDK**
  根据 [RuyiSDK 官方安装指南](https://ruyisdk.org/docs/Package-Manager/installation)，在开发板上完成 Ruyi 包管理器本身的安装部署

- **安装工具链**

  ```bash
  ruyi update
  ruyi install gnu-plct-xthead
  ```


## LicheePi 4A LED Blink 示例

- #### 示例描述和硬件环境准备

1. **示例用途**：使用 C 语言和 `libgpiod` 库控制 LicheePi 4A 的 GPIO 引脚输出高低电平，实现 LED 灯闪烁效果。本示例采用直接调用交叉编译工具链进行单文件编译的方式，无需 CMake 或 Makefile 等额外构建工具。

2. **硬件组件**：面包板、3V-3.2V 直插 LED 灯、公对公杜邦线 2-3 根、1kΩ/220Ω 电阻（可选，用于限流保护）。

3. **硬件接线**：按以下方式完成连接，确保回路闭合：

   ```tex
   LicheePi4A GPIO1_3引脚 → LED长脚（正极） → LED短脚（负极） → LicheePi4A GND引脚
   ```

   - 可选：在GPIO1_3与LED正极之间串联1kΩ电阻（限流保护，无电阻不影响测试）；

   - 确认：杜邦线两端完全插入开发板排针和LED引脚，无松动。

     ![LED_blink_1](https://github.com/zhiyao310/plct_works/blob/main/outcome_list/Licheepi4A/LED_flashing/images/LED_blink_1.jpg)

- #### 创建并激活 ruyi 虚拟环境

  ```bash
  # 创建虚拟环境，命名为 blink-venv，使用 sipeed-lpi4a profile
  ruyi venv -t gnu-plct-xthead sipeed-lpi4a blink-venv
  # 进入虚拟环境目录
  cd blink-venv
  # 激活虚拟环境
  source ./bin/ruyi-activate
  ```

- #### 使用 ruyi 工具链编译示例代码

  1. **创建工作目录并编写源码**

     ```bash
     # 创建工作目录
     mkdir -p ~/ruyi_led_example && cd ~/ruyi_led_example
     
     # 创建 C 语言源文件
     nano gpio_blink.c
     ```

     在打开的编辑器中粘贴以下完整代码（按 `Ctrl+O` 保存，按回车确认，按 `Ctrl+X` 退出）：

     ```c
     #include <stdio.h>
     #include <stdlib.h>
     #include <unistd.h>
     #include <gpiod.h>
     
     int main()
     {
         int i;
         int ret;
     
         struct gpiod_chip * chip;
         struct gpiod_line * line;
     
         chip = gpiod_chip_open("/dev/gpiochip1");
         if(chip == NULL)
         {
             printf("gpiod_chip_open error\n");
             return -1;
         }
     
         line = gpiod_chip_get_line(chip, 3);
         if(line == NULL)
         {
             printf("gpiod_chip_get_line error\n");
             gpiod_line_release(line);
         }
     
         ret = gpiod_line_request_output(line, "gpio", 0);
         if(ret < 0)
         {
             printf("gpiod_line_request_output error\n");
             gpiod_chip_close(chip);
         }
     
         printf("Starting LED blink on GPIO1_3 (num 427), press Ctrl+C to stop.\n");
         for(i = 0; i < 10; i++) 
         {
             gpiod_line_set_value(line, 1);
             printf("ON\n");
             sleep(1);
             gpiod_line_set_value(line, 0);
             printf("OFF\n");
             sleep(1);
         }
     
         gpiod_line_release(line);
         gpiod_chip_close(chip);
     
         return 0;
     }
     ```

  2. **编译**

     ```bash
     gcc gpio_blink.c -I /usr/include/ -L /usr/lib/riscv64-linux-gnu/ -lgpiod -o gpio_blink
     ls
     ```


- #### 运行程序

  ```bash
  sudo ./gpio_blink
  ```

  **输出结果**

  ```bash
  Starting LED blink on GPIO1_3 (num 427), press Ctrl+C to stop.
  ON
  OFF
  ON
  OFF
  ON
  OFF
  ON
  OFF
  ...
  ```

