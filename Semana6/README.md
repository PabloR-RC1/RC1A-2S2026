
---

## Parte 1: VTP, STP y Convergencia (STP vs Rapid-PVST)

### Topología
Tres switches en cascada. `Switch0` actúa como servidor VTP y raíz STP. `Switch1` y `Switch2` son clientes de acceso.

---

### Switch0 — VTP Server & STP Root

**Rol:** Crea y propaga las VLANs. Se configura como raíz primaria del árbol de expansión (STP).

```
enable
configure terminal

! 1. Configurar VTP como Server
vtp domain Redes1
vtp mode server

! 2. Crear las VLANs
vlan 10
 name Ventas
vlan 20
 name Compras
exit

! 3. Forzar que sea el Switch Raíz de STP
spanning-tree vlan 1,10,20 root primary

! 4. Configurar los enlaces hacia Switch1 y Switch2 como troncales
interface range FastEthernet 0/1 - 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
```

---

### Switch1 — VTP Client (Acceso Izquierda)

**Rol:** Recibe las VLANs por VTP. Da acceso a PC2 (VLAN 10 - Ventas) y PC0 (VLAN 20 - Compras).

```
enable
configure terminal

! 1. Configurar VTP como Cliente
vtp domain Redes1
vtp mode client

! 2. Configurar los enlaces entre switches como troncales
interface range FastEthernet 0/1 - 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

! 3. Asignar los puertos a las computadoras
interface FastEthernet 0/10
 switchport mode access
 switchport access vlan 10
exit

interface FastEthernet 0/11
 switchport mode access
 switchport access vlan 20
exit
```

---

### Switch2 — VTP Client (Acceso Derecha)

**Rol:** Igual que Switch1. Da acceso a PC1 (VLAN 10 - Ventas) y PC3 (VLAN 20 - Compras).

```
enable
configure terminal

! 1. Configurar VTP como Cliente
vtp domain Redes1
vtp mode client

! 2. Configurar los enlaces entre switches como troncales
interface range FastEthernet 0/1 - 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

! 3. Asignar los puertos a las computadoras
interface FastEthernet 0/10
 switchport mode access
 switchport access vlan 10
exit

interface FastEthernet 0/11
 switchport mode access
 switchport access vlan 20
exit
```

---

### Demostración de Convergencia STP → Rapid-PVST

1. Iniciar un **ping extendido** entre dos PCs de la misma VLAN.
2. **Desconectar un cable** de uplink y observar cómo el ping se recupera (convergencia STP).
3. Cambiar a **Rapid-PVST** en los tres switches y repetir el proceso para comparar la velocidad de convergencia.

Ejecutar en **cada switch**:

```
configure terminal
spanning-tree mode rapid-pvst
```

---

## Parte 2: EtherChannel (LACP, PAgP y Manual)

### Topología
Tres switches en triángulo. `SW1` es el servidor VTP y usa distintos protocolos de EtherChannel hacia cada vecino.

| Enlace | Switches | Protocolo | Modos |
|--------|----------|-----------|-------|
| SW1 ↔ SW2 | Fa0/1-2 | LACP | Active / Passive |
| SW1 ↔ SW3 | Fa0/3-4 | PAgP | Desirable / Auto |
| SW2 ↔ SW3 | Fa0/5-6 | Manual | On / On |

---

### SW1 — VTP Server, LACP Active, PAgP Desirable

```
enable
configure terminal
hostname SW1

! 1. VTP y VLANs
vtp domain Redes1
vtp mode server
vlan 10
 name ADMIN
vlan 20
 name ESTUDIANTES
exit

! 2. EtherChannel hacia SW2 (LACP) - Fa0/1 y Fa0/2
interface range FastEthernet 0/1 - 2
 channel-group 1 mode active
exit

! 3. EtherChannel hacia SW3 (PAgP) - Fa0/3 y Fa0/4
interface range FastEthernet 0/3 - 4
 channel-group 2 mode desirable
exit

! 4. Convertir los Port-Channels en Troncales
interface port-channel 1
 switchport mode trunk
exit
interface port-channel 2
 switchport mode trunk
exit

! 5. Puertos de Acceso para las PCs
interface FastEthernet 0/10
 switchport mode access
 switchport access vlan 10
exit
interface FastEthernet 0/20
 switchport mode access
 switchport access vlan 20
exit
```

---

### SW2 — VTP Client, LACP Passive, Manual ON hacia SW3

```
enable
configure terminal
hostname SW2

! 1. VTP (las VLANs llegan automáticamente desde SW1)
vtp domain Redes1
vtp mode client

! 2. EtherChannel hacia SW1 (LACP) - Fa0/1 y Fa0/2
interface range FastEthernet 0/1 - 2
 channel-group 1 mode passive
exit

! 3. EtherChannel hacia SW3 (Manual) - Fa0/5 y Fa0/6
interface range FastEthernet 0/5 - 6
 channel-group 3 mode on
exit

! 4. Convertir los Port-Channels en Troncales
interface port-channel 1
 switchport mode trunk
exit
interface port-channel 3
 switchport mode trunk
exit

! 5. Puertos de Acceso (esperar sincronización VTP)
interface FastEthernet 0/10
 switchport mode access
 switchport access vlan 10
exit
interface FastEthernet 0/20
 switchport mode access
 switchport access vlan 20
exit
```

---

### SW3 — VTP Client, PAgP Auto, Manual ON hacia SW2

```
enable
configure terminal
hostname SW3

! 1. VTP (las VLANs llegan automáticamente desde SW1)
vtp domain Redes1
vtp mode client

! 2. EtherChannel hacia SW1 (PAgP) - Fa0/3 y Fa0/4
interface range FastEthernet 0/3 - 4
 channel-group 2 mode auto
exit

! 3. EtherChannel hacia SW2 (Manual) - Fa0/5 y Fa0/6
interface range FastEthernet 0/5 - 6
 channel-group 3 mode on
exit

! 4. Convertir los Port-Channels en Troncales
interface port-channel 2
 switchport mode trunk
exit
interface port-channel 3
 switchport mode trunk
exit

! 5. Puertos de Acceso (esperar sincronización VTP)
interface FastEthernet 0/10
 switchport mode access
 switchport access vlan 10
exit
interface FastEthernet 0/20
 switchport mode access
 switchport access vlan 20
exit
```

---

### Eliminar EtherChannel Manual (Mode ON)

Para desagrupar las interfaces del canal 3 en **SW2 y SW3**:

```
enable
configure terminal
interface range FastEthernet 0/5 - 6
 no channel-group 3
exit
```

También se recomienda eliminar la interfaz lógica:

```
no interface port-channel 3
exit
```

---

### Verificación

Verificar el estado de todos los EtherChannels configurados:

```
show etherchannel summary
```
