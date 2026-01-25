# Проектная работа
# Тема: Разработка и настройка многофилиальной сети предприятия с применением протоколов L2/L3 и сервисов безопасности

# Введение

Современные предприятия часто имеют распределённую структуру: центральный офис и несколько филиалов, между которыми требуется организовать надёжный и безопасный обмен данными. В таких условиях к сети предъявляются требования по отказоустойчивости, масштабируемости, управляемости и информационной безопасности. Для решения этих задач используются технологии второго и третьего уровней модели OSI: сегментация сети с помощью VLAN, предотвращение петель и повышение отказоустойчивости на L2 (STP/RSTP, EtherChannel), динамическая маршрутизация на L3 (OSPFv2), а также инфраструктурные сервисы и механизмы защиты (DHCP, NAT, ACL, защищённый доступ по SSH).

Работа направлена на проектирование адресного пространства, построение логической схемы сети и реализацию набора ключевых сетевых технологий, обеспечивающих корректную маршрутизацию между площадками, централизованное обслуживание пользователей и базовую защиту периметра и сегментов сети.

# Цель работы

Разработать и реализовать в лабораторной среде модель многофилиальной сети предприятия (HQ + филиалы) с использованием протоколов L2/L3 и сервисов безопасности, обеспечив связность между подсетями, отказоустойчивость на втором уровне и контролируемый доступ во внешнюю сеть.

# Задачи работы

Для достижения поставленной цели необходимо решить следующие задачи:

1. Разработать топологию сети предприятия, определить состав оборудования и схему физических подключений.

2. Выполнить планирование адресного пространства IPv4: выделить подсети для пользовательских VLAN, административных VLAN, серверного сегмента, сети управления и транзитной сети между маршрутизаторами.

3. Реализовать сегментацию сети на уровне L2 с использованием VLAN и настроить trunk-порты для передачи нескольких VLAN.

4. Обеспечить отказоустойчивость и устранение петель на L2 с применением RSTP (Rapid PVST) и агрегирования каналов EtherChannel (LACP) в ядре сети.

5. Настроить межвлановую маршрутизацию по схеме Router-on-a-Stick на маршрутизаторах площадок.

6. Реализовать динамическую маршрутизацию OSPFv2 (Area 0) для обмена маршрутами между центральным офисом и филиалами.

7. Настроить инфраструктурные сервисы: DHCP (централизованная выдача адресов) и DHCP-relay для удалённых VLAN.

8. Реализовать выход пользователей во внешнюю сеть через пограничный маршрутизатор с помощью NAT overload (PAT), а также настроить публикацию внутреннего веб-ресурса посредством статического NAT (port-forward) при необходимости.

9. Внедрить механизмы безопасности: ACL для сегментации доступа (в том числе гостевого сегмента) и SSH для защищённого администрирования сетевых устройств.

10. Выполнить тестирование работоспособности сети (L2/L3, DHCP, NAT, ACL, SSH) и задокументировать результаты проверок.

# Этап 1. Подключения и адресный план
## 1.1 Ядро: Switch1 / Switch2 / Switch3 (EtherChannel)
### S1 ↔ S2 (Port-Channel 12, 2 линка)
* S1 Fa0/1 ↔ S2 Fa0/1
* S1 Fa0/2 ↔ S2 Fa0/2
### S1 ↔ S3 (Port-Channel 13, 2 линка)
* S1 Fa0/3 ↔ S3 Fa0/1
* S1 Fa0/4 ↔ S3 Fa0/2
### S2 ↔ S3 (Port-Channel 23, 2 линка)
* S2 Fa0/3 ↔ S3 Fa0/3
* S2 Fa0/4 ↔ S3 Fa0/4

## 1.2 Роутеры в ядро (Transit VLAN 999)
### Router1 (EDGE) ↔ Switch1 (trunk)
* Router1 G0/0/0 ↔ Switch1 Gi0/1
* Режим: TRUNK
* VLAN: 999
### Router2 (HQ) ↔ Switch1 (access port VLAN 999)
* Router2 G0/0/0 ↔ Switch1 Gi0/2
* Режим: ACCESS VLAN 999
### Router3 (BR1) ↔ Switch2 (access port VLAN 999)
* Router3 G0/0/0 ↔ Switch2 Gi0/1
* Режим: ACCESS VLAN 999
### Router4 (BR2) ↔ Switch3 (access port VLAN 999)
* Router4 G0/0/0 ↔ Switch3 Gi0/1
* Режим: ACCESS VLAN 999

## 1.3 Площадки (Access-switch ↔ Router trunk)
### HQ
* Router2 G0/0/1 ↔ Switch4 Gi0/1
* Режим: TRUNK
* VLAN’ы: 10 / 20 / 30 / 99
### Branch1
* Router3 G0/0/1 ↔ Switch5 Gi0/1
* Режим: TRUNK
* VLAN’ы: 10 / 20 / 40 / 99
### Branch2
* Router4 G0/0/1 ↔ Switch6 Gi0/1
* Режим: TRUNK
* VLAN’ы: 10 / 20 / 99

## 1.4 Конечные устройства
### Switch4 (HQ)
* Fa0/1 → PC-ADMIN1 (VLAN10)
* Fa0/2 → PC-USER1 (VLAN20)
* Fa0/3 → Server-DNS (VLAN30)
* Fa0/4 → Server-WEB (VLAN30)
### Switch5 (Branch1)
* Fa0/1 → PC-ADMIN2 (VLAN10)
* Fa0/2 → PC-USER2 (VLAN20)
* Fa0/3 → AccessPoint0 (VLAN40)
* Laptop подключается к AP по Wi-Fi (в гостевой VLAN40)
### Switch6 (Branch2)
* Fa0/1 → PC-ADMIN3 (VLAN10)
* Fa0/2 → PC-USER3 (VLAN20)

## 1.5 “Интернет” для NAT
* Router1 G0/0/1 ↔ InternetServer Fa0

## 1.6 VLAN сети (по площадкам)
### HQ (Router2 ↔ Switch4)
* VLAN10 ADMIN: 192.168.10.0/24, GW 192.168.10.1
* VLAN20 USERS: 192.168.11.0/24, GW 192.168.11.1
* VLAN30 SERVERS: 192.168.12.0/24, GW 192.168.12.1
* VLAN99 MGMT: 192.168.13.0/24, GW 192.168.13.1

### Branch1 (Router3 ↔ Switch5)
* VLAN10 ADMIN: 192.168.20.0/24, GW 192.168.20.1
* VLAN20 USERS: 192.168.21.0/24, GW 192.168.21.1
* VLAN40 GUEST: 192.168.22.0/24, GW 192.168.22.1
* VLAN99 MGMT: 192.168.23.0/24, GW 192.168.23.1
  
### Branch2 (Router4 ↔ Switch6)
* VLAN10 ADMIN: 192.168.30.0/24, GW 192.168.30.1
* VLAN20 USERS: 192.168.31.0/24, GW 192.168.31.1
* VLAN99 MGMT: 192.168.32.0/24, GW 192.168.32.1

## 1.7 Transit VLAN 999 (OSPF между роутерами)
### Сеть: 192.168.255.0/28 (255.255.255.240)
* Router1: 192.168.255.1/28
* Router2: 192.168.255.2/28
* Router3: 192.168.255.3/28
* Router4: 192.168.255.4/28
### Управление ядром (SVI VLAN999 на коммутаторах):
* Switch1: 192.168.255.11/28
* Switch2: 192.168.255.12/28
* Switch3: 192.168.255.13/28
* Default-gateway на S1/S2/S3: 192.168.255.1
  
## 1.8 Серверы HQ (VLAN30) - статический адрес
* DNS: 192.168.12.10/24, GW 192.168.12.1
* WEB: 192.168.12.20/24, GW 192.168.12.1

## 1.9 Внешняя сеть для NAT
### Сеть: 203.0.113.0/24
* Router1 outside: 203.0.113.2/24
* InternetServer: 203.0.113.10/24, GW 203.0.113.2

## 1.10 Схема топологии сети
<img width="1125" height="746" alt="image" src="https://github.com/user-attachments/assets/b22a9776-b79d-49f7-acaf-ec80242c23ba" />



# Этап 2. Базовая настройка 

На всех маршрутизаторах и коммутаторах необходимо выполнить базовые настройки. 
На примере Switch1:
```
Switch1>en
Switch1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch1(config)#no ip domain-lookup
Switch1(config)#service password-encryption
Switch1(config)#enable secret C1sco123!
Switch1(config)#banner motd ^CUnauthorized access prohibited^C
Switch1(config)#end
Switch1#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
Switch1#
```


# Этап 3. Настройка ядра сети (Switch1/Switch2/Switch3): VLAN 999 + EtherChannel + STP + Management
Необходимо построить отказоустойчивое L2-ядро с транзитной VLAN для маршрутизаторов и OSPF, исключив петли и обеспечив резервирование каналов.
## 3.1 Обоснование выбора оборудования Cisco Catalyst 3560

В ходе реализации выяснилось, что отдельные модели коммутаторов в Packet Tracer (особенно упрощённые/урезанные) могут:
* отклонять перевод порта в trunk при encapsulation auto;
* некорректно собирать EtherChannel из-за несовпадений trunk-параметров и ограничений CLI.

Коммутаторы Cisco Catalyst 3560 обеспечивают корректную поддержку:

* trunk encapsulation 802.1Q (dot1q)
* EtherChannel с протоколом LACP
* SVI для Management-адресов
* стабильные выводы диагностических команд (show etherchannel, show spanning-tree)

## 3.2 Топология соединений ядра (EtherChannel)

S1 ↔ S2 (Po12):

* Switch1 Fa0/1–Fa0/2 ↔ Switch2 Fa0/1–Fa0/2

S1 ↔ S3 (Po13):

* Switch1 Fa0/3–Fa0/4 ↔ Switch3 Fa0/1–Fa0/2

S2 ↔ S3 (Po23):

* Switch2 Fa0/3–Fa0/4 ↔ Switch3 Fa0/3–Fa0/4

## 3.3 Требования к порт-каналам

Чтобы EtherChannel корректно агрегировался и не происходило ошибок типа VLAN mask is different / Native VLAN mismatch, параметры должны быть одинаковыми на обеих сторонах:
* switchport trunk encapsulation dot1q
* switchport mode trunk
* switchport trunk native vlan 999
* switchport trunk allowed vlan 999
* switchport nonegotiate
* channel-group <id> mode active (LACP)

## 3.4 Конфигурация Switch1 (Root Primary)
### 3.4.1 Базовая настройка + VLAN999

```
Switch>enable
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname Switch1
Switch1(config)#no ip domain-lookup
Switch1(config)#spanning-tree mode rapid-pvst

Switch1(config)#vlan 999
Switch1(config-vlan)# name TRANSIT_OSPF
Switch1(config-vlan)#exit
```

### 3.4.2 EtherChannel S1↔S2 (Po12, Fa0/1-2)

```
Switch1(config)#interface port-channel 12
Switch1(config-if)#switchport trunk encapsulation dot1q
Switch1(config-if)#switchport mode trunk
Switch1(config-if)#switchport trunk native vlan 999
Switch1(config-if)#switchport trunk allowed vlan 999
Switch1(config-if)#switchport nonegotiate
Switch1(config-if)#no shutdown
Switch1(config-if)#exit
```

```
Switch1(config)#interface range fa0/1 - 2
Switch1(config-if-range)#switchport trunk encapsulation dot1q
Switch1(config-if-range)#switchport mode trunk
Switch1(config-if-range)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up
Switch1(config-if-range)#switchport trunk native vlan 999
Switch1(config-if-range)#switchport trunk allowed vlan 999
Switch1(config-if-range)#switchport nonegotiate
Switch1(config-if-range)#channel-group 12 mode active
Switch1(config-if-range)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up
Switch1(config-if-range)#no shutdown
Switch1(config-if-range)#exit
```

### 3.4.3 EtherChannel S1↔S3 (Po13, Fa0/3-4)

```
Switch1(config)#interface port-channel 13
Switch1(config-if)#switchport trunk encapsulation dot1q
Switch1(config-if)#switchport mode trunk
Switch1(config-if)#switchport trunk native vlan 999
Switch1(config-if)#switchport trunk allowed vlan 999
Switch1(config-if)#switchport nonegotiate
Switch1(config-if)#no shutdown
Switch1(config-if)#exit
```

```
Switch1(config)#interface range fa0/3 - 4
Switch1(config-if-range)#switchport trunk encapsulation dot1q
Switch1(config-if-range)#switchport mode trunk
Switch1(config-if-range)#switchport trunk native vlan 999
Switch1(config-if-range)#switchport trunk allowed vlan 999
Switch1(config-if-range)#switchport nonegotiate
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up
Switch1(config-if-range)#channel-group 13 mode active
Switch1(config-if-range)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up
Switch1(config-if-range)#no shutdown
Switch1(config-if-range)#exit
```

### 3.4.4 STP Root Primary VLAN999

```
Switch1(config)#spanning-tree vlan 999 root primary
```
### 3.4.5 Trunk к Router1-2
```
Switch1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch1(config)#interface gi0/1
Switch1(config-if)#switchport trunk encapsulation dot1q
Switch1(config-if)#switchport mode trunk
Switch1(config-if)#switchport trunk native vlan 999
Switch1(config-if)#switchport trunk allowed vlan 999
Switch1(config-if)#switchport nonegotiate
Switch1(config-if)#no shut
Switch1(config-if)#exit
```
```
Switch1(config)#interface gi0/2
Switch1(config-if)#switchport mode access
Switch1(config-if)#switchport access vlan 999
Switch1(config-if)#no shut
Switch1(config-if)#exit
Switch1(config)#end
Switch1#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
```
### 3.4.6 Management IP (SVI VLAN999)

```
Switch1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch1(config)#interface vlan 999
Switch1(config-if)#no ip address
Switch1(config-if)#ip address 192.168.255.11 255.255.255.240
Switch1(config-if)#no shutdown
Switch1(config-if)#exit
Switch1(config)#ip default-gateway 192.168.255.1
Switch1(config)#end
Switch1#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
```

## 3.5 Конфигурация Switch2 (Root Secondary)
### 3.5.1 Базовая настройка + VLAN999

```
Switch(config)#hostname Switch2
Switch2(config)#no ip domain-lookup
Switch2(config)#spanning-tree mode rapid-pvst
Switch2(config)#vlan 999
Switch2(config-vlan)# name TRANSIT_OSPF
Switch2(config-vlan)#exit
```

### 3.5.2 EtherChannel S2↔S1 (Po12, Fa0/1-2)

```
Switch2(config)#interface port-channel 12
Switch2(config-if)#switchport trunk encapsulation dot1q
Switch2(config-if)#switchport mode trunk
Switch2(config-if)#switchport trunk native vlan 999
Switch2(config-if)#switchport trunk allowed vlan 999
Switch2(config-if)#switchport nonegotiate
Switch2(config-if)#no shutdown
Switch2(config-if)#exit
```

```
Switch2(config)#interface range fa0/1 - 2
Switch2(config-if-range)#switchport trunk encapsulation dot1q
Switch2(config-if-range)#switchport mode trunk
Switch2(config-if-range)#switchport trunk native vlan 999
Switch2(config-if-range)#switchport trunk allowed vlan 999
Switch2(config-if-range)#switchport nonegotiate
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up
Switch2(config-if-range)#channel-group 12 mode active
Switch2(config-if-range)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up
%LINK-5-CHANGED: Interface Port-channel12, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel12, changed state to up
Switch2(config-if-range)#no shutdown
Switch2(config-if-range)#exit
```

### 3.5.3 EtherChannel S2↔S3 (Po23, Fa0/3-4)

```
Switch2(config)#interface port-channel 23
Switch2(config-if)#switchport trunk encapsulation dot1q
Switch2(config-if)#switchport mode trunk
Switch2(config-if)#switchport trunk native vlan 999
Switch2(config-if)#switchport trunk allowed vlan 999
Switch2(config-if)#switchport nonegotiate
Switch2(config-if)#no shutdown
Switch2(config-if)#exit
```

```
Switch2(config)#interface range fa0/3 - 4
Switch2(config-if-range)# switchport trunk encapsulation dot1q
Switch2(config-if-range)# switchport mode trunk
Switch2(config-if-range)# switchport trunk native vlan 999
Switch2(config-if-range)# switchport trunk allowed vlan 999
Switch2(config-if-range)# switchport nonegotiate
Switch2(config-if-range)# channel-group 23 mode active
Switch2(config-if-range)# no shutdown
Switch2(config-if-range)#exit
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up
```

### 3.5.4 STP Root Secondary VLAN999

```
Switch2(config)#spanning-tree vlan 999 root secondary
```
### 3.5.5 Trunk к Router3
```
Switch2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch2(config)#interface gi0/1
Switch2(config-if)#switchport mode access
Switch2(config-if)#switchport access vlan 999
Switch2(config-if)#no shut
Switch2(config-if)#end
Switch2#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
Switch2#
```
### 3.5.6 Management IP (SVI VLAN999)

```
Switch2(config)#interface vlan 999
Switch2(config-if)# ip address 192.168.255.12 255.255.255.240
Switch2(config-if)#no shutdown
Switch2(config-if)#exit
Switch2(config)#ip default-gateway 192.168.255.1
Switch2(config)#end
Switch2#wr
Building configuration...
[OK]
Switch2#
%LINK-5-CHANGED: Interface Vlan999, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan999, changed state to up
%SYS-5-CONFIG_I: Configured from console by console
```

## 3.6 Конфигурация Switch3
### 3.6.1 Базовая настройка + VLAN999

```
Switch>enable
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname Switch3
Switch3(config)#no ip domain-lookup
Switch3(config)#spanning-tree mode rapid-pvst
Switch3(config)#vlan 999
Switch3(config-vlan)#name TRANSIT_OSPF
Switch3(config-vlan)#exit
```

### 3.6.2 EtherChannel S3↔S1 (Po13, Fa0/1-2)

```
Switch3(config)#interface port-channel 13
Switch3(config-if)#switchport trunk encapsulation dot1q
Switch3(config-if)#switchport mode trunk
Switch3(config-if)#switchport trunk native vlan 999
Switch3(config-if)#switchport trunk allowed vlan 999
Switch3(config-if)#switchport nonegotiate
Switch3(config-if)#no shutdown
Switch3(config-if)#exit
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/3 (1), with Switch2 FastEthernet0/3 (999).
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/4 (1), with Switch2 FastEthernet0/4 (999).
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/1 (1), with Switch1 FastEthernet0/3 (999).
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/2 (1), with Switch1 FastEthernet0/4 (999).
```

```
Switch3(config)#interface range fa0/1 - 2
Switch3(config-if-range)#switchport trunk encapsulation dot1q
Switch3(config-if-range)#switchport mode trunk
Switch3(config-if-range)#switchport trunk native vlan 999
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up
Switch3(config-if-range)#switchport trunk allowed vlan 999
Switch3(config-if-range)#switchport nonegotiate
Switch3(config-if-range)#channel-group 13 mode active
Switch3(config-if-range)#no shutdown
Switch3(config-if-range)#exit
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up
%LINK-5-CHANGED: Interface Port-channel13, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel13, changed state to up
```

### 3.6.3 EtherChannel S3↔S2 (Po23, Fa0/3-4)

```
Switch3(config)#interface port-channel 23
Switch3(config-if)#switchport trunk encapsulation dot1q
Switch3(config-if)#switchport mode trunk
Switch3(config-if)#switchport trunk native vlan 999
Switch3(config-if)#switchport trunk allowed vlan 999
Switch3(config-if)#switchport nonegotiate
Switch3(config-if)#no shutdown
Switch3(config-if)#exit
```

```
Switch3(config)#interface range fa0/3 - 4
Switch3(config-if-range)# switchport trunk encapsulation dot1q
Switch3(config-if-range)# switchport mode trunk
Switch3(config-if-range)# switchport trunk native vlan 999
Switch3(config-if-range)# switchport trunk allowed vlan 999
Switch3(config-if-range)# switchport nonegotiate
Switch3(config-if-range)# channel-group 23 mode active
Switch3(config-if-range)# no shutdown
Switch3(config-if-range)#exit
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up
%LINK-5-CHANGED: Interface Port-channel23, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Port-channel23, changed state to up
```
### 3.6.5 Trunk к Router4
```
Switch3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch3(config)#interface gi0/1
Switch3(config-if)#switchport mode access
Switch3(config-if)#switchport access vlan 999
Switch3(config-if)#no shut
Switch3(config-if)#end
Switch3#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
Switch3#
```
### 3.6.6 Management IP (SVI VLAN999)

```
Switch3(config)#interface vlan 999
Switch3(config-if)# ip address 192.168.255.13 255.255.255.240
Switch3(config-if)#no shutdown
Switch3(config-if)#exit
Switch3(config)#ip default-gateway 192.168.255.1
Switch3(config)#end
Switch3#wr
Building configuration...
[OK]
%LINK-5-CHANGED: Interface Vlan999, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan999, changed state to up
%SYS-5-CONFIG_I: Configured from console by console
```

## 3.7 Проверка и результаты
### 3.7.1 Проверка EtherChannel на каждом коммутаторе

```
show etherchannel summary
```

<img width="408" height="280" alt="image" src="https://github.com/user-attachments/assets/fa110821-7f24-4aa4-8e38-ff654c8eacfc" />
<img width="407" height="280" alt="image" src="https://github.com/user-attachments/assets/cd6418ad-4e81-4888-90a4-956b5e4f6371" />
<img width="409" height="278" alt="image" src="https://github.com/user-attachments/assets/9f5e7a24-6fc0-47bb-a3f8-34f26a4d1532" />

Результат:

* На Switch1: Po12(SU) и Po13(SU), порты в составе помечены (P).
* На Switch2: Po12(SU) и Po23(SU), порты (P).
* На Switch3: Po13(SU) и Po23(SU), порты (P).

Это означает, что порт-каналы работают на L2 (S) и находятся в использовании (U), а физические порты агрегированы (P).

### 3.7.2 Проверка trunk на Port-Channel
```
show interfaces trunk
```
<img width="470" height="230" alt="image" src="https://github.com/user-attachments/assets/d56a6468-9402-41ea-abbd-326cacfc84dc" />
<img width="471" height="255" alt="image" src="https://github.com/user-attachments/assets/4f0c2a5d-d109-4941-b929-2248e9910f40" />
<img width="470" height="223" alt="image" src="https://github.com/user-attachments/assets/da335f73-23fe-4e9e-8d32-5c9b2dc4488c" />

Результат:

* Po12/Po13/Po23 работают в режиме trunking, encapsulation 802.1Q, native VLAN 999, allowed VLAN 999.

### 3.7.3 Проверка STP VLAN999

```
show spanning-tree vlan 999
```
Результат:

<img width="487" height="266" alt="image" src="https://github.com/user-attachments/assets/785620c0-b086-4c05-add2-caa7b69aac94" />
<img width="486" height="256" alt="image" src="https://github.com/user-attachments/assets/df7465e3-d455-482c-82d4-7c54c057176c" />
<img width="484" height="254" alt="image" src="https://github.com/user-attachments/assets/924e84f6-ba03-49a2-a5de-2b277a09a4b9" />

* Switch1 является Root Bridge (This bridge is the root), а Po12 и Po13 находятся в роли Designated Forwarding.
* На Switch2 корневой порт Po12 Root FWD, а Po23 Designated FWD.
* На Switch3 корневой порт Po13 Root FWD, а Po23 Altn BLK.

Ядро имеет топологию треугольника, что создаёт потенциальную L2-петлю. RSTP блокирует один из путей (на Switch3 - Po23 в состоянии Alternate/Blocking), предотвращая петли, но оставляя резервный маршрут. При отказе канала Po13 ожидается переведение Po23 в Forwarding, сохраняя связность сети.



# Этап 4. Access-switch: VLAN + access-порты + trunk к роутеру + MGMT VLAN99
На данном этапе выполняется логическая сегментация сети на уровне L2 с помощью VLAN, назначаются access-порты для конечных устройств (ПК и серверов), настраивается магистральный trunk-порт к маршрутизатору для последующей реализации "Роутер на палочке", а также создаётся отдельная VLAN управления (VLAN 99) с SVI-интерфейсом для удалённого администрирования коммутатора. Дополнительно на портах доступа включаются механизмы PortFast и BPDU Guard для повышения устойчивости сети и защиты от петель.
## 4.1 Switch4 (HQ)
### 4.1.1 Базовая настройка + VLAN

```
Switch4>enable
Switch4#conf t
Switch4(config)#hostname Switch4
Switch4(config)#no ip domain-lookup
Switch4(config)#spanning-tree mode rapid-pvst
```
```
Switch4(config)#vlan 10
Switch4(config-vlan)#name ADMIN
Switch4(config-vlan)#vlan 20
Switch4(config-vlan)#name USERS
Switch4(config-vlan)#vlan 30
Switch4(config-vlan)#name SERVERS
Switch4(config-vlan)#vlan 99
Switch4(config-vlan)#name MGMT
Switch4(config-vlan)#exit
```
### 4.1.2 Access-порты (ПК/Сервера) + PortFast/BPDU guard
```
Switch4(config)#interface fa0/1
Switch4(config-if)#switchport mode access
Switch4(config-if)#switchport access vlan 10
Switch4(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/1 but will only
have effect when the interface is in a non-trunking mode.
Switch4(config-if)#spanning-tree bpduguard enable
Switch4(config-if)#exit
```
```
Switch4(config)#interface fa0/2
Switch4(config-if)#switchport mode access
Switch4(config-if)#switchport access vlan 20
Switch4(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/2 but will only
have effect when the interface is in a non-trunking mode.
Switch4(config-if)#spanning-tree bpduguard enable
Switch4(config-if)#exit
```
```
Switch4(config)#interface fa0/3
Switch4(config-if)#switchport mode access
Switch4(config-if)#switchport access vlan 30
Switch4(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/3 but will only
have effect when the interface is in a non-trunking mode.
Switch4(config-if)#spanning-tree bpduguard enable
Switch4(config-if)#exit
```
```
Switch4(config)#interface fa0/4
Switch4(config-if)#switchport mode access
Switch4(config-if)#switchport access vlan 30
Switch4(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/4 but will only
have effect when the interface is in a non-trunking mode.
Switch4(config-if)#spanning-tree bpduguard enable
Switch4(config-if)#exit
```
### 4.1.3 Trunk к Router2
```
Switch4(config)#interface gi0/1
Switch4(config-if)#switchport mode trunk
Switch4(config-if)#switchport trunk allowed vlan 10,20,30,99
Switch4(config-if)#switchport nonegotiate
Switch4(config-if)#no shutdown
Switch4(config-if)#exit
```
### 4.1.4 MGMT SVI VLAN99 + default-gateway
```
Switch4(config)#interface vlan 99
Switch4(config-if)#ip address 192.168.13.2 255.255.255.0
Switch4(config-if)#no shutdown
Switch4(config-if)#exit
Switch4(config)#ip default-gateway 192.168.13.1
Switch4(config)#end
Switch4#wr
```
### 4.1.5 Проверки
```show vlan brief```

VLAN 10/20/30/99 есть:

<img width="571" height="259" alt="image" src="https://github.com/user-attachments/assets/a68c5023-c884-4e3a-8065-111e33046588" />

```show interfaces status```

Fa0/1-4 в нужных VLAN:

<img width="572" height="398" alt="image" src="https://github.com/user-attachments/assets/436ce954-57a7-441c-8786-10e5f173c775" />

## 4.2 Switch5 (Branch1)
### 4.2.1 Базовая настройка + VLAN
```
Switch>en
Switch#conf t
Switch(config)#hostname Switch5
Switch5(config)#no ip domain-lookup
Switch5(config)#spanning-tree mode rapid-pvst
```
```
Switch5(config)#vlan 10
Switch5(config-vlan)#name ADMIN
Switch5(config-vlan)#vlan 20
Switch5(config-vlan)#name USERS
Switch5(config-vlan)#vlan 40
Switch5(config-vlan)#name GUEST
Switch5(config-vlan)#vlan 99
Switch5(config-vlan)#name MGMT
Switch5(config-vlan)#exit
```
### 4.2.2 Access-порты
```
Switch5(config)#interface fa0/1
Switch5(config-if)#switchport mode access
Switch5(config-if)#switchport access vlan 10
Switch5(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/1 but will only
have effect when the interface is in a non-trunking mode.
Switch5(config-if)#spanning-tree bpduguard enable
Switch5(config-if)#exit
```
```
Switch5(config)#interface fa0/2
Switch5(config-if)#switchport mode access
Switch5(config-if)#switchport access vlan 20
Switch5(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/2 but will only
have effect when the interface is in a non-trunking mode.
Switch5(config-if)#spanning-tree bpduguard enable
Switch5(config-if)#exit
```
```
Switch5(config)#interface fa0/3
Switch5(config-if)#switchport mode access
Switch5(config-if)#switchport access vlan 40
Switch5(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/3 but will only
have effect when the interface is in a non-trunking mode.
Switch5(config-if)#spanning-tree bpduguard enable
Switch5(config-if)#exit
```
### 4.2.3 Trunk к Router3
```
Switch5(config)#interface gi0/1
Switch5(config-if)#switchport mode trunk
Switch5(config-if)#switchport trunk allowed vlan 10,20,40,99
Switch5(config-if)#switchport nonegotiate
Switch5(config-if)#no shutdown
Switch5(config-if)#exit
Switch5(config)#
```
### 4.2.4 MGMT SVI VLAN99
```
Switch5(config)#interface vlan 99
Switch5(config-if)#ip address 192.168.23.2 255.255.255.0
Switch5(config-if)#no shutdown
Switch5(config-if)#exit
Switch5(config)#ip default-gateway 192.168.23.1
Switch5(config)#end
Switch5#wr
```
### 4.2.5 Проверки
```show vlan brief```

VLAN 10/20/30/99 есть:

<img width="567" height="255" alt="image" src="https://github.com/user-attachments/assets/c7315d20-6bf0-44da-b87c-59a660d1448b" />

```show interfaces status```

Fa0/1-3 в нужных VLAN:

<img width="572" height="403" alt="image" src="https://github.com/user-attachments/assets/78175dcc-bf17-4291-b1fc-95f5fd5630b7" />


## 4.3 Switch6 (Branch2)
### 4.3.1 Базовая настройка + VLAN
```
Switch>en
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname Switch6
Switch6(config)#no ip domain-lookup
Switch6(config)#spanning-tree mode rapid-pvst
```
```
Switch6(config)#vlan 10
Switch6(config-vlan)#name ADMIN
Switch6(config-vlan)#vlan 20
Switch6(config-vlan)#name USERS
Switch6(config-vlan)#vlan 99
Switch6(config-vlan)#name MGMT
Switch6(config-vlan)#exit
```
### 4.3.2 Access-порты
```
Switch6(config)#interface fa0/1
Switch6(config-if)#switchport mode access
Switch6(config-if)#switchport access vlan 10
Switch6(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/1 but will only
have effect when the interface is in a non-trunking mode.
Switch6(config-if)#spanning-tree bpduguard enable
Switch6(config-if)#exit
```
```
Switch6(config)#interface fa0/2
Switch6(config-if)#switchport mode access
Switch6(config-if)#switchport access vlan 20
Switch6(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

%Portfast has been configured on FastEthernet0/2 but will only
have effect when the interface is in a non-trunking mode.
Switch6(config-if)#spanning-tree bpduguard enable
Switch6(config-if)#exit
```
### 4.3.3 Trunk к Router4
```
Switch6(config)#interface gi0/1
Switch6(config-if)#switchport mode trunk
Switch6(config-if)#switchport trunk allowed vlan 10,20,99
Switch6(config-if)#switchport nonegotiate
Switch6(config-if)#no shutdown
Switch6(config-if)#exit
```
### 4.3.4 MGMT SVI VLAN99
```
Switch6(config)#interface vlan 99
Switch6(config-if)#ip address 192.168.32.2 255.255.255.0
Switch6(config-if)#no shutdown
Switch6(config-if)#exit
Switch6(config)#ip default-gateway 192.168.32.1
Switch6(config)#end
Switch6#wr
```
### 4.3.5 Проверки
```show vlan brief```

VLAN 10/20/99 есть:

<img width="571" height="244" alt="image" src="https://github.com/user-attachments/assets/c46f0751-8a4f-4dc4-924b-e0692dc8d613" />


```show interfaces status```

Fa0/1-2 в нужных VLAN:

<img width="569" height="401" alt="image" src="https://github.com/user-attachments/assets/2b7ef4d3-25d1-4bce-aa17-befcbd57d1fa" />

# Этап 5. Роутеры: интерфейсы + ROAS (sub-interfaces) + OSPFv2
## 5.1 Router2 (HQ)
## 5.1.1 Шлюзы HQ VLAN + DHCP сервер
```
Router>en
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname Router2
Router2(config)#no ip domain-lookup
```
```
Router2(config)#interface g0/0/0
Router2(config-if)#ip address 192.168.255.2 255.255.255.240
Router2(config-if)#no shut
Router2(config-if)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up
Router2(config)#interface g0/0/1
Router2(config-if)#no shut
Router2(config-if)#exit
```
```
Router2(config)#interface g0/0/1.10
Router2(config-subif)#encapsulation dot1Q 10
Router2(config-subif)#ip address 192.168.10.1 255.255.255.0
Router2(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.10, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.10, changed state to up
Router2(config)#interface g0/0/1.20
Router2(config-subif)#encapsulation dot1Q 20
Router2(config-subif)#ip address 192.168.11.1 255.255.255.0
Router2(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.20, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.20, changed state to up
Router2(config)#interface g0/0/1.30
Router2(config-subif)#encapsulation dot1Q 30
Router2(config-subif)#ip address 192.168.12.1 255.255.255.0
Router2(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.30, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.30, changed state to up
Router2(config)#interface g0/0/1.99
Router2(config-subif)#encapsulation dot1Q 99
Router2(config-subif)#ip address 192.168.13.1 255.255.255.0
Router2(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.99, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.99, changed state to up
```
```
Router2(config)#router ospf 1
Router2(config-router)#router-id 2.2.2.2
Router2(config-router)#network 192.168.255.0 0.0.0.15 area 0
Router2(config-router)#network 192.168.10.0 0.0.0.255 area 0
Router2(config-router)#network 192.168.11.0 0.0.0.255 area 0
Router2(config-router)#network 192.168.12.0 0.0.0.255 area 0
Router2(config-router)#network 192.168.13.0 0.0.0.255 area 0
Router2(config-router)#exit
Router2(config)#end
Router2#wr
```
## 5.2 Router3 (Branch1)
### 5.2.1 Шлюзы BR1 VLAN + DHCP relay + ACL guest
```
Router>en
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname Router3
Router3(config)#no ip domain-lookup
```
```
Router3(config)#interface g0/0/0
Router3(config-if)#ip address 192.168.255.3 255.255.255.240
Router3(config-if)#no shut
Router3(config-if)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up
Router3(config)#interface g0/0/1
Router3(config-if)#no shut
Router3(config-if)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up
```
```
Router3(config)#interface g0/0/1.10
Router3(config-subif)#encapsulation dot1Q 10
Router3(config-subif)#ip address 192.168.20.1 255.255.255.0
Router3(config-subif)#ip helper-address 192.168.255.2
Router3(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.10, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.10, changed state to up
Router3(config)#interface g0/0/1.20
Router3(config-subif)#encapsulation dot1Q 20
Router3(config-subif)#ip address 192.168.21.1 255.255.255.0
Router3(config-subif)#ip helper-address 192.168.255.2
Router3(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.20, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.20, changed state to up
Router3(config)#interface g0/0/1.40
Router3(config-subif)#encapsulation dot1Q 40
Router3(config-subif)#ip address 192.168.22.1 255.255.255.0
Router3(config-subif)#ip helper-address 192.168.255.2
Router3(config-subif)#ip access-group GUEST_IN in
Router3(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.40, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.40, changed state to up
Router3(config)#interface g0/0/1.99
Router3(config-subif)#encapsulation dot1Q 99
Router3(config-subif)#ip address 192.168.23.1 255.255.255.0
Router3(config-subif)#ip helper-address 192.168.255.2
Router3(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.99, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.99, changed state to up
```
```
Router3(config)#router ospf 1
Router3(config-router)#router-id 3.3.3.3
Router3(config-router)#network 192.168.255.0 0.0.0.15 area 0
Router3(config-router)#network 192.168.20.0 0.0.0.255 area 0
Router3(config-router)#network 192.168.21.0 0.0.0.255 area 0
Router3(config-router)#network 192.168.22.0 0.0.0.255 area 0
Router3(config-router)#network 192.168.23.0 0.0.0.255 area 0
Router3(config-router)#exit
Router3(config)#end
Router3#wr
```
## 5.3 Router4 (Branch2)
### 5.3.1 Шлюзы BR2 VLAN + DHCP relay
```
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname Router4
Router4(config)#no ip domain-lookup
```
```
Router4(config)#interface g0/0/0
Router4(config-if)#ip address 192.168.255.4 255.255.255.240
Router4(config-if)#no shut
Router4(config-if)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up
Router4(config)#interface g0/0/1
Router4(config-if)#no shut
Router4(config-if)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up
```
```
Router4(config)#interface g0/0/1.10
Router4(config-subif)#encapsulation dot1Q 10
Router4(config-subif)#ip address 192.168.30.1 255.255.255.0
Router4(config-subif)#ip helper-address 192.168.255.2
Router4(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.10, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.10, changed state to up
Router4(config)#interface g0/0/1.20
Router4(config-subif)#encapsulation dot1Q 20
Router4(config-subif)#ip address 192.168.31.1 255.255.255.0
Router4(config-subif)#ip helper-address 192.168.255.2
Router4(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.20, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.20, changed state to up
Router4(config)#interface g0/0/1.99
Router4(config-subif)#encapsulation dot1Q 99
Router4(config-subif)#ip address 192.168.32.1 255.255.255.0
Router4(config-subif)#ip helper-address 192.168.255.2
Router4(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.99, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.99, changed state to up
```
```
Router4(config)#router ospf 1
Router4(config-router)#router-id 4.4.4.4
Router4(config-router)#network 192.168.255.0 0.0.0.15 area 0
Router4(config-router)#network 192.168.30.0 0.0.0.255 area 0
Router4(config-router)#network 192.168.31.0 0.0.0.255 area 0
Router4(config-router)#network 192.168.32.0 0.0.0.255 area 0
Router4(config-router)#exit
Router4(config)#end
Router4#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
```

# Этап 6. Router1 (EDGE): trunk VLAN999 + внешняя сеть + OSPF + NAT + WAN ACL
```
Router>en
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname Router1
Router1(config)#no ip domain-lookup
```
## 6.1 Внутрь в ядро: trunk на Switch1 Gi0/1
```
Router1(config)#interface g0/0/0
Router1(config-if)#no shut
Router1(config-if)#exit
Router1(config)#interface g0/0/0.999
Router1(config-subif)#encapsulation dot1Q 999 native
Router1(config-subif)#ip address 192.168.255.1 255.255.255.240
Router1(config-subif)#ip nat inside
Router1(config-subif)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0.999, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0.999, changed state to up
```
## 6.2 Наружу (к InternetServer/ISP-SW)
```
Router1(config)#interface g0/0/1
Router1(config-if)#ip address 203.0.113.2 255.255.255.0
Router1(config-if)#ip nat outside
Router1(config-if)#no shut
Router1(config-if)#exit
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up
```
## 6.3 OSPF для назначения пути к 203.0.113.0/24 через R1
```
Router1(config)#router ospf 1
Router1(config-router)#router-id 1.1.1.1
Router1(config-router)#network 192.168.255.0 0.0.0.15 area 0
Router1(config-router)#network 203.0.113.0 0.0.0.255 area 0
Router1(config-router)#exit
```
## 6.4 NAT overload для всего 192.168.0.0/16
```
Router1(config)#access-list 1 permit 192.168.0.0 0.0.255.255
Router1(config)#ip nat inside source list 1 interface g0/0/1 overload
```
## 6.5 Порт-форвардинг наружу на внутренний WEB
```
Router1(config)#ip nat inside source static tcp 192.168.12.20 80 203.0.113.2 80
Router1(config)#ip nat inside source static tcp 192.168.12.20 443 203.0.113.2 443
```
## 6.6 WAN ACL: разрешаем вход только 80/443 на публичный IP
```
Router1(config)#ip access-list extended WAN_IN
Router1(config-ext-nacl)#remark Allow return TCP (established) to NAT/PAT
Router1(config-ext-nacl)#permit tcp any host 203.0.113.2 established
Router1(config-ext-nacl)#remark Allow inbound to published WEB (port-forward)
Router1(config-ext-nacl)#permit tcp any host 203.0.113.2 eq 80
Router1(config-ext-nacl)#permit tcp any host 203.0.113.2 eq 443
Router1(config-ext-nacl)#permit icmp any host 203.0.113.2 echo-reply
Router1(config-ext-nacl)#permit udp any host 203.0.113.2 eq 53
Router1(config-ext-nacl)#permit udp any host 203.0.113.2 gt 1023
Router1(config-ext-nacl)#deny ip any any
Router1(config-ext-nacl)#exit
Router1(config)#interface g0/0/1
Router1(config-if)#ip access-group WAN_IN in
Router1(config-if)#exit
Router1(config)#end
Router1#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
```
## CDP на Router1
```
Router1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router1(config)#cdp run
Router1(config)#interface g0/0/0
Router1(config-if)#cdp enable
Router1(config-if)#end
Router1#
%SYS-5-CONFIG_I: Configured from console by console
Router1#wr
Building configuration...
[OK]
```

# Этап 7. DHCP (на Router2) + статические адреса серверов
## 7.1 Настройка адресов на серверах

DNS: 192.168.12.10/24, GW 192.168.12.1

<img width="704" height="266" alt="image" src="https://github.com/user-attachments/assets/382c338e-2599-44a5-847f-14a38d804d9b" />

WEB: 192.168.12.20/24, GW 192.168.12.1

<img width="703" height="266" alt="image" src="https://github.com/user-attachments/assets/4e0cdbf5-acaa-43e4-a0c8-d578a09fae61" />


## 7.2 DHCP на Router2 для VLAN
### 7.2.1 Исключения (шлюзы, сервера, mgmt IP свитчей)
```
Router2(config)#ip dhcp excluded-address 192.168.10.1 192.168.10.30
Router2(config)#ip dhcp excluded-address 192.168.11.1 192.168.11.30
Router2(config)#ip dhcp excluded-address 192.168.12.1 192.168.12.50
Router2(config)#ip dhcp excluded-address 192.168.13.1 192.168.13.20
```
```
Router2(config)#ip dhcp excluded-address 192.168.20.1 192.168.20.30
Router2(config)#ip dhcp excluded-address 192.168.21.1 192.168.21.30
Router2(config)#ip dhcp excluded-address 192.168.22.1 192.168.22.30
Router2(config)#ip dhcp excluded-address 192.168.23.1 192.168.23.20
```
```
Router2(config)#ip dhcp excluded-address 192.168.30.1 192.168.30.30
Router2(config)#ip dhcp excluded-address 192.168.31.1 192.168.31.30
Router2(config)#ip dhcp excluded-address 192.168.32.1 192.168.32.20
```
### 7.2.2 HQ
```
Router2(config)#ip dhcp pool HQ_ADMIN
Router2(dhcp-config)#network 192.168.10.0 255.255.255.0
Router2(dhcp-config)#default-router 192.168.10.1
Router2(dhcp-config)#dns-server 192.168.12.10
Router2(dhcp-config)#domain-name diploma.local
Router2(dhcp-config)#exit
Router2(config)#
Router2(config)#ip dhcp pool HQ_USERS
Router2(dhcp-config)#network 192.168.11.0 255.255.255.0
Router2(dhcp-config)#default-router 192.168.11.1
Router2(dhcp-config)#dns-server 192.168.12.10
Router2(dhcp-config)#domain-name diploma.local
Router2(dhcp-config)#exit
Router2(config)#ip dhcp pool HQ_SERVERS
Router2(dhcp-config)#network 192.168.12.0 255.255.255.0
Router2(dhcp-config)#default-router 192.168.12.1
Router2(dhcp-config)#dns-server 192.168.12.10
Router2(dhcp-config)#domain-name diploma.local
Router2(dhcp-config)#exit
```
### 7.2.3 Branch1 (Router3)
```
Router2(config)#ip dhcp pool BR1_ADMIN
Router2(dhcp-config)#network 192.168.20.0 255.255.255.0
Router2(dhcp-config)#default-router 192.168.20.1
Router2(dhcp-config)#dns-server 192.168.12.10
Router2(dhcp-config)#domain-name diploma.local
Router2(dhcp-config)#exit
Router2(config)#
Router2(config)#ip dhcp pool BR1_USERS
Router2(dhcp-config)#network 192.168.21.0 255.255.255.0
Router2(dhcp-config)#default-router 192.168.21.1
Router2(dhcp-config)#dns-server 192.168.12.10
Router2(dhcp-config)#domain-name diploma.local
Router2(dhcp-config)#exit
Router2(config)#
Router2(config)#ip dhcp pool BR1_GUEST
Router2(dhcp-config)#network 192.168.22.0 255.255.255.0
Router2(dhcp-config)#default-router 192.168.22.1
Router2(dhcp-config)#dns-server 192.168.12.10
Router2(dhcp-config)#domain-name diploma.local
Router2(dhcp-config)#exit
```
### 7.2.4 Branch2 (Router4)
```
Router2(config)#ip dhcp pool BR2_ADMIN
Router2(dhcp-config)#network 192.168.30.0 255.255.255.0
Router2(dhcp-config)#default-router 192.168.30.1
Router2(dhcp-config)#dns-server 192.168.12.10
Router2(dhcp-config)#domain-name diploma.local
Router2(dhcp-config)#exit
Router2(config)#
Router2(config)#ip dhcp pool BR2_USERS
Router2(dhcp-config)#network 192.168.31.0 255.255.255.0
Router2(dhcp-config)#default-router 192.168.31.1
Router2(dhcp-config)#dns-server 192.168.12.10
Router2(dhcp-config)#domain-name diploma.local
Router2(dhcp-config)#exit
Router2(config)#
Router2(config)#end
Router2#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
Router2#
```

# Этап 8. ACL для гостевого Wi-Fi на Router3

Guest (192.168.22.0/24) не должен ходить в 192.168.0.0/16, но должен:

* спрашивать DNS (192.168.12.10)
* обращаться к серверу во внешней сети (InternetServer 203.0.113.10)
* ping до внешней сети
```
Router3>en
Router3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router3(config)#ip access-list extended GUEST_IN
Router3(config-ext-nacl)#permit udp any eq bootpc any eq bootps
Router3(config-ext-nacl)#permit udp any eq bootps any eq bootpc
Router3(config-ext-nacl)#permit udp 192.168.22.0 0.0.0.255 host 192.168.12.10 eq 53
Router3(config-ext-nacl)#permit tcp 192.168.22.0 0.0.0.255 host 192.168.12.10 eq 53
Router3(config-ext-nacl)#permit tcp 192.168.22.0 0.0.0.255 host 203.0.113.10 eq 80
Router3(config-ext-nacl)#permit tcp 192.168.22.0 0.0.0.255 host 203.0.113.10 eq 443
Router3(config-ext-nacl)#permit icmp 192.168.22.0 0.0.0.255 host 203.0.113.10
Router3(config-ext-nacl)#deny   ip 192.168.22.0 0.0.0.255 192.168.0.0 0.0.255.255
Router3(config-ext-nacl)#permit ip 192.168.22.0 0.0.0.255 any
Router3(config-ext-nacl)#exit
Router3(config)#end
Router3#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
Router3#
```

# ЭТАП 9. SSH + ограничение доступа (VTY ACL)
## 9.1 Включаем SSH на всех роутерах и свитчах
```
Router1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router1(config)#hostname Router1
Router1(config)#ip domain-name diploma.local
Router1(config)#username admin privilege 15 secret Admin12345
Router1(config)#crypto key generate rsa
The name for the keys will be: Router1.diploma.local
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
Router1(config)#ip ssh version 2
*Mar 1 3:37:7.747: %SSH-5-ENABLED: SSH 2 has been enabled
Router1(config)#line vty 0 4
Router1(config-line)#login local
Router1(config-line)#transport input ssh
Router1(config-line)#exec-timeout 10 0
Router1(config-line)#end
Router1#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
Router1#
```
## 9.2 Ограничиваем SSH только с ADMIN VLAN (VTY ACL)
На всех устройствах:
```
Router1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router1(config)#ip access-list standard VTY_ADMIN_ONLY
Router1(config-std-nacl)#permit 192.168.10.0 0.0.0.255
Router1(config-std-nacl)#permit 192.168.20.0 0.0.0.255
Router1(config-std-nacl)#permit 192.168.30.0 0.0.0.255
Router1(config-std-nacl)#deny any
Router1(config-std-nacl)#exit
Router1(config)#
Router1(config)#line vty 0 4
Router1(config-line)#access-class VTY_ADMIN_ONLY in
Router1(config-line)#exit
Router1(config)#end
Router1#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
Router1#
```

В итоге: SSH на любое устройство будет работать только если подключиться с ПК из VLAN10 (ADMIN) на любой площадке.

# Этап 10. Проверка работоспособности
## 10.1 L2: VLAN / Trunk / EtherChannel / STP
#### Switch1/2/3:
```
Switch1>en
Password: 
Switch1#show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port
Number of channel-groups in use: 2
Number of aggregators:           2
Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

12     Po12(SU)           LACP   Fa0/1(P) Fa0/2(P) 
13     Po13(SU)           LACP   Fa0/3(P) Fa0/4(P) 
Switch1#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Po12        on           802.1q         trunking      999
Po13        on           802.1q         trunking      999
Gig0/1      on           802.1q         trunking      999

Port        Vlans allowed on trunk
Po12        999
Po13        999
Gig0/1      999

Port        Vlans allowed and active in management domain
Po12        999
Po13        999
Gig0/1      999

Port        Vlans in spanning tree forwarding state and not pruned
Po12        999
Po13        999
Gig0/1      999

Switch1#show spanning-tree vlan 999
VLAN0999
  Spanning tree enabled protocol rstp
  Root ID    Priority    25575
             Address     00D0.9799.12AA
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    25575  (priority 24576 sys-id-ext 999)
             Address     00D0.9799.12AA
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Desg FWD 4         128.25   P2p
Gi0/2            Desg FWD 4         128.26   P2p
Po13             Desg FWD 9         128.28   Shr
Po12             Desg FWD 9         128.27   Shr

Switch1#show cdp neighbors
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone
Device ID    Local Intrfce   Holdtme    Capability   Platform    Port ID
Router2      Gig 0/2          159            R       ISR4300     Gig 0/0/0
Router1      Gig 0/1          160            R       ISR4300     Gig 0/0/0
Switch3      Por 13           160                    3560        Fas 0/1
Switch3      Por 13           159                    3560        Fas 0/2
Router1      Gig 0/1          160            R       ISR4300     Gig 0/0/0.999
Switch3      Por 13           160                    3560        Por 13
Switch2      Por 12           160                    3560        Fas 0/1
Switch2      Por 12           160                    3560        Fas 0/2
Switch2      Por 12           160                    3560        Por 12
Switch1#
```
```
Switch2#show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

12     Po12(SU)           LACP   Fa0/1(P) Fa0/2(P) 
23     Po23(SU)           LACP   Fa0/3(P) Fa0/4(P) 
Switch2#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Po12        on           802.1q         trunking      999
Po23        on           802.1q         trunking      999

Port        Vlans allowed on trunk
Po12        999
Po23        999

Port        Vlans allowed and active in management domain
Po12        999
Po23        999

Port        Vlans in spanning tree forwarding state and not pruned
Po12        999
Po23        999

Switch2#show spanning-tree vlan 999
VLAN0999
  Spanning tree enabled protocol rstp
  Root ID    Priority    25575
             Address     00D0.9799.12AA
             Cost        9
             Port        27(Port-channel12)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    29671  (priority 28672 sys-id-ext 999)
             Address     000B.BE6C.D00D
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Desg FWD 4         128.25   P2p
Po12             Root FWD 9         128.27   Shr
Po23             Desg FWD 9         128.28   Shr

Switch2#show cdp neighbors
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone
Device ID    Local Intrfce   Holdtme    Capability   Platform    Port ID
Router3      Gig 0/1          133            R       ISR4300     Gig 0/0/0
Switch1      Por 12           133                    3560        Fas 0/1
Switch1      Por 12           133                    3560        Fas 0/2
Switch1      Por 12           133                    3560        Por 12
Switch3      Por 23           133                    3560        Fas 0/3
Switch3      Por 23           133                    3560        Fas 0/4
Switch3      Por 23           133                    3560        Por 23
Switch2#
```
```
Switch3#show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

13     Po13(SU)           LACP   Fa0/1(P) Fa0/2(P) 
23     Po23(SU)           LACP   Fa0/3(P) Fa0/4(P) 
Switch3#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Po13        on           802.1q         trunking      999
Po23        on           802.1q         trunking      999

Port        Vlans allowed on trunk
Po13        999
Po23        999

Port        Vlans allowed and active in management domain
Po13        999
Po23        999

Port        Vlans in spanning tree forwarding state and not pruned
Po13        999
Po23        none

Switch3#show spanning-tree vlan 999
VLAN0999
  Spanning tree enabled protocol rstp
  Root ID    Priority    25575
             Address     00D0.9799.12AA
             Cost        9
             Port        27(Port-channel13)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    33767  (priority 32768 sys-id-ext 999)
             Address     0004.9A98.5207
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/1            Desg FWD 4         128.25   P2p
Po13             Root FWD 9         128.27   Shr
Po23             Altn BLK 9         128.28   Shr

Switch3#show cdp neighbors
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone
Device ID    Local Intrfce   Holdtme    Capability   Platform    Port ID
Switch1      Por 13           121                    3560        Fas 0/3
Switch1      Por 13           121                    3560        Fas 0/4
Switch1      Por 13           121                    3560        Por 13
Switch2      Por 23           121                    3560        Fas 0/3
Switch2      Por 23           121                    3560        Fas 0/4
Switch2      Por 23           121                    3560        Por 23
Switch3#
```

#### Switch4/5/6:
```
Switch4#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/2
10   ADMIN                            active    Fa0/1
20   USERS                            active    Fa0/2
30   SERVERS                          active    Fa0/3, Fa0/4
99   MGMT                             active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
Switch4#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Gig0/1      10,20,30,99

Port        Vlans allowed and active in management domain
Gig0/1      10,20,30,99

Port        Vlans in spanning tree forwarding state and not pruned
Gig0/1      10,20,30,99

Switch4#show spanning-tree vlan 10
VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    32778
             Address     0001.42CC.0C93
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     0001.42CC.0C93
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Gi0/1            Desg FWD 4         128.25   P2p

Switch4#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    32788
             Address     0001.42CC.0C93
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     0001.42CC.0C93
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Gi0/1            Desg FWD 4         128.25   P2p

Switch4#
```
```
Switch5#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/4, Fa0/5, Fa0/6, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/2
10   ADMIN                            active    Fa0/1
20   USERS                            active    Fa0/2
40   GUEST                            active    Fa0/3
99   MGMT                             active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
Switch5#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Gig0/1      10,20,40,99

Port        Vlans allowed and active in management domain
Gig0/1      10,20,40,99

Port        Vlans in spanning tree forwarding state and not pruned
Gig0/1      10,20,40,99

Switch5#show spanning-tree vlan 10
VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    32778
             Address     0006.2A1E.8574
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     0006.2A1E.8574
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Gi0/1            Desg FWD 4         128.25   P2p

Switch5#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    32788
             Address     0006.2A1E.8574
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     0006.2A1E.8574
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Gi0/1            Desg FWD 4         128.25   P2p

Switch5#
```
```
Switch6#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/3, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/2
10   ADMIN                            active    Fa0/1
20   USERS                            active    Fa0/2
99   MGMT                             active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
Switch6#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Gig0/1      10,20,99

Port        Vlans allowed and active in management domain
Gig0/1      10,20,99

Port        Vlans in spanning tree forwarding state and not pruned
Gig0/1      10,20,99

Switch6#show spanning-tree vlan 10
VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    32778
             Address     00E0.8F12.DA21
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     00E0.8F12.DA21
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Gi0/1            Desg FWD 4         128.25   P2p

Switch6#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol rstp
  Root ID    Priority    32788
             Address     00E0.8F12.DA21
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     00E0.8F12.DA21
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Gi0/1            Desg FWD 4         128.25   P2p

Switch6#
```
## 10.2 OSPFv2: соседство и маршруты
#### R1–R4:
```
Router1#show ip ospf neighbor


Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   2WAY/DROTHER    00:00:38    192.168.255.2   GigabitEthernet0/0/0.999
4.4.4.4           1   FULL/DR         00:00:38    192.168.255.4   GigabitEthernet0/0/0.999
3.3.3.3           1   FULL/BDR        00:00:38    192.168.255.3   GigabitEthernet0/0/0.999
Router1#show ip route ospf
O    192.168.10.0 [110/2] via 192.168.255.2, 00:16:53, GigabitEthernet0/0/0.999
O    192.168.11.0 [110/2] via 192.168.255.2, 00:16:53, GigabitEthernet0/0/0.999
O    192.168.12.0 [110/2] via 192.168.255.2, 00:16:53, GigabitEthernet0/0/0.999
O    192.168.13.0 [110/2] via 192.168.255.2, 00:16:53, GigabitEthernet0/0/0.999
O    192.168.20.0 [110/2] via 192.168.255.3, 00:16:53, GigabitEthernet0/0/0.999
O    192.168.21.0 [110/2] via 192.168.255.3, 00:16:53, GigabitEthernet0/0/0.999
O    192.168.22.0 [110/2] via 192.168.255.3, 00:16:53, GigabitEthernet0/0/0.999
O    192.168.23.0 [110/2] via 192.168.255.3, 00:16:53, GigabitEthernet0/0/0.999
O    192.168.30.0 [110/2] via 192.168.255.4, 00:16:53, GigabitEthernet0/0/0.999
O    192.168.31.0 [110/2] via 192.168.255.4, 00:16:53, GigabitEthernet0/0/0.999
O    192.168.32.0 [110/2] via 192.168.255.4, 00:16:53, GigabitEthernet0/0/0.999

Router1#show ip protocols

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 1.1.1.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    203.0.113.0 0.0.0.255 area 0
    192.168.255.0 0.0.0.15 area 0
  Routing Information Sources:  
    Gateway         Distance      Last Update 
    1.1.1.1              110      00:17:07
    2.2.2.2              110      00:17:07
    3.3.3.3              110      00:17:07
    4.4.4.4              110      00:17:07
  Distance: (default is 110)
```
```
Router2#show ip ospf neighbor


Neighbor ID     Pri   State           Dead Time   Address         Interface
4.4.4.4           1   FULL/DR         00:00:30    192.168.255.4   GigabitEthernet0/0/0
3.3.3.3           1   FULL/BDR        00:00:30    192.168.255.3   GigabitEthernet0/0/0
1.1.1.1           1   2WAY/DROTHER    00:00:30    192.168.255.1   GigabitEthernet0/0/0
Router2#show ip route ospf
O    192.168.20.0 [110/2] via 192.168.255.3, 00:18:01, GigabitEthernet0/0/0
O    192.168.21.0 [110/2] via 192.168.255.3, 00:18:01, GigabitEthernet0/0/0
O    192.168.22.0 [110/2] via 192.168.255.3, 00:18:01, GigabitEthernet0/0/0
O    192.168.23.0 [110/2] via 192.168.255.3, 00:18:01, GigabitEthernet0/0/0
O    192.168.30.0 [110/2] via 192.168.255.4, 00:18:01, GigabitEthernet0/0/0
O    192.168.31.0 [110/2] via 192.168.255.4, 00:18:01, GigabitEthernet0/0/0
O    192.168.32.0 [110/2] via 192.168.255.4, 00:18:01, GigabitEthernet0/0/0
O    203.0.113.0 [110/2] via 192.168.255.1, 00:18:01, GigabitEthernet0/0/0

Router2#show ip protocols

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 2.2.2.2
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.10.0 0.0.0.255 area 0
    192.168.11.0 0.0.0.255 area 0
    192.168.12.0 0.0.0.255 area 0
    192.168.13.0 0.0.0.255 area 0
    192.168.255.0 0.0.0.15 area 0
  Routing Information Sources:  
    Gateway         Distance      Last Update 
    1.1.1.1              110      00:18:15
    2.2.2.2              110      00:18:15
    3.3.3.3              110      00:18:15
    4.4.4.4              110      00:18:15
  Distance: (default is 110)
```
```
Router3#show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DROTHER    00:00:31    192.168.255.2   GigabitEthernet0/0/0
4.4.4.4           1   FULL/DR         00:00:31    192.168.255.4   GigabitEthernet0/0/0
1.1.1.1           1   FULL/DROTHER    00:00:31    192.168.255.1   GigabitEthernet0/0/0
Router3#show ip route ospf
O    192.168.10.0 [110/2] via 192.168.255.2, 00:19:18, GigabitEthernet0/0/0
O    192.168.11.0 [110/2] via 192.168.255.2, 00:19:18, GigabitEthernet0/0/0
O    192.168.12.0 [110/2] via 192.168.255.2, 00:19:18, GigabitEthernet0/0/0
O    192.168.13.0 [110/2] via 192.168.255.2, 00:19:18, GigabitEthernet0/0/0
O    192.168.30.0 [110/2] via 192.168.255.4, 00:19:18, GigabitEthernet0/0/0
O    192.168.31.0 [110/2] via 192.168.255.4, 00:19:18, GigabitEthernet0/0/0
O    192.168.32.0 [110/2] via 192.168.255.4, 00:19:18, GigabitEthernet0/0/0
O    203.0.113.0 [110/2] via 192.168.255.1, 00:19:18, GigabitEthernet0/0/0

Router3#show ip protocols
Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 3.3.3.3
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.20.0 0.0.0.255 area 0
    192.168.21.0 0.0.0.255 area 0
    192.168.22.0 0.0.0.255 area 0
    192.168.23.0 0.0.0.255 area 0
    192.168.255.0 0.0.0.15 area 0
  Routing Information Sources:  
    Gateway         Distance      Last Update 
    1.1.1.1              110      00:19:32
    2.2.2.2              110      00:19:32
    3.3.3.3              110      00:19:32
    4.4.4.4              110      00:19:32
  Distance: (default is 110)
```
```
Router4#show ip ospf neighbor


Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DROTHER    00:00:31    192.168.255.2   GigabitEthernet0/0/0
3.3.3.3           1   FULL/BDR        00:00:31    192.168.255.3   GigabitEthernet0/0/0
1.1.1.1           1   FULL/DROTHER    00:00:31    192.168.255.1   GigabitEthernet0/0/0
Router4#show ip route ospf
O    192.168.10.0 [110/2] via 192.168.255.2, 00:23:35, GigabitEthernet0/0/0
O    192.168.11.0 [110/2] via 192.168.255.2, 00:23:35, GigabitEthernet0/0/0
O    192.168.12.0 [110/2] via 192.168.255.2, 00:23:35, GigabitEthernet0/0/0
O    192.168.13.0 [110/2] via 192.168.255.2, 00:23:35, GigabitEthernet0/0/0
O    192.168.20.0 [110/2] via 192.168.255.3, 00:23:35, GigabitEthernet0/0/0
O    192.168.21.0 [110/2] via 192.168.255.3, 00:23:35, GigabitEthernet0/0/0
O    192.168.22.0 [110/2] via 192.168.255.3, 00:23:35, GigabitEthernet0/0/0
O    192.168.23.0 [110/2] via 192.168.255.3, 00:23:35, GigabitEthernet0/0/0
O    203.0.113.0 [110/2] via 192.168.255.1, 00:23:35, GigabitEthernet0/0/0

Router4#show ip protocols

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set 
  Incoming update filter list for all interfaces is not set 
  Router ID 4.4.4.4
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    192.168.30.0 0.0.0.255 area 0
    192.168.31.0 0.0.0.255 area 0
    192.168.32.0 0.0.0.255 area 0
    192.168.255.0 0.0.0.15 area 0
  Routing Information Sources:  
    Gateway         Distance      Last Update 
    1.1.1.1              110      00:23:43
    2.2.2.2              110      00:23:43
    3.3.3.3              110      00:23:43
    4.4.4.4              110      00:23:43
  Distance: (default is 110)
```
## 10.3 DHCP: выдача адресов + relay
#### Router2 (DHCP сервер):
```
Router2#show ip dhcp pool

Pool HQ_ADMIN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 1
 Excluded addresses             : 11
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.10.1         192.168.10.1     - 192.168.10.254    1    / 11    / 254

Pool HQ_USERS :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 1
 Excluded addresses             : 11
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.11.1         192.168.11.1     - 192.168.11.254    1    / 11    / 254

Pool BR1_ADMIN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 1
 Excluded addresses             : 11
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.20.1         192.168.20.1     - 192.168.20.254    1    / 11    / 254

Pool BR1_USERS :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 1
 Excluded addresses             : 11
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.21.1         192.168.21.1     - 192.168.21.254    1    / 11    / 254

Pool BR1_GUEST :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 1
 Excluded addresses             : 11
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.22.1         192.168.22.1     - 192.168.22.254    1    / 11    / 254

Pool BR2_ADMIN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 1
 Excluded addresses             : 11
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.30.1         192.168.30.1     - 192.168.30.254    1    / 11    / 254

Pool BR2_USERS :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 1
 Excluded addresses             : 11
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.31.1         192.168.31.1     - 192.168.31.254    1    / 11    / 254

Pool HQ_SERVERS :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 0
 Excluded addresses             : 11
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.12.1         192.168.12.1     - 192.168.12.254    0    / 11    / 254
Router2#show ip dhcp binding
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.10.31    00E0.8FCA.B709           --                     Automatic
192.168.11.31    00E0.8F3A.C3E4           --                     Automatic
192.168.20.33    0006.2A3D.2427           --                     Automatic
192.168.21.33    0090.0C28.935C           --                     Automatic
192.168.22.31    00D0.BA2E.7B8A           --                     Automatic
192.168.30.31    0003.E4B3.BAB3           --                     Automatic
192.168.31.31    0030.A39B.29E7           --                     Automatic
```
#### Router3/Router4 (relay):
```
Router3#show running-config | include helper-address
 ip helper-address 192.168.255.2
 ip helper-address 192.168.255.2
 ip helper-address 192.168.255.2
 ip helper-address 192.168.255.2
Router3#show ip interface brief | include 0/0/1
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.10192.168.20.1    YES manual up                    up 
GigabitEthernet0/0/1.20192.168.21.1    YES manual up                    up 
GigabitEthernet0/0/1.40192.168.22.1    YES manual up                    up 
GigabitEthernet0/0/1.99192.168.23.1    YES manual up                    up 
```
```
Router4#show running-config | include helper-address
 ip helper-address 192.168.255.2
 ip helper-address 192.168.255.2
 ip helper-address 192.168.255.2
Router4#show ip interface brief | include 0/0/1
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.10192.168.30.1    YES manual up                    up 
GigabitEthernet0/0/1.20192.168.31.1    YES manual up                    up 
GigabitEthernet0/0/1.99192.168.32.1    YES manual up                    up
```
#### ПК/Ноутбуки получают IP/MASK/GW/DNS:
<img width="367" height="294" alt="image" src="https://github.com/user-attachments/assets/c350c8cd-bc9c-4010-b1e0-1e81cd07f9f0" />
<img width="353" height="291" alt="image" src="https://github.com/user-attachments/assets/60833cd2-9f0c-48c3-a00d-db4f4ab74784" />
<img width="359" height="290" alt="image" src="https://github.com/user-attachments/assets/9878d883-c449-44fe-9aca-0e1a4467f87f" />
<img width="346" height="292" alt="image" src="https://github.com/user-attachments/assets/48372d5a-3d7c-44f4-8b0a-6c0ea02a7da6" />

## 10.4 NAT: PAT + port-forward
#### R1:
```
Router1#show ip nat statistics
Total translations: 2 (2 static, 0 dynamic, 2 extended)
Outside Interfaces: GigabitEthernet0/0/1
Inside Interfaces: GigabitEthernet0/0/0.999
Hits: 0  Misses: 0
Expired translations: 0
Dynamic mappings:
Router1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
tcp 203.0.113.2:443    192.168.12.20:443  ---                ---
tcp 203.0.113.2:80     192.168.12.20:80   ---                ---

Router1#show access-lists 1
Standard IP access list 1
    permit 192.168.0.0 0.0.255.255

Router1#show access-lists WAN_IN
Extended IP access list WAN_IN
    permit icmp any host 203.0.113.2 echo-reply
    permit udp any host 203.0.113.2 eq domain
    permit udp any host 203.0.113.2 gt 1023
    permit tcp any host 203.0.113.2 established
    permit tcp any host 203.0.113.2 eq www
    permit tcp any host 203.0.113.2 eq 443
    deny ip any any
```
#### Тест:
#### ping 203.0.113.10 (InternetServer)
<img width="434" height="357" alt="image" src="https://github.com/user-attachments/assets/e6512987-2636-483f-b487-61c97e2bf5af" />

#### С InternetServer открыть браузер на http://203.0.113.2 (проверка port-forward на WEB)
<img width="695" height="301" alt="image" src="https://github.com/user-attachments/assets/0ea9db99-7dd1-43f0-bde9-0ef77cfe82e3" />

## 10.5 ACL: GUEST ограничен, но интернет/DNS работает
#### R3:
```
Router3#show access-lists GUEST_IN
Extended IP access list GUEST_IN
    permit udp any eq bootpc any eq bootps (3 match(es))
    permit udp any eq bootps any eq bootpc
    permit udp 192.168.22.0 0.0.0.255 host 192.168.12.10 eq domain
    permit tcp 192.168.22.0 0.0.0.255 host 192.168.12.10 eq domain
    permit tcp 192.168.22.0 0.0.0.255 host 203.0.113.10 eq www
    permit tcp 192.168.22.0 0.0.0.255 host 203.0.113.10 eq 443
    permit icmp 192.168.22.0 0.0.0.255 host 203.0.113.10
    deny ip 192.168.22.0 0.0.0.255 192.168.0.0 0.0.255.255
    permit ip 192.168.22.0 0.0.0.255 any

Router3#show ip interface g0/0/1.40
GigabitEthernet0/0/1.40 is up, line protocol is up (connected)
  Internet address is 192.168.22.1/24
  Broadcast address is 255.255.255.255
  Address determined by setup command
  MTU is 1500 bytes
  Helper address is 192.168.255.2
  Directed broadcast forwarding is disabled
  Outgoing access list is not set
  Inbound  access list is GUEST_IN
  Proxy ARP is enabled
  Security level is default
  Split horizon is enabled
  ICMP redirects are always sent
  ICMP unreachables are always sent
  ICMP mask replies are never sent
  IP fast switching is disabled
  IP fast switching on the same interface is disabled
  IP Flow switching is disabled
  IP Fast switching turbo vector
  IP multicast fast switching is disabled
  IP multicast distributed fast switching is disabled
  Router Discovery is disabled
  IP output packet accounting is disabled
  IP access violation accounting is disabled
  TCP/IP header compression is disabled
  RTP/IP header compression is disabled
  Probe proxy name replies are disabled
  Policy routing is disabled
  Network address translation is disabled
  BGP Policy Mapping is disabled
  Input features: MCI Check
  WCCP Redirect outbound is disabled
  WCCP Redirect inbound is disabled
  WCCP Redirect exclude is disabled
```
#### Тест с Laptop (VLAN40):
 * ping до внутренних 192.168.x.x, пример 192.168.12.10 - запрещён
 * ping до 203.0.113.10 - работает
<img width="436" height="526" alt="image" src="https://github.com/user-attachments/assets/5339d507-5bf4-4dae-a629-b29efb16aa97" />

## 10.6 SSH: доступ только с ADMIN VLAN
```
Switch4#show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
Switch4#show running-config | section line vty
line vty 0 4
 access-class VTY_ADMIN_ONLY in
 login local
 transport input ssh
line vty 5 15
 login
Switch4#show access-lists VTY_ADMIN_ONLY
Standard IP access list VTY_ADMIN_ONLY
    permit 192.168.10.0 0.0.0.255
    permit 192.168.20.0 0.0.0.255
    permit 192.168.30.0 0.0.0.255
    deny any
```
#### Тест:
#### С ПК ADMIN доступ есть, с ПК USER доступ закрыт.

<img width="367" height="213" alt="image" src="https://github.com/user-attachments/assets/97f67ae1-8623-4bdf-9402-37b3dac90ce0" />
<img width="357" height="159" alt="image" src="https://github.com/user-attachments/assets/85e9f87b-1c2e-4951-9953-06b9775eb223" />

## Вывод

В ходе работы я спроектировал адресное пространство и топологию многофилиальной сети. Я реализовал сегментацию на L2 с помощью VLAN, собрал отказоустойчивое ядро на EtherChannel и обеспечил защиту от петель через RSTP. Я настроил межвлановую маршрутизацию по схеме "Роутер на палочке" и организовал динамическую маршрутизацию между площадками с помощью OSPFv2 Area 0. Централизовал выдачу IP-адресов через DHCP на HQ и настроил DHCP-relay для удаленных VLAN. Для выхода в интернет настроил NAT overload, а для публикации WEB сервера применил статический NAT (port-forward). Для повышения безопасности внедрил ACL для гостевого сегмента и WAN, а также включил SSH и ограничил доступ к управлению через VTY ACL. После настройки проверил работоспособность L2/L3, OSPF, DHCP, NAT, ACL и SSH командами диагностики и тестами связности.
























