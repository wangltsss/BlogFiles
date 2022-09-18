---
title: Notes on using Pyserial
tags:
  - Python
categories: Programming & Algorithms
date: 2021-08-17 11:55:32
---


# About ***Pyserial***
This module encapsulates the access methods for serial ports. It provides backends for Python running on Windows, OSX, Linux, BSD (possibly any POSIX compliant system) and IronPython. The module named “serial” automatically selects the appropriate backend.
<!--more-->

# Installing ***Pyserial***
Install using ***pip***：
```bash
pip install pyserial
```

# Serial object
The object is ```serial.Serial```.
Due to business needs, this app requires compatibility with Windows and MacOS.
However, port names are different in Windows and MacOS.
In Windows, port names have a format of [COMx], where x is an integer；
In MacOS, port names are like [/dev/xxxxx], where x is the specific port name.
If you have similar needs, remember to import the ```platform``` package.
```python3
import serial

	def create_connection(self, port):
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
```

## Common properties
Listed below are all properties of a ```Serial``` object：
```python3
port: str #port name
baudrate: int
bytesize #possible values：FIVEBITS, SIXBITS, SEVENBITS, EIGHTBITS
parity #possible values：PARITY_NONE, PARITY_EVEN, PARITY_ODD PARITY_MARK, PARITY_SPACE
stopbits #possible values：STOPBITS_ONE, STOPBITS_ONE_POINT_FIVE, STOPBITS_TWO
timeout: float 
xonxoff: bool #switch of software flow control
rtscts: bool #switch of hardware(RTS/CTS) flow control
dsrdtr: bool #switch of hardware(DSR/DTR) flow control
write_timeout: float 
inter_byte_timeout: float #set to None to disable
in_waiting: int 
out_waiting: int 

```

- [port], [baudrate], [bytesize], [parity], [stopbits], [in_waiting]are very commonly used.
- Setting of [timeout] will impact the behavior of ```read()``` method：
  * When set to None, ```read()``` will keep trying to read data from buffer until specified number of characters are read.
  * When set to 0, ```read()``` will immediately return all characters read ranges from 0 to specified number.
  * When set to x (x is an integer), ```read()``` will return specified number of characters before x seconds have elapsed, or ```read()``` will return all characters read after x seconds have elapsed.
  
## Common methods
```python3

def serial.tool.list_ports()
"""return all ports scanned
"""

serial.Serial.open()
"""open serial port
"""

serial.Serial.close()
"""shut serial port
"""

serial.Serial.read(size=1)
"""read at most n=size characters from the port
"""

serial.Serial.write(data)
"""send what's inside data to the port. The type of data must be bytes, bytearray or str.
"""

serial.Serial.flush()
"""Wait for all data to be sent.
"""
```

# Example
Encapsulation instance：
```python3
import serial

import serial.tools.list_ports

import platform

from time import sleep


class PortManager(object):
	"""The class that manages the port

    Attributes:
        _system: str  #The system where the app is running
        ser: serial.Serial  #Serial port object
        _serial_port: str  #Serial port name
    """

    _system = platform.system()
    ser: serial.Serial
    _serial_port: str

    def set_port(self, port):
    	"""_serial_port setter"""
        self._serial_port = port

    def get_port(self):
    	"""_serial_port getter"""
        return self._serial_port

    def list_ports(self):
    	"""Get and return all ports scanned"""
        list_p = list(serial.tools.list_ports.comports())
        list_ports_name = []
        if self._system.lower() == "darwin":
            list_ports_name = [str(i.name) for i in list_p]
        elif self._system.lower() == "windows":
            list_ports_name = [str(i.device) for i in list_p]
        return list_ports_name

    def create_connection(self, port):
    	"""Create serial port connection"""
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
    	"""Send data to the port"""
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
    	"""Read data from the port"""
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
    	"""close the port"""
        if ser:
            ser[0].close()
        else:
            self.ser.close()
```

# Conclusion
This is a web app built by me using ***Bootstrap***, which is mainly used to test the customized relay boards of the company I worked in. This app utilized pyserial to communicate through serial ports.

Configure group mode：
![Configure group mode](https://img-blog.csdnimg.cn/77e1cb3e8a1f44a9bc17ebff2dd9ce4f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDczMzMz,size_16,color_FFFFFF,t_70#pic_center)


Board control panel：
![Board control panel](https://img-blog.csdnimg.cn/7ab9122042ec4ef789ae30cf09504336.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDczMzMz,size_16,color_FFFFFF,t_70#pic_center)

Board configure panel：
![Board configure panel](https://img-blog.csdnimg.cn/13190e0c95464b2f924fdde87921cd47.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDczMzMz,size_16,color_FFFFFF,t_70#pic_center)


Connection error handling：
![Error handling](https://img-blog.csdnimg.cn/83a747f39605441f80378d995b55463d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDczMzMz,size_16,color_FFFFFF,t_70#pic_center)


Change target board in multiple boards cascading：
![Change target board in multiple boards cascading](https://img-blog.csdnimg.cn/cae4af3ecaab4583a4fadaa91a2e8575.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDczMzMz,size_16,color_FFFFFF,t_70#pic_center)

