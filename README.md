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
    
    这个代码是基于第4种类型的I2C函数。应用是 双机通信中作为从机的带有Repeat Start操作的I2C通信.
  
  1.HAL_I2C_Slave_Sequential_XX_IT()的程序流图  
  ![](https://github.com/stonechenSJ/HAL_I2C/blob/master/HAL_I2C%20structure.png)
  
  2.在I2C通信中，提供的中断源有接收寄存器，发送寄存器，stop位，NACK位，ADDR位匹配.  
  在I2C的ISR中,可以看到有5个去处,其中4个分别对应器件地址匹配,发送完成中断,接收完成中断,STOP置位中断.  
  可以看到,其对应的从机接收程序中的中断位置.
  ![](https://github.com/stonechenSJ/HAL_I2C/blob/master/SlaveRx.PNG) .
  sequential类型的通信的一个特征是,每当完成一个阶段的通信后,会关闭相关的中断,然后提供Callback函数做处理,如果主机没有发出停止位,那么从机需要在Callback函数中利用HAL_I2C_Slave_Sequential_XX_IT()函数再次打开中断，知道主机发出停止位。  
  那么也就是说，接收端可以每次接收1 Byte数据然后做处理再在slaveRxCpltCallback()中打开中断循环接收1Byte的数据，直到从机指定的长度或者主机发出停止位。
