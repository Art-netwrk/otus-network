<img width="443" height="213" alt="image" src="https://github.com/user-attachments/assets/b45f145b-7085-47d6-92da-2634f2705f46" />

<img width="433" height="210" alt="image" src="https://github.com/user-attachments/assets/6709ec48-071f-447e-b012-4cc152445729" />
```
Switch3#show spanning-tree vlan 999
VLAN0999
  Spanning tree enabled protocol rstp
  Root ID    Priority    25575
             Address     00D0.9799.12AA
             Cost        9
             Port        28(Port-channel13)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    33767  (priority 32768 sys-id-ext 999)
             Address     0004.9A98.5207
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Po23             Altn BLK 9         128.29   Shr
Po13             Root FWD 9         128.28   Shr
```
```
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
Po12             Root FWD 9         128.27   Shr
Po23             Desg FWD 9         128.29   Shr

```
```
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
Po13             Desg FWD 9         128.29   Shr
Po12             Desg FWD 9         128.27   Shr
```
```
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

Port        Vlans allowed on trunk
Po12        999
Po13        999

Port        Vlans allowed and active in management domain
Po12        999
Po13        999

Port        Vlans in spanning tree forwarding state and not pruned
Po12        999
Po13        999
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
```
```
Switch1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch1(config)#interface vlan 999
Switch1(config-if)# ip address 192.168.255.5 255.255.255.248
Switch1(config-if)# no shut
Switch1(config-if)#exit
Switch1(config)#ip default-gateway 192.168.255.1
Switch1(config)#end
Switch1#wr
Building configuration...
[OK]
```
```
Switch2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch2(config)#interface vlan 999
Switch2(config-if)# ip address 192.168.255.6 255.255.255.248
Switch2(config-if)# no shut
Switch2(config-if)#exit
Switch2(config)#ip default-gateway 192.168.255.1
Switch2(config)#end
Switch2#wr
Building configuration...
[OK]
```
```
Switch3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch3(config)#interface vlan 999
Switch3(config-if)# ip address 192.168.255.7 255.255.255.248
Bad mask /29 for address 192.168.255.7
Switch3(config-if)# no shut
Switch3(config-if)#exit
Switch3(config)#ip default-gateway 192.168.255.1
Switch3(config)#end
Switch3#wr
Building configuration...
[OK]
```



# 3.4 Конфигурация Switch1 (Root Primary)
## 3.4.1 Базовая настройка + VLAN999
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
## 3.4.2 EtherChannel S1↔S2 (Po12, Fa0/1-2)
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
## 3.4.3 EtherChannel S1↔S3 (Po13, Fa0/3-4)
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
## 3.4.4 STP Root Primary VLAN999
```
Switch1(config)#spanning-tree vlan 999 root primary
```
## 3.4.5 Management IP (SVI VLAN999)
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
# 3.5 Конфигурация Switch2 (Root Secondary)
## 3.5.1 Базовая настройка + VLAN999
```
Switch(config)#hostname Switch2
Switch2(config)#no ip domain-lookup
Switch2(config)#spanning-tree mode rapid-pvst
Switch2(config)#vlan 999
Switch2(config-vlan)# name TRANSIT_OSPF
Switch2(config-vlan)#exit
```
## 3.5.2 EtherChannel S2↔S1 (Po12, Fa0/1-2)
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
## 3.5.3 EtherChannel S2↔S3 (Po23, Fa0/3-4)
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
## 3.5.4 STP Root Secondary VLAN999
```
Switch2(config)#spanning-tree vlan 999 root secondary
```
## 3.5.5 Management IP (SVI VLAN999)
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
# 3.6 Конфигурация Switch3
## 3.6.1 Базовая настройка + VLAN999
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
## 3.6.2 EtherChannel S3↔S1 (Po13, Fa0/1-2)
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
## 3.6.3 EtherChannel S3↔S2 (Po23, Fa0/3-4)
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
## 3.6.4 Management IP (SVI VLAN999)
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
# 3.7 Проверка и результаты
## 3.7.1 Проверка EtherChannel на каждом коммутаторе
```
show etherchannel summary
```
<img width="408" height="280" alt="image" src="https://github.com/user-attachments/assets/fa110821-7f24-4aa4-8e38-ff654c8eacfc" />
<img width="407" height="280" alt="image" src="https://github.com/user-attachments/assets/cd6418ad-4e81-4888-90a4-956b5e4f6371" />
<img width="409" height="278" alt="image" src="https://github.com/user-attachments/assets/9f5e7a24-6fc0-47bb-a3f8-34f26a4d1532" />
### Результат:
#### На Switch1: Po12(SU) и Po13(SU), порты в составе помечены (P).
#### На Switch2: Po12(SU) и Po23(SU), порты (P).
#### На Switch3: Po13(SU) и Po23(SU), порты (P).

#### Это означает, что порт-каналы работают на L2 (S) и находятся в использовании (U), а физические порты агрегированы (P).



























