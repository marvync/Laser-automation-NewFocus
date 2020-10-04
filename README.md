# Automation for Laser NewFocus TLB-6700 using Python

Jupyter notebook (.ipynb) to operate remotely the Tunable Diode Laser TLB-6700 using python.

This script is a first version and some improvements must be done! 

As an ideia, we can create a class called **lasers** and unify the control of all lasers that are connected in the intranet or internet.


**Requirements:**

- NewFocus laser driver (.exe) has to be installed.
- The code was tested to run on the [conda environment](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html) attached (.yml).

## NewFocus TLB-6700


```python
import os
import sys
import ipdb
import numpy as np

import clr

from time import sleep
from clr import System
from System.Text import StringBuilder
from System import Int32
from System.Reflection import Assembly
```


```python
sys.path.append('C:\\Program Files\\New Focus\\New Focus Tunable Laser Application\\')
clr.AddReference('UsbDllWrap')

import Newport
tlb = Newport.USBComm.USB()
```


```python
answer = StringBuilder(64)

ProductID = 4106
DeviceKey = '6700 SN1012'

def tlb_open():
    tlb.OpenDevices(ProductID, True)

def tlb_close():
    tlb.CloseDevices()

# tab = tlb.GetDeviceTable()
# Empty buffer
# out = tlb.Read(DeviceKey, answer)
```


```python
tlb_open()

def tlb_query(msg):
    answer.Clear()
    tlb.Query(DeviceKey, msg, answer)
    return answer.ToString()

tlb_query('*RST') # Performs a soft reset of the instrument.
tlb_query('*IDN?')
```




    'New_Focus 6700 v2.4 03/19/14 SN1012'



### TLB Properties


```python
def tlb_set_power(P):
    # P in mW
    tlb_query('SOURce:POWer:DIODe {}'.format(P))
    P_current = tlb_query('SOURce:POWer:DIODe?')
    return print('P_current = {} mW'.format(P_current))

def tlb_set_wavelength(λ):
    # λ in nm
    tlb_query('SOURce:WAVElength {}'.format(λ))
    tlb_query('OUTPut:TRACK 1')
    λ_current = tlb_query('SOURCE:WAVELENGTH?')
    return print('λ_current = {} nm'.format(λ_current))
    
def tlb_set_scan_limits(λi, λf):
    # λi, λf in nm
    tlb_query('SOURce:WAVElength:START {}'.format(λi))
    tlb_query('SOURce:WAVElength:STOP {}'.format(λf))
    return print('λ_init = {} nm'.format(tlb_query('SOURce:WAVElength:START?'))  + '\n' +  
                 'λ_final = {} nm'.format(tlb_query('SOURce:WAVElength:STOP?')))

def tlb_set_scan_speeds(forward, backward):
    # forward, backward in nm/s
    tlb_query('SOURce:WAVE:SLEW:FORWard {}'.format(forward))
    tlb_query('SOURce:WAVE:SLEW:RETurn {}'.format(backward))
    return print('Forward speed = {} nm/s'.format(tlb_query('SOURce:WAVE:SLEW:FORWard?')) + '\n' + 
                 'Backward speed = {} nm/s'.format(tlb_query('SOURce:WAVE:SLEW:RETurn?')))

def tlb_scan(boolean):
    # boolean is True or False
    tlb_query('SOUR:WAVE:DESSCANS 50')
    if boolean:
        tlb_query('OUTPut:SCAN:START')
    else:
        tlb_query('OUTPut:SCAN:STOP')
```


```python
# tlb_set_wavelength(1550)
# tlb_set_scan_limits(1550,1560)
tlb_set_scan_speeds(0.1, 5)
tlb_scan(True)
```

    Forward speed = 0.10 nm/s
    Backward speed = 5.00 nm/s



```python
tlb_scan(False)
```


```python
# print(tlb_query('OUTPut:SCAN:START?'))
tlb_query('*OPC?')
```




    '0'




```python
# tlb_open()

# tlb_set_wavelength(1550)
tlb_set_scan_limits(1550,1560)
# tlb_scan_limits()

# tlb_query('SOURce:WAVE:SCANCFG')
# tlb_query('OUTPut:TRACK 1')
```

    λ_init = 1550.00 nm
    λ_final = 1560.00 nm



```python
print('Current wavelength : '+tlb_query('SOURCE:WAVELENGTH?')+' nm')
print('Laser state : '+tlb_query('OUTPut:STATe?'))
```

    Current wavelength : 1553.000 nm
    Laser state : 0



```python
tlb_query('OUTPut:STATe 0')
```




    'OK'




```python
tlb_close()
```
