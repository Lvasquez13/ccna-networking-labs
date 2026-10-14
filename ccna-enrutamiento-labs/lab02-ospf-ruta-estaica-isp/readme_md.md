# 🌐 Lab 03: Enrutamiento Híbrido OSPF y Ruta Estática por Defecto hacia el ISP

## 📌 Resumen del Laboratorio

En este laboratorio práctico para la certificación **CCNA 200-301**, se implementa un escenario híbrido de enrutamiento que combina un protocolo de estado de enlace dinámico (**OSPF Área 0**) para la conectividad interna de la empresa, junto con una **Ruta Estática por Defecto** y el comando `default-information originate` en el router de frontera para proveer acceso simulado a Internet hacia un Proveedor de Servicios (ISP).

Además, se profundiza en la diferenciación de direccionamiento **Público vs. Privado (RFC 1918)** y en la lógica de reenvío de paquetes mediante el algoritmo de Dijkstra y el criterio de *Longest Match*.

## 📐 Topología de Red

```text
[ LAN HQ ] (192.168.10.0/24)
    │
 (Gi0/0/0)
  [ R1-HQ ] ── (Gi0/0/1) ─────────── (Gi0/0/1) [ R2-BRANCH ] ── (Gi0/0/0) ── [ LAN BRANCH ] (192.168.20.0/24)
 172.16.10.1    10.1.12.1/30          10.1.12.2/30   172.16.20.1
    │ (Gi0/0/2) 
    │ 203.0.113.1/30 (WAN ISP)
    │
 (Gi0/0/0) 203.0.113.2/30
  [ ISP-ROUTER ] ── (Loopback0: 8.8.8.8/32)
```

## 📋 Tabla de Direccionamiento IP y Clasificación

| Dispositivo | Interfaz | Dirección IP | Máscara / CIDR | Tipo de IP | Función / Nota |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R1-HQ** | `Gi0/0/0` | `192.168.10.1` | `255.255.255.0` (/24) | **PRIVADA** | Gateway LAN Sede Central |
| **R1-HQ** | `Gi0/0/1` | `10.1.12.1` | `255.255.255.252` (/30) | **PRIVADA** | Enlace WAN OSPF hacia R2 |
| **R1-HQ** | `Gi0/0/2` | `203.0.113.1` | `255.255.255.252` (/30) | **PÚBLICA** | Enlace WAN hacia el Proveedor ISP |
| **R2-BRANCH**| `Gi0/0/1` | `10.1.12.2` | `255.255.255.252` (/30) | **PRIVADA** | Enlace WAN OSPF hacia R1 |
| **R2-BRANCH**| `Gi0/0/0` | `192.168.20.1` | `255.255.255.0` (/24) | **PRIVADA** | Gateway LAN Sucursal |
| **ISP-ROUTER**| `Gi0/0/0` | `203.0.113.2` | `255.255.255.252` (/30) | **PÚBLICA** | Interfaz del Proveedor de Internet |
| **ISP-ROUTER**| `Loopback0`| `8.8.8.8` | `255.255.255.255` (/32)| **PÚBLICA** | Servidor DNS Externo (Simula Internet) |

## 🎯 Objetivos del Laboratorio

1. Configurar interfaces físicas y lógicas con asignación correcta de direcciones IP públicas y privadas.
2. Implementar OSPFv2 (Área 0) en R1 y R2 para automatizar el enrutamiento interno.
3. Configurar una ruta estática por defecto en R1 apuntando al ISP y propagarla dinámicamente mediante `default-information originate`.
4. Habilitar rutas estáticas de retorno en el ISP para compensar entornos de laboratorio sin NAT activo.

---

## ⚙️ Archivos de Configuración Comentados

### 1. R1-HQ (Router de Borde y Sede Central)
```text
enable
configure terminal
hostname R1-HQ

! Configuración de interfaces LAN y WAN (Privadas y Públicas)
interface GigabitEthernet0/0/0
 description LAN-HQ
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/0/1
 description WAN-OSPF-R2
 ip address 10.1.12.1 255.255.255.252
 no shutdown

interface GigabitEthernet0/0/2
 description WAN-ISP-Publica
 ip address 203.0.113.1 255.255.255.252
 no shutdown

! Ruta estática por defecto hacia el ISP (Salida a Internet)
ip route 0.0.0.0 0.0.0.0 203.0.113.2

! Configuración de OSPF Proceso 1
router ospf 1
 router-id 1.1.1.1
 network 10.1.12.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.255.255 area 0
 default-information originate
end
write
```

### 2. R2-BRANCH (Router de Sucursal)
```text
enable
configure terminal
hostname R2-BRANCH

! Configuración de interfaces LAN y WAN OSPF
interface GigabitEthernet0/0/0
 description LAN-BRANCH
 ip address 192.168.20.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/0/1
 description WAN-OSPF-R1
 ip address 10.1.12.2 255.255.255.252
 no shutdown

! Configuración de OSPF Proceso 1
router ospf 1
 router-id 2.2.2.2
 network 10.1.12.0 0.0.0.3 area 0
 network 192.168.20.0 0.0.255.255 area 0
end
write
```

### 3. ISP-ROUTER (Proveedor de Internet Simulado)
```text
enable
configure terminal
hostname ISP-ROUTER

! Interfaz conectada a la empresa y Loopback de prueba (Públicas)
interface GigabitEthernet0/0/0
 description ENLACE-WAN-EMPRESA
 ip address 203.0.113.2 255.255.255.252
 no shutdown

interface Loopback0
 description SIMULACION-INTERNET-8.8.8.8
 ip address 8.8.8.8 255.255.255.255
 no shutdown

! Rutas estáticas de retorno para redes internas y enlaces WAN (entorno sin NAT)
ip route 192.168.10.0 255.255.255.0 203.0.113.1
ip route 192.168.20.0 255.255.255.0 203.0.113.1
ip route 10.1.12.0 255.255.255.252 203.0.113.1
end
write
```

---

## 🧪 Verificación y Comandos Clave (CCNA)

1. **Verificar vecinos OSPF activos:**
   ```text
   R1-HQ# show ip ospf neighbor
   ```
2. **Verificar la ruta por defecto inyectada por OSPF en R2:**
   ```text
   R2-BRANCH# show ip route ospf
   ```
   *(Debe aparecer como `O*E2  0.0.0.0/0 [110/1] via 10.1.12.1`)*

3. **Verificar conectividad end-to-end hacia Internet:**
   ```text
   R2-BRANCH# ping 8.8.8.8
   ```