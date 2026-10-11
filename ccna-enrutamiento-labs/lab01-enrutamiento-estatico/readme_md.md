# 🌐 Lab 02: Ruteo Estático Multisitio y Rutas Flotantes de Respaldo

[![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-blue?logo=cisco&logoColor=white)](https://www.netacad.com/courses/packet-tracer)
[![CCNA Domain](https://img.shields.io/badge/CCNA%20Domain-3.0%20IP%20Routing%20Technologies-green)](https://www.cisco.com)

## 📌 Resumen del Laboratorio
En este laboratorio práctico de la certificación **CCNA 200-301**, se implementa un esquema de enrutamiento estático redundante sobre una topología en triángulo interconectando tres routers Cisco (`R1`, `R2` y `R3`). 

El objetivo principal es asegurar la conectividad end-to-end entre sitios remotos e implementar **Rutas Estáticas Flotantes** (*Floating Static Routes*) manipulando la **Distancia Administrativa (AD)** para proporcionar tolerancia a fallos (*failover*) automática ante la caída del enlace WAN principal.

---

## 📐 Topología de Red

```text
                  [ R2 ] (Sede Regional)
                 /      \
  172.16.12.0/30 /        \ 172.16.23.0/30
                /          \
  [ R1 ] (HQ) ─────────────── [ R3 ] (Branch Office)
              172.16.13.0/30
    │                               │
[ LAN 1 ]                       [ LAN 3 ]
192.168.10.0/24                 192.168.30.0/24
(PC-HQ)                         (SRV-DATA)
```

---

## 📋 Tabla de Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Máscara / CIDR | Gateway Predeterminado |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | Gi0/0/0 | `192.168.10.1` | `255.255.255.0` (/24) | N/A |
| **R1** | Gi0/0/1 | `172.16.12.1` | `255.255.255.252` (/30) | N/A |
| **R1** | Gi0/0/2 | `172.16.13.1` | `255.255.255.252` (/30) | N/A |
| **R2** | Gi0/0/1 | `172.16.12.2` | `255.255.255.252` (/30) | N/A |
| **R2** | Gi0/0/0 | `172.16.23.1` | `255.255.255.252` (/30) | N/A |
| **R3** | Gi0/0/2 | `172.16.13.2` | `255.255.255.252` (/30) | N/A |
| **R3** | Gi0/0/0 | `172.16.23.2` | `255.255.255.252` (/30) | N/A |
| **R3** | Gi0/0/1 | `192.168.30.1` | `255.255.255.0` (/24) | N/A |
| **PC-HQ** | NIC | `192.168.10.10` | `255.255.255.0` (/24) | `192.168.10.1` |
| **SRV-DATA** | NIC | `192.168.30.100` | `255.255.255.0` (/24) | `192.168.30.1` |

---

## 🎯 Objetivos y Reglas del Laboratorio
1. Configurar direcciones IP y activar todas las interfaces en `R1`, `R2` y `R3`.
2. Establecer el enlace directo `R1 <-> R3` como la **ruta estática principal** (AD 1 por defecto) para el tráfico entre LAN 1 y LAN 3.
3. Configurar **rutas estáticas flotantes** a través de `R2` asignando una **Distancia Administrativa de 130**.
4. Validar la convergencia automática (*Failover*) simulando una caída de enlace (`shutdown`) en el tramo principal.

---

## ⚙️ Configuración Paso a Paso (Cisco IOS)

### 1. Router R1 (HQ)
```text
enable
configure terminal
hostname R1

! Interfaces IP
interface GigabitEthernet0/0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/0/1
 ip address 172.16.12.1 255.255.255.252
 no shutdown

interface GigabitEthernet0/0/2
 ip address 172.16.13.1 255.255.255.252
 no shutdown
exit

! Ruta Estática Principal (AD = 1)
ip route 192.168.30.0 255.255.255.0 172.16.13.2

! Ruta Estática Flotante de Respaldo (AD = 130)
ip route 192.168.30.0 255.255.255.0 172.16.12.2 130

end
write
```

### 2. Router R2 (Sede Transit)
```text
enable
configure terminal
hostname R2

! Interfaces IP
interface GigabitEthernet0/0/0
 ip address 172.16.23.1 255.255.255.252
 no shutdown

interface GigabitEthernet0/0/1
 ip address 172.16.12.2 255.255.255.252
 no shutdown
exit

! Rutas de Tránsito
ip route 192.168.10.0 255.255.255.0 172.16.12.1
ip route 192.168.30.0 255.255.255.0 172.16.23.2

end
write
```

### 3. Router R3 (Branch Office)
```text
enable
configure terminal
hostname R3

! Interfaces IP
interface GigabitEthernet0/0/0
 ip address 172.16.23.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/0/2
 ip address 172.16.13.2 255.255.255.252
 no shutdown
exit

! Ruta Estática Principal hacia LAN 1 (AD = 1)
ip route 192.168.10.0 255.255.255.0 172.16.13.1

! Ruta Estática Flotante de Respaldo (AD = 130)
ip route 192.168.10.0 255.255.255.0 172.16.23.1 130

end
write
```

---

## 🧪 Pruebas de Verificación y Convergencia

### 1. Estado Normal: Enlace Principal Operativo

#### 🔹 Trazado de Ruta (`tracert`) desde PC-HQ (`192.168.10.10`)
```text
C:\> tracert 192.168.30.100

Tracing route to 192.168.30.100 over a maximum of 30 hops:

  1    1 ms    1 ms    1 ms  192.168.10.1 (Gateway R1)
  2    2 ms    2 ms    2 ms  172.16.13.2 (Interfaz WAN R3)
  3    3 ms    2 ms    3 ms  192.168.30.100 (SRV-DATA)

Trace complete.
```
> **Análisis:** El tráfico toma el camino directo `R1 -> R3` a través de la red `172.16.13.0/30`.

#### 🔹 Inspección de la Tabla de Enrutamiento en R1
```text
R1# show ip route static
Codes: L - local, C - connected, S - static

S    192.168.30.0/24 [1/0] via 172.16.13.2
```
> **Nota:** La ruta flotante con AD 130 permanece oculta en la base de datos y no entra a la tabla de enrutamiento activa mientras la principal (AD 1) esté activa.

---

### 2. Prueba de Failover: Simulación de Caída de Enlace WAN

Desactivamos intencionalmente la interfaz `Gi0/0/2` en R1 para simular la ruptura de la fibra:

```text
R1(config)# interface GigabitEthernet0/0/2
R1(config-if)# shutdown
```

#### 🔹 Inspección de la Tabla de Enrutamiento tras la Caída (R1)
```text
R1# show ip route static
Codes: L - local, C - connected, S - static

S    192.168.30.0/24 [130/0] via 172.16.12.2
```
> **Resultado:** Al caer la interfaz física, la ruta principal es removida automáticamente e ingresa la **ruta flotante con AD 130**.

#### 🔹 Trazado de Ruta Redirigido (`tracert`)
```text
C:\> tracert 192.168.30.100

Tracing route to 192.168.30.100 over a maximum of 30 hops:

  1    1 ms    1 ms    1 ms  192.168.10.1 (Gateway R1)
  2    2 ms    2 ms    2 ms  172.16.12.2 (Interfaz WAN R2)
  3    4 ms    3 ms    4 ms  172.16.23.2 (Interfaz WAN R3)
  4    5 ms    4 ms    5 ms  192.168.30.100 (SRV-DATA)

Trace complete.
```
> **Conclusión:** ✅ **Failover Exitoso**. El tráfico conmuta automáticamente a través del nodo intermedio `R2`, garantizando alta disponibilidad sin intervención manual.

---

## 📂 Archivos del Laboratorio
* `lab02-rutas-estaticas.pkt`: Archivo interactivo de Cisco Packet Tracer.
* `topology.png`: Esquema visual de la topología en triángulo.
* `configs/R1.cfg`: Running-config final de R1.
* `configs/R2.cfg`: Running-config final de R2.
* `configs/R3.cfg`: Running-config final de R3.