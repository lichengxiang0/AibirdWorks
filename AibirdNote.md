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


# MB2天宝板卡  
MB2板卡使用需要注意版本号，原来使用的是3.04版本，这个版本最高串口波特率为115200，不能使用230400，同时在写3.04驱动时，配置报文增加GSV对RTCM流的速率有影响，所以我们在3.04这个版本在配置报文时关掉了GSV。在3.04以后的版本，比如我们使用的3.62，在配置这个串口波特率时都是230400.但是后面的版本出现最大的问题就是飞点，会有几秒种的时间GGA不出经度和纬度的数据，导致飞机无法正常飞行。使用双GPS另外加一个M8N只是让飞机出现飞点后安全返回。M8N精度不高，成本低，不属于高精度板卡。  



```c++
const char comm[]= "$PASHS,POP,20\r\n"\
                    "$PASHS,NME,ZDA,B,ON,3\r\n"\
                    "$PASHS,NME,GST,B,ON,3\r\n"\
                    "$PASHS,NME,GSV,B,OFF\r\n"\
                    "$PASHS,NME,GGA,B,OFF\r\n"\
                    "$PASHS,NME,POS,B,ON,0.05\r\n";


```  
GGA和POS不能同时开启，因为飞控GPS驱动就是从POS取数据，当两者都开启了，仍然从POS获取数据。  
MB2板卡的配置报文开头加了"$PASHS,POP,20\r\n"，这一句表示内部数据跟新速率为20HZ，后面的3和0.05单位都是秒。  
