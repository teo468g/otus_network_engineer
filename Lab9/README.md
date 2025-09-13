## BGP. Основы

1. Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.  
2. Настроите eBGP между провайдерами Киторн и Ламас.  
3. Настроите eBGP между Ламас и Триада.  
4. Настроите eBGP между офисом С.-Петербург и провайдером Триада.  
5. Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.  

Приведем пример настройки bgp на паре R14 и R22.  
R14# sh run | sec bgp   
router bgp 1001  
 bgp log-neighbor-changes  
 network 10.0.0.14 mask 255.255.255.255  
 network 192.168.1.0 mask 255.255.255.254  
 network 192.168.1.6 mask 255.255.255.254  
 redistribute ospf 1  
 neighbor 10.0.0.15 remote-as 1001  
 neighbor 192.168.1.1 remote-as 101  


R22# router bgp 101  
 bgp log-neighbor-changes  
 network 10.0.0.22 mask 255.255.255.255  
 network 192.168.1.0  
 neighbor 192.168.1.0 remote-as 1001  
 neighbor 192.168.1.18 remote-as 301  

BGP соседство присутствует, маршруты принимаются:  

R14#sh bgp ipv4 unicast summary  
BGP router identifier 10.0.0.14, local AS number 1001  

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd  
10.0.0.15       4         1001      72      90       76    0    0 00:53:00       10  
192.168.1.1     4          101      98     106       76    0    0 01:09:00        9
 

R14#sh bgp ipv4 unicast neighbors 192.168.1.1 routes  
BGP table version is 20, local router ID is 10.0.0.14


     Network          Next Hop            Metric LocPrf Weight Path
 *   10.0.0.18/32     192.168.1.1                            0 101 301 520 2042 i
 *   10.0.0.21/32     192.168.1.1                            0 101 301 i
 *>  10.0.0.22/32     192.168.1.1              0             0 101 i
 *   10.0.0.24/32     192.168.1.1                            0 101 301 520 i
 *   10.0.0.26/32     192.168.1.1                            0 101 301 520 i
 *   192.168.1.4/31   192.168.1.1                            0 101 301 520 2042 i
 *   192.168.1.6/31   192.168.1.1                            0 101 301 520 2042 i
 *   192.168.1.12/31  192.168.1.1                            0 101 301 520 i

Total number of prefixes 9

Конфигурационные файлы  

[R14](../Lab9/R14.txt)  
[R15](../Lab9/R15.txt)  
[R18](../Lab9/R18.txt)  
[R21](../Lab9/R21.txt)  
[R22](../Lab9/R22.txt)  
[R24](../Lab9/R24.txt)  
[R26](../Lab9/R26.txt)


## Итог: успешный обмен пакетами между офисами в Москве и СПб.  
R18>ping 10.0.0.14  
Type escape sequence to abort.  
Sending 5, 100-byte ICMP Echos to 10.0.0.14, timeout is 2 seconds:  
!!!!!  
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms  

R14#ping 10.0.0.18  
Type escape sequence to abort.  
Sending 5, 100-byte ICMP Echos to 10.0.0.18, timeout is 2 seconds:  
!!!!!  
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/3/6 ms  



