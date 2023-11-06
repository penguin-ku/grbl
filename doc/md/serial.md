Serial.c 注解

对外职责：

| 接口                       | 参数 | 返回值 | 说明                                                         |
| -------------------------- | ---- | ------ | ------------------------------------------------------------ |
| serial_init                | void | void   | 初始化串口<br /><br />初始化串口的参数：波特率<br />使能接受与发送<br />启用接收的中断<br /><br />此处可以继续增加停止位/字节长/校验位等参数初始化 |
| serial_write               | void | byte   | 发送数据<br /><br />实现：<br />1、写入本地ringbuffer<br />2、启用发送空闲中断<br />3、发送空闲中断从ringbuffer获取数据，写入物理针脚UDR0<br /> |
| serial_read                | byte | void   | 从接收RingBuffer中读取数据<br />主程序调用                   |
| serial_get_rx_buffer_count | void | byte   | 接收缓冲区堆积的字节数                                       |
| serial_get_tx_buffer_count | void | byte   | 发送缓冲区堆积的字节数                                       |
| serial_reset_read_buffer   | void | void   | 清空接收缓冲区，用于紧停与重置                               |

内部实现

- 发送

  - 提供128个字节长度的发送缓冲区serial_tx_buffer(借助头、尾两个游标化身环形缓冲区)
  - 外部的发送数据都是先写到serial_tx_buffer，然后检查是否启用了发送空闲中断，若未启用，则启用
  - 发送空闲的中断中从serial_tx_buffer中按照FIFO的规格逐个获取字节，然后写入物理通道UDR0。当serial_tx_buffer为空时，关闭发送空闲中断
  - 外部发送数据时，若发现发送缓冲区满，则触发系统重启

- 接收

  - 提供128个字节长度的发送缓冲区serial_rx_buffer(借助头、尾两个游标化身环形缓冲区)

  - 初始化之后，启用接收中断

  - 接收中断触发时，读取物理通道UDR0上的数据

  - 对接收到的数据进行判断

    - '?'：紧停。系统状态位sys_rt_exec_state设置紧停标志

    - '~'：循环加工。系统状态位sys_rt_exec_state设置循环加工标志

    - '!'：锁机床。系统状态位sys_rt_exec_state设置锁机床标志

    - '@'：安全门。系统状态位sys_rt_exec_state设置安全门标志

    - 0x18：重置。直接调用重置例程。

    - 其他数据追加到serial_rx_buffer。如果serial_rx_buffer满，则不读数据

      <font color="green">此处设定很神奇，为何不和发送缓冲区满一样触发重置</font>
      
      <font color="red">另外此处的?、~、!、@不交给system.c去执行，而是直接此处开始执行的原因在于，此四种的优先级高。如果放到队列中等待protocol的循环再派发给system.c去执行命令行，会带来时延，而这几个操作对于时延是非常敏感的。所以此处才如此实现（虽然丑陋，但相对高效，当然也可以引入优先级的概念，即buffer分高低两种。grbl的作者相对而言，追求简易而非优雅）</font>
  
  - 外部读取数据时，从serial_rx_buffer按照FIFO原则返回数据。如果serial_rx_buffer为空，则返回0xFF。

