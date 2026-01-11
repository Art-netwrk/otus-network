# Лабораторная работа. Развертывание коммутируемой сети с резервными каналами (STP)
<img width="822" height="265" alt="image" src="https://github.com/user-attachments/assets/9dd2bc91-7215-4a01-b39a-036bc4413a9b" />

## Таблица адресации

| Устройство | Интерфейс | IP-адрес | Маска |
|-----------|-----------|---------|-------|
| S1 | VLAN 1 | 192.168.1.1 | 255.255.255.0 |
| S2 | VLAN 1 | 192.168.1.2 | 255.255.255.0 |
| S3 | VLAN 1 | 192.168.1.3 | 255.255.255.0 |

## Цель работы
#### Часть 1. Создание сети и настройка основных параметров устройства
#### Часть 2. Выбор корневого моста
#### Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
#### Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

## Часть 1:	Создание сети и настройка основных параметров устройства
В части 1 вам предстоит настроить топологию сети и основные параметры маршрутизаторов.

### Шаг 1:	Создайте сеть согласно топологии.
Подключите устройства, как показано в топологии, и подсоедините необходимые кабели.

### Шаг 2:	Выполните инициализацию и перезагрузку коммутаторов.

### Шаг 3:	Настройте базовые параметры каждого коммутатора.

a.	Отключите поиск DNS.

b.	Присвойте имена устройствам в соответствии с топологией.

c.	Назначьте class в качестве зашифрованного пароля доступа к привилегированному режиму.

d.	Назначьте cisco в качестве паролей консоли и VTY и активируйте вход для консоли и VTY каналов.

e.	Настройте logging synchronous для консольного канала.


f.	Настройте баннерное сообщение дня (MOTD) для предупреждения пользователей о запрете несанкционированного доступа.

g.	Задайте IP-адрес, указанный в таблице адресации для VLAN 1 на всех коммутаторах.

h.	Скопируйте текущую конфигурацию в файл загрузочной конфигурации.

```
Настройка S1 и S3 на примере S2:

Switch>enable
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#no ip domain-lookup
Switch(config)#hostname S2
S2(config)#enable secret class
S2(config)#line console 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#logging synchronous
S2(config-line)#exit
S2(config)#line vty 0 4
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#service password-encryption
S2(config)#banner motd "ADMIN ONLY"
S2(config)#interface vlan 1
S2(config-if)#ip address 192.168.1.2 255.255.255.0
S2(config-if)#no shutdown
S2(config-if)#
%LINK-5-CHANGED: Interface Vlan1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
S2(config-if)#exit
S2(config)#end
S2#
%SYS-5-CONFIG_I: Configured from console by console
S2#write memory
Building configuration...
[OK]
```
### Шаг 4:	Проверьте связь.
Проверьте способность компьютеров обмениваться эхо-запросами.

Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S2?	Да
```
S1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```
Успешно ли выполняется эхо-запрос от коммутатора S1 на коммутатор S3?	Да
```
S1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/3 ms
```
Успешно ли выполняется эхо-запрос от коммутатора S2 на коммутатор S3?	Да
```
S2#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/1/6 ms
```

Выполняйте отладку до тех пор, пока ответы на все вопросы не будут положительными.

## Часть 2: Определение корневого моста
### Шаг 1:	Отключите все порты на коммутаторах.
```
S1#configure terminal
S1(config)#interface range fa0/1-4
S1(config-if-range)#shutdown
S1(config-if-range)#
%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down
%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down
%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to administratively down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to down
S1(config-if-range)#end
```
<img width="424" height="225" alt="image" src="https://github.com/user-attachments/assets/c3f1c842-b8fb-43fe-9a63-211ed1fbfab4" />

### Шаг 2:	Настройте подключенные порты в качестве транковых.
```
S1(config)#interface range fa0/1-4
S1(config-if-range)#switchport mode trunk 
S1(config-if-range)#end
```
### Шаг 3:	Включите порты F0/2 и F0/4 на всех коммутаторах.
```
S1(config)#interface fa0/2
S1(config-if)#no shutdown 
%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to down
S1(config-if)#end
S1(config)#interface fa0/4
S1(config-if)#no shutdown
%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to down
S1(config-if)#end
```
### Шаг 4:	Отобразите данные протокола spanning-tree.
```
S1#show spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E69.2354
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0005.5E69.2354
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Desg FWD 19        128.4    P2p
Fa0/2            Desg FWD 19        128.2    P2p
```
```
S2#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E69.2354
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000D.BDAA.42C8
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Desg FWD 19        128.4    P2p
Fa0/2            Root FWD 19        128.2    P2p
```
```
S3#show spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E69.2354
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0060.2F5B.461C
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```

С учетом выходных данных, поступающих с коммутаторов, ответьте на следующие вопросы.
Какой коммутатор является корневым мостом? 
```
В нашем случае корневым является коммутатор S1
```
Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?
```
Корневым коммутатором является устройство с наименьшим идентификатором моста (Bridge ID).
Если приоритеты равны, выбор осуществляется по наименьшему MAC-адресу.
```
Какие порты на коммутаторе являются корневыми портами? 
```
На коммутаторе S2 корневым является порт Fa0/2
На коммутаторе S3 корневым является порт Fa0/4
```
Какие порты на коммутаторе являются назначенными портами?
```
На коммутаторе S1 назначенными портами являются порт Fa0/4 и Fa0/2
На коммутаторе S2 назначенным портом является порт Fa0/4
```
Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?
```
На коммутаторе S3 порт Fa0/2 альтернативный и заблокирован
```
Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?
```
Порт S3 Fa0/2 был выбран STP как невыделенный и переведён в состояние заблокированного, потому что между S2 и S3 два равнозначных пути к корневому коммутатору S1 (стоимость пути у обоих 19). При равной стоимости STP выбирает назначенный порт по меньшему Bridge ID.
```
## Часть 3:	Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
### Шаг 1:	Определите коммутатор с заблокированным портом
```
S3#show spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E69.2354
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0060.2F5B.461C
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```
### Шаг 2:	Измените стоимость порта.
```
S3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S3(config)#interface fa0/4
S3(config-if)#spanning-tree vlan 1 cost 18
S3(config-if)#end
```
### Шаг 3:	Просмотрите изменения протокола spanning-tree.
```
S3#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E69.2354
             Cost        18
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0060.2F5B.461C
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg LRN 19        128.2    P2p
Fa0/4            Root FWD 18        128.4    P2p
```
```
S2#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E69.2354
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000D.BDAA.42C8
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Altn BLK 19        128.4    P2p
Fa0/2            Root FWD 19        128.2    P2p
```
#### Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?
```
STP выбирает путь с наименьшей суммарной стоимостью до корневого коммутатора.
После уменьшения cost дерево перестраивается, и блокировка может перейти на другой линк.
```
### Шаг 4:	Удалите изменения стоимости порта.
```
S3(config)#interface f0/4
S3(config-if)#no spanning-tree vlan 1 cost 18
S3(config-if)#exit
```
```
S3#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E69.2354
             Cost        19
             Port        4(FastEthernet0/4)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0060.2F5B.461C
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Root FWD 19        128.4    P2p
```
## Часть 4:	Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов
### a.	Включите порты F0/1 и F0/3 на всех коммутаторах.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface fa0/1
S1(config-if)#no shutdown 
%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to down
S1(config-if)#exit
S1(config)#interface fa0/3
S1(config-if)#no shutdown
%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to down
S1(config-if)#end
```
### b.	Подождите 30 секунд, чтобы протокол STP завершил процесс перевода порта, после чего выполните команду show spanning-tree на коммутаторах некорневого моста. Обратите внимание, что порт корневого моста переместился на порт с меньшим номером, связанный с коммутатором корневого моста, и заблокировал предыдущий порт корневого моста.
```
S2#show spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E69.2354
             Cost        19
             Port        1(FastEthernet0/1)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000D.BDAA.42C8
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Desg FWD 19        128.4    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/1            Root FWD 19        128.1    P2p
```
```
S3#show spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E69.2354
             Cost        19
             Port        3(FastEthernet0/3)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0060.2F5B.461C
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Altn BLK 19        128.1    P2p
Fa0/3            Root FWD 19        128.3    P2p
Fa0/2            Altn BLK 19        128.2    P2p
Fa0/4            Altn BLK 19        128.4    P2p
```
Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста? 
```
Root Port на S2 это Fa0/1
Root Port на S3 это Fa0/3
```
Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?
```
Root Port — это порт на некорневом коммутаторе, который даёт лучший путь к Root Bridge.
Либо суммарная стоимость пути до корневого коммутатора (одинакова)
Меньший BID соседа (если одинаков)
Меньший Port ID

Выбрал по меньшеиу номеру порта
```

### Вопросы для повторения
#### 1.	Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?
```
Стоимость пути (Path Cost)
```
#### 2.	Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?
```
Bridge ID
```
#### 3.	Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?
```
По приоритету и номеру порта
```

