# Лабораторная работа. Настройка и проверка расширенных списков контроля доступа.

## Топология

<img width="697" height="286" alt="image" src="https://github.com/user-attachments/assets/05a24086-98a7-4a55-ad94-01e439fd210f" />

<img width="703" height="373" alt="image" src="https://github.com/user-attachments/assets/a5abe31a-96e8-4d6a-b207-32ae8cc3863e" />

<img width="693" height="225" alt="image" src="https://github.com/user-attachments/assets/e5890201-3712-44c5-b07c-1a1f8842c6e3" />

## Инструкции

## Часть 1. Создание сети и настройка основных параметров устройства

### Шаг 1. Создайте сеть согласно топологии.
Подключите устройства, как показано в топологии, и подсоедините необходимые кабели.

### Шаг 2. Произведите базовую настройку маршрутизаторов.

a.	Назначьте маршрутизатору имя устройства.

b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким 
образом, как будто они являются именами узлов.

c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.

d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.

e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.

f.	Зашифруйте открытые пароли.

g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.

h.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

#### R1 и R2 аналогично
```
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R1
R1(config)#no ip domain-lookup
R1(config)#enable secret class
R1(config)#line con 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#logging synchronous
R1(config-line)#exit
R1(config)#line vty 0 15
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
R1(config)#service password-encryption
R1(config)#banner motd "ADMIN ONLY"
R1(config)#end
%SYS-5-CONFIG_I: Configured from console by console
R1#wr
Building configuration...
[OK]
```
### Шаг 3. Настройте базовые параметры каждого коммутатора.

a.	Присвойте коммутатору имя устройства.

b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.

d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.

e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.

f.	Зашифруйте открытые пароли.

g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.

h.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

#### S1 и S2 аналогично
```
S1#enable
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#hostname S1
S1(config)#no ip domain-lookup
S1(config)#enable secret class
S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#logging synchronous
S1(config-line)#exit
S1(config)#line vty 0 15
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption
S1(config)#banner motd "ADMIN ONLY"
S1(config)#end
S1#wr
Building configuration...
[OK]
```
## Часть 2. Настройка сетей VLAN на коммутаторах.

### Шаг 1. Создайте сети VLAN на коммутаторах.

a.	Создайте необходимые VLAN и назовите их на каждом коммутаторе из приведенной выше таблицы.

#### S1 и S2 аналогично
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#vlan 20
S1(config-vlan)#name Management
S1(config-vlan)#vlan 30
S1(config-vlan)#name Operations
S1(config-vlan)#vlan 40
S1(config-vlan)#name Sales
S1(config-vlan)#vlan 999
S1(config-vlan)#name ParkingLot
S1(config-vlan)#vlan 1000
S1(config-vlan)#name Native
S1(config-vlan)#end
```
b.	Настройте интерфейс управления и шлюз по умолчанию на каждом коммутаторе, используя информацию об IP-адресе в таблице адресации. 
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface vlan 20
S1(config-if)#ip address 10.20.0.2 255.255.255.0
S1(config-if)#no shutdown
S1(config-if)#exit
S1(config)#ip default-gateway 10.20.0.1
S1(config)#end
S1#wr
Building configuration...
[OK]
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface vlan 20
S2(config-if)#ip address 10.20.0.3 255.255.255.0
S2(config-if)#no shutdown
S2(config-if)#exit
S2(config)#ip default-gateway 10.20.0.1
S2(config)#end
S2#wr
```
c.	Назначьте все неиспользуемые порты коммутатора VLAN Parking Lot, настройте их для статического режима доступа и административно деактивируйте их.
Примечание. Команда interface range полезна для выполнения этой задачи с помощью необходимого количества команд. 
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface range fa0/2-4, fa0/7-24, gi0/1-2
S1(config-if-range)#switchport mode access
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#shutdown
S1(config-if-range)#end
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface range fa0/2-4, fa0/6-17, fa0/19-24, gi0/1-2
S2(config-if-range)#switchport mode access
S2(config-if-range)#switchport access vlan 999
S2(config-if-range)#shutdown
S2(config-if-range)#end
```
### Шаг 2. Назначьте сети VLAN соответствующим интерфейсам коммутатора.

a.	Назначьте используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и настройте их для режима статического доступа.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface fa0/6
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 30
S1(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION
%Portfast has been configured on FastEthernet0/6 but will only
have effect when the interface is in a non-trunking mode.
S1(config-if)#no shutdown
S1(config-if)#end
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface fa0/18
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 40
S2(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface  when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION
%Portfast has been configured on FastEthernet0/18 but will only
have effect when the interface is in a non-trunking mode.
S2(config-if)#no shutdown
S2(config-if)#end
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface fa0/5
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 20
S2(config-if)#no shutdown
S2(config-if)#end
```
b.	Выполните команду show vlan brief, чтобы убедиться, что сети VLAN назначены правильным интерфейсам.
```
S1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/5
20   Management                       active    
30   Operations                       active    Fa0/6
40   Sales                            active    
999  ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
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
1    default                          active    Fa0/1
20   Management                       active    Fa0/5
30   Operations                       active    
40   Sales                            active    Fa0/18
999  ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active
```

## Часть 3. ·Настройте транки (магистральные каналы).

### Шаг 1. Вручную настройте магистральный интерфейс F0/1.

a.	Измените режим порта коммутатора на интерфейсе F0/1, чтобы принудительно создать магистральную связь. Не забудьте сделать это на обоих коммутаторах.

b.	В рамках конфигурации транка установите для native vlan значение 1000 на обоих коммутаторах. При настройке двух интерфейсов для разных собственных VLAN сообщения об ошибках могут отображаться временно.

c.	В качестве другой части конфигурации транка укажите, что VLAN 20, 30, 40 и 1000 разрешены в транке.

d.	Выполните команду show interfaces trunk для проверки портов магистрали, собственной VLAN и разрешенных VLAN через магистраль.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface fa0/1
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 1000
S1(config-if)#switchport trunk allowed vlan 20,30,40,1000
S1(config-if)#no shutdown
S1(config-if)#end
```
```
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#interface fa0/1
S2(config-if)#switchport mode trunk
S2(config-if)#switchport trunk native vlan 1000
S2(config-if)#switchport trunk allowed vlan 20,30,40,1000
S2(config-if)#no shutdown
S2(config-if)#end
```
```
S1>en
Password: 
S1#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000
Port        Vlans allowed on trunk
Fa0/1       20,30,40,1000
Port        Vlans allowed and active in management domain
Fa0/1       20,30,40,1000
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       20,30,40,1000
```
```
S2#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000
Port        Vlans allowed on trunk
Fa0/1       20,30,40,1000
Port        Vlans allowed and active in management domain
Fa0/1       20,30,40,1000
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       20,30,40,1000
```

### Шаг 2. Вручную настройте магистральный интерфейс F0/5 на коммутаторе S1.

a.	Настройте интерфейс S1 F0/5 с теми же параметрами транка, что и F0/1. Это транк до маршрутизатора.

b.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

c.	Используйте команду show interfaces trunk для проверки настроек транка.
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#interface fa0/5
S1(config-if)#switchport mode trunk
S1(config-if)#switchport trunk native vlan 1000
S1(config-if)#switchport trunk allowed vlan 20,30,40,1000
S1(config-if)#no shutdown
S1(config-if)#end
S1#wr
Building configuration...
[OK]
```


## Часть 4. Настройте маршрутизацию.

### Шаг 1. Настройка маршрутизации между сетями VLAN на R1.

a.	Активируйте интерфейс G0/0/1 на маршрутизаторе.
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#interface g0/0/1
R1(config-if)#no ip address
R1(config-if)#no shutdown
R1(config-if)#end
```
b.	Настройте подинтерфейсы для каждой VLAN, как указано в таблице IP-адресации. Все подинтерфейсы используют инкапсуляцию 802.1Q. Убедитесь, что подинтерфейс для собственной VLAN не имеет назначенного IP-адреса. Включите описание для каждого подинтерфейса.
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#interface g0/0/1.20
R1(config-subif)#description VLAN20_Management
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#ip address 10.20.0.1 255.255.255.0
R1(config-subif)#exit
R1(config)#interface g0/0/1.30
R1(config-subif)#description VLAN30_Operations
R1(config-subif)#encapsulation dot1Q 30
R1(config-subif)#ip address 10.30.0.1 255.255.255.0
R1(config-subif)#exit
R1(config)#interface g0/0/1.40
R1(config-subif)#description VLAN40_Sales
R1(config-subif)#encapsulation dot1Q 40
R1(config-subif)#ip address 10.40.0.1 255.255.255.0
R1(config-subif)#exit
R1(config)#interface g0/0/1.1000
R1(config-subif)#description Native_VLAN1000
R1(config-subif)#encapsulation dot1Q 1000 native
R1(config-subif)#no ip address
R1(config-subif)#exit
R1(config)#end
```
c.	Настройте интерфейс Loopback 1 на R1 с адресацией из приведенной выше таблицы.
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#interface loopback1
R1(config-if)#ip address 172.16.1.1 255.255.255.0
R1(config-if)#end
R1#wr
```
d.	С помощью команды show ip interface brief проверьте конфигурацию подынтерфейса.
```
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.2010.20.0.1       YES manual up                    up 
GigabitEthernet0/0/1.3010.30.0.1       YES manual up                    up 
GigabitEthernet0/0/1.4010.40.0.1       YES manual up                    up 
GigabitEthernet0/0/1.1000unassigned      YES unset  up                    up 
Loopback1              172.16.1.1      YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

### Шаг 2. Настройка интерфейса R2 g0/0/1 с использованием адреса из таблицы и маршрута по умолчанию с адресом следующего перехода 10.20.0.1
```
R2#enable
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#interface g0/0/1
R2(config-if)#ip address 10.20.0.4 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#ip route 0.0.0.0 0.0.0.0 10.20.0.1
R2(config)#end
R2#wr
```
```
R2#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 10.20.0.1 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.20.0.0/24 is directly connected, GigabitEthernet0/0/1
L       10.20.0.4/32 is directly connected, GigabitEthernet0/0/1
S*   0.0.0.0/0 [1/0] via 10.20.0.1
```
## Часть 5. Настройте удаленный доступ

### Шаг 1. Настройте все сетевые устройства для базовой поддержки SSH.

a.	Создайте локального пользователя с именем пользователя SSHadmin и зашифрованным паролем $cisco123!

b.	Используйте ccna-lab.com в качестве доменного имени.

c.	Генерируйте криптоключи с помощью 1024 битного модуля.

d.	Настройте первые пять линий VTY на каждом устройстве, чтобы поддерживать только SSH-соединения и с локальной 
аутентификацией.
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#username SSHadmin secret $cisco123!
R1(config)#ip domain-name ccna-lab.com
R1(config)#end
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#crypto key generate rsa
The name for the keys will be: R1.ccna-lab.com
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.
How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
R1(config)#ip ssh version 2
*Mar 2 3:29:36.171: %SSH-5-ENABLED: SSH 2 has been enabled
R1(config)#line vty 0 4
R1(config-line)#transport input ssh
R1(config-line)#login local
R1(config-line)#end
R1#wr
```
```
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#username SSHadmin secret $cisco123!
S1(config)#ip domain-name ccna-lab.com
S1(config)#end
S1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S1(config)#crypto key generate rsa
The name for the keys will be: S1.ccna-lab.com
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.
How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
S1(config)#ip ssh version 2
S1(config)#
S1(config)#line vty 0 4
S1(config-line)#transport input ssh
S1(config-line)#login local
S1(config-line)#end
S1#wr
%SYS-5-CONFIG_I: Configured from console by console
S1#wr
```
```
S2#enable
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#username SSHadmin secret $cisco123!
S2(config)#ip domain-name ccna-lab.com
S2(config)#end
S2#
%SYS-5-CONFIG_I: Configured from console by console
S2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
S2(config)#crypto key generate rsa
The name for the keys will be: S2.ccna-lab.com
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.
How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
S2(config)#ip ssh version 2
*Mar 2 3:40:2.551: %SSH-5-ENABLED: SSH 2 has been enabled
S2(config)#line vty 0 4
S2(config-line)#transport input ssh
S2(config-line)#login local
S2(config-line)#end
S2#wr
Building configuration...
[OK]
```
```
R2#enable
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#username SSHadmin secret $cisco123!
R2(config)#ip domain-name ccna-lab.com
R2(config)#end
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#crypto key generate rsa
The name for the keys will be: R2.ccna-lab.com
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.
How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
R2(config)#ip ssh version 2
*Mar 2 3:42:57.800: %SSH-5-ENABLED: SSH 1.99 has been enabled
R2(config)#line vty 0 4
R2(config-line)#transport input ssh
R2(config-line)#login local
R2(config-line)#end
R2#wr
Building configuration...
[OK]
```
### Шаг 2. Включите защищенные веб-службы с проверкой подлинности на R1.

a.	Включите сервер HTTPS на R1.
R1(config)# ip http secure-server 

b.	Настройте R1 для проверки подлинности пользователей, пытающихся подключиться к веб-серверу.
R1(config)# ip http authentication local
```
На используемой модели маршрутизатора/образе IOS в Packet Tracer отсутствует поддержка встроенного HTTP/HTTPS сервера. Поэтому включение HTTPS и локальной аутентификации веб-сервера выполнить невозможно. Удалённый доступ реализован через SSH (Шаг 1), а политики по WEB (HTTP/HTTPS) будут проверяться и обеспечиваться ACL по портам 80/443.
```
## Часть 6. Проверка подключения

### Шаг 1. Настройте узлы ПК.
Адреса ПК можно посмотреть в таблице адресации.

<img width="458" height="301" alt="image" src="https://github.com/user-attachments/assets/335e1730-51d6-42ae-9c14-5d60804406e3" />

<img width="460" height="303" alt="image" src="https://github.com/user-attachments/assets/eb749c85-db6a-4955-8bc4-32bb63ff1251" />

### Шаг 2. Выполните следующие тесты. Эхозапрос должен пройти успешно.
Примечание. Возможно, вам придется отключить брандмауэр ПК для работы ping

<img width="665" height="272" alt="image" src="https://github.com/user-attachments/assets/d58f8bf7-b20e-4085-aa04-88a9f715f31a" />

### PC-A

<img width="405" height="191" alt="image" src="https://github.com/user-attachments/assets/c8ea5bad-6c1c-4202-8f2b-75cc8d4e124c" />

<img width="426" height="209" alt="image" src="https://github.com/user-attachments/assets/3d81d4d1-6c87-494b-bbb4-6edb75362292" />

### PC-B

<img width="408" height="610" alt="image" src="https://github.com/user-attachments/assets/08fab593-8189-4c4e-b13f-9aef889d8065" />

<img width="223" height="100" alt="image" src="https://github.com/user-attachments/assets/f9bc20e8-8664-4d44-bec1-f9b78de44486" />

<img width="231" height="99" alt="image" src="https://github.com/user-attachments/assets/77474a48-a477-4cdc-baeb-9690eeecf1d2" />

## Часть 7. Настройка и проверка списков контроля доступа (ACL)
При проверке базового подключения компания требует реализации следующих политик безопасности:

Политика1. Сеть Sales не может использовать SSH в сети Management (но в  другие сети SSH разрешен). 

Политика 2. Сеть Sales не имеет доступа к IP-адресам в сети Management с помощью любого веб-протокола (HTTP/HTTPS). Сеть Sales также не имеет доступа к интерфейсам R1 с помощью любого веб-протокола. Разрешён весь другой веб-трафик (обратите внимание — Сеть Sales  может получить доступ к интерфейсу Loopback 1 на R1).

Политика3. Сеть Sales не может отправлять эхо-запросы ICMP в сети Operations или Management. Разрешены эхо-запросы ICMP к другим адресатам. 

Политика 4: Cеть Operations  не может отправлять ICMP эхозапросы в сеть Sales. Разрешены эхо-запросы ICMP к другим 
адресатам. 

### Шаг 1. Проанализируйте требования к сети и политике безопасности для планирования реализации ACL.
 
### Шаг 2. Разработка и применение расширенных списков доступа, которые будут соответствовать требованиям политики безопасности.
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#ip access-list extended SALES-IN
R1(config-ext-nacl)#remark Policy1: block SSH from Sales to Management
R1(config-ext-nacl)#deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 22
R1(config-ext-nacl)#remark Policy2: block HTTP/HTTPS from Sales to Management
R1(config-ext-nacl)#deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 80
R1(config-ext-nacl)#deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 443
R1(config-ext-nacl)#remark Policy2: block HTTP/HTTPS from Sales to R1 routed interfaces (SVIs on router-on-a-stick)
R1(config-ext-nacl)#deny tcp 10.40.0.0 0.0.0.255 host 10.20.0.1 eq 80
R1(config-ext-nacl)#deny tcp 10.40.0.0 0.0.0.255 host 10.20.0.1 eq 443
R1(config-ext-nacl)#deny tcp 10.40.0.0 0.0.0.255 host 10.30.0.1 eq 80
R1(config-ext-nacl)#deny tcp 10.40.0.0 0.0.0.255 host 10.30.0.1 eq 443
R1(config-ext-nacl)#deny tcp 10.40.0.0 0.0.0.255 host 10.40.0.1 eq 80
R1(config-ext-nacl)#deny tcp 10.40.0.0 0.0.0.255 host 10.40.0.1 eq 443
R1(config-ext-nacl)#remark Policy3: block ICMP echo from Sales to Operations and Management
R1(config-ext-nacl)#deny icmp 10.40.0.0 0.0.0.255 10.30.0.0 0.0.0.255 echo
R1(config-ext-nacl)#deny icmp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 echo
R1(config-ext-nacl)#remark Allow everything else (includes SSH to other nets and web/ssh to 172.16.1.1)
R1(config-ext-nacl)#permit ip 10.40.0.0 0.0.0.255 any
R1(config-ext-nacl)#exit
```
```
R1(config)#ip access-list extended OPERATIONS-IN
R1(config-ext-nacl)#remark Policy4: block ICMP echo from Operations to Sales
R1(config-ext-nacl)#deny icmp 10.30.0.0 0.0.0.255 10.40.0.0 0.0.0.255 echo
R1(config-ext-nacl)#remark Allow everything else
R1(config-ext-nacl)#permit ip 10.30.0.0 0.0.0.255 any
R1(config-ext-nacl)#exit
```
```
R1(config)#interface g0/0/1.40
R1(config-subif)#ip access-group SALES-IN in
R1(config-subif)#exit
R1(config)#interface g0/0/1.30
R1(config-subif)#ip access-group OPERATIONS-IN in
R1(config-subif)#end
R1#wr
Building configuration...
[OK]
```
### Шаг 3. Убедитесь, что политики безопасности применяются развернутыми списками доступа.
Выполните следующие тесты. Ожидаемые результаты показаны в таблице:

<img width="677" height="270" alt="image" src="https://github.com/user-attachments/assets/4600cad5-ea21-4536-8a75-7e64e2f29c95" />

### PC-A

<img width="415" height="164" alt="image" src="https://github.com/user-attachments/assets/f3a4e00f-5cb9-4c5b-931b-cb763f64a2ff" />

<img width="402" height="190" alt="image" src="https://github.com/user-attachments/assets/17a763b5-b0ae-4d07-b42c-8ec5080c8606" />

### PC-B

<img width="411" height="162" alt="image" src="https://github.com/user-attachments/assets/51dd32ef-0094-48d5-991d-343c4d3f9d81" />

<img width="415" height="166" alt="image" src="https://github.com/user-attachments/assets/4472450f-00fb-46f1-9c4c-623b3cb83dcd" />

<img width="398" height="189" alt="image" src="https://github.com/user-attachments/assets/57861196-c4f2-4bfa-88b6-f216c6f86a3e" />

<img width="360" height="47" alt="image" src="https://github.com/user-attachments/assets/44d9e20f-eb90-476d-895b-f962af212f1e" />

<img width="220" height="155" alt="image" src="https://github.com/user-attachments/assets/17e5ca9a-bde8-402c-82f1-e5b6e15a7bef" />

```
R1#show access-lists
Extended IP access list SALES-IN
    10 deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 22 (12 match(es))
    20 deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq www
    30 deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 443
    40 deny tcp 10.40.0.0 0.0.0.255 host 10.20.0.1 eq www
    50 deny tcp 10.40.0.0 0.0.0.255 host 10.20.0.1 eq 443
    60 deny tcp 10.40.0.0 0.0.0.255 host 10.30.0.1 eq www
    70 deny tcp 10.40.0.0 0.0.0.255 host 10.30.0.1 eq 443
    80 deny tcp 10.40.0.0 0.0.0.255 host 10.40.0.1 eq www
    90 deny tcp 10.40.0.0 0.0.0.255 host 10.40.0.1 eq 443
    100 deny icmp 10.40.0.0 0.0.0.255 10.30.0.0 0.0.0.255 echo (4 match(es))
    110 deny icmp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 echo (4 match(es))
    120 permit ip 10.40.0.0 0.0.0.255 any (29 match(es))
Extended IP access list OPERATIONS-IN
    10 deny icmp 10.30.0.0 0.0.0.255 10.40.0.0 0.0.0.255 echo (4 match(es))
    20 permit ip 10.30.0.0 0.0.0.255 any (4 match(es))
```
```
HTTPS: На используемой модели R1 команды включения HTTPS-сервера (ip http secure-server, ip http authentication local) не поддерживаются (Invalid input), поэтому проверка через браузер ограничена. Политика 2 подтверждается ACL SALES-IN: HTTPS (tcp/443) из сети Sales 10.40.0.0/24 в сеть Management 10.20.0.0/24 и к интерфейсу R1 10.20.0.1 запрещён, при этом доступ к Loopback1 172.16.1.1 не запрещён и разрешается правилом permit ip.
```
