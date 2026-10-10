# 🧪 Registro de Pruebas de Verificación (Lab 01 - ACLs)

Este documento contiene las dos pruebas de conectividad desde los endpoints y la validación en la CLI para confirmar el funcionamiento de la **ACL Extendida (`FILTRO-LAN1`)**.

---

## 🔹 Prueba 1: Acceso Permitido (PC-ADMIN ➔ SRV-WEB)

Se ejecuta un test de conectividad ICMP desde **PC-ADMIN** (`192.168.10.10`) con destino al servidor web (`10.0.0.100`).

### Salida de Consola (PC-ADMIN)

```text
C:\> ping 10.0.0.100

Pinging 10.0.0.100 with 32 bytes of data:
Reply from 10.0.0.100: bytes=32 time=1ms TTL=126
Reply from 10.0.0.100: bytes=32 time=1ms TTL=126
Reply from 10.0.0.100: bytes=32 time=1ms TTL=126
Reply from 10.0.0.100: bytes=32 time=1ms TTL=126

Ping statistics for 10.0.0.100:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 1ms, Average = 1ms

```

## 🔹 Prueba 1: Acceso Denegado (PC-USER ➔ SRV-WEB)

Se ejecuta un test de conectividad ICMP desde **PC-ADMIN** (`192.168.10.20`) con destino al servidor web (`10.0.0.100`).

### Salida de Consola (PC-ADMIN)

```text
C:\> ping 10.0.0.100

Pinging 10.0.0.100 with 32 bytes of data:
Reply from 192.168.10.1: Destination host unreachable.
Reply from 192.168.10.1: Destination host unreachable.
Reply from 192.168.10.1: Destination host unreachable.
Reply from 192.168.10.1: Destination host unreachable.

Ping statistics for 10.0.0.100:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

Resultado: ❌ ÉXITO (BLOQUEADO). El router R1 intercepta el paquete en su interfaz de entrada Gi0/0/0, lo descarta por la regla 20 (deny ip host 192.168.10.20 host 10.0.0.100) y responde con Destination host unreachable.
