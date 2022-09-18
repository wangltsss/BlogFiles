---
title: Pyserial的使用心得
date: 2021-08-17 11:55:32
categories: Programming & Algorithms
tags:
- Python
---

# 关于Pyserial
Pyserial库封装了访问串口的方法。它为在Windows，OSX，Linux，BSD和IronPython上运行的Python提供后端。```serial```对象会自动选择合适的后端。
This module encapsulates the access for the serial port. It provides backends for Python running on Windows, OSX, Linux, BSD (possibly any POSIX compliant system) and IronPython. The module named “serial” automatically selects the appropriate backend.
<!--more-->

# Pyserial库的安装
将python版本更新到2.7及以上，或3.4及以上，然后直接使用pip安装：
```bash
pip install pyserial
```

# Serial对象
导入pyserial包后使用serial.Serial创建。
此处因为业务需要，需要同时在windows和linux或macos下访问此应用。
在windows和linux中，端口名称不相同。
在windows下，端口名形如【COMx】（x为整数）；
在linux下，端口名形如【/dev/xxxxx】（x为具体端口名）。
因此还引入了platform来判断系统以获取不同系统下的端口。
```python3
import serial

	def create_connection(self, port):
        if self._system.lower() == "darwin":
            self.ser = serial.Serial(port="/dev/{}".format(port),
                                     baudrate=115200, #波特率
                                     bytesize=8, #字节大小
                                     stopbits=1) #停止位
        elif self._system.lower() == "windows":
            self.ser = serial.Serial(port=port,
                                     baudrate=115200,
                                     bytesize=8,
                                     stopbits=1)
```
此处应根据实际业务需求设置【校验位】，【超时时间】等属性。
## 常用属性
以下为Serial对象所有的属性：
```python3
port: str #端口名
baudrate: int #波特率
bytesize #可能的值：FIVEBITS, SIXBITS, SEVENBITS, EIGHTBITS
parity #可能的值：PARITY_NONE, PARITY_EVEN, PARITY_ODD PARITY_MARK, PARITY_SPACE
stopbits #可能的值：STOPBITS_ONE, STOPBITS_ONE_POINT_FIVE, STOPBITS_TWO
timeout: float #超时时长
xonxoff: bool #软件流控制的开关
rtscts: bool #硬件(RTS/CTS)流控制的开关
dsrdtr: bool #硬件(DSR/DTR)流控制的开关
write_timeout: float #输出的超时时间
inter_byte_timeout: float #字符间隔超时时长，默认设置为None以禁用
in_waiting: int #返回输入缓冲区（待接收）中的字符数量
out_waiting: int #返回输出缓冲区（待发送）中的字符数量

```
- 其中【port】【baudrate】【bytesize】【parity】【stopbits】【in_waiting】很常用。
- 【timeout】的设置会影响read()方法的行为：
  * 设置为None时，read()会一直从缓冲区中读取数据，直到指定的字符数量已被读取。
  * 设置为0时，read()会立即返回读到的从0到指定数量的字符。
  * 设置为x（x为整数）时，read()会在达到x秒时返回读到的所有字符，或在此之前返回指定数		量的字符。
## 常用方法
```python3

def serial.tool.list_ports()
"""返回所有检测到的串口
"""

serial.Serial.open()
"""打开串口
"""

serial.Serial.close()
"""关闭串口
"""

serial.Serial.read(size=1)
"""从串口中读取最多【size】个的字符
"""

serial.Serial.write(data)
"""向串口发送数据【data】,【data】的类型为bytes, bytearray或str。
"""

serial.Serial.flush()
"""等待所有数据发送完毕
"""
```

# 例子
封装案例：
```python3
import serial

import serial.tools.list_ports

import platform

from time import sleep


class PortManager(object):
	"""操作串口的类

    Attributes:
        _system: str - 当前应用所在环境的系统类型
        ser: serial.Serial - 串口链接的对象
        _serial_port: str - 串口名称
    """

    _system = platform.system()
    ser: serial.Serial
    _serial_port: str

    def set_port(self, port):
    	"""_serial_port的setter"""
        self._serial_port = port

    def get_port(self):
    	"""_serial_port的getter"""
        return self._serial_port

    def list_ports(self):
    	"""获取并返回所有的串口"""
        list_p = list(serial.tools.list_ports.comports())
        list_ports_name = []
        if self._system.lower() == "darwin":
            list_ports_name = [str(i.name) for i in list_p]
        elif self._system.lower() == "windows":
            list_ports_name = [str(i.device) for i in list_p]
        return list_ports_name

    def create_connection(self, port):
    	"""创建串口链接"""
        if self._system.lower() == "darwin":
            self.ser = serial.Serial(port="/dev/{}".format(port),
                                     baudrate=115200,
                                     bytesize=8,
                                     stopbits=1)
        elif self._system.lower() == "windows":
            self.ser = serial.Serial(port=port,
                                     baudrate=115200,
                                     bytesize=8,
                                     stopbits=1)

    def send_data(self, value):
    	"""向串口发送数据"""
        write_data = bytearray.fromhex(value)
        try:
            sleep(0.1)
            if self.ser.out_waiting:
                self.ser.reset_output_buffer()
            if self.ser.write(write_data):
                self.ser.flush()
            else:
                if self.ser.write(write_data):
                    self.ser.flush()
            self.ser.reset_output_buffer()
            return True
        except Exception as e:
            print(e)
            self.ser.reset_output_buffer()
            return False

    def read_data(self):
    	"""从串口读取数据"""
        try:
            sleep(0.1)
            if self.ser.in_waiting:
                bs = self.ser.read(self.ser.in_waiting).hex()
                self.ser.reset_input_buffer()
                res = ''
                for i in range(len(bs)):
                    res += bs[i]
                    if i % 2 == 1:
                        res += ' '
                res = res.rstrip(' ')
                return res
            else:
                sleep(0.1)
                if self.ser.in_waiting:
                    bs = self.ser.read(self.ser.in_waiting).hex()
                    self.ser.reset_input_buffer()
                    res = ''
                    for i in range(len(bs)):
                        res += bs[i]
                        if i % 2 == 1:
                            res += ' '
                    res = res.rstrip(' ')
                    return res
                else:
                    self.ser.reset_input_buffer()
                    return ""
        except Exception as e:
            print(e)
            self.ser.reset_input_buffer()
            return None

    def close(self, *ser):
    	"""关闭串口"""
        if ser:
            ser[0].close()
        else:
            self.ser.close()
```
# 小结
这是用bootstrap构建的一个web app，主要用途是配置测试公司客制化的继电器板卡。
此app就是利用pyserial与串口通信。

配置组模式：
![配置组模式](https://img-blog.csdnimg.cn/77e1cb3e8a1f44a9bc17ebff2dd9ce4f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDczMzMz,size_16,color_FFFFFF,t_70#pic_center)


板卡控制面板：
![板卡控制面板](https://img-blog.csdnimg.cn/7ab9122042ec4ef789ae30cf09504336.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDczMzMz,size_16,color_FFFFFF,t_70#pic_center)

板卡配置面板：
![板卡配置面板](https://img-blog.csdnimg.cn/13190e0c95464b2f924fdde87921cd47.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDczMzMz,size_16,color_FFFFFF,t_70#pic_center)


连接错误处理：
![错误处理](https://img-blog.csdnimg.cn/83a747f39605441f80378d995b55463d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDczMzMz,size_16,color_FFFFFF,t_70#pic_center)


多板卡级联时的板卡切换：
![多板卡级联的切换板卡](https://img-blog.csdnimg.cn/cae4af3ecaab4583a4fadaa91a2e8575.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDczMzMz,size_16,color_FFFFFF,t_70#pic_center)

