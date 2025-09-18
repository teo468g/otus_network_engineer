## iBGP  

## 1.Настроите iBGP в офисом Москва между маршрутизаторами R14 и R15

R14#sh run | sec bgp  
router bgp 1001  
 bgp log-neighbor-changes 
 network 10.0.0.14 mask 255.255.255.255  
 network 192.168.1.0 mask 255.255.255.254  
 network 192.168.1.6 mask 255.255.255.254  
 redistribute ospf 1  
 neighbor 10.0.0.15 remote-as 1001  
 neighbor 192.168.1.1 remote-as 101  

R15#sh run | sec bgp  
router bgp 1001  
 bgp log-neighbor-changes  
 bgp default local-preference 500  
 network 10.0.0.15 mask 255.255.255.255  
 network 192.168.1.2 mask 255.255.255.254  
 neighbor 10.0.0.14 remote-as 1001  
 neighbor 10.0.0.14 next-hop-self  
 neighbor 172.16.7.14 remote-as 1001  
 neighbor 192.168.1.3 remote-as 301 

 R14#sh bgp su  
Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.0.15       4         1001     174     178       29    0    0 02:30:12       13
 192.168.1.1     4          101     181     175       29    0    0 02:30:12       11

 R14#sh bgp ipv4 u nei 10.0.0.15
BGP neighbor is 10.0.0.15,  remote AS 1001, internal link
  BGP version 4, remote router ID 10.0.0.15
  BGP state = Established, up for 02:31:56

  BGP соседство присутствует

 ## 2. Настроите iBGP в провайдере Триада, с использованием RR.

 Выберем в качестве отражателя маршрутов R26. На остальных роутерах указываем его в качетве нейбора. 
 
 R26#sh run | sec bgp  
router bgp 520  
 bgp log-neighbor-changes  
 bgp listen range 172.19.0.0/16 peer-group TRIADA  
 network 10.0.0.26 mask 255.255.255.255  
 network 172.19.1.0 mask 255.255.255.0  
 neighbor TRIADA peer-group  
 neighbor TRIADA remote-as 520  
 neighbor TRIADA route-reflector-client  
 neighbor TRIADA next-hop-self  
 neighbor 172.19.2.23 peer-group TRIADA  
 neighbor 172.19.3.24 peer-group TRIADA  
 neighbor 172.19.4.25 peer-group TRIADA  
 neighbor 192.168.1.6 remote-as 2042

 Проверим результат на R24. Сессия с R26 установлена, все loopback по bgp получены:

 R24#sh bgp su

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd  
172.19.3.26     4          520     263     258      103    0    0 03:41:17        7    

     Network          Next Hop            Metric LocPrf Weight Path  

 r>i 10.0.0.23/32     172.19.2.23              0    100      0 i  
 *>  10.0.0.24/32     0.0.0.0                  0         32768 i  
 r>i 10.0.0.25/32     172.19.4.25              0    100      0 i  
 r>i 10.0.0.26/32     172.19.3.26              0    100      0 i  

 ## 3. Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.  

После увеличения local-preference на R15 приоритетным выходом из AS 1001 становится R21.   
R15(config-router)# bgp default local-preference 500   

R14#  sh ip bgp  

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.0.0.14/32     0.0.0.0                  0         32768 i
 *>  10.0.0.15/32     172.16.7.15             11         32768 ?
 * i                  10.0.0.15                0    500      0 i
 *   10.0.0.18/32     192.168.1.1                            0 101 301 520 2042 i
 *>i                  192.168.1.3              0    500      0 301 520 2042 i
 *   10.0.0.21/32     192.168.1.1                            0 101 301 i
 *>i                  192.168.1.3              0    500      0 301 i
 *   10.0.0.22/32     192.168.1.1              0             0 101 i
 *>i                  192.168.1.3              0    500      0 301 101 i
 *   10.0.0.23/32     192.168.1.1                            0 101 301 520 i
 *>i                  192.168.1.3              0    500      0 301 520 i
 *   10.0.0.24/32     192.168.1.1                            0 101 301 520 i
 *>i                  192.168.1.3              0    500      0 301 520 i

R14#sh ip route 10.0.0.22  
Routing entry for 10.0.0.22/32  
  Known via "bgp 1001", distance 200, metric 0  
  Tag 301, type internal  
  Last update from 192.168.1.3 03:12:07 ago  
  Routing Descriptor Blocks:  
  * 192.168.1.3, from 10.0.0.15, 03:12:07 ago  
      Route metric is 0, traffic share count is 1  
      AS Hops 2  
      Route tag 301  
      MPLS label: none  

Итог: маршрут из R14 на connected R22 идет через R21 Ламас.

## 4. Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.  

На R18 даем команду maximum-paths 2, получаем ECMP (Equal-Cost Multi-Path).  

R18#sh ip bgp 10.0.0.24  
BGP routing table entry for 10.0.0.24/32, version 59  
Paths: (2 available, best #1, table default)  
Multipath: eBGP  
  Advertised to update-groups:  
     1  
  Refresh Epoch 2  
  520  
    192.168.1.5 from 192.168.1.5 (10.0.0.24)  
      Origin IGP, metric 0, localpref 100, valid, external, multipath, best  
      rx pathid: 0, tx pathid: 0x0  
  Refresh Epoch 1  
  520  
    192.168.1.7 from 192.168.1.7 (10.0.0.26) 
      Origin IGP, localpref 100, valid, external, multipath(oldest)  
      rx pathid: 0, tx pathid: 0  

R18#traceroute 10.0.0.24  
Type escape sequence to abort.  
Tracing the route to 10.0.0.24  
VRF info: (vrf in name/id, vrf out name/id)  
  1 192.168.1.5 25 msec  
    192.168.1.7 0 msec  
    192.168.1.5 1 msec  

    BGP все-равно выбирает один маршрут лучшим, то пакеты ходят по двум линкам. 







 





 

