# Лабораторная работа - Конфигурация безопасности коммутатора
<img width="614" height="382" alt="image" src="https://github.com/user-attachments/assets/c68ee331-6dee-4814-ab4a-75143c233048" />
<img width="680" height="235" alt="image" src="https://github.com/user-attachments/assets/498df30c-93f6-49ef-acef-fe8af2111d19" />

## Часть 1. Настройка основного сетевого устройства
### Шаг 1. Создайте сеть.

#### a.	Создайте сеть согласно топологии.

#### b.	Инициализация устройств

### Шаг 2. Настройте маршрутизатор R1.

#### a.	Загрузите следующий конфигурационный скрипт на R1.

```
enable
configure terminal
hostname R1
no ip domain lookup
ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp excluded-address 192.168.10.201 192.168.10.202
ip dhcp relay information trust-all
!
ip dhcp pool Students
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 domain-name CCNA2.Lab-11.6.1
!
interface Loopback0
 ip address 10.10.1.1 255.255.255.0
!
interface GigabitEthernet0/0/1
 description Link to S1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
!
line con 0
 logging synchronous
 exec-timeout 0 0
```
#### Результат
```
R1(config-line)#
%LINK-5-CHANGED: Interface Loopback0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up
```
#### b.	Проверьте текущую конфигурацию на R1, используя следующую команду:
```
R1# show ip interface brief
```
#### c.	Убедитесь, что IP-адресация и интерфейсы находятся в состоянии up / up (при необходимости устраните неполадки).
```
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   192.168.10.1    YES manual up                    down 
GigabitEthernet0/0/2   unassigned      YES unset  administratively down down 
Loopback0              10.10.1.1       YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```
### Шаг 3. Настройка и проверка основных параметров коммутатора

#### a.	Настройте имя хоста для коммутаторов S1 и S2.
#### b.	Запретите нежелательный поиск в DNS.
#### c.	Настройте описания интерфейса для портов, которые используются в S1 и S2.
#### d.	Установите для шлюза по умолчанию для VLAN управления значение 192.168.10.1 на обоих коммутаторах.
```
Switch>en
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname S1
S1(config)#no ip domain lookup
S1(config)#ip default-gateway 192.168.10.1
S1(config)#interface fa0/1
S1(config-if)#description Link to S2
S1(config-if)#interface fa0/5
S1(config-if)# description Link to R1
S1(config-if)#interface fa0/6
S1(config-if)# description Link to PC-A
S1(config-if)#end
S1#
%SYS-5-CONFIG_I: Configured from console by console
S1#wr
Building configuration...
[OK]
```
```
Switch>en
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname S2
S2(config)#no ip domain lookup
S2(config)#ip default-gateway 192.168.10.1
S2(config)#interface fa0/1
S2(config-if)# description Link to S1
S2(config-if)#interface fa0/18
S2(config-if)# description Link to PC-B
S2(config-if)#end
S2#write memory
Building configuration...
[OK]
S2#
%SYS-5-CONFIG_I: Configured from console by console
```

## Часть 2. Настройка сетей VLAN на коммутаторах.

### Шаг 1. Сконфигруриуйте VLAN 10.
#### Добавьте VLAN 10 на S1 и S2 и назовите VLAN - Management.

### Шаг 2. Сконфигруриуйте SVI для VLAN 10.
#### Настройте IP-адрес в соответствии с таблицей адресации для SVI для VLAN 10 на S1 и S2. Включите интерфейсы SVI и предоставьте описание для интерфейса.

### Шаг 3. Настройте VLAN 333 с именем Native на S1 и S2.

### Шаг 4. Настройте VLAN 999 с именем ParkingLot на S1 и S2.

#### Создание VLAN для S1 и S2
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#vlan 10
S1(config-vlan)# name Management
S1(config-vlan)#vlan 333
S1(config-vlan)# name Native
S1(config-vlan)#vlan 999
S1(config-vlan)# name ParkingLot
S1(config-vlan)#end
S1#wr
Building configuration...
[OK]
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#vlan 10
S2(config-vlan)# name Management
S2(config-vlan)#vlan 333
S2(config-vlan)# name Native
S2(config-vlan)#vlan 999
S2(config-vlan)# name ParkingLot
S2(config-vlan)#end
S2#wr
Building configuration...
[OK]
```
#### Конфигурация SVI для VLAN 10
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface vlan 10
S1(config-if)# description Management SVI
S1(config-if)# ip address 192.168.10.201 255.255.255.0
S1(config-if)# no shutdown
S1(config-if)#end
S1#wr
Building configuration...
[OK]
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface vlan 10
S2(config-if)# description Management SVI
S2(config-if)# ip address 192.168.10.202 255.255.255.0
S2(config-if)# no shutdown
S2(config-if)#end
S2#wr
Building configuration...
[OK]
```


## Часть 3. Настройки безопасности коммутатора.

### Шаг 1. Релизация магистральных соединений 802.1Q.

#### a.	Настройте все магистральные порты Fa0/1 на обоих коммутаторах для использования VLAN 333 в качестве native VLAN.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface fa0/1
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 333
S1(config-if)# switchport nonegotiate
S1(config-if)#end
S1#wr
Building configuration...
[OK]
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface fa0/1
S2(config-if)# switchport mode trunk
S2(config-if)# switchport trunk native vlan 333
S2(config-if)# switchport nonegotiate
S2(config-if)#end
S2#wr
Building configuration...
[OK]
```

#### b.	Убедитесь, что режим транкинга успешно настроен на всех коммутаторах.
```
S1#show interface trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      333
Port        Vlans allowed on trunk
Fa0/1       1-1005
Port        Vlans allowed and active in management domain
Fa0/1       1,10,333,999
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,333,999
```
```
S2#show interface trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      333
Port        Vlans allowed on trunk
Fa0/1       1-1005
Port        Vlans allowed and active in management domain
Fa0/1       1,10,333,999
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,333,999
```
#### c.	Отключить согласование DTP F0/1 на S1 и S2. 
```
S1(config-if)# switchport nonegotiate
```
```
S2(config-if)# switchport nonegotiate
```
#### d.	Проверьте с помощью команды show interfaces.
```
S1# show interfaces f0/1 switchport | include Negotiation
Negotiation of Trunking: Off
```
```
S2# show interfaces f0/1 switchport | include Negotiation
Negotiation of Trunking: Off
```
### Шаг 2. Настройка портов доступа
#### a.	На S1 настройте F0/5 и F0/6 в качестве портов доступа и свяжите их с VLAN 10.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface range f0/5 - 6
S1(config-if-range)# switchport mode access
S1(config-if-range)# switchport access vlan 10
S1(config-if-range)#end
S1#wr
Building configuration...
[OK]
```
#### b.	На S2 настройте порт доступа Fa0/18 и свяжите его с VLAN 10.
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface f0/18
S2(config-if)# switchport mode access
S2(config-if)# switchport access vlan 10
S2(config-if)#end
S2#wr
Building configuration...
[OK]
```
### Шаг 3. Безопасность неиспользуемых портов коммутатора
#### a.	На S1 и S2 переместите неиспользуемые порты из VLAN 1 в VLAN 999 и отключите неиспользуемые порты.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface range f0/2 - 4, f0/7 - 24, gi0/1 - 2
S1(config-if-range)# switchport mode access
S1(config-if-range)# switchport access vlan 999
S1(config-if-range)# shutdown
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface range f0/2 - 17, f0/19 - 24, gi0/1 - 2
S2(config-if-range)# switchport mode access
S2(config-if-range)# switchport access vlan 999
S2(config-if-range)# shutdown
```

#### b.	Убедитесь, что неиспользуемые порты отключены и связаны с VLAN 999, введя команду  show.
```
S1#show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     Link to S2         connected    trunk      auto    auto  10/100BaseTX
Fa0/2                        disabled 999        auto    auto  10/100BaseTX
Fa0/3                        disabled 999        auto    auto  10/100BaseTX
Fa0/4                        disabled 999        auto    auto  10/100BaseTX
Fa0/5     Link to R1         notconnect   10         auto    auto  10/100BaseTX
Fa0/6     Link to PC-A       connected    10         auto    auto  10/100BaseTX
Fa0/7                        disabled 999        auto    auto  10/100BaseTX
Fa0/8                        disabled 999        auto    auto  10/100BaseTX
Fa0/9                        disabled 999        auto    auto  10/100BaseTX
Fa0/10                       disabled 999        auto    auto  10/100BaseTX
Fa0/11                       disabled 999        auto    auto  10/100BaseTX
Fa0/12                       disabled 999        auto    auto  10/100BaseTX
Fa0/13                       disabled 999        auto    auto  10/100BaseTX
Fa0/14                       disabled 999        auto    auto  10/100BaseTX
Fa0/15                       disabled 999        auto    auto  10/100BaseTX
Fa0/16                       disabled 999        auto    auto  10/100BaseTX
Fa0/17                       disabled 999        auto    auto  10/100BaseTX
Fa0/18                       disabled 999        auto    auto  10/100BaseTX
Fa0/19                       disabled 999        auto    auto  10/100BaseTX
Fa0/20                       disabled 999        auto    auto  10/100BaseTX
Fa0/21                       disabled 999        auto    auto  10/100BaseTX
Fa0/22                       disabled 999        auto    auto  10/100BaseTX
Fa0/23                       disabled 999        auto    auto  10/100BaseTX
Fa0/24                       disabled 999        auto    auto  10/100BaseTX
Gig0/1                       disabled 999        auto    auto  10/100BaseTX
Gig0/2                       disabled 999        auto    auto  10/100BaseTX

```
```
S2#show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1     Link to S1         connected    trunk      auto    auto  10/100BaseTX
Fa0/2                        disabled 999        auto    auto  10/100BaseTX
Fa0/3                        disabled 999        auto    auto  10/100BaseTX
Fa0/4                        disabled 999        auto    auto  10/100BaseTX
Fa0/5                        disabled 999        auto    auto  10/100BaseTX
Fa0/6                        disabled 999        auto    auto  10/100BaseTX
Fa0/7                        disabled 999        auto    auto  10/100BaseTX
Fa0/8                        disabled 999        auto    auto  10/100BaseTX
Fa0/9                        disabled 999        auto    auto  10/100BaseTX
Fa0/10                       disabled 999        auto    auto  10/100BaseTX
Fa0/11                       disabled 999        auto    auto  10/100BaseTX
Fa0/12                       disabled 999        auto    auto  10/100BaseTX
Fa0/13                       disabled 999        auto    auto  10/100BaseTX
Fa0/14                       disabled 999        auto    auto  10/100BaseTX
Fa0/15                       disabled 999        auto    auto  10/100BaseTX
Fa0/16                       disabled 999        auto    auto  10/100BaseTX
Fa0/17                       disabled 999        auto    auto  10/100BaseTX
Fa0/18    Link to PC-B       connected    10         auto    auto  10/100BaseTX
Fa0/19                       disabled 999        auto    auto  10/100BaseTX
Fa0/20                       disabled 999        auto    auto  10/100BaseTX
Fa0/21                       disabled 999        auto    auto  10/100BaseTX
Fa0/22                       disabled 999        auto    auto  10/100BaseTX
Fa0/23                       disabled 999        auto    auto  10/100BaseTX
Fa0/24                       disabled 999        auto    auto  10/100BaseTX
Gig0/1                       disabled 999        auto    auto  10/100BaseTX
Gig0/2                       disabled 999        auto    auto  10/100BaseTX
```
### Шаг 4. Документирование и реализация функций безопасности порта.
#### Интерфейсы F0/6 на S1 и F0/18 на S2 настроены как порты доступа. На этом шаге вы также настроите безопасность портов на этих двух портах доступа.
#### a.	На S1, введите команду show port-security interface f0/6  для отображения настроек по умолчанию безопасности порта для интерфейса F0/6. Запишите свои ответы ниже.

<img width="644" height="285" alt="image" src="https://github.com/user-attachments/assets/19eaf6d8-59c1-4c4f-a2e5-4122e6d99f84" />

#### b.	На S1 включите защиту порта на F0 / 6 со следующими настройками:
o	Максимальное количество записей MAC-адресов: 3
o	Режим безопасности: restrict
o	Aging time: 60 мин.
o	Aging type: неактивный

```
В используемом образе Cisco IOS коммутатора Catalyst 2960 (Packet Tracer) команда switchport port-security aging type inactivity не поддерживается.
```
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface f0/6
S1(config-if)# switchport port-security
S1(config-if)# switchport port-security maximum 3
S1(config-if)# switchport port-security violation restrict
S1(config-if)# switchport port-security aging time 60
S1(config-if)#end
S1#wr
Building configuration...
[OK]
```
#### c.	Verify port security on S1 F0/6.
```
S1#show port-security interface f0/6
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 3
Total MAC Addresses        : 0
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0000.0000.0000:0
Security Violation Count   : 0

S1#show port-security address
               Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports   Remaining Age
                                                                   (mins)
----    -----------       ----                          -----   -------------
-----------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 1024
```
#### d.	Включите безопасность порта для F0 / 18 на S2. Настройте каждый активный порт доступа таким образом, чтобы он автоматически добавлял адреса МАС, изученные на этом порту, в текущую конфигурацию.
#### e.	Настройте следующие параметры безопасности порта на S2 F / 18:
o	Максимальное количество записей MAC-адресов: 2
o	Тип безопасности: Protect
o	Aging time: 60 мин.
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface f0/18
S2(config-if)# switchport port-security
S2(config-if)# switchport port-security mac-address sticky
S2(config-if)# switchport port-security maximum 2
S2(config-if)# switchport port-security violation protect
S2(config-if)# switchport port-security aging time 60
S2(config-if)#end
S2#wr
Building configuration...
[OK]
```
#### f.	Проверка функции безопасности портов на S2 F0/18.
```
S2#show port-security interface f0/18
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Protect
Aging Time                 : 60 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 2
Total MAC Addresses        : 0
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 0
Last Source Address:Vlan   : 0000.0000.0000:0
Security Violation Count   : 0

S2#show port-security address
               Secure Mac Address Table
-----------------------------------------------------------------------------
Vlan    Mac Address       Type                          Ports   Remaining Age
                                                                   (mins)
----    -----------       ----                          -----   -------------
-----------------------------------------------------------------------------
Total Addresses in System (excluding one mac per port)     : 0
Max Addresses limit in System (excluding one mac per port) : 1024
```
### Шаг 5. Реализовать безопасность DHCP snooping.
#### a.	На S2 включите DHCP snooping и настройте DHCP snooping во VLAN 10.
#### b.	Настройте магистральные порты на S2 как доверенные порты.
#### c.	Ограничьте ненадежный порт Fa0/18 на S2 пятью DHCP-пакетами в секунду.
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#ip dhcp snooping
S2(config)#ip dhcp snooping vlan 10
S2(config)#
S2(config)#interface f0/1
S2(config-if)# ip dhcp snooping trust
S2(config-if)#
S2(config-if)#interface f0/18
S2(config-if)# ip dhcp snooping limit rate 5
S2(config-if)#end
S2#wr
Building configuration...
[OK]
```
#### d.	Проверка DHCP Snooping на S2.

```
S2#show ip dhcp snooping
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
10
Insertion of option 82 is enabled
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled
Interface                  Trusted    Rate limit (pps)
-----------------------    -------    ----------------
FastEthernet0/18           no         5               
FastEthernet0/1            yes        unlimited   
```
#### e.	В командной строке на PC-B освободите, а затем обновите IP-адрес.
```
C:\Users\Student> ipconfig /release
C:\>ipconfig /renew

   IP Address......................: 192.168.10.10
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: 192.168.10.1
   DNS Server......................: 0.0.0.0
```
#### f.	Проверьте привязку отслеживания DHCP с помощью команды show ip dhcp snooping binding.
```
S2#show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  -----------------
00:E0:F7:1B:7C:72   192.168.10.10    0           dhcp-snooping  10    FastEthernet0/18
Total number of bindings: 1
```
### Шаг 6. Реализация PortFast и BPDU Guard
#### a.	Настройте PortFast на всех портах доступа, которые используются на обоих коммутаторах.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface range f0/5 - 6
S1(config-if-range)# spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/5 but will only
have effect when the interface is in a non-trunking mode.
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/6 but will only
have effect when the interface is in a non-trunking mode.
S1(config-if-range)#end
S1#wr
Building configuration...
[OK]
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface f0/18
S2(config-if)# spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION
%Portfast has been configured on FastEthernet0/18 but will only
have effect when the interface is in a non-trunking mode.
S2(config-if)#end
S2#wr
Building configuration...
[OK]
```
#### b.	Включите защиту BPDU на портах доступа VLAN 10 S1 и S2, подключенных к PC-A и PC-B.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface f0/6
S1(config-if)# spanning-tree bpduguard enable
S1(config-if)#end
S1#wr
Building configuration...
[OK]
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface f0/18
S2(config-if)# spanning-tree bpduguard enable
S2(config-if)#end
S2#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
```
#### c.	Убедитесь, что защита BPDU и PortFast включены на соответствующих портах.
```
S1#show spanning-tree interface f0/6 detail
Port 6 (FastEthernet0/6) of VLAN0010 is designated forwarding
  Port path cost 19, Port priority 128, Port Identifier 128.6
  Designated root has priority 32778, address 00D0.9795.0163
  Designated bridge has priority 32778, address 00D0.9795.0163
  Designated port id is 128.6, designated path cost 19
  Timers: message age 16, forward delay 0, hold 0
  Number of transitions to forwarding state: 1
  The port is in the portfast mode
  Link type is point-to-point by default

S1#show running-config | include bpduguard
 spanning-tree bpduguard enable
```
### Шаг 7. Проверьте наличие сквозного ⁪подключения.

#### Проверьте PING свзяь между всеми устройствами в таблице IP-адресации. В случае сбоя проверки связи может потребоваться отключить брандмауэр на хостах.
PC:
```
C:\>ping 192.168.10.1

Pinging 192.168.10.1 with 32 bytes of data:

Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time=1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.10.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

C:\>ping 192.168.10.202

Pinging 192.168.10.202 with 32 bytes of data:

Reply from 192.168.10.202: bytes=32 time<1ms TTL=255
Reply from 192.168.10.202: bytes=32 time<1ms TTL=255
Reply from 192.168.10.202: bytes=32 time<1ms TTL=255
Reply from 192.168.10.202: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.10.202:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.10.201

Pinging 192.168.10.201 with 32 bytes of data:

Reply from 192.168.10.201: bytes=32 time<1ms TTL=255
Reply from 192.168.10.201: bytes=32 time=1ms TTL=255
Reply from 192.168.10.201: bytes=32 time<1ms TTL=255
Reply from 192.168.10.201: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.10.201:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```
S1:
```
S1#ping 192.168.10.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.10.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
```
S2:
```
S2>ping 192.168.10.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.10.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/4 ms
```

### Вопросы для повторения
#### 1.	С точки зрения безопасности порта на S2, почему нет значения таймера для оставшегося возраста в минутах, когда было сконфигурировано динамическое обучение - sticky?
#### 2.	Что касается безопасности порта на S2, если вы загружаете скрипт текущей конфигурации на S2, почему порту 18 на PC-B никогда не получит IP-адрес через DHCP?
#### 3.	Что касается безопасности порта, в чем разница между типом абсолютного устаревания и типом устаревание по неактивности?


























































