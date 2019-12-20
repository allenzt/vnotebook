# Inside_ble_2th_梳理

## 12涨
### 1. 一个attribute的结构
![att_struct](_v_images/20191220164626942_271562027.png)

### 2. 所有pdu分为6类，包格式相同
![pdu](_v_images/20191220165540765_279128390.png)
The length of a variable length field in the PDU is implicitly given by the length of the packet that carries this PDU