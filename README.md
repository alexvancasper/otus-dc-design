
# Лабораторная работа по теме "Оптимизация таблиц маршрутизации"

### Цель:
- разобрать EVPN route-type 5 и его применение;
- настройка route-type для оптимизации маршрутизации.

### Топология

![Топология](topology-evpn-esi.png "Топология сети")

## Реализация

Топология и настройки всех узлов сохранены с прошлой работы ESI LAG.
Цель EVPN Type-5 обеспечение связности между дата-центрами, подами.
Идея маршрута в том, что в отличии от Type-2 анонс Type-5 отделяется от MAC адреса 
и анонсируется только L3 префикс, который в дальнейшем может использовать и обычном l3-vrf.

Пример конфигурации EVPN Type-5 для v10-vrf, v20-vrf, v30-vrf
```
set routing-instances v10-vrf protocols evpn ip-prefix-routes advertise direct-nexthop
set routing-instances v10-vrf protocols evpn ip-prefix-routes encapsulation vxlan
set routing-instances v10-vrf protocols evpn ip-prefix-routes vni 9910

set routing-instances v20-vrf protocols evpn ip-prefix-routes advertise direct-nexthop
set routing-instances v20-vrf protocols evpn ip-prefix-routes encapsulation vxlan
set routing-instances v20-vrf protocols evpn ip-prefix-routes vni 9920

set routing-instances v30-vrf protocols evpn ip-prefix-routes advertise direct-nexthop
set routing-instances v30-vrf protocols evpn ip-prefix-routes encapsulation vxlan
set routing-instances v30-vrf protocols evpn ip-prefix-routes vni 9930

```
После добавления конфигурации выше, в сети появляются новые маршруты EVPN Type-5

```
2:10.255.254.1:1::0::2c:6b:f5:18:6f:f0::10.5.5.100/304 MAC/IP (1 entry, 0 announced)
        *EVPN   Preference: 170
                Next hop type: Indirect, Next hop index: 0
                Address: 0x89fc7ac
                Next-hop reference count: 33, key opaque handle: 0x0
                Protocol next hop: 10.255.254.1
                Indirect next hop: 0x0 - INH Session ID: 0
                State: <Secondary Active Int Ext>
                Age: 23:43
                Validation State: unverified
                Task: v10-evpn
                AS path: I
                Communities: target:42011:10 target:42011:1010 encapsulation:vxlan(0x8) evpn-default-gateway router-mac:2c:6b:f5:18:6f:f0
                Route Label: 10010
                Route Label: 9910
                ESI: 00:00:00:00:00:00:00:00:00:00
                Primary Routing Table: v10.evpn.0
                Thread: junos-main

5:10.255.254.1:100::0::10.5.5.0::24/248 (1 entry, 0 announced)
        *EVPN   Preference: 170
                Next hop type: Fictitious, Next hop index: 0
                Address: 0x89fc5fc
                Next-hop reference count: 12, key opaque handle: 0x0
                Next hop:
                State: <Secondary Active Int Ext>
                Age: 23:43
                Validation State: unverified
                Task: v10-vrf-EVPN-L3-context
                AS path: I
                Communities: target:42011:1010 encapsulation:vxlan(0x8) router-mac:2c:6b:f5:18:6f:f0
                Route Label: 9910
                Overlay gateway address: 0.0.0.0
                ESI 00:00:00:00:00:00:00:00:00:00
                Primary Routing Table: v10-vrf.evpn.0
                Thread: junos-main
```

## Связность

Доступность с vEOS до всех других клиентов сохраняется.
```
Server-MH#ping 10.5.5.10 repeat 1
PING 10.5.5.10 (10.5.5.10) 72(100) bytes of data.
80 bytes from 10.5.5.10: icmp_seq=1 ttl=254 time=14.9 ms

--- 10.5.5.10 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 14.920/14.920/14.920/0.000 ms
Server-MH#ping 10.5.5.20 repeat 1
PING 10.5.5.20 (10.5.5.20) 72(100) bytes of data.
80 bytes from 10.5.5.20: icmp_seq=1 ttl=253 time=17.8 ms

--- 10.5.5.20 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 17.897/17.897/17.897/0.000 ms
Server-MH#ping 172.17.0.1 repeat 1
PING 172.17.0.1 (172.17.0.1) 72(100) bytes of data.
80 bytes from 172.17.0.1: icmp_seq=1 ttl=253 time=22.8 ms

--- 172.17.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 22.819/22.819/22.819/0.000 ms
Server-MH#ping 172.17.0.2 repeat 1
PING 172.17.0.2 (172.17.0.2) 72(100) bytes of data.
80 bytes from 172.17.0.2: icmp_seq=1 ttl=253 time=16.5 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 16.578/16.578/16.578/0.000 ms
Server-MH#
```


Useful links:
- [Understanding EVPN Pure Type 5 Routes](https://www.juniper.net/documentation/us/en/software/junos/evpn-vxlan/topics/concept/evpn-route-type5-understanding.html)
- [EVPN Type-5](https://www.juniper.net/documentation/us/en/software/junos/evpn-vxlan/topics/concept/evpn-vxlan-encapsulation.html)  
- [EVPN Type 2 and Type 5 Route Coexistence](https://www.juniper.net/documentation/us/en/software/junos/evpn-vxlan/topics/concept/evpn-t2-t5-coexist-evpn-vxlan.html)
- [EVPN Type 5 Routing over VXLAN Tunnels](https://www.juniper.net/documentation/us/en/software/cloud-native-router23.3/cloud-native-router-user/topics/concept/l3-evpn-type5-routing-over-vxlan-tunnels.html)
- [EVPN Type-5 MPLS](https://bgphelp.com/2017/04/23/evpn-type-5-configuration-example-juniper-mx/)  