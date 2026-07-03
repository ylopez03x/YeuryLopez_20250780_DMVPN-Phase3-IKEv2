# DMVPN Fase 3 — IKEv2 con OSPF
**Autor:** Yeury Lopez | **Matrícula:** 20250780 | **Repositorio:** YeuryLopez_20250780_DMVPN-Phase3-IKEv2

---

## Video de demostración

[![Ver en YouTube](https://img.shields.io/badge/YouTube-Ver%20Video-red?logo=youtube)](https://youtu.be/462RfNMWNbk)

**Enlace directo:** https://youtu.be/462RfNMWNbk

---

## 1. Objetivo
Configurar una VPN Hub and Spoke DMVPN Fase 3 con IKEv2 y OSPF. En la Fase 3, el Hub usa `ip nhrp redirect` y los Spokes usan `ip nhrp shortcut` para instalar rutas dinámicas directas entre Spokes sin pasar por el Hub. IKEv2 reemplaza a IKEv1 como mecanismo de negociación.

---

## 2. Topología


> <img width="940" height="589" alt="image" src="https://github.com/user-attachments/assets/33e4a6d6-8c70-4070-903e-f996efdcc2ee" />

---

## 3. Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Rol |
|---|---|---|---|
| HUB | Fa0/0 | 192.168.7.1/24 | LAN |
| HUB | Fa1/0 | 10.7.82.1/30 | WAN → ISP |
| HUB | Tunnel0 | 172.16.78.1/24 | DMVPN NHS |
| ISP | Fa0/0 | 10.7.82.2/30 | WAN → HUB |
| ISP | Fa3/0 | 10.7.83.1/30 | WAN → SPOKE1 |
| ISP | Fa1/0 | 10.7.84.1/30 | WAN → SPOKE2 |
| SPOKE1 | Fa0/0 | 10.7.83.2/30 | WAN → ISP |
| SPOKE1 | Tunnel0 | 172.16.78.2/24 | DMVPN |
| SPOKE2 | Fa0/0 | 10.7.84.2/30 | WAN → ISP |
| SPOKE2 | Tunnel0 | 172.16.78.3/24 | DMVPN |

---

## 4. Parámetros VPN

| Parámetro | Valor |
|---|---|
| Tipo | DMVPN Fase 3 IKEv2 |
| Protocolo Túnel | mGRE (Multipoint GRE) |
| NHRP Network-ID | 780 |
| Comando Hub | ip nhrp redirect |
| Comando Spoke | ip nhrp shortcut |
| Enrutamiento Dinámico | OSPF Proceso 1 — Area 0 |
| IKEv2 — Cifrado | AES-CBC-128 |
| IKEv2 — Integridad | SHA-1 |
| Pre-Shared Key | Yeury0780 |

---

## 5. Configuración

### 5.1 Configuración HUB

> 📸 **SCREENSHOT:** Insertar captura del `show running-config` del HUB mostrando ikev2, Tunnel0 con `ip nhrp redirect` y OSPF

```
interface Tunnel0
 ip address 172.16.78.1 255.255.255.0
 no ip redirects
 ip nhrp map multicast dynamic
 ip nhrp network-id 780
 ip nhrp redirect
 tunnel source FastEthernet1/0
 tunnel mode gre multipoint
 tunnel protection ipsec profile DMVPN-PROFILE
 ip ospf network point-to-multipoint
 ip ospf priority 255
```

### 5.2 Configuración SPOKE1

> 📸 **SCREENSHOT:** Insertar captura del `show running-config` del SPOKE1 mostrando `ip nhrp shortcut`

```
interface Tunnel0
 ip address 172.16.78.2 255.255.255.0
 ip nhrp map 172.16.78.1 10.7.82.1
 ip nhrp map multicast 10.7.82.1
 ip nhrp network-id 780
 ip nhrp nhs 172.16.78.1
 ip nhrp shortcut
 tunnel source FastEthernet0/0
 tunnel mode gre multipoint
 ip ospf network point-to-multipoint
 ip ospf priority 0
```

---

## 6. Verificación y Funcionamiento

### 6.1 Estado DMVPN en HUB

Ejecutar en HUB:
```
show dmvpn
```
> <img width="488" height="447" alt="image" src="https://github.com/user-attachments/assets/14a39a99-b604-4166-a2fb-cc5aa2d8042f" />


### 6.2 Estado IKEv2 SA en HUB

Ejecutar en HUB:
```
show crypto ikev2 sa
```
> <img width="940" height="286" alt="image" src="https://github.com/user-attachments/assets/fe28033e-96d4-4b20-aef6-46800e4d348f" />


### 6.3 Estado OSPF en HUB

Ejecutar en HUB:
```
show ip ospf neighbor
```
> <img width="940" height="134" alt="image" src="https://github.com/user-attachments/assets/54d9746a-a2e1-4ebe-a5fe-9c409300c199" />


### 6.4 Tabla de rutas en HUB

Ejecutar en HUB:
```
show ip route
```
> <img width="940" height="458" alt="image" src="https://github.com/user-attachments/assets/7e14cdd8-2ce4-4722-94f0-c8e86c810f8e" />


### 6.5 Demostración de conectividad

Ejecutar en Linux-SP1:
```
ping -c 4 192.168.7.2
ping -c 4 192.168.78.2
```
> <img width="940" height="212" alt="image" src="https://github.com/user-attachments/assets/3e2d1cbd-ca28-4adb-8993-58b7106efc1e" />


---
