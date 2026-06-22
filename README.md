# VPN-site-to-site-punto-a-punto-basado-en-enrutamiento-IPSec-IKEv1


**Juan Francisco Burgos Hiciano – 2023-1981**

📹 Video demostración: https://youtu.be/ECPA58dufoM

---

## 📌 Información General

| Campo | Detalle |
|------|--------|
| Tipo de solución | VPN Site-to-Site basada en rutas (Route-Based / VTI) |
| Protocolo | IPsec con IKEv1 (ISAKMP) |
| Topología | Hub de tránsito (IOU1) entre Site A (IOU2) y Site B (IOU3) |
| Mecanismo de túnel | Tunnel Interface (VTI) con IPsec Profile |
| Cifrado Fase 1 / Fase 2 | AES-256 / AES-256 con SHA-256 |
| Fecha | 19 de junio de 2026 |

---

## 🌐 1. Introducción

Este proyecto implementa una **VPN Site-to-Site basada en enrutamiento (Route-Based VPN)** utilizando interfaces de túnel virtual (VTI) protegidas con **IPsec e IKEv1**.

El objetivo es permitir comunicación segura entre dos redes LAN remotas (Site A y Site B) a través de un ISP simulado (IOU1), garantizando confidencialidad, integridad y autenticación del tráfico.

---

## 🖼️ Topología

![Topología VPN](https://github.com/jburgoshiciano-source/VPN-site-to-site-punto-a-punto-basado-en-pol-ticas-IKEv1/blob/d23e0d65d52c49780f8bd311b8fe0ae018c5a713/vpn.png)

---

## 🧱 2. Topología de Red

- **IOU1 (ISP):** Router de tránsito entre ambos sitios.
- **IOU2 (Site A):** Router de borde con LAN 192.168.1.0/24.
- **IOU3 (Site B):** Router de borde con LAN 192.168.2.0/24.
- **VPCS:** Hosts finales para verificar conectividad extremo a extremo.

📌 IOU1 no participa en el túnel VPN.

---

## 🗺️ 3. Esquema de Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Función |
|------------|----------|--------------|----------|
| IOU1 | e0/0 | 200.0.0.2/30 | WAN hacia Site A |
| IOU1 | e0/1 | 200.0.0.6/30 | WAN hacia Site B |
| IOU2 | e0/0 | 200.0.0.1/30 | WAN hacia ISP |
| IOU2 | e0/1 | 192.168.1.1/24 | LAN Site A |
| IOU2 | Tunnel0 | 10.0.0.1/30 | VPN hacia Site B |
| IOU3 | e0/1 | 200.0.0.5/30 | WAN hacia ISP |
| IOU3 | e0/2 | 192.168.2.1/24 | LAN Site B |
| IOU3 | Tunnel0 | 10.0.0.2/30 | VPN hacia Site A |

---

## ⚙️ 4. Configuración

### 🔹 IOU1 (ISP)

```bash
enable
configure terminal

interface Ethernet0/0
 description Hacia IOU2 (Site A)
 ip address 200.0.0.2 255.255.255.252
 no shutdown

interface Ethernet0/1
 description Hacia IOU3 (Site B)
 ip address 200.0.0.6 255.255.255.252
 no shutdown

end
```

---

### 🔹 IOU2 (Site A)

```bash
enable
configure terminal

interface Ethernet0/0
 description WAN hacia ISP
 ip address 200.0.0.1 255.255.255.252
 no shutdown

interface Ethernet0/1
 description LAN Site A
 ip address 192.168.1.1 255.255.255.0
 no shutdown

ip route 0.0.0.0 0.0.0.0 200.0.0.2

crypto isakmp policy 10
 encr aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400

crypto isakmp key VPN_S3CR3T address 200.0.0.5

crypto ipsec transform-set TS_VTI esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile PROF_VTI_IKEv1
 set transform-set TS_VTI
 set pfs group14

interface Tunnel0
 description VPN hacia Site B
 ip address 10.0.0.1 255.255.255.252
 tunnel source Ethernet0/0
 tunnel destination 200.0.0.5
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile PROF_VTI_IKEv1
 no shutdown

ip route 192.168.2.0 255.255.255.0 10.0.0.2

end
```

---

### 🔹 IOU3 (Site B)

```bash
enable
configure terminal

interface Ethernet0/1
 description WAN hacia ISP
 ip address 200.0.0.5 255.255.255.252
 no shutdown

interface Ethernet0/2
 description LAN Site B
 ip address 192.168.2.1 255.255.255.0
 no shutdown

ip route 0.0.0.0 0.0.0.0 200.0.0.6

crypto isakmp policy 10
 encr aes 256
 hash sha256
 authentication pre-share
 group 14
 lifetime 86400

crypto isakmp key VPN_S3CR3T address 200.0.0.1

crypto ipsec transform-set TS_VTI esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile PROF_VTI_IKEv1
 set transform-set TS_VTI
 set pfs group14

interface Tunnel0
 description VPN hacia Site A
 ip address 10.0.0.2 255.255.255.252
 tunnel source Ethernet0/1
 tunnel destination 200.0.0.1
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile PROF_VTI_IKEv1
 no shutdown

ip route 192.168.1.0 255.255.255.0 10.0.0.1

end
```

---

## 🔍 5. Verificación

```bash
show crypto isakmp sa
show crypto ipsec sa
show interface tunnel 0
show ip route
```

### Pruebas de conectividad:
```bash
ping 192.168.2.1 source 192.168.1.1
ping 192.168.1.1 source 192.168.2.1
```

---

## 🧠 6. Conclusión

- La VPN Route-Based simplifica el manejo del tráfico cifrado mediante interfaces de túnel.
- IKEv1 permite autenticación segura mediante fases de negociación.
- El tráfico entre sitios viaja cifrado a través de IPsec sin exposición en el ISP.
- La solución es escalable y más flexible que VPN basada en políticas.

---
