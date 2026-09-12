# Configuración de Topología — Semana 8: Ruteo Estático con VLANs

Topología de dos sitios conectados por un enlace WAN punto a punto (`/30`). Cada sitio tiene su propio switch con dos VLANs separadas (PCs y Laptops), y la comunicación entre sitios se realiza mediante rutas estáticas.

---

## Routers

### Router0 — Sitio A (Izquierda)

Enlace WAN con una subred `/30` hacia Router1. El puerto hacia el switch opera en modo trunk mediante subinterfaces, una por cada VLAN local. Las rutas estáticas apuntan al otro extremo del enlace WAN para alcanzar las redes del Sitio B.

```bash
enable
configure terminal
hostname Router0

! Enlace WAN punto a punto hacia Router1
interface FastEthernet0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown
 exit

! Puerto troncal físico hacia Switch0
interface FastEthernet1/0
 no shutdown
 exit

! Subinterfaz para VLAN 10 — PCs del Sitio A
interface FastEthernet1/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

! Subinterfaz para VLAN 20 — Laptops del Sitio A
interface FastEthernet1/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit

! Rutas estáticas hacia las redes del Sitio B
ip route 192.168.30.0 255.255.255.0 10.0.0.2
ip route 192.168.40.0 255.255.255.0 10.0.0.2
```

---

### Router1 — Sitio B (Derecha)

Misma lógica que Router0 pero para el Sitio B. Las subinterfaces corresponden a las VLANs 30 y 40, y las rutas estáticas apuntan de vuelta al Sitio A.

```bash
enable
configure terminal
hostname Router1

! Enlace WAN punto a punto hacia Router0
interface FastEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
 exit

! Puerto troncal físico hacia Switch1
interface FastEthernet1/0
 no shutdown
 exit

! Subinterfaz para VLAN 30 — PCs del Sitio B
interface FastEthernet1/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 exit

! Subinterfaz para VLAN 40 — Laptops del Sitio B
interface FastEthernet1/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
 exit

! Rutas estáticas hacia las redes del Sitio A
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.20.0 255.255.255.0 10.0.0.1
```

---

## Switches

### Switch0 — Sitio A (Izquierda)

Maneja VLAN 10 (PCs) y VLAN 20 (Laptops). El puerto hacia el router se configura en modo trunk para transportar ambas VLANs etiquetadas. Los puertos de los equipos finales van en modo acceso, uno por VLAN.

```bash
enable
configure terminal
hostname Switch0

! Definición de VLANs
vlan 10
 name PCs_SitioA
vlan 20
 name Laptops_SitioA
exit

! Puerto troncal hacia Router0 (Fa0/1)
interface FastEthernet0/1
 switchport mode trunk
 no shutdown
 exit

! Puerto de acceso — PC0 (VLAN 10)
interface FastEthernet2/1
 switchport mode access
 switchport access vlan 10
 no shutdown
 exit

! Puerto de acceso — Laptop0 (VLAN 20)
interface FastEthernet3/1
 switchport mode access
 switchport access vlan 20
 no shutdown
 exit
```

---

### Switch1 — Sitio B (Derecha)

Maneja VLAN 30 (PCs) y VLAN 40 (Laptops). Configuración espejo a Switch0 pero para el Sitio B.

```bash
enable
configure terminal
hostname Switch1

! Definición de VLANs
vlan 30
 name PCs_SitioB
vlan 40
 name Laptops_SitioB
exit

! Puerto troncal hacia Router1 (Fa0/1)
interface FastEthernet0/1
 switchport mode trunk
 no shutdown
 exit

! Puerto de acceso — PC1 (VLAN 30)
interface FastEthernet2/1
 switchport mode access
 switchport access vlan 30
 no shutdown
 exit

! Puerto de acceso — Laptop1 (VLAN 40)
interface FastEthernet3/1
 switchport mode access
 switchport access vlan 40
 no shutdown
 exit
```

---

## Tabla de direccionamiento

| Dispositivo     | Interfaz         | Dirección IP      | Máscara           | Gateway          |
|-----------------|------------------|-------------------|-------------------|------------------|
| Router0         | Fa0/0            | 10.0.0.1          | 255.255.255.252   | —                |
| Router0         | Fa1/0.10         | 192.168.10.1      | 255.255.255.0     | —                |
| Router0         | Fa1/0.20         | 192.168.20.1      | 255.255.255.0     | —                |
| Router1         | Fa0/0            | 10.0.0.2          | 255.255.255.252   | —                |
| Router1         | Fa1/0.30         | 192.168.30.1      | 255.255.255.0     | —                |
| Router1         | Fa1/0.40         | 192.168.40.1      | 255.255.255.0     | —                |
| PC0 (Sitio A)   | —                | 192.168.10.x      | 255.255.255.0     | 192.168.10.1     |
| Laptop0 (Sitio A) | —              | 192.168.20.x      | 255.255.255.0     | 192.168.20.1     |
| PC1 (Sitio B)   | —                | 192.168.30.x      | 255.255.255.0     | 192.168.30.1     |
| Laptop1 (Sitio B) | —              | 192.168.40.x      | 255.255.255.0     | 192.168.40.1     |
