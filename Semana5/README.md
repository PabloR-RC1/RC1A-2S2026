# Manual de Configuración VTP y VLANs

Este manual describe paso a paso cómo configurar el protocolo VTP (VLAN Trunking Protocol) en una red con múltiples switches. VTP permite administrar las VLANs de forma centralizada, propagando automáticamente la configuración desde un servidor hacia los clientes.

---

## Conceptos Básicos

| Modo VTP | Descripción |
|----------|-------------|
| **Server** | Crea, modifica y elimina VLANs. Propaga la información a otros switches. |
| **Client** | Recibe y sincroniza las VLANs del servidor. No puede crear VLANs localmente. |
| **Transparent** | No participa en la sincronización VTP. Puede crear VLANs locales sin propagarlas. |

---

## 1. Configuración del Switch Server

El switch en modo **Server** es el encargado de crear las VLANs y distribuirlas a toda la red. Es el punto central de administración.

```
enable
configure terminal
```
> Ingresa al modo privilegiado y luego al modo de configuración global.

```
vtp domain Redes1
vtp mode server
```
> Define el dominio VTP como "Redes1" y establece el switch como servidor.

```
vlan 10
 name ADMIN
vlan 20
 name ESTUDIANTES
exit
```
> Crea las VLANs 10 (ADMIN) y 20 (ESTUDIANTES) que serán propagadas a los clientes.

```
interface range FastEthernet 0/1 - 3
 switchport mode trunk
exit
```
> Configura los puertos Fa0/1 al Fa0/3 como enlaces troncales para transportar múltiples VLANs.

---

## 2. Configuración del Switch Cliente 1

Este switch recibe automáticamente las VLANs del servidor y asigna los puertos de acceso a las computadoras.

```
enable
configure terminal
```

```
vtp domain Redes1
vtp mode client
```
> Se une al dominio "Redes1" como cliente. Recibirá las VLANs del servidor automáticamente.

```
interface FastEthernet 0/1
 switchport mode trunk
exit
```
> Configura el puerto Fa0/1 como troncal para conectar con el servidor.

```
interface FastEthernet 0/2
 switchport mode access
 switchport access vlan 10
exit
```
> Asigna el puerto Fa0/2 a la VLAN 10 (ADMIN) para conectar un equipo.

```
interface range FastEthernet 0/3 - 4
 switchport mode access
 switchport access vlan 20
exit
```
> Asigna los puertos Fa0/3 y Fa0/4 a la VLAN 20 (ESTUDIANTES).

---

## 3. Configuración del Switch Cliente 2

Similar al Cliente 1, este switch sincroniza las VLANs desde el servidor.

```
enable
configure terminal
```

```
vtp domain Redes1
vtp mode client
```
> Se configura como cliente del dominio "Redes1".

```
interface FastEthernet 0/2
 switchport mode trunk
exit
```
> Puerto troncal para la conexión con otros switches.

```
interface FastEthernet 0/1
 switchport mode access
 switchport access vlan 20
exit
```
> Puerto de acceso asignado a la VLAN 20.

```
interface FastEthernet 0/3
 switchport mode access
 switchport access vlan 10
exit
```
> Puerto de acceso asignado a la VLAN 10.

---

## 4. Configuración del Switch Transparente

El modo **Transparent** es útil cuando se necesita crear VLANs locales sin afectar ni ser afectado por la sincronización VTP del resto de la red.

```
enable
configure terminal
```

```
vtp domain Redes1
vtp mode transparent
```
> Se une al dominio pero en modo transparente: no sincroniza VLANs con el servidor.

```
vlan 30
 name SOPORTE
exit
```
> Crea la VLAN 30 (SOPORTE) de forma local. Esta VLAN NO será propagada a otros switches.

```
interface FastEthernet 0/3
 switchport mode trunk
exit
```
> Puerto troncal para permitir el paso del tráfico de otras VLANs.

```
interface range FastEthernet 0/1 - 2
 switchport mode access
 switchport access vlan 30
exit
```
> Puertos de acceso asignados a la VLAN 30 local.

---

## Comandos de Verificación

| Comando | Descripción |
|---------|-------------|
| `show vlan brief` | Muestra un resumen de las VLANs configuradas en el switch. |
| `show vtp status` | Muestra el modo VTP, dominio y número de revisión. |
| `show interfaces trunk` | Lista los puertos configurados como troncales. |

---

## Pruebas del Laboratorio

1. **Ping entre VLANs iguales:** Realizar ping entre equipos de la misma VLAN (ej: PC Admin1 ↔ Laptop Admin2). Debe ser exitoso.

2. **Verificar sincronización:** En cualquier switch cliente, ejecutar `show vlan brief`. Las VLANs 10 y 20 deben aparecer automáticamente.

3. **Confirmar aislamiento:** En el switch Server, ejecutar `show vlan brief`. La VLAN 30 **NO** debe existir, confirmando que el switch Transparente no propaga sus VLANs locales.
