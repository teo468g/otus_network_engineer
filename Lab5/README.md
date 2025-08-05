Маршрутизация на основе политик (PBR) 

![](topology.PNG) 

Задание:

1. Настроить политику маршрутизации для сетей офиса.
2. Распределить трафик между двумя линками с провайдером.
3. Настроить отслеживание линка через технологию IP SLA.
4. Настройть для офиса Лабытнанги маршрут по-умолчанию.


На R28 создадим маршрутную карту Balance для распределения трафика от VPC30 на R26 и VPC31 на R25.  

route-map Balance permit 10  
 match ip address Vlan30  
 set ip default next-hop 192.168.1.16  
  
route-map Balance permit 20  
 match ip address Vlan31  
 set ip default next-hop 192.168.1.14  

 Создадим access листы Vlan30 для VPC30 и Vlan31 для VPC31

ip access-list standard Vlan30  
 permit 172.18.1.0 0.0.0.255  
ip access-list standard Vlan31  
 permit 172.18.2.0 0.0.0.255  

 Применим route-map на входящем интерфейсе R28

 interface Ethernet0/2  
 description to SW29  
 no ip address  
 ip policy route-map Balance  

Настроим отслеживание линка R28 e0/2 - SW29 e0/2 

ip sla 1   
 icmp-echo 172.18.1.29 source-ip 172.18.1.28  
 frequency 5  
ip sla schedule 1 life forever start-time now  

Статистика:

R28#show ip sla statistics  
IPSLAs Latest Operation Statistics  

IPSLA operation id: 1  
        Latest RTT: 1 milliseconds  
Latest operation start time: 11:39:45 UTC Tue Aug 5 2025  
Latest operation return code: OK  
Number of successes: 513  
Number of failures: 1  
Operation time to live: Forever  

Маршрут по-умолчанию для офиса Лабытнанги:

ip default-gateway 192.168.1.9





