# 串口

	关于PX4的串口8接收不到数据，现在无法解释。
	关于串口的open函数，需要加上open("/dev/ttyS6",O_RDWR|O_NOCTTY|O_NONBLOCK )
对于O_NONBLOCK这个参数一定要加上，否则每次会阻塞，每当读到数据时才会发送一个数据，否则一直等待。

# GPS双串口  
	在PX4中，默认的一个GPS串口是串口4（ttyS3），在rcs启动脚本中，默认添加了 gps start 所以在上电时会自动开启，可以在rcs启动脚本中注释掉，这样上电的时候就不会自己启动，可以更好的看到GPS的启动现象。选取第二个串口作为gps的双串口，我们选择TELEM2(ttyS2)。上电接上飞控，开启QGC，在控制台上输入 gps start -d /dev/ttyS3 -dualgps /dev/ttyS2  有可能会输入两次，看到控制台提示两个GPS都已经看起就可以了。然后输入gps status，就可以看到两个GPS的数据。如果看不到，需要进入nuttx-configs/px4fmu-v2/nsh/defconfig 打开defconfig  然后比较原始的串口4和选取的第二个GPS串口的配置时候相同，配置如下：  
```C++

#
# USART3 Configuration
#
CONFIG_USART3_RXBUFSIZE=300
CONFIG_USART3_TXBUFSIZE=300
CONFIG_USART3_BAUD=57600
CONFIG_USART3_BITS=8
CONFIG_USART3_PARITY=0
CONFIG_USART3_2STOP=0
#CONFIG_USART3_IFLOWCONTROL=y
#CONFIG_USART3_OFLOWCONTROL=y
# CONFIG_USART3_DMA is not set

#
# UART4 Configuration
#
CONFIG_UART4_RXBUFSIZE=300
CONFIG_UART4_TXBUFSIZE=300
CONFIG_UART4_BAUD=57600
CONFIG_UART4_BITS=8
CONFIG_UART4_PARITY=0
CONFIG_UART4_2STOP=0
# CONFIG_UART4_IFLOWCONTROL is not set
# CONFIG_UART4_OFLOWCONTROL is not set
# CONFIG_UART4_DMA is not set

```  

需要注释掉UART3的最后两句，保持和串口4相同，然后一定要 make clean 清理一下，在重新编译，下载程序验证。  
需要注意几个文件：第一，启动脚本 rcs，里面添加了gps start，会在上电时自动启动。  第二，配置文件defconfig，里面有关于需要用到模块的配置。  
第三，gps的驱动程序，里面介绍了GPS驱动的实现。  
