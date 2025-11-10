## VPN. GRE. DmVPN  

### Настроить GRE между офисами Москва и С.-Петербург  
Создадим туннельный интерфейс, overlay  адрес, src и dst underlay адреса:  
R15  
interface Tunnel0 
 ip address 192.168.2.15 255.255.255.0  
 tunnel source 192.168.1.2  
 tunnel destination 192.168.1.4  
 tunnel key 111  

Шлюз по умолчанию ptp стык с R21
 Gateway of last resort is 192.168.1.3 to network 0.0.0.0  

Статический маршрут направляет пакеты на loopback адрес R18 через туннульный адрес R18
 ip route 10.0.0.18 255.255.255.255 192.168.2.18    

На R18 настройки симметричные:  

interface Tunnel0  
 ip address 192.168.2.18 255.255.255.0  
 tunnel source 192.168.1.4  
 tunnel destination 192.168.1.2  

B*    0.0.0.0/0 [20/0] via 192.168.1.7, 00:34:35  
                [20/0] via 192.168.1.5, 00:34:35  

 ip route 10.0.0.15 255.255.255.255 192.168.2.15

### Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги

R15  

interface Tunnel1  
 ip address 192.168.3.15 255.255.255.0  
 no ip redirects  
 ip nhrp map multicast dynamic  
 ip nhrp network-id 1  
 tunnel source Ethernet0/2  
 tunnel mode gre multipoint  
 tunnel key 999  

 R28

 interface Tunnel1  
 ip address 192.168.3.28 255.255.255.0  
 no ip redirects  
 ip nhrp map multicast 192.168.1.2  
 ip nhrp map 192.168.3.15 192.168.1.2  
 ip nhrp network-id 1  
 ip nhrp nhs 192.168.3.15  
 ip nhrp registration no-unique  
 tunnel source Ethernet0/0  
 tunnel mode gre multipoint  
 tunnel key 999  

  R27  

 interface Tunnel1  
 ip address 192.168.3.27 255.255.255.0  
 no ip redirects  
 ip nhrp map multicast 192.168.1.2  
 ip nhrp map 192.168.3.15 192.168.1.2 
 ip nhrp network-id 1  
 ip nhrp nhs 192.168.3.15  
 ip nhrp registration no-unique  
 tunnel source Ethernet0/0  
 tunnel mode gre multipoint  
 tunnel key 999  

 









