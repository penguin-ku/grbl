system.c 注释

对外接口

| 接口                               | 参数  | 返回值                         | 说明                                                         |
| ---------------------------------- | ----- | ------------------------------ | ------------------------------------------------------------ |
| system_init                        | void  | void                           | 配置外部控制针脚<br />启用控制针脚变化引发的中断             |
| system_check_safety_door_ajar      | void  | byte                           | 返回安全门的状态<br />此状态依赖于安全门针脚的状态，也即外部输入<br />默认为false，表示关<br />如果未启用安全门功能（ 编译选项），则一直返回关 |
| system_excute_line                 | byte* | byte                           | 执行系统命令<br />上位机通过串口发送的命令行，首字符为'\$'<br />"\$\0"：请求帮助，回复帮助信息<br />"\$\$\0"：输出grbl的设置<br />"\$G\0"：输出当前GParser的模态<br />"\$C\0"：只有在grbl空闲以及就绪的状况。 |
| system_execute_startup             | void  | byte*                          | 执行预置的g代码块，存储在EEPROM中                            |
| system_convert_axis_steps_to_mpos  | float | int* step<br />byte idex       | 获取指定轴的机器位置                                         |
| system_convert_array_steps_to_mpos | void  | float* pos[out]<br />int* step | 获取所有轴的机器位置                                         |
|                                    |       |                                |                                                              |

命令行规格

系统命令必须'$'开头

| 内容       | 意义                                                         |
| ---------- | ------------------------------------------------------------ |
| $          | 请求帮助，回复帮助信息                                       |
| $$         | 请求grbl的设置，回复设置信息                                 |
| $CG        | 请求g-code当前模态，回复g-code模态                           |
| $C         | 切换g-code模式，处于check模式，锁住所有的规划以及执行（运动、主轴等），即处于预分析状态，不产生实际加工。<br />只有grbl处于就绪且空闲状态可以使用该命令切换到check模式，再次输入\$C切换回正常加工模式<br /><br /><font color="green">此处为何用Check，而不是Toggle。grbl的作者觉得Toggle也模凌两可，还不如Check。个人觉得他在强词夺理</font> |
| $X         | 报警解除。此处特别的是对于安全门（锁机床），这两个无法被解除 |
| $#         | 查看NGC参数（坐标、偏置等）                                  |
| $H         | 回原点。安全门打开或者机床锁状态，无法回原点                 |
| $I         | 查看构建信息，构建信息存储在EEPROM                           |
| $I=xxxx    | 设置构建信息，构建信息存储在EEPROM                           |
| \$RST=\0\$ | 恢复出厂设置，脉冲频率、速度等等。具体见default_generic.h。这些设置都存储在EEPROM中 |
| \$RST=\0#  | 恢复参数，工件坐标信息重置为空。这些信息存储在EEPROM中。     |
| \$RST=\0*  | 恢复所有，设置、参数、构建信息等。这些信息存储在EEPROM中。   |
| \$N\0      | grbl每次上电或断电自动执行的g代码块。可以用于设置g代码默认模态或执行固定的任务。<br />此命令为查询grbl提供的两个g代码块。返回值为<br />\$N0=xxxxx<br />\$N1=xxxxx |
| $N0=xxxx   | 设定第一个g代码块                                            |
| $N1=xxxx   | 设定第二个g代码块                                            |
| \$x=val    | 参数设定。此处具体参见https://github.com/grbl/grbl/wiki/Configuring-Grbl-v0.9 （实在有点多） |

