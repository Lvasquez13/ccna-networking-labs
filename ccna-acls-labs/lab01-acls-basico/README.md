# 🔐 Lab 01: Implementación de ACL Extendida en Cisco IOS

## 📌 Objetivos
* Configurar direccionamiento IP IPv4 y enrutamiento estático entre dos routers Cisco.
* Crear una **ACL Extendida Nombrada** para filtrar tráfico L3/L4.
* Aplicar las mejores prácticas de Cisco: aplicar ACLs Extendidas lo más cerca del origen (`in`).
* Validar el comportamiento mediante comandos de verificación (`show ip access-lists`).

---

## 📐 Topología

![Topología de Red](topology.png)

```text
[ PC-ADMIN ] (192.168.10.10/24) 
    │
[ Switch-LAN1 ] ── (Gi0/0/0) [ R1 ] (Gi0/0/1) ─── (Gi0/0/1) [ R2 ] ── (Gi0/0/0) [ SRV-WEB ] (10.0.0.100/24)
    │
[ PC-USER  ] (192.168.10.20/24)
