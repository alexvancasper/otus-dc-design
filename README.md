
# Лабораторная работа по теме "Построение Underlay сети (OSPF)"

### Цель:
- исследовать построение Underlay сети с использованием OSPF

### Топология

![Топология](target_topo.avif "Топология")

## Реализация

В конфигурацию устройств был добавлен OSPF протокол.
Интервалы `Hello=1` и `Dead=4`.
Интерфейсы все в `point-to-point`.
Лупбек адреса добавлены как `passive`.
Все интерфейсы в `area 0`.

Пример конфигурации OSPF для Leaf-1
```
set protocols ospf area 0.0.0.0 interface xe-0/0/0.0 interface-type p2p
set protocols ospf area 0.0.0.0 interface xe-0/0/0.0 hello-interval 1
set protocols ospf area 0.0.0.0 interface xe-0/0/0.0 dead-interval 4
set protocols ospf area 0.0.0.0 interface xe-0/0/1.0 interface-type p2p
set protocols ospf area 0.0.0.0 interface xe-0/0/1.0 hello-interval 1
set protocols ospf area 0.0.0.0 interface xe-0/0/1.0 dead-interval 4
set protocols ospf area 0.0.0.0 interface lo0.0 passive
set policy-options policy-statement ecmp term 1 then load-balance per-packet
```

Пример конфигурации OSPF для Spine-1
```
set protocols ospf area 0.0.0.0 interface xe-0/0/0.0 interface-type p2p
set protocols ospf area 0.0.0.0 interface xe-0/0/0.0 hello-interval 1
set protocols ospf area 0.0.0.0 interface xe-0/0/0.0 dead-interval 4
set protocols ospf area 0.0.0.0 interface xe-0/0/1.0 interface-type p2p
set protocols ospf area 0.0.0.0 interface xe-0/0/1.0 hello-interval 1
set protocols ospf area 0.0.0.0 interface xe-0/0/1.0 dead-interval 4
set protocols ospf area 0.0.0.0 interface lo0.0 passive
set policy-options policy-statement ecmp term 1 then load-balance per-packet
```

Команда `set policy-options policy-statement ecmp term 1 then load-balance per-packet` нужна для включения ECMP per-flow.

[Understanding Per-Packet Load Balancing](https://www.juniper.net/documentation/us/en/software/junos/sampling-forwarding-monitoring/topics/concept/policy-per-packet-load-balancing-overview.html)