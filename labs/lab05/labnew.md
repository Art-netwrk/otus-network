# Настройка протоколов CDP, LLDP и NTP

## Задачи

### Часть 1. Создание сети и настройка основных параметров устройства
### Часть 2. Обнаружение сетевых ресурсов с помощью протокола CDP
### Часть 3. Обнаружение сетевых ресурсов с помощью протокола LLDP
### Часть 4. Настройка и проверка NTP

---


## Часть 1. Создание сети и настройка основных параметров устройства
### Шаг 1. Создайте сеть согласно топологии.

Топология

<img width="644" height="123" alt="image" src="https://github.com/user-attachments/assets/7e3f5742-b96e-4a2d-a908-c64068a9de0a" />


Таблица адресации сетевой топологии.

| Устройство	 | Интерфейс	 |  IP-адрес	  | Маска подсети	 | Шлюз по умолчанию |
|:-----------:|:----------:|:-----------:|:--------------:|:-----------------:|
|     R1	     | Loopback1	 | 172.16.1.1	 | 255.255.255.0	 |         —         |
|     R1	     |  G0/0/1	   | 10.22.0.1	  | 255.255.255.0	 |         —         |
|     S1	     |    SVI     |  VLAN 1 	   |   10.22.0.2	   |  255.255.255.0	   | 10.22.0.1 |
|     S2	     |    SVI     |   VLAN 1	   |   10.22.0.3	   |  255.255.255.0	   | 10.22.0.1 |



### Шаг 2. Настройте базовые параметры для маршрутизатора.

```
a.	Назначьте маршрутизатору имя устройства.
b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.
c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.
d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.
e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.
f.	Зашифруйте открытые пароли.
g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.
h.	Настройка интерфейсов, перечисленных в таблице выше
i.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.
```

``R1``

```
en
conf t
no ip domain-lookup
hostname R1
banner motd #####R1 Router ############
line  0
logging synchronous
password cisco
login
exi
enable secret class
line vty 0 15
password cisco
login
exi
service password-encryption
interface G 0/0/1
ip add 10.22.0.1 255.255.255.0
no sh
ex
interface  Loopback1
ip add 172.16.1.1 255.255.255.0
no sh
ex

copy running-config startup-config
```

### Шаг 3. Настройте базовые параметры каждого коммутатора.

```
a.	Присвойте коммутатору имя устройства.
b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.
c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.
d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.
e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.
f.	Зашифруйте открытые пароли.
g.	Создайте баннер, который предупреждает всех, кто обращается к устройству, видит баннерное сообщение «Только авторизованные пользователи!».  
h.	Отключите неиспользуемые интерфейсы
i.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.
```

``S1``

```
en
conf t
no ip domain-lookup
hostname S1
banner motd "######## S1 Switch ########"
line  0
logging synchronous
password cisco
login
exit
enable secret class
line vty 0 15
password cisco
login
exi
service password-encryption
ex
interface range f0/2-4, f0/6-24, g0/1-2
sh
ex

copy running-config startup-config
```

``S2``

```
en
conf t
no ip domain-lookup
hostname S2
banner motd "######## S2 Switch ########"
line  0
logging synchronous
password cisco
login
exi
enable secret class
line vty 0 15
password cisco
login
exi
service password-encryption
ex
interface range f0/2-24, g0/1-2
sh
ex

copy running-config startup-config
```

## Часть 2. Обнаружение сетевых ресурсов с помощью протокола CDP

a. На R1 используйте соответствующую команду show cdp, чтобы определить, сколько интерфейсов включено CDP, сколько из
них включено и сколько отключено.

``R1``

```
R1>show cdp interface
Vlan1 is administratively down, line protocol is down
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
GigabitEthernet0/0/0 is administratively down, line protocol is down
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
GigabitEthernet0/0/1 is up, line protocol is up
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
GigabitEthernet0/0/2 is administratively down, line protocol is down
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
```

`Вопрос: Сколько интерфейсов участвует в объявлениях CDP? Какие из них активны?`  
Используется один интерфейс GigabitEthernet0/0/1

b. На R1 используйте соответствующую команду show cdp, чтобы определить версию IOS, используемую на S1.

```
R1>show cdp entry  S1

Device ID: S1
Entry address(es): 
Platform: cisco 2960, Capabilities: Switch
Interface: GigabitEthernet0/0/1, Port ID (outgoing port): FastEthernet0/5
Holdtime: 153

Version :
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen

advertisement version: 2
Duplex: full
```

`Какая версия IOS используется на S1?`
15.0(2)SE4

c. На S1 используйте соответствующую команду show cdp, чтобы определить, сколько пакетов CDP было выданных.

``S1``

```
S1>show cdp traffic
            ^
% Invalid input detected at '^' marker.
S1> show cdp ?
  entry      Information for specific neighbor entry
  interface  CDP interface status and configuration
  neighbors  CDP neighbor entries
  <cr>
```

Нет такой команды в CPT

d. Настройте SVI для VLAN 1 на S1 и S2, используя IP-адреса, указанные в таблице адресации выше. Настройте шлюз по
умолчанию для каждого коммутатора на основе таблицы адресов.

``S1``

```
en
conf t
int vlan 1
description SVI VLAN 1
ip add 10.22.0.2 255.255.255.0
no sh
ex
ip default-gateway 10.22.0.1
```

``S2``

```
en
conf t
int vlan 1
description SVI VLAN 1
ip add 10.22.0.3 255.255.255.0
no sh
ex
ip default-gateway 10.22.0.1
```

e. На R1 выполните команду `show cdp entry S1` .

``R1``

```
R1>show cdp entry S1

Device ID: S1
Entry address(es): 
  IP address : 10.22.0.2
Platform: cisco 2960, Capabilities: Switch
Interface: GigabitEthernet0/0/1, Port ID (outgoing port): FastEthernet0/5
Holdtime: 177

Version :
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen

advertisement version: 2
Duplex: full
```

`Вопрос: Какие дополнительные сведения доступны теперь?`
Дополнительно показан IP адрес ( IP address : 10.22.0.2)

f. Отключить CDP глобально на всех устройствах.

``R1/S1/S2``

```
R1>en
Password: 
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#no cdp run
R1(config)#ex
R1#
%SYS-5-CONFIG_I: Configured from  by 

R1#show cdp
% CDP is not enabled
```

```
S1(config)#no cdp run
S1(config)#ex
S1#
%SYS-5-CONFIG_I: Configured from  by 

S1#show cdp 
% CDP is not enabled
```

```
S2(config)#no cdp run
S2(config)#ex
S2#
%SYS-5-CONFIG_I: Configured from  by 

S2#show cdp 
% CDP is not enabled
```

## Часть 3. Обнаружение сетевых ресурсов с помощью протокола LLDP

a. Введите соответствующую команду lldp, чтобы включить LLDP на всех устройствах в топологии.

``R1/S1/S2``

```
R1#show lldp
% LLDP is not enabled
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#lldp run
R1(config)#ex
R1#
%SYS-5-CONFIG_I: Configured from  by 

R1#show lldp

Global LLDP Information:
    Status: ACTIVE
    LLDP advertisements are sent every 30 seconds
    LLDP hold time advertised is 120 seconds
    LLDP interface reinitialisation delay is 2 seconds
R1#
```

```
S1#show lldp
% LLDP is not enabled
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#lldp run
S1(config)#ex
S1#
%SYS-5-CONFIG_I: Configured from  by 

S1#show lldp

Global LLDP Information:
    Status: ACTIVE
    LLDP advertisements are sent every 30 seconds
    LLDP hold time advertised is 120 seconds
    LLDP interface reinitialisation delay is 2 seconds
```

```
S2#show lldp
% LLDP is not enabled
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#lldp run
S2(config)#ex
S2#
%SYS-5-CONFIG_I: Configured from  by 

S2#show lldp

Global LLDP Information:
    Status: ACTIVE
    LLDP advertisements are sent every 30 seconds
    LLDP hold time advertised is 120 seconds
    LLDP interface reinitialisation delay is 2 seconds
```

b. На S1 выполните соответствующую команду lldp, чтобы предоставить подробную информацию о S2.

``S1``

```
S1#show lldp entry S2
             ^
% Invalid input detected at '^' marker.
	
S1#show lldp ?
  neighbors  LLDP neighbor entries
  <cr>
```

Команда не работает в CPT

`Что такое chassis ID для коммутатора S2?`
MAC-адрес устройства которое за портом Fa0/1

c. Соединитесь через консоль на всех устройствах и используйте команды LLDP, необходимые для отображения топологии
физической сети только из выходных данных команды show.

``R1/S1/S2``

```
R1#show lldp neighbors
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID           Local Intf     Hold-time  Capability      Port ID
S1                  Gig0/0/1       120        B               Fa0/5

Total entries displayed: 1
```

```
S1#show lldp neighbors
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID           Local Intf     Hold-time  Capability      Port ID
S2                  Fa0/1          120        B               Fa0/1
R1                  Fa0/5          120        R               Gig0/0/1

Total entries displayed: 2
```

```
S2#show lldp neighbors
Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
Device ID           Local Intf     Hold-time  Capability      Port ID
S1                  Fa0/1          120        B               Fa0/1

Total entries displayed: 1
```

## Часть 4. Настройка и проверка NTP

### Шаг 1. Выведите на экран текущее время.

``R1``

```
R1#show clock detail 
*0:36:4.624 UTC Mon Mar 1 1993
Time source is hardware calendar
```

### Шаг 2. Установите время.

С помощью команды clock set установите время на маршрутизаторе R1. Введенное время должно быть в формате UTC.

``R1``

```
R1#clock set 20:20:00 Jan 28 2026
R1#show clock detail 
20:20:2.992 UTC Wed Jan 28 2026
Time source is user configuration
``` 

### Шаг 3. Настройте главный сервер NTP.

Настройте R1 в качестве хозяина NTP с уровнем слоя 4.

``R1``

```
R1(config)#ntp server 172.16.1.1
R1(config)#ntp master 4
R1(config)#ntp update-calendar
```

### Шаг 4. Настройте клиент NTP.

a. Выполните соответствующую команду на S1 и S2, чтобы просмотреть настроенное время. Запишите текущее время, в
следующей таблице.

``S1/S2``

```
show clock detail 
```

|     Дата	      |   Время	    | Часовой пояс	 |         Источник времени         |
|:--------------:|:-----------:|:-------------:|:--------------------------------:| 
| Mon Mar 1 1993 | 0:51:49.742 |      UTC      | Time source is hardware calendar |
| Mon Mar 1 1993 | 0:53:12.588 |      UTC      | Time source is hardware calendar |

b. Настройте S1 и S2 в качестве клиентов NTP. Используйте соответствующие команды NTP для получения времени от
интерфейса G0/0/1 R1, а также для периодического обновления календаря или аппаратных часов коммутатора.

``S1/S2``

```
S1(config)#ntp server 10.22.0.1
S1(config)#ntp update-calendar
               ^
% Invalid input detected at '^' marker.
```

```
S2(config)#ntp server 10.22.0.1
```

### Шаг 5. Проверьте настройку NTP.

a. Используйте соответствующую команду show , чтобы убедиться, что S1 и S2 синхронизированы с R1. Примечание.
Синхронизация метки времени на маршрутизаторе R2 с меткой времени на маршрутизаторе R1 может занять несколько минут.

``S1/S2``
```
S1#show ntp status
Clock is synchronized, stratum 5, reference is 10.22.0.1
nominal freq is 250.0000 Hz, actual freq is 249.9990 Hz, precision is 2**24
reference time is FFFFFFFFE9DBD136.000002AA (20:21:10.682 UTC Wed Jan 28 2026)
clock offset is 3.00 msec, root delay is 0.00  msec
root dispersion is 10.08 msec, peer dispersion is 0.12 msec.
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is - 0.000001193 s/s system poll interval is 4, last update was 1 sec ago.
```

```
S2#show ntp status
Clock is synchronized, stratum 5, reference is 10.22.0.1
nominal freq is 250.0000 Hz, actual freq is 249.9990 Hz, precision is 2**24
reference time is FFFFFFFFE9DBD19C.0000000F (20:22:52.015 UTC Wed Jan 28 2026)
clock offset is 3.00 msec, root delay is 0.00  msec
root dispersion is 10.08 msec, peer dispersion is 0.12 msec.
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is - 0.000001193 s/s system poll interval is 4, last update was 10 sec ago.
```

b. Выполните соответствующую команду на S1 и S2, чтобы просмотреть настроенное время и сравнить ранее записанное время.

``S1/S2``

```
S1#show clock detail 
*20:22:32.837 UTC Wed Jan 28 2026
Time source is hardware calendar
```

```
S2#show clock detail 
20:23:26.433 UTC Wed Jan 28 2026
Time source is NTP
```

Вопрос для повторения  
`Для каких интерфейсов в пределах сети не следует использовать протоколы обнаружения сетевых ресурсов? Поясните ответ.`

```
1) Внешние/WAN-интерфейсы (к провайдеру/Интернету, intersite через чужую инфраструктуру)
   Пояснение: CDP/LLDP раскрывают модель устройства, имя (hostname), порты, адреса управления, возможности (capabilities).
   Это упрощает разведку и подбор атак.

2) Интерфейсы на границе доверия внутри организации (Guest VLAN, BYOD, общедоступные розетки, переговорные, холлы)
   Пояснение: любой подключившийся пользователь может увидеть данные о сетевом оборудовании и понять, куда он подключён,
   что облегчает обход сегментации и атаки на инфраструктуру.

3) Линки к сторонним организациям/подрядчикам/партнёрам (B2B, аутсорс, совместные площадки)
   Пояснение: вы отдаёте сведения о своей внутренней сети наружу. Если нужно взаимодействие — лучше давать минимум информации.

4) DMZ-пограничные интерфейсы (особенно “наружная сторона” DMZ)
   Пояснение: устройства/сервисы в DMZ потенциально подвергаются атакам, и “подсказки” про внутреннюю инфраструктуру
   повышают риск компрометации.

5) Беспроводные сегменты с низким уровнем доверия (открытый Wi-Fi, гостевой Wi-Fi)
   Пояснение: в радио-среде проще перехватывать кадры, а информация LLDP/CDP полезна для атакующего как карта сети.
```
