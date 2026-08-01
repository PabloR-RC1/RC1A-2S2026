# Manual Clase 2: Operaciones Básicas en Cisco Packet Tracer

### 1. Despliegue de Nodos en la Topología

La arquitectura de cualquier red en el simulador parte de la correcta selección de los equipos. En la interfaz inferior izquierda del simulador, el hardware se clasifica en dos familias principales:

* **End Devices (Dispositivos Finales):** Estaciones de trabajo donde se origina o termina el tráfico de red (ej. PC, Laptop, Server).
* **Network Devices (Dispositivos de Red):** Equipos intermedios encargados de la conmutación y enrutamiento de los paquetes:
* **Switches:** Dispositivos de conmutación de Capa 2 (ej. modelo 2960).
* **Routers:** Dispositivos de enrutamiento de Capa 3 (ej. modelos 4331 o 1941).

---

### 2. Estándares de Interconexión Física (Capa 1)

Para establecer los enlaces físicos, se debe acceder a la sección Connections (representada por el icono de un rayo). La selección del medio de transmisión no es arbitraria y obedece a las reglas de interoperabilidad del Modelo OSI:

**Cable Cruzado (Copper Crossover)**

* Norma Técnica: Se emplea para interconectar dispositivos que procesan información en la misma capa del modelo OSI.
* Escenarios de Conexión: PC a PC, Switch a Switch, Router a Router, PC a Router (ambos procesan Capa 3).

**Cable Directo (Copper Straight-Through)**

* Norma Técnica: Se emplea para interconectar dispositivos que operan en distintas capas del modelo OSI.
* Escenarios de Conexión: PC a Switch, Laptop a Switch, Switch a Router.

**Asignación Automática (Automatically Choose Connection Type)**

* Norma Técnica: Herramienta (icono de rayo naranja) que deduce y aplica el cable correcto basándose en los puertos disponibles.
* Escenarios de Conexión: Se recomienda su uso para agilizar pruebas de concepto (PoC) iniciales.

---

### 3. Aprovisionamiento Lógico y Direccionamiento IP

Para que los dispositivos finales puedan participar en la red, es imperativo asignarles un identificador lógico. El procedimiento para la asignación de una IP estática es el siguiente:

1. Seleccionar el dispositivo final (PC o Laptop) en la topología.
2. Navegar a la pestaña **Desktop** (entorno de escritorio del equipo).
3. Ingresar al módulo **IP Configuration**.
4. Asegurarse de que la opción **Static** esté marcada y completar los siguientes parámetros:
* **IPv4 Address:** Dirección lógica del nodo (ej. `192.168.1.10`).
* **Subnet Mask:** Máscara de subred correspondiente a la arquitectura diseñada (ej. `255.255.255.0`).
* **Default Gateway:** Dirección IP de la interfaz del Router que servirá como puerta de enlace hacia otras redes.



---

### 4. Auditoría de Conectividad y Diagnóstico (Troubleshooting)

Una vez establecida la capa física y lógica, se debe validar la comunicación de extremo a extremo (End-to-End) utilizando el protocolo ICMP:

1. Acceder al dispositivo de origen.
2. Dirigirse a la pestaña **Desktop** y ejecutar la herramienta **Command Prompt** (Consola de comandos).
3. Ingresar el comando `ping` seguido de la dirección IP del nodo de destino:
```bash
ping 192.168.1.20

```


4. **Análisis de Resultados:**
* **Tráfico exitoso:** El sistema reportará *`Reply from [IP de destino]...`*, confirmando la bidireccionalidad del enlace.
* **Fallo de comunicación:** El sistema reportará *`Request timed out`* (Tiempo de espera agotado) o *`Destination host unreachable`*. Ante este escenario, el estudiante deberá auditar el cableado (Paso 2) y el direccionamiento (Paso 3) antes de escalar la duda.
