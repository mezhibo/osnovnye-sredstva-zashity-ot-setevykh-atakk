![alt text](https://github.com/mezhibo/security-methods/blob/f3c95789b4f014251d77bdf7e27b6ed5a56ab61e/IMG/1.png)


**Задание 1**

На картинке изображена схема офисной сети:

![alt text](https://github.com/mezhibo/osnovnye-sredstva-zashity-ot-setevykh-atakk/blob/70376c341fad4887f26e49125a4bca5b00030de4/IMG/1.png)


Каким образом необходимо настроить все 4 свитча, чтобы в сети корректно работал только один DHCP сервер - маршрутизатор R1. Очень важна корректная конфигурация портов на всех свитчах.

Перечислите список свитчей и список команд, которые необходимо выполнить.



**Решение 1**

Для выполнения задания в эмуляторе строится сеть:


![alt text](https://github.com/mezhibo/osnovnye-sredstva-zashity-ot-setevykh-atakk/blob/70376c341fad4887f26e49125a4bca5b00030de4/IMG/2.png)

Настраиваются два DHCP сервера на маршрутизаторах R1 и R2:

```
---R1--
Router(config)#host R1
R1(config)#int e0/0
R1(config-if)#ip addr 192.168.1.1 255.255.255.0
R1(config-if)#no sh

R1(config)#ip dhcp excluded-address 192.168.1.1
R1(config)#ip dhcp pool MY-POOL
R1(dhcp-config)#network 192.168.1.0 255.255.255.0
R1(dhcp-config)#default-router 192.168.1.1

---R2---
Router(config)#host R2
R2(config)#int e0/0
R2(config-if)#ip addr 10.10.10.1 255.255.255.0
R2(config-if)#no sh

R2(config-if)#exi
R2(config)#ip dhcp excluded-address 10.10.10.1
R2(config)#ip dhcp pool MY-POOL-10
R2(dhcp-config)#network 10.10.10.0 255.255.255.0
R2(dhcp-config)#default-router 10.10.10.1
```

При данной конфигурации в сети работают два сервера DHCP, устройства получают адреса разных сетей, что не является приемлемым с точки зрения безопасности:


![alt text](https://github.com/mezhibo/osnovnye-sredstva-zashity-ot-setevykh-atakk/blob/70376c341fad4887f26e49125a4bca5b00030de4/IMG/3.png)


Для того, чтобы в сети корректно работал только один DHCP сервер на R1, необходимо использовать механизм DHCP snooping на всех коммутаторах. Интерфейс SW1, подключенный к DHCP серверу (R1), является доверенным, а остальные интерфейсы – недоверенными. На остальных коммутаторах доверенным является интерфейс в сторону SW1.

Настойка осуществляется следующими командами:

```
---SW1---
SW1(config)#ip dhcp snooping //активация DHCP snooping на свитче
SW1(config)#ip dhcp snooping vlan 1
SW1(config)#no ip dhcp snooping information option //отключаем опцию 82
SW1(config)#int e0/2
SW1(config-if)#ip dhcp snooping trust //конфигурация trust-порта

---SW2---
SW2(config)#ip dhcp snooping
SW2(config)#ip dhcp snooping vlan 1
SW2(config)#no ip dhcp snooping information option
SW2(config)#int e0/0
SW2(config-if)#ip dhcp snooping trust

---SW3---
SW3(config)#ip dhcp snooping
SW3(config)#ip dhcp snooping vlan 1
SW3(config)#no ip dhcp snooping information option
SW3(config-if)#int e0/2
SW3(config-if)#ip dhcp snooping trust

---SW4---
SW4(config)#ip dhcp snooping
SW4(config)#ip dhcp snooping vlan 1
SW4(config)#no ip dhcp snooping information option
SW4(config)#int e0/1
SW4(config-if)#ip dhcp snooping trust
```

После настройки хосты всегда получают адрес от доверенного DHCP сервера.


![alt text](https://github.com/mezhibo/osnovnye-sredstva-zashity-ot-setevykh-atakk/blob/70376c341fad4887f26e49125a4bca5b00030de4/IMG/4.png)




**Задание 2**

По топологии из задания 1 необходимо на SW2 настроить ARP Inspection и IP Source guard для Client9 и Client10, подключенных к SW2. Client9 получает адрес по DHCP, Client10 ip адрес задан статически. DHCP Snooping уже настроен по первому заданию.

Перечислите список команд, которые необходимо применить на SW2


**Решение 2**

Изменяем схему в соответствии с заданием:


![alt text](https://github.com/mezhibo/osnovnye-sredstva-zashity-ot-setevykh-atakk/blob/70376c341fad4887f26e49125a4bca5b00030de4/IMG/5.png)


Настройка SW2:

```
SW2(config)#ip arp inspection vlan 1 //Включаем Dynamic ARP Inspection на коммутаторе
SW2(config)#int e0/0
SW2(config-if)#ip arp inspection trust //доверяем порту, к которому подключён другой коммутатор
SW2(config-if)#int e0/2
SW2(config-if)#ip arp inspection trust

--Включаем IP Source Guard на интерфейсах, к которым подключены хосты

SW2(config-if)#int e0/1
SW2(config-if)#ip verify source

-- Закрепление определенного хоста с статикой за интерфейсом, внесение записи в БД
SW2(config)#ip source binding 0050.7966.680a vlan 1 192.168.1.10 interface e0/1

SW2(config-if)#int e0/3
SW2(config-if)#ip verify source
```


Настройки обеспечивают базовую политику безопасности на уровне L2 коммутатора, защищая от атак, связанных с ARP и подделкой источников IP-адресов.
