# **Manual de Laboratorio: Configuración Básica de Switch Cisco**

**Dispositivo:** Cisco Catalyst Switch 2960

## **Descripción General**

Este documento detalla los comandos fundamentales del sistema operativo Cisco IOS para la configuración inicial de un switch.

El objetivo es establecer la identidad del dispositivo, aplicar medidas básicas de seguridad y asegurar la persistencia de la configuración.

---

## **1. Modos de Operación y Acceso**

El sistema operativo presenta una jerarquía de modos. Es necesario escalar privilegios para realizar cambios en el dispositivo.

### **Acceso al Modo Privilegiado**

El switch inicia en **Modo Usuario** (identificado por el símbolo `>`), el cual solo permite visualización básica. Para ejecutar comandos de configuración y diagnóstico avanzado, se debe ingresar al **Modo Privilegiado**.

**Comando:**

```bash
Switch> enable
Switch#
```

*Nota: El cambio del símbolo `>` a `#` indica que se han obtenido privilegios de administración.*

### **Acceso al Modo de Configuración Global**

Este modo permite modificar la configuración activa del dispositivo.

**Comando:**

```bash
Switch# configure terminal
Switch(config)#
```

---

## **2. Identidad y Gestión del Dispositivo**

### **Asignación de Nombre de Host (Hostname)**

Es indispensable identificar cada dispositivo en la red para facilitar su administración y monitoreo.

**Comando:**

```bash
Switch(config)# hostname NuevoNombre
NuevoNombre(config)#
```

*El prompt del sistema se actualizará inmediatamente reflejando el nuevo nombre asignado.*

Es correcto. Para fines educativos es excelente mostrar la diferencia entre **texto plano** y **cifrado**.

Los dos comandos son:

1. `enable password` (Obsoleto/Inseguro): Guarda la contraseña en texto legible.
2. `enable secret` (Estándar/Seguro): Guarda la contraseña cifrada (hash MD5).

El comando para **verlas** y compararlas es `show running-config`.

Aquí tienes la sección actualizada para tu `README.md`. Agrégala justo después del punto **3. Seguridad del Dispositivo** o reemplaza esa sección con esta versión más completa que incluye la comparativa y el comando de encriptación de servicio.

---

```markdown
## 3. Seguridad del Dispositivo y Tipos de Contraseña

Existen dos métodos para asegurar el modo privilegiado. Es fundamental comprender la diferencia de seguridad entre ambos.

### a. Método Inseguro (Texto Plano)
El comando `enable password` asigna una contraseña, pero esta se almacena sin cifrar en el archivo de configuración. Cualquier persona que pueda ver la configuración podrá leer la contraseña.

**Comando:**
```bash
Switch(config)# enable password CiscoFacil
```

### **b. Método Seguro (Cifrado)**

El comando `enable secret` asigna una contraseña y le aplica un cifrado fuerte (MD5) inmediatamente. Aunque alguien vea la configuración, no podrá descifrar cuál es la clave real.

**Comando:**

```bash
Switch(config)# enable secret CiscoSeguro
```

*Nota: Si se configuran ambas, `enable secret` tiene prioridad y sobrescribe a `enable password`.*

### **c. Cómo ver las contraseñas (Auditoría)**

Para verificar cómo están guardadas las contraseñas en la memoria RAM, utilizamos el comando de visualización.

**Comando:**

```bash
Switch# show running-config
```

**Lo que verás en pantalla:**

- `enable secret 5 $1$mERr$hx5rVt7rPNoS4wqbXHX7` (La contraseña "CiscoSeguro" es ilegible).
- `enable password CiscoFacil` (La contraseña es totalmente visible y vulnerable).

### **Salida de Modos**

Para regresar un nivel en la jerarquía de comandos (de Configuración Global a Modo Privilegiado).

**Comando:**

```bash
NuevoNombre(config)# exit
NuevoNombre#
```

---

## **4. Persistencia de la Configuración**

Los cambios realizados se almacenan en la memoria RAM (`running-config`), la cual es volátil. Para evitar la pérdida de datos ante un reinicio o corte de energía, se debe guardar la configuración en la memoria NVRAM (`startup-config`).

**Comando:**

```bash
NuevoNombre# write memory
```

*Salida esperada:* `Building configuration... [OK]`

---

## **5. Verificación y Diagnóstico**

### **Verificación de Configuración en Ejecución**

Muestra la configuración actual que reside en la memoria RAM.

**Comando:**

```bash
NuevoNombre# show running-config
```

### **Verificación de Configuración de Inicio**

Muestra la configuración guardada en la NVRAM, la cual se cargará en el próximo reinicio.

**Comando:**

```bash
NuevoNombre# show startup-config
```

### **Estado de las Interfaces**

Proporciona un resumen del estado físico y lógico de los puertos del switch. Es fundamental para verificar conectividad de capa 1 y capa 2.

**Comando:**

```bash
NuevoNombre# show interfaces status
```

**Interpretación de columnas clave:**

- **Port:** Identificador de la interfaz física.
- **Status:** Estado de la conexión (`connected` indica enlace activo).
- **Vlan:** VLAN a la que pertenece el puerto (Valor por defecto: 1).
- **Duplex/Speed:** Negociación automática de velocidad y modo de transmisión.

---

### **Resumen de Comandos**

| Acción | Comando | Modo Requerido |
| --- | --- | --- |
| Elevar privilegios | `enable` | Usuario `>` |
| Configuración global | `configure terminal` | Privilegiado `#` |
| Cambiar nombre | `hostname [Nombre]` | Global `(config)#` |
| Evitar búsqueda DNS | `do no ip domain-lookup` | Global `(config)#` |
| Contraseña cifrada | `enable secret [Pass]` | Global `(config)#` |
| Guardar cambios | `write memory` | Privilegiado `#` |
| Ver estado interfaces | `show interfaces status` | Privilegiado `#` |