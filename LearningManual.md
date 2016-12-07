# BLDC_DRV11873
A BLDC Control Project With DRV11873 Driver, STM32F030F4P6 MCU, OLED, Speed and Current Detection etc.

# Bug
- 板子回来后， 发现Top Layer的一些封装没有出来， 如图所示： ![](http://7xtauc.com1.z0.glb.clouddn.com/B2.png)
没办法， 用刀片一条条刮开， 3块刮了一个多小时， 手疼... 原因大概是我勾选了：
![](http://7xtauc.com1.z0.glb.clouddn.com/Bug1.png)
不该勾选的...

- 把DRV11873的22uF滤波电容改成了25V， 10uF的3528钽电容, 已在V1.1原理图中修正.  
- 5V转3.3V的SP6205的1 3引脚忘记连起来， 飞了一根线， 已在V1.1 PCB中修正.
- DRV11873的CS引脚电阻R10, 从10k换成了3.3k，已在V1.1原理图修正.

# 调试
- 程序中printf乱码， 原因是 **晶振用的12M**， 而不是8M.
- 由于加了S8050隔离3.3V和5V， 起了一个反相器的作用，**PWM是反相的**， MCU PWM值越小， Motor转速越高.  
- 大电机(3线)转， 小电机(4线)不转， 原因是SH1.0-4接口COM引脚虚焊.  
- 抖/不转， DRV11873引脚虚焊或短路.
- OLED用两面粘粘贴到了PCB上.  
- BLDC任意交换两个引脚， 转向相反.  
- 拨码开关5 6: 默认关电机有COM; 闭合表示无COM; 实测3/4线均ON即可.
- 官方评估板原理图如下:　![](http://7xtauc.com1.z0.glb.clouddn.com/DRV11873%E5%AE%98%E6%96%B9%E8%AF%84%E4%BC%B0%E6%9D%BF%E5%8E%9F%E7%90%86%E5%9B%BE.png)
图中的FS, FG, RD, FR, COM的连接方式值得参考.

# UART Printf
这一节实现
