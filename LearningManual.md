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

# 开发环境
主要使用STM32的HAL库进行开发， 需要安装的软件：
- Keil MDK V5
- [STM32CubeMX](http://www.st.com/en/development-tools/stm32cubemx.html), 安装完成后, 在Help -> Install New Libraries中找到 Firmware Package for Family STM32F0, 勾选并安装. 或者到 [keil网址](http://www.keil.com/dd2/pack/) 下载后安装.

# UART Printf
**MCU选型**  
- 打开STM32CubeMX -> New Project -> MCU找到STM32F030F4Px后双击.  

**配置时钟**    
- Pinout选项卡->Peripherals -> RCC -> High Speed Clock(HSE) 选择 Crystal/Ceramic Resonator.
- Clock Configuration选项卡配置如图:
![Clock Configuration](http://7xtauc.com1.z0.glb.clouddn.com/Clock%20Configuration.png)  

**配置USART**   
- Pinout选项卡 -> USART -> Mode 选择 Asynchronous(异步的).  
- Configuration -> USART1 -> Parameter Settings选项卡 -> BaudRate 设置为 115200 Bits/s, 其他不变, 依次点击Apply, OK.  

**生成工程**  
- Project菜单 -> Code Generator选项卡 -> 勾选Generator peripheral initialization as a pair of '.c/.h' files per peripheral:
![](http://7xtauc.com1.z0.glb.clouddn.com/Code%20Generator.png)
- Project选项卡: 填写工程名, 选择工程位置, 选择IDE, 如图:    
![](http://7xtauc.com1.z0.glb.clouddn.com/Project.png)
- Open Project.

**添加printf支持**   
- usart.c 中 `/* USER CODE DEGIN 1 */` 和 `/* USER CODE END 1 */` 之间添加如下代码:  

```C
/* USER CODE BEGIN 1 */
#ifdef __GNUC__
  /* With GCC/RAISONANCE, small printf (option LD Linker->Libraries->Small printf
     set to 'Yes') calls __io_putchar() */
  #define PUTCHAR_PROTOTYPE int __io_putchar(int ch)
#else
  #define PUTCHAR_PROTOTYPE int fputc(int ch, FILE *f)
#endif /* __GNUC__ */
/**
  * @brief  Retargets the C library printf function to the USART.
  * @param  None
  * @retval None
  */
PUTCHAR_PROTOTYPE
{
  /* Place your implementation of fputc here */
  /* e.g. write a character to the EVAL_COM1 and Loop until the end of transmission */
  HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xFFFF);

  return ch;
}
/* USER CODE END 1 */
```

**修改main.c**  
- main.c添加:  

```C
while (1)
{
/* USER CODE END WHILE */

/* USER CODE BEGIN 3 */
  printf("Hello, World!\n");
  HAL_Delay(1000);
}
/* USER CODE END 3 */
```  

**CP2102驱动安装**  
- 连接USB和 J-link 如图:  
![J-link](http://7xtauc.com1.z0.glb.clouddn.com/J-link.png)  
 联网的win10电脑会自动安装CP2102或者J-link的驱动, CP2102的驱动也可以到 [USB to UART Bridge VCP Drivers](https://www.silabs.com/products/mcu/Pages/USBtoUARTBridgeVCPDrivers.aspx)下载CP2102的驱动.

**Keil设置**  
- keil配置如下:  
![Debug](http://7xtauc.com1.z0.glb.clouddn.com/Debug.png)  
![Flash Download](http://7xtauc.com1.z0.glb.clouddn.com/Flash.png)
OK, OK.  

**编译下载验证**  
先编译, 0 error 0 warning.  后下载:  
![](http://7xtauc.com1.z0.glb.clouddn.com/%E7%BC%96%E8%AF%91%E4%B8%8B%E8%BD%BD.png)  
下载后打开XCOM软件, 选择串口, 设置波特率115200, 打开串口, 即可看到每秒打印出一个 Hello, World!   
![XCOM](http://7xtauc.com1.z0.glb.clouddn.com/XCOM.png)
