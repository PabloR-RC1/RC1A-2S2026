## Topologia General

- Lado izquierdo: VLANs con ruteo en `Router0` usando RIP.
- Lado derecho: VLANs con ruteo en `Router1` usando OSPF.
- Router central: `Router2` redistribuye rutas entre RIP y OSPF.

---

## 1) Configuracion de Switches (Capa 2)

### Switch1 (Lado Izquierdo - Red RIP)

```bash
enable
configure terminal
hostname Switch1

! Creacion de VLANs
vlan 10
 name PCs_RIP
vlan 20
 name Laptops_RIP
exit

! Enlace troncal hacia Router0
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit

! Puertos de acceso
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 no shutdown
exit
```

### Switch3 (Lado Derecho - Red OSPF)

```bash
enable
configure terminal
hostname Switch3

! Creacion de VLANs
vlan 30
 name PCs_OSPF
vlan 40
 name Laptops_OSPF
exit

! Enlace troncal hacia Router1
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit

! Puertos de acceso
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 30
 no shutdown
exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 40
 no shutdown
exit
```

---

## 2) Configuracion de Routers de Extremo (Router-on-a-stick + Ruteo)

### Router0 (Lado Izquierdo - RIP)

```bash
enable
configure terminal
hostname Router0

! Encender puerto fisico hacia el switch
interface GigabitEthernet6/0
 no shutdown
exit

! Subinterfaz VLAN 10
interface GigabitEthernet6/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

! Subinterfaz VLAN 20
interface GigabitEthernet6/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit

! Enlace WAN hacia Router2
interface FastEthernet0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit

! Configuracion RIP
router rip
 version 2
 network 10.0.0.0
 network 192.168.10.0
 network 192.168.20.0
 no auto-summary
exit
```

### Router1 (Lado Derecho - OSPF)

```bash
enable
configure terminal
hostname Router1

! Encender puerto fisico hacia el switch
interface GigabitEthernet6/0
 no shutdown
exit

! Subinterfaz VLAN 30
interface GigabitEthernet6/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit

! Subinterfaz VLAN 40
interface GigabitEthernet6/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
exit

! Enlace WAN hacia Router2
interface FastEthernet0/0
 ip address 11.0.0.2 255.255.255.252
 no shutdown
exit

! Configuracion OSPF
router ospf 1
 network 11.0.0.0 0.0.0.3 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
exit
```

---

## 3) Configuracion del Router Central (Redistribucion)

### Router2 (Puente entre RIP y OSPF)

```bash
enable
configure terminal
hostname Router2

! Enlace WAN izquierdo (hacia RIP)
interface FastEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit

! Enlace WAN derecho (hacia OSPF)
interface FastEthernet1/0
 ip address 11.0.0.1 255.255.255.252
 no shutdown
exit

! Inyectar rutas OSPF en RIP
router rip
 version 2
 network 10.0.0.0
 redistribute ospf 1 metric 5
exit

! Inyectar rutas RIP en OSPF
router ospf 1
 network 11.0.0.0 0.0.0.3 area 0
 redistribute rip subnets
exit
```

---

## 4) Verificacion Rapida

1. Verifica que todas las interfaces esten `up/up`.
2. Haz ping entre equipos de VLANs del lado izquierdo y derecho.
3. Revisa tablas de ruteo en los routers:

```bash
show ip route
```
### 4.1) Verifica que las rutas RIP y OSPF se esten redistribuyendo correctamente.

```bash
show ip protocols
```