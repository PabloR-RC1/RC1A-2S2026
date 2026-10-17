# Configuración

## Switch izquierdo

```bash
enable
configure terminal
vlan 10
name VLAN10
vlan 20
name VLAN20

interface fa0/1
switchport mode access
switchport access vlan 10

interface fa0/2
switchport mode access
switchport access vlan 20

interface g0/1
switchport mode trunk
switchport trunk allowed vlan 10,20
```

## Router izquierdo

```bash
enable
configure terminal

interface g0/0
ip address 10.0.0.1 255.255.255.252
no shutdown

interface g0/1
no shutdown

interface g0/1.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface g0/1.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
```

## Switch derecho

```bash
enable
configure terminal
vlan 30
name VLAN30
vlan 40
name VLAN40

interface fa0/1
switchport mode access
switchport access vlan 30

interface fa0/2
switchport mode access
switchport access vlan 40

interface g0/1
switchport mode trunk
switchport trunk allowed vlan 30,40
```

## Router derecho

```bash
enable
configure terminal

interface g0/0
ip address 10.0.0.2 255.255.255.252
no shutdown

interface g0/1
no shutdown

interface g0/1.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0

interface g0/1.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0
```

---

# Rutas entre routers

Como son dos routers, cada uno debe saber cómo llegar a las redes del otro.

## En router izquierdo

```bash
ip route 192.168.30.0 255.255.255.0 10.0.0.2
ip route 192.168.40.0 255.255.255.0 10.0.0.2
```

## En router derecho

```bash
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.20.0 255.255.255.0 10.0.0.1
```

---

# IPs de los hosts

## Lado izquierdo

* PC0: 192.168.10.10 /24 — gateway 192.168.10.1
* PC1: 192.168.20.10 /24 — gateway 192.168.20.1

## Lado derecho

* PC2: 192.168.30.10 /24 — gateway 192.168.30.1
* Laptop0: 192.168.40.10 /24 — gateway 192.168.40.1

