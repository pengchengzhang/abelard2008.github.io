---
layout: post
status: publish
published: true
title: "StarterWare How to build the first example LEDBlink "
author: Dillon Peng
date: Mon Jul  2 17:36:43 CST 2018
comments: []
---

  有一块AM3354的开发板和一个不太稳定的XDS100V3，提供了原理图，u-boot可执行文件MLO和u-boot.img（不提供源代码和裸板开发，裸板开发可理解为基于TI的StareWare开发小应用)，部分linux驱动源码，要想从源码移植U-boot到这个板子上，基本思路是，在TI的CCS里编译一个StarterWare小例子，如题，让板子上的一个LED闪烁，然后到TI找AM335X的u-boot SDK源码，根据板子的硬件资源一步步调整，直到可以装载和运行Linux。
  这篇文章讨论如何编译第一个StartWare小例子，来熟悉AM335X的开发环境CCS。以下的讨论都是在Fedora core 26 x64下完成的。

### **安装CCS8.0和StarterWare**
  下载链接

   [CCS 8.0.0.00016](http://processors.wiki.ti.com/index.php/Download_CCS#Code_Composer_Studio_Version_8_Downloads)


   [StartWare 02_00_01_01](http://software-dl.ti.com/dsps/dsps_public_sw/am_bu/starterware/latest/index_FDS.html)

  为了能使用XDS100V3，在安装好CCS后，需要安装驱动，执行安装目录下的一个脚本*install_drivers.sh*,具体位置为`~/ti/ccsv8/install_scripts/install_drivers.sh`。


在开始后续工作之前，可以看一个视屏[How to use StarterWare](https://youtu.be/w_XFGKHXqbw)和一个文档`AM335X_StarterWare_02_00_01_01/docs/UserGuide_02_00_01_01.pdf`

### **开发gpioLEDBlink步骤**
1. 从桌面或其他位置进入CCS开发环境，选定一个心仪的workspace位置。


2. 导入gpioLEDBlink项目需要的相关项目

    从File->Import打开Import窗口，选择Code Composer Studio -> CCS Projects, 点击下一步后，进入 `Import CCS Eclipse Projects`, 在单选项`Select search directory`旁边，指定StarterWare例程项目所在的目录：`AM335X_StarterWare_02_00_01_01/build/armv7a/cgt_ccs/`后，在Discovered Porject下方将列出所有项目，勾选本次需要的boot, drivers, platform,system和utils，gpioLEDBlink，点击finish按钮后，就会在左侧的工作区显示这几个项目

3. 设置编译版本
    在gpioLEDBlink项目上，点击右键，选择Build Configurations -> Set Active -> Debug
    
4. 选定编译器和链接命令文件
    在gpioLEDBlink项目上，点击右键，选择最后一项属性properties，在General项的右则，需要在Device栏为选择Variant为`AM33x - Cortex A8`和`AM3354 [Cortex A]`; 在Tool-chain栏内，Compiler version选为：`TI v18.1.1.LTS`, Linker command file选为AM335x.cmd(这个文件来自CCS安装目录ti/ccsv8/ccs_base/arm/include，为了根据硬件条件修改，最后拷贝到其它目录下)。

5. 编译辅助项目
    除了gpioLEDBlink项目外，在每个项目上点击右键，选择build Project，将会在`AM335x_StarterWare_02_00_01_01`安装的主目录的binary目录下生成相应的*.lib文件，如platform项目生成了：`binary/armv7a/cgt_ccs/am335x/beaglebone/platform/Debug/platform.lib`

6. gpioLEDBlink项目的头文件目录和lib目录
    打开项目的属性页，找到Build -> ARM Compiler -> Include Options添加"soc_AM335x.h", "evmskAM335x.h", "gpio_v2.h"所在的目录，比如"soc_AM335x.h"在“AM335X_StarterWare_02_00_01_01/include/hw”目录下，则需要将该目录添加进来，可以使用绝对路径和相对路径，StarterWare原始设置使用的相对路径，也可以使用如"${CG_TOOL_ROOT}/include"。

    找到Build->ARM Linker -> File Search Paths, 在右侧的Include Library file or command file as input(--library, -L), 添加各个辅助lib的路径和文件名，如platform.lib, `../../../../../../../binary/armv7a/cgt_ccs/am335x/beaglebone/platform/${ConfigName}/platform.lib`

7. 设置post build steps
    打开gpioLEDBlink的属性也，打开Build，选择右侧的Steps的下方，有一栏Post-build steps，原始内容如下：

       "${CCS_INSTALL_ROOT}/utils/tiobj2bin/tiobj2bin.bat"  "binary/armv7a/cgt_ccs/am335x/beaglebone/gpio/${ConfigName}/${ProjName}.out"  "binary/armv7a/cgt_ccs/am335x/beaglebone/gpio/${ConfigName}/${ProjName}.bin"  "${CG_TOOL_ROOT}/bin/armofd.exe"  "${CG_TOOL_ROOT}/bin/armhex.exe"  "${CCS_INSTALL_ROOT}/utils/tiobj2bin/mkhex4bin.exe" & "D:/ti/AM335X_StarterWare_02_00_01_01/tools/ti_image/tiimage.exe" "0x80000000" "NONE" "binary/armv7a/cgt_ccs/am335x/beaglebone/gpio/${ConfigName}/${ProjName}.bin" "binary/armv7a/cgt_ccs/am335x/beaglebone/gpio/${ConfigName}/${ProjName}_ti.bin"


   这里的命令都是用的windows下的名称，需要改称Linux下的命令，如去掉`.bat`,`.exe`等后缀，如果遇到相对路径，也需要作相应的调整：
        
       "${CCS_INSTALL_ROOT}/utils/tiobj2bin/tiobj2bin"  "../../../../../../../binary/armv7a/cgt_ccs/am335x/beaglebone/gpio/${ConfigName}/${ProjName}.out"  "../../../../../../../binary/armv7a/cgt_ccs/am335x/beaglebone/gpio/${ConfigName}/${ProjName}.bin"  "${CG_TOOL_ROOT}/bin/armofd"  "${CG_TOOL_ROOT}/bin/armhex"  "${CCS_INSTALL_ROOT}/utils/tiobj2bin/mkhex4bin"
"../../../../../../../tools/ti_image/tiimage" "0x80000$000" "NONE" "../../../../../../../binary/armv7a/cgt_ccs/am335x/beaglebone/gpio/${ConfigName}/${ProjName}.bin" "../../../../../../../binary/armv7a/cgt_ccs/am335x/beaglebone/gpio/${ConfigName}/${ProjName}_ti.bin"
@echo "finished successfully"

8. 修改LED对应的管脚
打开板子的原理图，发现led在GPIO3的第16个pin上，因此需要修改定义，同时，需要修改`main()`的语句为：


       #define GPIO_INSTANCE_ADDRESS           (SOC_GPIO_3_REGS)
       #define GPIO_INSTANCE_PIN_NUMBER        (16)

       int main()
        {

        /* Enabling functional clocks for GPIO3 instance. */
        GPIO3ModuleClkConfig();

       ...

9. 连接，下载和调试
    打开Window -> Show View -> Target Configurations, 创建一个New Target Configurations，之后，双击新建的ccxml文件，然后选择`Texas Instruments XDS100v3 USB Debug Probe` 和 `AM3354`,点击`Save Configuration`的save按钮，之后，点击`Test Connection`, 如果链接成功，则可以Run -> load -> gpioLEDBlink, 然后在gpioLEDBlink，右键选择Debug As-> 1. Code Composer Debug Session, 之后将停在`main()`函数的入口出，可以全速运行，看看是否闪烁。