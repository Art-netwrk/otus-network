# Лабораторная работа - Реализация DHCPv4 
<img width="723" height="125" alt="image" src="https://github.com/user-attachments/assets/50567992-a3ee-464c-868d-25052ae811fd" />
<img width="690" height="589" alt="image" src="https://github.com/user-attachments/assets/35acdfc4-04d4-4ed4-9af9-9e7ce51cd7a2" />

## 3.	Задачи

### Часть 1. Создание сети и настройка основных параметров устройства

### Часть 2. Настройка и проверка двух серверов DHCPv4 на R1

### Часть 3. Настройка и проверка DHCP-ретрансляции на R2

## Инструкции

### Часть 1.	Создание сети и настройка основных параметров устройства
В первой части лабораторной работы вам предстоит создать топологию сети и настроить базовые параметры для узлов ПК и коммутаторов.

#### Шаг 1.	Создание схемы адресации
Подсеть сети 192.168.1.0/24 в соответствии со следующими требованиями:

a.	Одна подсеть «Подсеть A», поддерживающая 58 хостов (клиентская VLAN на R1).
Подсеть A
Запишите первый IP-адрес в таблице адресации для R1 G0/0/1.100 . 

b.	Одна подсеть «Подсеть B», поддерживающая 28 хостов (управляющая VLAN на R1). 
Подсеть B:
Запишите первый IP-адрес в таблице адресации для R1 G0/0/1.200. Запишите второй IP-адрес в таблице адресов для S1 VLAN 200 и введите соответствующий шлюз по умолчанию.

c.	Одна подсеть «Подсеть C», поддерживающая 12 узлов (клиентская сеть на R2).
Подсеть C:
Запишите первый IP-адрес в таблице адресации для R2 G0/0/1.

#### Шаг 2.	Создайте сеть согласно топологии.
Подключите устройства, как показано в топологии, и подсоедините необходимые кабели.
<img width="633" height="158" alt="image" src="https://github.com/user-attachments/assets/7f932b51-d61d-44db-9582-d5b3561fbd71" />


#### Шаг 3.	Произведите базовую настройку маршрутизаторов.

a.	Назначьте маршрутизатору имя устройства.

b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.

d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.

e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.

f.	Зашифруйте открытые пароли.

g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.

h.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.
```
enable
conf t
 hostname R2
 no ip domain-lookup
 enable secret class

 line con 0
  password cisco
  login
 exit

 line vty 0 15
  password cisco
  login
 exit

 service password-encryption
 banner motd "ADMIN ONLY"
end
wr
```
#### Аналогичная настройка для R2

i.	Установите часы на маршрутизаторе на сегодняшнее время и дату.
Примечание. Вопросительный знак (?) позволяет открыть справку с правильной последовательностью параметров, необходимых для выполнения этой команды.
```
R1#clock set 15:00:00 14 Jan 2026
```
```
R2#clock set 15:00:00 14 Jan 2026
```
Шаг 4.	Настройка маршрутизации между сетями VLAN на маршрутизаторе R1

a.	Активируйте интерфейс G0/0/1 на маршрутизаторе.

b.	Настройте подинтерфейсы для каждой VLAN в соответствии с требованиями таблицы IP-адресации. Все субинтерфейсы используют инкапсуляцию 802.1Q и назначаются первый полезный адрес из вычисленного пула IP-адресов. Убедитесь, что подинтерфейсу для native VLAN не назначен IP-адрес. Включите описание для каждого подинтерфейса.
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)# interface g0/0/1
R1(config-if)#  no shutdown
R1(config-if)# exit
R1(config)# interface g0/0/1.100
R1(config-subif)#  description VLAN 100 Clients
R1(config-subif)#  encapsulation dot1Q 100
R1(config-subif)#  ip address 192.168.1.1 255.255.255.192
R1(config-subif)# exit
R1(config)# interface g0/0/1.200
R1(config-subif)#  description VLAN 200 Management
R1(config-subif)#  encapsulation dot1Q 200
R1(config-subif)#  ip address 192.168.1.65 255.255.255.224
R1(config-subif)# exit
R1(config)# interface g0/0/1.1000
R1(config-subif)#  description Native VLAN 1000
R1(config-subif)#  encapsulation dot1Q 1000 native
R1(config-subif)#  no ip address
R1(config-subif)# exit
R1(config)#end
R1#write memory
Building configuration...
[OK]
```
c.	Убедитесь, что вспомогательные интерфейсы работают.
```
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.100192.168.1.1     YES manual up                    up 
GigabitEthernet0/0/1.200192.168.1.65    YES manual up                    up 
GigabitEthernet0/0/1.1000unassigned      YES unset  up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

#### Шаг 5.	Настройте G0/1 на R2, затем G0/0/0 и статическую маршрутизацию для обоих маршрутизаторов

a.	Настройте G0/0/1 на R2 с первым IP-адресом подсети C, рассчитанным ранее.

b.	Настройте интерфейс G0/0/0 для каждого маршрутизатора на основе приведенной выше таблицы IP-адресации.

c.	Настройте маршрут по умолчанию на каждом маршрутизаторе, указываемом на IP-адрес G0/0/0 на другом маршрутизаторе.

d.	Убедитесь, что статическая маршрутизация работает с помощью пинга до адреса G0/0/1 R2 от R1.

e.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

#### Подсеть C
```
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)# interface g0/0/1
R2(config-if)#  description R2 Client LAN (Subnet C)
R2(config-if)#  ip address 192.168.1.97 255.255.255.240
R2(config-if)#  no shutdown
R2(config-if)# exit
```
#### Линк между R1 и R2
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)# interface g0/0/0
R1(config-if)#  ip address 10.0.0.1 255.255.255.252
R1(config-if)#  no shutdown
R1(config-if)# exit
```
```
R2#conf t
R2(config)# interface g0/0/0
R2(config-if)#  ip address 10.0.0.2 255.255.255.252
R2(config-if)#  no shutdown
R2(config-if)# exit
```
#### Маршрут по умолчанию
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2
R1(config)#end
R1#write memory
Building configuration...
[OK]
```
```
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1
R2(config)#end
R2#write memory
Building configuration...
[OK]
```
#### Проверка
```
R1#ping 192.168.1.97

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```
#### Шаг 6.	Настройте базовые параметры каждого коммутатора.

a.	Присвойте коммутатору имя устройства.

b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.

d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.

e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.

f.	Зашифруйте открытые пароли.

g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.

h.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.
#### Аналогично на S1 и S2
```
Switch>en
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)# hostname S1
S1(config)# no ip domain-lookup
S1(config)# enable secret class
S1(config)# line con 0
S1(config-line)#  password cisco
S1(config-line)#  login
S1(config-line)# exit
S1(config)# line vty 0 15
S1(config-line)#  password cisco
S1(config-line)#  login
S1(config-line)# exit
S1(config)# service password-encryption
S1(config)# banner motd "ADMIN ONLY"
S1(config)#end
S1#wr
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
```
i.	Установите часы на маршрутизаторе на сегодняшнее время и дату.
Примечание. Вопросительный знак (?) позволяет открыть справку с правильной последовательностью параметров, необходимых для выполнения этой команды.
```
clock set 16:00:00 14 Jan 2026
```
j.	Скопируйте текущую конфигурацию в файл загрузочной конфигурации.

#### Шаг 7.	Создайте сети VLAN на коммутаторе S1.
Примечание. S2 настроен только с базовыми настройками. 

a.	Создайте необходимые VLAN на коммутаторе 1 и присвойте им имена из приведенной выше таблицы.

b.	Настройте и активируйте интерфейс управления на S1 (VLAN 200), используя второй IP-адрес из подсети, рассчитанный ранее. Кроме того установите шлюз по умолчанию на S1.

c.	Настройте и активируйте интерфейс управления на S2 (VLAN 1), используя второй IP-адрес из подсети, рассчитанный ранее. Кроме того, установите шлюз по умолчанию на S2

d.	Назначьте все неиспользуемые порты S1 VLAN Parking_Lot, настройте их для статического режима доступа и административно деактивируйте их. На S2 административно деактивируйте все неиспользуемые порты.
Примечание. Команда interface range полезна для выполнения этой задачи с минимальным количеством команд.

#### S1: создаём VLAN 100/200/999/1000 и SVI для VLAN 200
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)# vlan 100
S1(config-vlan)#  name Clients
S1(config-vlan)# vlan 200
S1(config-vlan)#  name Management
S1(config-vlan)# vlan 999
S1(config-vlan)#  name Parking_Lot
S1(config-vlan)# vlan 1000
S1(config-vlan)#  name Native
S1(config-vlan)# exit
S1(config)# interface vlan 200
S1(config-if)#  ip address 192.168.1.66 255.255.255.224
S1(config-if)#  no shutdown
S1(config-if)# exit
S1(config)# ip default-gateway 192.168.1.65
S1(config)#end
S1#write memory
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)# interface vlan 1
S2(config-if)#  ip address 192.168.1.98 255.255.255.240
S2(config-if)#  no shutdown
S2(config-if)# exit
S2(config)# ip default-gateway 192.168.1.97
S2(config)#end
S2#write memory
Building configuration...
[OK]
```
```
conf t
interface range f0/1-4, f0/6-17, f0/19-24, g0/1-2
 shutdown
end
write memory
```
```
#### Шаг 8.	Назначьте сети VLAN соответствующим интерфейсам коммутатора.

a.	Назначьте используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и настройте их для режима статического доступа.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)# interface f0/6
S1(config-if)#  switchport mode access
S1(config-if)#  switchport access vlan 100
S1(config-if)#  spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION
%Portfast has been configured on FastEthernet0/6 but will only
have effect when the interface is in a non-trunking mode.
S1(config-if)# exit
S1(config)#end
S1#write memory
%SYS-5-CONFIG_I: Configured from console by console
Building configuration...
[OK]
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)# interface f0/18
S2(config-if)#  switchport mode access
S2(config-if)#  switchport access vlan 1
S2(config-if)#  spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION
%Portfast has been configured on FastEthernet0/18 but will only
have effect when the interface is in a non-trunking mode.
S2(config-if)# exit
S2(config)#end
S2#write memory
Building configuration...
[OK]
```
b.	Убедитесь, что VLAN назначены на правильные интерфейсы.
```
S1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15, Fa0/16, Fa0/17
                                                Fa0/18, Fa0/19, Fa0/20, Fa0/21
                                                Fa0/22, Fa0/23, Fa0/24, Gig0/1
                                                Gig0/2
100  Clients                          active    Fa0/6
200  Management                       active    
999  Parking_Lot                      active    
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active   
```
```
S2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```

Вопрос:
Почему интерфейс F0/5 указан в VLAN 1?
```
Потому что VLAN 1 — VLAN по умолчанию на Cisco-коммутаторах, и все порты автоматически принадлежат VLAN 1, пока их не назначат вручную в другую VLAN или не переведут порт в trunk. Поэтому до настройки trunk порт S1 F0/5 отображается в VLAN 1.
```

#### Шаг 9.	Вручную настройте интерфейс S1 F0/5 в качестве транка 802.1Q.
a.	Измените режим порта коммутатора, чтобы принудительно создать магистральный канал.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)# interface f0/5
S1(config-if)#  switchport mode trunk
S1(config-if)# exit
```
b.	В рамках конфигурации транка  установите для native  VLAN значение 1000.
```
S1(config)# interface f0/5
S1(config-if)#  switchport trunk native vlan 1000
S1(config-if)# exit
```

c.	В качестве другой части конфигурации магистрали укажите, что VLAN 100, 200 и 1000 могут проходить по транку.
```
S1(config)# interface f0/5
S1(config-if)#  switchport trunk allowed vlan 100,200,1000
S1(config-if)# exit
S1(config)#end
S1#write memory
Building configuration...
[OK]
```

d.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

e.	Проверьте состояние транка.
```
S1#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/5       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/5       100,200,1000

Port        Vlans allowed and active in management domain
Fa0/5       100,200,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/5       100,200,1000
```

Вопрос:
Какой IP-адрес был бы у ПК, если бы он был подключен к сети с помощью DHCP?
```
PC-A (VLAN 100, 192.168.1.0/26) получил бы адрес из пула подсети A, то есть 192.168.1.6-192.168.1.62 (первые 5 адресов исключены).

PC-B (подсеть C, 192.168.1.96/28) получил бы адрес из пула подсети C, то есть 192.168.1.102-192.168.1.110 (исключены 192.168.1.97-192.168.1.101).
```

### Часть 2.	Настройка и проверка двух серверов DHCPv4 на R1
В части 2 необходимо настроить и проверить сервер DHCPv4 на R1. Сервер DHCPv4 будет обслуживать две подсети, подсеть A и подсеть C.

#### Шаг 1.	Настройте R1 с пулами DHCPv4 для двух поддерживаемых подсетей. Ниже приведен только пул DHCP для подсети A

a.	Исключите первые пять используемых адресов из каждого пула адресов.
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.5
R1(config)# ip dhcp excluded-address 192.168.1.97 192.168.1.101
```
b.	Создайте пул DHCP (используйте уникальное имя для каждого пула).
```
R1(config)# ip dhcp pool R1_Client_LAN
R1(dhcp-config)# network 192.168.1.0 255.255.255.192
R1(dhcp-config)# default-router 192.168.1.1
R1(dhcp-config)# domain-name CCNA-lab.com
R1(dhcp-config)# exit
```
```
R1(dhcp-config)#lease ?
% Unrecognized command
```
c.	Укажите сеть, поддерживающую этот DHCP-сервер.

d.	В качестве имени домена укажите CCNA-lab.com.

e.	Настройте соответствующий шлюз по умолчанию для каждого пула DHCP.

f.	Настройте время аренды на 2 дня 12 часов и 30 минут.

g.	Затем настройте второй пул DHCPv4, используя имя пула R2_Client_LAN и вычислите сеть, маршрутизатор по умолчанию, и используйте то же имя домена и время аренды, что и предыдущий пул DHCP.
```
R1(config)#ip dhcp pool R2_Client_LAN
R1(dhcp-config)# network 192.168.1.96 255.255.255.240
R1(dhcp-config)# default-router 192.168.1.97
R1(dhcp-config)# domain-name CCNA-lab.com
R1(dhcp-config)# exit
```
#### Шаг 2.	Сохраните конфигурацию.
Сохраните текущую конфигурацию в файл загрузочной конфигурации.
```
R1(config)#end
R1#write memory
Building configuration...
[OK]
```

#### Шаг 3.	Проверка конфигурации сервера DHCPv4

a.	Чтобы просмотреть сведения о пуле, выполните команду show ip dhcp pool.
```
R1#show ip dhcp pool

Pool R1_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 0
 Excluded addresses             : 2
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.1          192.168.1.1      - 192.168.1.62      0    / 2     / 62

Pool R2_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 14
 Leased addresses               : 0
 Excluded addresses             : 2
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.97         192.168.1.97     - 192.168.1.110     0    / 2     / 14
```
b.	Выполните команду show ip dhcp bindings для проверки установленных назначений адресов DHCP.
```
R1#show ip dhcp binding
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
```

c.	Выполните команду show ip dhcp server statistics для проверки сообщений DHCP.
```
R1#show ip dhcp server statistics
                ^
% Invalid input detected at '^' marker.

Статистика DHCP-сервера недоступна в используемой версии IOS.
```

#### Шаг 4.	Попытка получить IP-адрес от DHCP на PC-A

a.	Из командной строки компьютера PC-A выполните команду ipconfig /all.

b.	После завершения процесса обновления выполните команду ipconfig для просмотра новой информации об IP-адресе.

c.	Проверьте подключение с помощью пинга IP-адреса интерфейса R0 G0/0/1.
```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 00D0.FFD0.617B
   Link-local IPv6 Address.........: FE80::2D0:FFFF:FED0:617B
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-36-CE-2D-CD-00-D0-FF-D0-61-7B
   DNS Servers.....................: ::
                                     0.0.0.0
```
```
C:\>ipconfig /renew

   IP Address......................: 192.168.1.6
   Subnet Mask.....................: 255.255.255.192
   Default Gateway.................: 192.168.1.1
   DNS Server......................: 0.0.0.0
```
```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: CCNA-lab.com
   Link-local IPv6 Address.........: FE80::2D0:FFFF:FED0:617B
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.6
   Subnet Mask.....................: 255.255.255.192
   Default Gateway.................: ::
                                     192.168.1.1
```
```

C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
### Часть 3.	Настройка и проверка DHCP-ретрансляции на R2
В части 3 настраивается R2 для ретрансляции DHCP-запросов из локальной сети на интерфейсе G0/0/1 на DHCP-сервер (R1). 

#### Шаг 1.	Настройка R2 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1

a.	Настройте команду ip helper-address на G0/0/1, указав IP-адрес G0/0/0 R1.

b.	Сохраните конфигурацию.
```
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#interface g0/0/1
R2(config-if)# ip helper-address 10.0.0.1
R2(config-if)#end
R2#write memory
Building configuration...
[OK]
```
#### Шаг 2.	Попытка получить IP-адрес от DHCP на PC-B

a.	Из командной строки компьютера PC-B выполните команду ipconfig /all.

b.	После завершения процесса обновления выполните команду ipconfig для просмотра новой информации об IP-адресе.

c.	Проверьте подключение с помощью пинга IP-адреса интерфейса R1 G0/0/1.

d.	Выполните show ip dhcp binding для R1 для проверки назначений адресов в DHCP.

e.	Выполните команду show ip dhcp server statistics для проверки сообщений DHCP.
```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0006.2A31.2D87
   Link-local IPv6 Address.........: FE80::206:2AFF:FE31:2D87
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-25-30-55-C0-00-06-2A-31-2D-87
   DNS Servers.....................: ::
                                     0.0.0.0
```
```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: CCNA-lab.com
   Link-local IPv6 Address.........: FE80::206:2AFF:FE31:2D87
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.102
   Subnet Mask.....................: 255.255.255.240
   Default Gateway.................: ::
                                     192.168.1.97
```
```
C:\>ping 192.168.1.97

Pinging 192.168.1.97 with 32 bytes of data:

Reply from 192.168.1.97: bytes=32 time<1ms TTL=255
Reply from 192.168.1.97: bytes=32 time<1ms TTL=255
Reply from 192.168.1.97: bytes=32 time<1ms TTL=255
Reply from 192.168.1.97: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.97:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
```
R1#show ip dhcp binding
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      00D0.FFD0.617B           --                     Automatic
192.168.1.102    0006.2A31.2D87           --                     Automatic
```















