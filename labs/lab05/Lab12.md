# Лабораторная работа 12 - Настройка NAT для IPv4

### Топология

![image](https://user-images.githubusercontent.com/89464074/176600189-ee6b2619-0ed4-4a2d-bdfb-fa1625eb0fee.png)

![image](https://user-images.githubusercontent.com/89464074/176591086-1b70d1b5-a3b1-4166-a200-c545a4bb3c0e.png)

**Таблица адресации**

| Устройство | Интерфейс | IP-адрес | Маска подсети |
|---|---|---|---|
| R1 | G0/0/0 | 209.165.200.230 | 255.255.255.248 |
| R1 | G0/0/1 | 192.168.1.1 | 255.255.255.0 |
| R2 | G0/0/0 | 209.165.200.225 | 255.255.255.248 |
| R2 | Lo1 | 209.165.200.1 | 255.255.255.224 |
| S1 | VLAN 1 | 192.168.1.11 | 255.255.255.0 |
| S2 | VLAN 1 | 192.168.1.12 | 255.255.255.0 |
| PC-A | NIC | 192.168.1.2 | 255.255.255.0 |
| PC-B | NIC | 192.168.1.3 | 255.255.255.0 |

---

### Часть 1. Базовые настройки сетевого оборудования и узлов

- Настроим коммутаторы S1 и S2, присвоим адреса согласно таблице адресации и активируем VLANы

![image](https://user-images.githubusercontent.com/89464074/176597100-3a7b810a-47f7-4a1a-b34f-5856e2ca7e44.png)

```bash
S1(config)#
S1(config)#int vlan 1
S1(config-if)#ip add
S1(config-if)#ip address 192.168.1.11 255.255.255.0
S1(config-if)#no sh

S1(config-if)#
%LINK-5-CHANGED: Interface Vlan1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
```

```bash
Switch(config)#hostname S2
S2(config)#
S2(config)#int vlan 1
S2(config-if)#ip add
S2(config-if)#ip address 192.168.1.12 255.255.255.0
S2(config-if)#no sh

S2(config-if)#
%LINK-5-CHANGED: Interface Vlan1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
```

- Настроим статические ip-адреса для узлов

![image](https://user-images.githubusercontent.com/89464074/176597459-edf262d5-e9ad-4a09-a4ad-97d37077d9f2.png)

```bash
Packet Tracer PC Command Line 1.0
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..:
   Link-local IPv6 Address.........: FE80::260:70FF:FECB:C658
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.3
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     0.0.0.0
```

```bash
Packet Tracer PC Command Line 1.0
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..:
   Link-local IPv6 Address.........: FE80::202:4AFF:FE77:D5D6
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.2
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     0.0.0.0
```

- Настроим роутеры R1 и R2, присвоим адреса и активируем интерфейсы

![image](https://user-images.githubusercontent.com/89464074/176598750-34c71a07-7e6d-4457-b614-3c2fd1400c12.png)

![image](https://user-images.githubusercontent.com/89464074/176599792-8a483d8f-826a-4490-9393-7cbde1bd9a0c.png)

```bash
Router(config)#no ip domain-lookup
Router(config)#hostname R1
R1(config)#
R1(config)#int g0/0/0
R1(config-if)#ip add
R1(config-if)#ip address 209.165.200.230 255.255.255.248
R1(config-if)#no sh

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

R1(config-if)#int g0/0/1
R1(config-if)#ip ad
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#
R1(config-if)#no sh

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

R1(config-if)#
R1(config-if)#end
R1#
%SYS-5-CONFIG_I: Configured from console by console
```

```bash
Router(config)#no ip domain-lookup
Router(config)#
Router(config)#hostname R2
R2(config)#
R2(config)#int g0/0/0
R2(config-if)#ip add
R2(config-if)#ip address 209.165.200.225 255.255.255.248
R2(config-if)#no sh

R2(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up
```

- На R2 добавим интерфейс Loopback1 и маршрут по-умолчанию до R1

![image](https://user-images.githubusercontent.com/89464074/176599960-76338d43-ba52-433d-bba0-ed1f831e3d9d.png)
![image](https://user-images.githubusercontent.com/89464074/176600050-0f8109d8-cb0d-43ba-a0e5-09c22bffc094.png)

```bash
R2(config-if)#
%LINK-5-CHANGED: Interface Loopback1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback1, changed state to up

R2(config-if)#ip add
R2(config-if)#ip address 209.165.200.1 255.255.255.224
R2(config-if)#end
```

```bash
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#ip route 0.0.0.0 0.0.0.0 209.165.200.230
R2(config)#end
R2#
```

---

### Часть 2

- Настроим NAT на R1, используя пул из трех адресов 209.165.200.226 - 209.165.200.228.

![image](https://user-images.githubusercontent.com/89464074/176601255-da907373-01ec-43d1-bdd1-84db2488619d.png)

```bash
R1(config)#
R1(config)#acc
R1(config)#access-list 1 per
R1(config)#access-list 1 permit 192.168.1.0 0.0.0.255
R1(config)#
R1(config)#ip nat pool PUBLIC_ACCESS 209.165.200.226 209.165.200.228 net
R1(config)#ip nat pool PUBLIC_ACCESS 209.165.200.226 209.165.200.228 netmask 255.255.255.248
R1(config)#
R1(config)#ip nat ins
R1(config)#ip nat inside source list 1 pool PUBLIC_ACCESS
R1(config)#
R1(config)#int g0/0/1
R1(config-if)#ip nat ins
R1(config-if)#ip nat inside
R1(config-if)#
R1(config-if)#int g0/0/0
R1(config-if)#
R1(config-if)#ip nat ou
R1(config-if)#ip nat outside
R1(config-if)#end
R1#
%SYS-5-CONFIG_I: Configured from console by console
```

- Проверим конфигурацию  
Пинг с PC-B На Loopback1 изначально не прошел, пока на R1 не был прописан маршрут ip route 209.165.200.0 255.255.255.224 209.165.200.225, для связи целевой подсети с интерфейсом роутера R1.

![image](https://user-images.githubusercontent.com/89464074/176647110-70b21f36-911f-4920-ac4d-6c771c1b54aa.png)

```bash
Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

C:\>
C:\>
C:\>ping 209.165.200.1

Pinging 209.165.200.1 with 32 bytes of data:

Request timed out.
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time=9ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 9ms, Average = 3ms

C:\>ping 209.165.200.1

Pinging 209.165.200.1 with 32 bytes of data:

Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time=1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

C:\>
```

- Рассмотрим таблицу NAT на R1 после эхо-запросов с PC-B и c PC-A

![image](https://user-images.githubusercontent.com/89464074/176647444-13078d6d-a5ca-4a99-ad84-4c9f1369150b.png)

![image](https://user-images.githubusercontent.com/89464074/176649512-e616660a-1631-4868-8f43-c4a0ebadae34.png)

```bash
R1#show ip nat translations
Pro  Inside global         Inside local         Outside local        Outside global
icmp 209.165.200.227:33192 192.168.1.3:33       209.165.200.1:33     209.165.200.1:33
icmp 209.165.200.227:34192 192.168.1.3:34       209.165.200.1:34     209.165.200.1:34
icmp 209.165.200.227:35192 192.168.1.3:35       209.165.200.1:35     209.165.200.1:35
icmp 209.165.200.227:36192 192.168.1.3:36       209.165.200.1:36     209.165.200.1:36
```

```bash
R1#sh ip nat translations
Pro  Inside global         Inside local         Outside local        Outside global
icmp 209.165.200.226:37192 192.168.1.2:37       209.165.200.1:37     209.165.200.1:37
icmp 209.165.200.226:38192 192.168.1.2:38       209.165.200.1:38     209.165.200.1:38
icmp 209.165.200.226:39192 192.168.1.2:39       209.165.200.1:39     209.165.200.1:39
icmp 209.165.200.226:40192 192.168.1.2:40       209.165.200.1:40     209.165.200.1:40
icmp 209.165.200.228:41192 192.168.1.3:41       209.165.200.1:41     209.165.200.1:41
icmp 209.165.200.228:42192 192.168.1.3:42       209.165.200.1:42     209.165.200.1:42
icmp 209.165.200.228:43192 192.168.1.3:43       209.165.200.1:43     209.165.200.1:43
icmp 209.165.200.228:44192 192.168.1.3:44       209.165.200.1:44     209.165.200.1:44
icmp 209.165.200.228:45192 192.168.1.3:45       209.165.200.1:45     209.165.200.1:45
```

(примечание - Директива verbose для команды sh ip nat translations выдавала обшибку. работа выполнялась в CPT 8)

---

### Часть 3

- Настроим PAT на R1

![image](https://user-images.githubusercontent.com/89464074/176653299-e43642e2-d393-4882-b1de-c8d1870f0160.png)

```bash
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#no ip nat inside source list 1 pool PUBLIC_ACCESS
R1(config)#ip nat inside source list 1 pool PUBLIC_ACCESS ove
R1(config)#ip nat inside source list 1 pool PUBLIC_ACCESS overload
R1(config)#
```

- Наблюдаем за работой PAT при помощи безостановочного пинга с узлов

![image](https://user-images.githubusercontent.com/89464074/176654979-1c84557a-e9fa-469a-b2ae-3eb32c87f504.png)

```bash
R1#
R1#sh ip nat translations
Pro  Inside global         Inside local         Outside local        Outside global
icmp 209.165.200.226:1024  192.168.1.2:53       209.165.200.1:53     209.165.200.1:1024
icmp 209.165.200.226:1025  192.168.1.2:54       209.165.200.1:54     209.165.200.1:1025
icmp 209.165.200.226:1026  192.168.1.2:55       209.165.200.1:55     209.165.200.1:1026
icmp 209.165.200.226:1027  192.168.1.2:56       209.165.200.1:56     209.165.200.1:1027
icmp 209.165.200.226:1028  192.168.1.2:57       209.165.200.1:57     209.165.200.1:1028
icmp 209.165.200.226:4519  192.168.1.2:45       209.165.200.1:45     209.165.200.1:45
icmp 209.165.200.226:4619  192.168.1.2:46       209.165.200.1:46     209.165.200.1:46
icmp 209.165.200.226:4719  192.168.1.2:47       209.165.200.1:47     209.165.200.1:47
icmp 209.165.200.226:4819  192.168.1.2:48       209.165.200.1:48     209.165.200.1:48
icmp 209.165.200.226:4919  192.168.1.2:49       209.165.200.1:49     209.165.200.1:49
icmp 209.165.200.226:5019  192.168.1.2:50       209.165.200.1:50     209.165.200.1:50
icmp 209.165.200.226:5119  192.168.1.2:51       209.165.200.1:51     209.165.200.1:51
icmp 209.165.200.226:5219  192.168.1.2:52       209.165.200.1:52     209.165.200.1:52
icmp 209.165.200.226:5319  192.168.1.2:53       209.165.200.1:53     209.165.200.1:53
icmp 209.165.200.226:5419  192.168.1.3:54       209.165.200.1:54     209.165.200.1:54
icmp 209.165.200.226:5519  192.168.1.3:55       209.165.200.1:55     209.165.200.1:55
icmp 209.165.200.226:5619  192.168.1.3:56       209.165.200.1:56     209.165.200.1:56
icmp 209.165.200.226:5719  192.168.1.3:57       209.165.200.1:57     209.165.200.1:57
icmp 209.165.200.226:5819  192.168.1.3:58       209.165.200.1:58     209.165.200.1:58
icmp 209.165.200.226:5919  192.168.1.3:59       209.165.200.1:59     209.165.200.1:59

R1#sh ip nat translations
Pro  Inside global         Inside local         Outside local        Outside global
icmp 209.165.200.226:1024  192.168.1.2:53       209.165.200.1:53     209.165.200.1:1024
icmp 209.165.200.226:1025  192.168.1.2:54       209.165.200.1:54     209.165.200.1:1025
icmp 209.165.200.226:1026  192.168.1.2:55       209.165.200.1:55     209.165.200.1:1026
icmp 209.165.200.226:1027  192.168.1.2:56       209.165.200.1:56     209.165.200.1:1027
icmp 209.165.200.226:1028  192.168.1.2:57       209.165.200.1:57     209.165.200.1:1028
icmp 209.165.200.226:1029  192.168.1.2:58       209.165.200.1:58     209.165.200.1:1029
icmp 209.165.200.226:1030  192.168.1.2:59       209.165.200.1:59     209.165.200.1:1030
icmp 209.165.200.226:1031  192.168.1.2:60       209.165.200.1:60     209.165.200.1:1031
icmp 209.165.200.226:1032  192.168.1.2:61       209.165.200.1:61     209.165.200.1:1032
icmp 209.165.200.226:1033  192.168.1.2:62       209.165.200.1:62     209.165.200.1:1033
icmp 209.165.200.226:1034  192.168.1.2:63       209.165.200.1:63     209.165.200.1:1034
icmp 209.165.200.226:1035  192.168.1.2:64       209.165.200.1:64     209.165.200.1:1035
icmp 209.165.200.226:1036  192.168.1.2:65       209.165.200.1:65     209.165.200.1:1036
icmp 209.165.200.226:1037  192.168.1.2:66       209.165.200.1:66     209.165.200.1:1037
icmp 209.165.200.226:1038  192.168.1.2:67       209.165.200.1:67     209.165.200.1:1038
icmp 209.165.200.226:1039  192.168.1.2:68       209.165.200.1:68     209.165.200.1:1039
icmp 209.165.200.226:1040  192.168.1.2:69       209.165.200.1:69     209.165.200.1:1040
icmp 209.165.200.226:1041  192.168.1.2:70       209.165.200.1:70     209.165.200.1:1041
icmp 209.165.200.226:1042  192.168.1.2:71       209.165.200.1:71     209.165.200.1:1042
icmp 209.165.200.226:4519  192.168.1.2:45       209.165.200.1:45     209.165.200.1:45
```

- Очистим трансляции и статистику

![image](https://user-images.githubusercontent.com/89464074/176655898-a0f5bb79-7201-41ad-84f0-95c3ad55c520.png)

```bash
R1#clear ip nat translation *
R1#
R1#clear ip nat sta ?
% Unrecognized command
R1#clear ip nat sta?
% Unrecognized command
R1#clear ip nat ?
trans  translation  Clear dynamic translation
R1#clear ip nat
% Incomplete command.
R1#
```

(примечание - команду clear ip nat statistics выполнить не удалось)

- Далее по заданию удаляем команды преобразования

![image](https://user-images.githubusercontent.com/89464074/176657427-4913842d-4422-450b-af5f-2a42de2ba729.png)

```bash
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#
R1(config)#no ip nat inside source list 1 pool PUBLIC_ACCESS ove
R1(config)#no ip nat inside source list 1 pool PUBLIC_ACCESS overload
%Dynamic mapping not found
R1(config)#
R1(config)#no ip nat pool PUBLIC_ACCESS
R1(config)#
R1(config)#
```

- Добавляем команду PAT overload с указанием внешнего интерфейся R1

![image](https://user-images.githubusercontent.com/89464074/176657947-d7bea8d1-2443-4797-92f9-f73a2e1be895.png)

```bash
R1(config)#
R1(config)#ip nat inside sou
R1(config)#ip nat inside source list 1 int g0/0/0 overload
R1(config)#
```

- Проверяем работу пингом с PC-B на loopback1 R2

![image](https://user-images.githubusercontent.com/89464074/176658302-cc546942-32f1-4f10-b754-8476d6b6e8b0.png)

```bash
R1#
R1#sh ip nat translations
Pro  Inside global         Inside local         Outside local        Outside global
icmp 209.165.200.230:1091  192.168.1.3:109      209.165.200.1:109    209.165.200.1:109
icmp 209.165.200.230:1101  192.168.1.3:110      209.165.200.1:110    209.165.200.1:110
icmp 209.165.200.230:1111  192.168.1.3:111      209.165.200.1:111    209.165.200.1:111
icmp 209.165.200.230:1121  192.168.1.3:112      209.165.200.1:112    209.165.200.1:112
```

- Тест с задействованием двух узлов с безостановочным пингом

![image](https://user-images.githubusercontent.com/89464074/176658672-fa2b3e98-3618-4628-a038-c3f3bacb2067.png)

```bash
R1#sh ip nat translations
Pro  Inside global         Inside local         Outside local        Outside global
icmp 209.165.200.230:1024  192.168.1.2:113      209.165.200.1:113    209.165.200.1:1024
icmp 209.165.200.230:1025  192.168.1.2:114      209.165.200.1:114    209.165.200.1:1025
icmp 209.165.200.230:1026  192.168.1.2:115      209.165.200.1:115    209.165.200.1:1026
icmp 209.165.200.230:1027  192.168.1.2:116      209.165.200.1:116    209.165.200.1:1027
icmp 209.165.200.230:1028  192.168.1.2:117      209.165.200.1:117    209.165.200.1:1028
icmp 209.165.200.230:1029  192.168.1.2:118      209.165.200.1:118    209.165.200.1:1029
icmp 209.165.200.230:1051  192.168.1.2:105      209.165.200.1:105    209.165.200.1:105
icmp 209.165.200.230:1061  192.168.1.2:106      209.165.200.1:106    209.165.200.1:106
icmp 209.165.200.230:1071  192.168.1.2:107      209.165.200.1:107    209.165.200.1:107
icmp 209.165.200.230:1081  192.168.1.2:108      209.165.200.1:108    209.165.200.1:108
icmp 209.165.200.230:1091  192.168.1.2:109      209.165.200.1:109    209.165.200.1:109
icmp 209.165.200.230:1101  192.168.1.3:110      209.165.200.1:110    209.165.200.1:110
icmp 209.165.200.230:1111  192.168.1.3:111      209.165.200.1:111    209.165.200.1:111
icmp 209.165.200.230:1121  192.168.1.3:112      209.165.200.1:112    209.165.200.1:112
icmp 209.165.200.230:1131  192.168.1.3:113      209.165.200.1:113    209.165.200.1:113
icmp 209.165.200.230:1141  192.168.1.3:114      209.165.200.1:114    209.165.200.1:114
icmp 209.165.200.230:1151  192.168.1.3:115      209.165.200.1:115    209.165.200.1:115
icmp 209.165.200.230:1161  192.168.1.3:116      209.165.200.1:116    209.165.200.1:116
icmp 209.165.200.230:1171  192.168.1.3:117      209.165.200.1:117    209.165.200.1:117
icmp 209.165.200.230:1181  192.168.1.3:118      209.165.200.1:118    209.165.200.1:118
icmp 209.165.200.230:1191  192.168.1.3:119      209.165.200.1:119    209.165.200.1:119
icmp 209.165.200.230:1201  192.168.1.3:120      209.165.200.1:120    209.165.200.1:120
icmp 209.165.200.230:1211  192.168.1.3:121      209.165.200.1:121    209.165.200.1:121
icmp 209.165.200.230:1221  192.168.1.3:122      209.165.200.1:122    209.165.200.1:122
icmp 209.165.200.230:1231  192.168.1.3:123      209.165.200.1:123    209.165.200.1:123
icmp 209.165.200.230:1241  192.168.1.3:124      209.165.200.1:124    209.165.200.1:124
icmp 209.165.200.230:1251  192.168.1.3:125      209.165.200.1:125    209.165.200.1:125
icmp 209.165.200.230:1261  192.168.1.3:126      209.165.200.1:126    209.165.200.1:126
icmp 209.165.200.230:1271  192.168.1.3:127      209.165.200.1:127    209.165.200.1:127
icmp 209.165.200.230:1281  192.168.1.3:128      209.165.200.1:128    209.165.200.1:128
icmp 209.165.200.230:1291  192.168.1.3:129      209.165.200.1:129    209.165.200.1:129
icmp 209.165.200.230:1301  192.168.1.3:130      209.165.200.1:130    209.165.200.1:130
```

---

### Часть 4. Настройка статического NAT

- Очистим трансляции и настроим статическое сопоставление внутреннего и внешнего адресов:

![image](https://user-images.githubusercontent.com/89464074/176659742-6769ac27-8cb0-4c0a-b155-035ade57146e.png)

```bash
R1#clear ip nat ?
trans  translation  Clear dynamic translation
R1#clear ip nat translation
% Incomplete command.
R1#clear ip nat translation *
R1#

R1#
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#ip nat inside source static 192.168.1.2 209.165.200.229
R1(config)#end
R1#
%SYS-5-CONFIG_I: Configured from console by console

R1#sh ip nat nta
R1#sh ip nat tran
R1#sh ip nat translations
Pro  Inside global      Inside local      Outside local      Outside global
---  209.165.200.229    192.168.1.2       ---               ---
```

- Запустим проверку пингом с R2 на адрес 209.165.200.229

![image](https://user-images.githubusercontent.com/89464074/176660021-7faa9308-6893-4447-bded-b2b204e6a88f.png)

```bash
R2#
R2#ping 209.165.200.229

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 209.165.200.229, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms

R2#ping 209.165.200.229

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 209.165.200.229, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

R2#
```

- Теперь рассмотрим таблицу NAT на R1 для подтверждения работы статического NAT

![image](https://user-images.githubusercontent.com/89464074/176660307-df6e720d-9421-4621-81e4-55422f0f89e3.png)

```bash
R1#
R1#sh ip nat translations
Pro  Inside global         Inside local         Outside local          Outside global
icmp 209.165.200.229:17192 192.168.1.2:17       209.165.200.225:17     209.165.200.225:17
icmp 209.165.200.229:18192 192.168.1.2:18       209.165.200.225:18     209.165.200.225:18
icmp 209.165.200.229:19192 192.168.1.2:19       209.165.200.225:19     209.165.200.225:19
icmp 209.165.200.229:20192 192.168.1.2:20       209.165.200.225:20     209.165.200.225:20
icmp 209.165.200.229:21192 192.168.1.2:21       209.165.200.225:21     209.165.200.225:21
icmp 209.165.200.229:22192 192.168.1.2:22       209.165.200.225:22     209.165.200.225:22
icmp 209.165.200.229:23192 192.168.1.2:23       209.165.200.225:23     209.165.200.225:23
icmp 209.165.200.229:24192 192.168.1.2:24       209.165.200.225:24     209.165.200.225:24
icmp 209.165.200.229:25192 192.168.1.2:25       209.165.200.225:25     209.165.200.225:25
---  209.165.200.229      192.168.1.2          ---                  ---
```
