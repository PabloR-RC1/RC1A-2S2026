# Guía de Configuración: HSRP + OSPF con Subinterfaces

## 1. Descripción General

Este laboratorio implementa:

- Redundancia de gateway usando HSRP
- Enrutamiento dinámico usando OSPF
- Segmentación mediante VLANs y subinterfaces (Router-on-a-Stick)
- Enlaces punto a punto para backbone

---

## 2. Topología

- Dos routers (Router 0 y Router 1)
- Enlace punto a punto entre routers (red 10.0.0.0/30)
- Switch de acceso con VLANs
- Subinterfaces en routers para VLAN 10 y VLAN 20
- HSRP configurado en VLANs
- OSPF para intercambio de rutas

---

## 3. Direccionamiento IP

### Backbone (Punto a punto)
| Dispositivo | Interfaz | IP |
|------------|---------|----|
| Router 0   | G0/0    | 10.0.0.1/30 |
| Router 1   | G0/0    | 10.0.0.2/30 |

### VLAN 10
| Dispositivo | IP |
|------------|----|
| Router 0   | 192.168.10.2 |
| Router 1   | 192.168.10.3 |
| Gateway virtual | 192.168.10.1 |

### VLAN 20
| Dispositivo | IP |
|------------|----|
| Router 0   | 124.172.20.2 |
| Router 1   | 124.172.20.3 |
| Gateway virtual | 124.172.20.1 |

---

## 4. Configuración de Interfaces

### Router 0

```bash
interface g0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown

interface g0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.2 255.255.255.0

interface g0/1.20
 encapsulation dot1Q 20
 ip address 124.172.20.2 255.255.255.0
````

---

### Router 1

```bash
interface g0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown

interface g0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.3 255.255.255.0

interface g0/1.20
 encapsulation dot1Q 20
 ip address 124.172.20.3 255.255.255.0
```

---

## 5. Configuración HSRP

### VLAN 10

#### Router 0

```bash
interface g0/1.10
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt
```

#### Router 1

```bash
interface g0/1.10
 standby 10 ip 192.168.10.1
 standby 10 priority 100
 standby 10 preempt
```

---

### VLAN 20

#### Router 0

```bash
interface g0/1.20
 standby 20 ip 124.172.20.1
 standby 20 priority 110
 standby 20 preempt
```

#### Router 1

```bash
interface g0/1.20
 standby 20 ip 124.172.20.1
 standby 20 priority 100
 standby 20 preempt
```

---

## 6. Configuración OSPF

### Router 0

```bash
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 124.172.20.0 0.0.0.255 area 0
 passive-interface g0/1.10
 passive-interface g0/1.20
```

---

### Router 1

```bash
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 124.172.20.0 0.0.0.255 area 0
 passive-interface g0/1.10
 passive-interface g0/1.20
```

---

## 7. Verificación

### HSRP

```bash
show standby brief
```

Debe mostrar:

* Un router en estado Active
* Otro en Standby
* IP virtual configurada

---

### OSPF

```bash
show ip ospf neighbor
```

Debe mostrar:

* Router vecino en estado FULL

---

### Tabla de rutas

```bash
show ip route
```

Debe mostrar rutas con:

* C (connected)
* O (OSPF)

---

### Prueba de conectividad

Desde un PC:

```bash
ping 192.168.10.1
ping 124.172.20.1
```

---

## 8. Prueba de Failover

1. Apagar interfaz del router activo:

```bash
interface g0/1
 shutdown
```

2. Verificar que:

* El otro router toma el rol Active
* El gateway sigue respondiendo

---

## 9. Consideraciones

* HSRP proporciona redundancia de gateway
* OSPF permite el intercambio dinámico de rutas
* Ambos protocolos trabajan de forma complementaria
* Las interfaces LAN deben ser configuradas como passive en OSPF

---

## 10. Conclusión

La implementación permite:

* Alta disponibilidad mediante HSRP
* Enrutamiento dinámico mediante OSPF
* Segmentación de red mediante VLANs
* Tolerancia a fallos en el gateway

---
