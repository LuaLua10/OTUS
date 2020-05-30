Лабораторная работа. Маршрутизация на основе политики (PBR). 
---------

Топология
---------

![](media/073df55cf8a389967d537a5c28c4e16e.png)

Задачи
---------
Настроить OSPF офисе Москва. Разделить сеть на зоны. Настроить фильтрацию между зонами.
1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone;
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по-умолчанию;
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию;
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101;
5. План работы и изменения зафиксированы в документации.

```
Примечание: так как согластно выбранной архитектуре сети, коммутаторы SW4 и SW5 будут участвовать в маршрутизации,
на них будет запущен процесс OSPF. Коммутаторы будут находится в Area 10.
```

Решение
---------

#### Настроим OSPF на маршрутизаторах R14 и R15

##### Конфигурация R15:
 
```
ip prefix-list DEAD_area101 seq 5 deny 100.1.0.0/30
ip prefix-list DEAD_area101 seq 10 permit 0.0.0.0/0 le 32
router ospf 1
 router-id 15.15.15.15
 area 102 filter-list prefix DEAD_area101 in
 passive-interface Ethernet0/2
 network 100.1.0.12 0.0.0.3 area 0
 network 100.1.0.16 0.0.0.3 area 0
 network 100.1.0.20 0.0.0.3 area 102
```

##### Таблица маршрутизации R15:

```
R15#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      30.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        30.1.0.0/30 is directly connected, Ethernet0/2
L        30.1.0.2/32 is directly connected, Ethernet0/2
      100.0.0.0/8 is variably subnetted, 16 subnets, 4 masks
O IA     100.1.0.0/30 [110/30] via 100.1.0.18, 03:18:54, Ethernet0/0
                      [110/30] via 100.1.0.14, 03:22:04, Ethernet0/1
O        100.1.0.4/30 [110/20] via 100.1.0.14, 03:22:04, Ethernet0/1
O        100.1.0.8/30 [110/20] via 100.1.0.18, 03:18:54, Ethernet0/0
C        100.1.0.12/30 is directly connected, Ethernet0/1
L        100.1.0.13/32 is directly connected, Ethernet0/1
C        100.1.0.16/30 is directly connected, Ethernet0/0
L        100.1.0.17/32 is directly connected, Ethernet0/0
C        100.1.0.20/30 is directly connected, Ethernet0/3
L        100.1.0.21/32 is directly connected, Ethernet0/3
O IA     100.1.0.24/30 [110/20] via 100.1.0.14, 03:21:17, Ethernet0/1
O IA     100.1.0.28/30 [110/20] via 100.1.0.14, 03:20:59, Ethernet0/1
O IA     100.1.0.32/30 [110/20] via 100.1.0.18, 03:17:54, Ethernet0/0
O IA     100.1.0.36/30 [110/20] via 100.1.0.18, 02:03:11, Ethernet0/0
O IA     100.1.0.64/26 [110/21] via 100.1.0.18, 02:03:57, Ethernet0/0
                       [110/21] via 100.1.0.14, 02:03:57, Ethernet0/1
O IA     100.1.1.0/24 [110/21] via 100.1.0.18, 02:03:57, Ethernet0/0
                      [110/21] via 100.1.0.14, 02:03:57, Ethernet0/1
O IA     100.1.2.0/24 [110/21] via 100.1.0.18, 02:03:57, Ethernet0/0
                      [110/21] via 100.1.0.14, 02:03:57, Ethernet0/1
```

##### Конфигурация R14:

```
router ospf 1
 router-id 14.14.14.14
 area 101 stub no-summary
 passive-interface Ethernet0/2
 network 100.1.0.0 0.0.0.3 area 101
 network 100.1.0.4 0.0.0.3 area 0
 network 100.1.0.8 0.0.0.3 area 0
```


##### Таблица маршрутизации R14:

```
R14#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      100.0.0.0/8 is variably subnetted, 16 subnets, 4 masks
C        100.1.0.0/30 is directly connected, Ethernet0/3
L        100.1.0.1/32 is directly connected, Ethernet0/3
C        100.1.0.4/30 is directly connected, Ethernet0/0
L        100.1.0.5/32 is directly connected, Ethernet0/0
C        100.1.0.8/30 is directly connected, Ethernet0/1
L        100.1.0.9/32 is directly connected, Ethernet0/1
O        100.1.0.12/30 [110/20] via 100.1.0.6, 01:59:28, Ethernet0/0
O        100.1.0.16/30 [110/20] via 100.1.0.10, 01:59:28, Ethernet0/1
O IA     100.1.0.20/30 [110/30] via 100.1.0.10, 01:59:28, Ethernet0/1
                       [110/30] via 100.1.0.6, 01:59:28, Ethernet0/0
O IA     100.1.0.24/30 [110/20] via 100.1.0.6, 01:59:28, Ethernet0/0
O IA     100.1.0.28/30 [110/20] via 100.1.0.6, 01:59:28, Ethernet0/0
O IA     100.1.0.32/30 [110/20] via 100.1.0.10, 01:59:28, Ethernet0/1
O IA     100.1.0.36/30 [110/20] via 100.1.0.10, 01:59:28, Ethernet0/1
O IA     100.1.0.64/26 [110/21] via 100.1.0.10, 01:59:28, Ethernet0/1
                       [110/21] via 100.1.0.6, 01:59:28, Ethernet0/0
O IA     100.1.1.0/24 [110/21] via 100.1.0.10, 01:59:28, Ethernet0/1
                      [110/21] via 100.1.0.6, 01:59:28, Ethernet0/0
O IA     100.1.2.0/24 [110/21] via 100.1.0.10, 01:59:28, Ethernet0/1
                      [110/21] via 100.1.0.6, 01:59:28, Ethernet0/0
      101.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        101.0.0.0/30 is directly connected, Ethernet0/2
L        101.0.0.2/32 is directly connected, Ethernet0/2
```

#### Настроим OSPF на маршрутизаторах R12 и R13

##### Конфигурация R12:

```
router ospf 1
 router-id 12.12.12.12
 area 10 stub
 network 100.1.0.4 0.0.0.3 area 0
 network 100.1.0.12 0.0.0.3 area 0
 network 100.1.0.24 0.0.0.3 area 10
 network 100.1.0.28 0.0.0.3 area 10
```

##### Таблица маршрутизации R12:

```
R12#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      100.0.0.0/8 is variably subnetted, 17 subnets, 4 masks
O IA     100.1.0.0/30 [110/20] via 100.1.0.5, 02:10:10, Ethernet0/2
C        100.1.0.4/30 is directly connected, Ethernet0/2
L        100.1.0.6/32 is directly connected, Ethernet0/2
O        100.1.0.8/30 [110/20] via 100.1.0.5, 02:10:10, Ethernet0/2
C        100.1.0.12/30 is directly connected, Ethernet0/3
L        100.1.0.14/32 is directly connected, Ethernet0/3
O        100.1.0.16/30 [110/20] via 100.1.0.13, 02:10:10, Ethernet0/3
O IA     100.1.0.20/30 [110/20] via 100.1.0.13, 02:10:10, Ethernet0/3
C        100.1.0.24/30 is directly connected, Ethernet0/0
L        100.1.0.25/32 is directly connected, Ethernet0/0
C        100.1.0.28/30 is directly connected, Ethernet0/1
L        100.1.0.29/32 is directly connected, Ethernet0/1
O        100.1.0.32/30 [110/20] via 100.1.0.26, 02:06:32, Ethernet0/0
O        100.1.0.36/30 [110/20] via 100.1.0.30, 02:05:56, Ethernet0/1
O        100.1.0.64/26 [110/11] via 100.1.0.30, 02:06:06, Ethernet0/1
                       [110/11] via 100.1.0.26, 02:06:42, Ethernet0/0
O        100.1.1.0/24 [110/11] via 100.1.0.30, 02:06:06, Ethernet0/1
                      [110/11] via 100.1.0.26, 02:06:42, Ethernet0/0
O        100.1.2.0/24 [110/11] via 100.1.0.30, 02:06:06, Ethernet0/1
                      [110/11] via 100.1.0.26, 02:06:42, Ethernet0/0
```

##### Конфигурация R13:

```
router ospf 1
 router-id 13.13.13.13
 area 10 stub
 network 100.1.0.8 0.0.0.3 area 0
 network 100.1.0.16 0.0.0.3 area 0
 network 100.1.0.32 0.0.0.3 area 10
 network 100.1.0.36 0.0.0.3 area 10
```

##### Таблица маршрутизации R13:

```
R13#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      100.0.0.0/8 is variably subnetted, 17 subnets, 4 masks
O IA     100.1.0.0/30 [110/20] via 100.1.0.9, 02:11:20, Ethernet0/3
O        100.1.0.4/30 [110/20] via 100.1.0.9, 02:11:20, Ethernet0/3
C        100.1.0.8/30 is directly connected, Ethernet0/3
L        100.1.0.10/32 is directly connected, Ethernet0/3
O        100.1.0.12/30 [110/20] via 100.1.0.17, 02:11:20, Ethernet0/2
C        100.1.0.16/30 is directly connected, Ethernet0/2
L        100.1.0.18/32 is directly connected, Ethernet0/2
O IA     100.1.0.20/30 [110/20] via 100.1.0.17, 02:11:20, Ethernet0/2
O        100.1.0.24/30 [110/20] via 100.1.0.34, 02:08:21, Ethernet0/1
O        100.1.0.28/30 [110/20] via 100.1.0.38, 02:07:45, Ethernet0/0
C        100.1.0.32/30 is directly connected, Ethernet0/1
L        100.1.0.33/32 is directly connected, Ethernet0/1
C        100.1.0.36/30 is directly connected, Ethernet0/0
L        100.1.0.37/32 is directly connected, Ethernet0/0
O        100.1.0.64/26 [110/11] via 100.1.0.38, 02:07:45, Ethernet0/0
                       [110/11] via 100.1.0.34, 02:08:31, Ethernet0/1
O        100.1.1.0/24 [110/11] via 100.1.0.38, 02:07:45, Ethernet0/0
                      [110/11] via 100.1.0.34, 02:08:31, Ethernet0/1
O        100.1.2.0/24 [110/11] via 100.1.0.38, 02:07:45, Ethernet0/0
                      [110/11] via 100.1.0.34, 02:08:31, Ethernet0/1
```

#### Настроим OSPF на L3 коммутаторах SW4 и SW5

##### Конфигурация SW4:

```
router ospf 1
 router-id 4.4.4.4
 area 10 stub
 passive-interface default
 no passive-interface Ethernet1/0
 no passive-interface Ethernet1/1
 network 100.1.0.24 0.0.0.3 area 10
 network 100.1.0.32 0.0.0.3 area 10
 network 100.1.0.64 0.0.0.63 area 10
 network 100.1.1.0 0.0.0.255 area 10
 network 100.1.2.0 0.0.0.255 area 10
```

##### Конфигурация SW5:

```
router ospf 1
 router-id 5.5.5.5
 area 10 stub
 passive-interface default
 no passive-interface Ethernet1/0
 no passive-interface Ethernet1/1
 network 100.1.0.28 0.0.0.3 area 10
 network 100.1.0.36 0.0.0.3 area 10
 network 100.1.0.64 0.0.0.63 area 10
 network 100.1.1.0 0.0.0.255 area 10
 network 100.1.2.0 0.0.0.255 area 10
```

>Таблицы маршрутизации на L3 коммутаторах SW4 и SW5 будут аналогичны другим маршрутизаторам этой зоны, кроме прямых соединений.

#### Настроим OSPF на маршрутизаторах R19 и R20

##### Конфигурация R19:

```
router ospf 1
 router-id 19.19.19.19
 area 101 stub
 network 100.1.0.0 0.0.0.3 area 101
```

##### Таблица маршрутизации R19:

```
R19#show ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 100.1.0.1 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 100.1.0.1, 01:42:28, Ethernet0/0
```

##### Конфигурация R20:

```
router ospf 1
 router-id 20.20.20.20
 network 100.1.0.20 0.0.0.3 area 102
```

##### Таблица маршрутизации R20:

```
R20#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is not set

      100.0.0.0/8 is variably subnetted, 13 subnets, 4 masks
O IA     100.1.0.4/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.8/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.12/30 [110/20] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.16/30 [110/20] via 100.1.0.21, 01:21:57, Ethernet0/0
C        100.1.0.20/30 is directly connected, Ethernet0/0
L        100.1.0.22/32 is directly connected, Ethernet0/0
O IA     100.1.0.24/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.28/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.32/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.36/30 [110/30] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.0.64/26 [110/31] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.1.0/24 [110/31] via 100.1.0.21, 01:21:57, Ethernet0/0
O IA     100.1.2.0/24 [110/31] via 100.1.0.21, 01:21:57, Ethernet0/0
```


