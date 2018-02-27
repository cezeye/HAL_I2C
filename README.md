# HAL_I2C
V0.1 对从机模式下的HAL库I2C 函数HAL_I2C_Slave_Sequential_Receive_IT的讨论

例程位于 ...\STM32Cube_FW_F0_V1.9.0\Projects\STM32F072RB-Nucleo\Examples\I2C\I2C_TwoBoards_RestartAdvComIT
平台为STM32F0 HAL库

首先对I2C协议做一个介绍
硬件：I2C总线驱动是漏极开路，也就是说接受发送端只有拉低总线的操作而没有拉高总线的操作.
协议：在最基础的模式下I2C通信分为地址帧和数据帧.
      地址帧7位的从机地址+R/W位 数据帧位N Bytes数据，每发送完1 Byte接收端就要发出ACK表示接收成功.
      主机占用总线发出START开始通信.数据发送完毕后主机发送STOP标志释放总线

库函数：
  HAL库的I2C函数有大概4大类型
    1.普通的 Blocking 型
    2.普通的 Non-Blocking 型
    3.DMA 型
    4.Sequential 型
    前三种是基于Start to Stop整个过程。函数需要设置TIMEOUT就可以直接使用。对于MCU操作Sensor,Bootload其他设备等操作而言很方便，但是对于双机通信的情况则兼容性不强.
    而第四种函数则是基于帧过程。那么只要主机没有释放总线, 接收端在接收信息时就可以根据情况转变数据传输方向.
  
  
这个代码是基于第4种类型的I2C函数。应用是 双机通信中的带有Repeat Start操作的I2C通信.

1.HAL_I2C_Slave_Sequential_XX_IT()的程序流图
![](https://github.com/stonechenSJ/HAL_I2C/blob/master/HAL_I2C%20structure.png)

2.
