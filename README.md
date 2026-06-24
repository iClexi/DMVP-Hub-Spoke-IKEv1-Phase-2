# DMVPN Hub and Spoke Fase 2 con IKEv1 y EIGRP

<p align="center">
  <img src="https://img.shields.io/badge/Plataforma-GNS3-blue" />
  <img src="https://img.shields.io/badge/VPN-DMVPN-orange" />
  <img src="https://img.shields.io/badge/Fase-2-success" />
  <img src="https://img.shields.io/badge/IKE-IKEv1-red" />
  <img src="https://img.shields.io/badge/IPsec-AES--256-important" />
  <img src="https://img.shields.io/badge/Routing-EIGRP-purple" />
  <img src="https://img.shields.io/badge/Modelo-Hub--and--Spoke-darkgreen" />
</p>

---

## Información del proyecto

| Campo | Detalle |
|---|---|
| Autor | Michael David Robles Fermín |
| Matrícula | 2025-0845 |
| Asignatura | Seguridad de Redes |
| Práctica | DMVPN Hub and Spoke Fase 2 |
| VPN | DMVPN punto a multipunto |
| Seguridad | IKEv1/IPsec |
| Enrutamiento | EIGRP |
| Repositorio | https://github.com/iClexi/DMVP-Hub-Spoke-IKEv1-Phase-2 |
| Video | https://youtu.be/W1ePrYkyxNs |

---

## Documentación técnica

La documentación técnica profesional se encuentra en la carpeta [`docs/`](docs/).

| Archivo | Descripción |
|---|---|
| [`docs/Documentacion_Tecnica_Profesional_DMVPN_Fase2_IKEv1.pdf`](docs/Documentacion_Tecnica_Profesional.pdf) | Documento técnico profesional en PDF |


---

## Índice

- [Descripción general](#descripción-general)
- [Objetivo del laboratorio](#objetivo-del-laboratorio)
- [Topología implementada](#topología-implementada)
- [Direccionamiento IP](#direccionamiento-ip)
- [VLANs utilizadas](#vlans-utilizadas)
- [Parámetros usados](#parámetros-usados)
- [Explicación técnica resumida](#explicación-técnica-resumida)
- [Configs de configuración](#configs-de-configuración)
- [Evidencias](#evidencias)
- [Comandos de verificación](#comandos-de-verificación)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Conclusión](#conclusión)

---

## Descripción general

Este laboratorio implementa una VPN **DMVPN Fase 2** con diseño **Hub and Spoke**. La topología tiene un router central llamado **R1-HUB**, dos routers remotos llamados **R2-SPOKE1** y **R3-SPOKE2**, y un router **ISP** que representa la red pública.

El ISP solamente conecta las direcciones WAN. Las redes privadas no están configuradas en el ISP, por lo que la comunicación entre las LANs se logra mediante la nube DMVPN protegida con **IKEv1/IPsec**.

La práctica usa:

- **GRE multipunto**, para usar una sola interfaz de túnel con varios peers.
- **NHRP**, para resolver IP de túnel contra IP WAN.
- **IKEv1/IPsec**, para cifrar y proteger el tráfico.
- **EIGRP**, para anunciar dinámicamente las LANs.
- **DMVPN Fase 2**, para permitir comunicación directa entre Spokes.

---

## Objetivo del laboratorio

Configurar una VPN **Hub and Spoke punto a multipunto DMVPN Fase 2 con IKEv1 y enrutamiento dinámico**.

El laboratorio demuestra:

- Registro de Spokes en el Hub mediante NHRP.
- Cifrado de tráfico con IPsec.
- Negociación IKEv1 en estado `QM_IDLE`.
- Aprendizaje de rutas mediante EIGRP.
- Comunicación Hub-to-Spoke.
- Comunicación Spoke-to-Spoke, característica principal de DMVPN Fase 2.

---

## Topología implementada

<p align="center">
  <img src="images/01_topologia_general.png" alt="Topología general DMVPN" width="850">
</p>

**Figura 1. Topología general DMVPN con 1 Hub, 2 Spokes, un ISP, tres switches IOSvL2 y tres VPCS.**

### Conexiones principales

| Desde | Interfaz | Hacia | Interfaz |
|---|---:|---|---:|
| PC-HUB | e0 | SW-HUB | Gi0/1 |
| SW-HUB | Gi0/0 | R1-HUB | Gi0/1 |
| R1-HUB | Gi0/0 | ISP | Gi0/0 |
| ISP | Gi0/1 | R2-SPOKE1 | Gi0/0 |
| R2-SPOKE1 | Gi0/1 | SW-SPOKE1 | Gi0/0 |
| SW-SPOKE1 | Gi0/1 | PC-SPOKE1 | e0 |
| ISP | Gi0/2 | R3-SPOKE2 | Gi0/0 |
| R3-SPOKE2 | Gi0/1 | SW-SPOKE2 | Gi0/0 |
| SW-SPOKE2 | Gi0/1 | PC-SPOKE2 | e0 |

---

## Direccionamiento IP

### WAN

| Equipo | Interfaz | IP |
|---|---:|---:|
| ISP | Gi0/0 | 20.25.8.45/30 |
| R1-HUB | Gi0/0 | 20.25.8.46/30 |
| ISP | Gi0/1 | 20.25.8.49/30 |
| R2-SPOKE1 | Gi0/0 | 20.25.8.50/30 |
| ISP | Gi0/2 | 20.25.8.53/30 |
| R3-SPOKE2 | Gi0/0 | 20.25.8.54/30 |

### LAN

| Sitio | Gateway | PC | Red |
|---|---:|---:|---:|
| Hub | 192.168.45.1 | 192.168.45.10 | 192.168.45.0/24 |
| Spoke 1 | 192.168.84.1 | 192.168.84.10 | 192.168.84.0/24 |
| Spoke 2 | 192.168.58.1 | 192.168.58.10 | 192.168.58.0/24 |

### Túnel DMVPN

| Router | Tunnel0 |
|---|---:|
| R1-HUB | 172.16.45.1/24 |
| R2-SPOKE1 | 172.16.45.2/24 |
| R3-SPOKE2 | 172.16.45.3/24 |

---

## VLANs utilizadas

| Switch | VLAN | Nombre |
|---|---:|---|
| SW-HUB | 45 | LAN_HUB |
| SW-SPOKE1 | 84 | LAN_SPOKE1 |
| SW-SPOKE2 | 58 | LAN_SPOKE2 |

Los switches son IOSvL2 y funcionan como capa 2. No participan en el cifrado ni en el enrutamiento DMVPN.

---

## Parámetros usados

| Parámetro | Valor |
|---|---|
| Tipo de VPN | DMVPN |
| Diseño | Hub and Spoke |
| Fase | Fase 2 |
| Seguridad | IKEv1/IPsec |
| PSK | ITLA20250845 |
| Cifrado | AES 256 |
| Hash/Integridad | SHA / SHA-HMAC |
| DH Group | 5 |
| Transform-set | TS-DMVPN-IKEV1 |
| Perfil IPsec | PROF-DMVPN-IKEV1 |
| Modo IPsec | Transport |
| Túnel | GRE multipunto |
| NHRP Authentication | DMVPN45 |
| NHRP Network-ID | 45 |
| Routing dinámico | EIGRP |
| Proceso EIGRP | 45 |

---

## Explicación técnica resumida

### Hub and Spoke

El modelo **Hub and Spoke** usa un router central, R1-HUB, que funciona como punto principal de registro y control. Los routers R2 y R3 son Spokes y se registran contra el Hub para formar parte de la nube DMVPN.

### Punto a multipunto

La VPN es punto a multipunto porque se usa una sola interfaz `Tunnel0` con `tunnel mode gre multipoint`, en vez de crear un túnel punto a punto por cada par de routers.

### DMVPN Fase 2

En DMVPN Fase 2, los Spokes pueden comunicarse directamente entre ellos. El Hub ayuda con el registro y la resolución, pero el tráfico entre R2 y R3 puede viajar directamente mediante la nube DMVPN.

En R1-HUB se configuró:

```cisco
no ip split-horizon eigrp 45
no ip next-hop-self eigrp 45
```

Estos comandos permiten que R2 aprenda la red de R3 con next-hop `172.16.45.3`, y que R3 aprenda la red de R2 con next-hop `172.16.45.2`.

### IKEv1/IPsec

La seguridad se configuró con IKEv1 e IPsec. IKEv1 negocia las asociaciones de seguridad usando la clave precompartida `ITLA20250845`. IPsec protege el tráfico GRE mediante el perfil `PROF-DMVPN-IKEV1`.

### NHRP

NHRP permite mapear una IP de túnel con una IP WAN. En el Hub se aceptan registros dinámicos, mientras que en los Spokes se configura manualmente el Hub como NHS.

### EIGRP

EIGRP proceso 45 anuncia las LANs de cada sitio por el túnel DMVPN, evitando la necesidad de rutas estáticas entre las redes privadas.

---

## Configs de configuración

Todos los archivos de configuración están en la carpeta [`configs/`](configs/) y usan extensión `.cfg`.

| Dispositivo | Archivo | Descripción |
|---|---|---|
| R1-HUB | [`configs/R1-HUB.cfg`](configs/R1-HUB.cfg) | Configuración del Hub con IKEv1, IPsec, NHRP dinámico, GRE multipunto y EIGRP |
| R2-SPOKE1 | [`configs/R2-SPOKE1.cfg`](configs/R2-SPOKE1.cfg) | Configuración del Spoke 1 con mapeo NHRP hacia el Hub y EIGRP |
| R3-SPOKE2 | [`configs/R3-SPOKE2.cfg`](configs/R3-SPOKE2.cfg) | Configuración del Spoke 2 con mapeo NHRP hacia el Hub y EIGRP |
| ISP | [`configs/ISP.cfg`](configs/ISP.cfg) | Configuración de enlaces WAN del ISP |
| SW-HUB | [`configs/SW-HUB.cfg`](configs/SW-HUB.cfg) | VLAN 45 y puertos access de la LAN del Hub |
| SW-SPOKE1 | [`configs/SW-SPOKE1.cfg`](configs/SW-SPOKE1.cfg) | VLAN 84 y puertos access de la LAN del Spoke 1 |
| SW-SPOKE2 | [`configs/SW-SPOKE2.cfg`](configs/SW-SPOKE2.cfg) | VLAN 58 y puertos access de la LAN del Spoke 2 |
| PC-HUB | [`configs/PC-HUB.cfg`](configs/PC-HUB.cfg) | IP estática de PC-HUB |
| PC-SPOKE1 | [`configs/PC-SPOKE1.cfg`](configs/PC-SPOKE1.cfg) | IP estática de PC-SPOKE1 |
| PC-SPOKE2 | [`configs/PC-SPOKE2.cfg`](configs/PC-SPOKE2.cfg) | IP estática de PC-SPOKE2 |

---

## Evidencias

### 1. Topología general

<p align="center">
  <img src="images/01_topologia_general.png" alt="Topología general" width="850">
</p>

### 2. Ping desde PC-HUB hacia PC-SPOKE1

<p align="center">
  <img src="images/02_ping_pc_hub_a_pc_spoke1.png" alt="Ping PC-HUB a PC-SPOKE1" width="850">
</p>

Este ping demuestra conectividad entre la LAN del Hub y la LAN del Spoke 1. Los primeros paquetes pueden fallar mientras NHRP e IPsec terminan de negociar.

### 3. Interfaces activas en R1-HUB

<p align="center">
  <img src="images/03_r1_show_ip_interface_brief.png" alt="show ip interface brief R1" width="850">
</p>

Se observa `GigabitEthernet0/0`, `GigabitEthernet0/1` y `Tunnel0` en estado `up/up`.

### 4. Tabla NHRP en R1-HUB

<p align="center">
  <img src="images/04_r1_show_ip_nhrp.png" alt="show ip nhrp R1" width="850">
</p>

R1 muestra a R2 y R3 registrados dinámicamente en la nube DMVPN.

### 5. Estado IKEv1 en R3-SPOKE2

<p align="center">
  <img src="images/05_r3_show_crypto_isakmp_sa.png" alt="show crypto isakmp sa R3" width="850">
</p>

El estado `QM_IDLE` confirma que IKEv1 negoció correctamente.

### 6. Estado IPsec en R2-SPOKE1

<p align="center">
  <img src="images/06_r2_show_crypto_ipsec_sa_parte1.png" alt="show crypto ipsec sa parte 1" width="850">
</p>

La salida muestra paquetes encapsulados y desencapsulados por IPsec.

### 7. Detalle de SAs IPsec

<p align="center">
  <img src="images/07_r2_show_crypto_ipsec_sa_parte2.png" alt="show crypto ipsec sa parte 2" width="850">
</p>

Se observan SAs entrantes y salientes en estado activo.

### 8. Configuración de Tunnel0 en R1-HUB

<p align="center">
  <img src="images/08_r1_show_running_config_tunnel0.png" alt="show running-config Tunnel0" width="850">
</p>

La evidencia muestra los comandos principales de DMVPN Fase 2 en el Hub.

---

## Comandos de verificación

### Interfaces

```cisco
show ip interface brief
```

### DMVPN y NHRP

```cisco
show dmvpn
show ip nhrp
```

### IKEv1

```cisco
show crypto isakmp sa
```

El estado esperado es:

```text
QM_IDLE
```

### IPsec

```cisco
show crypto ipsec sa
```

Revisar que suban:

```text
#pkts encaps
#pkts decaps
```

### EIGRP

```cisco
show ip eigrp neighbors
show ip route eigrp
```

### Verificación Fase 2

En R2:

```cisco
show ip route 192.168.58.0
```

La ruta debe ir vía `172.16.45.3`.

En R3:

```cisco
show ip route 192.168.84.0
```

La ruta debe ir vía `172.16.45.2`.

### Pings finales

```bash
PC-HUB> ping 192.168.84.10
PC-HUB> ping 192.168.58.10
PC-SPOKE1> ping 192.168.58.10
PC-SPOKE2> ping 192.168.84.10
```

---

## Estructura del repositorio

```text
DMVP-Hub-Spoke-IKEv1-Phase-2/
│
├── README.md
├── Links_Video_Repositorio.txt
│
├── docs/
│   ├── Documentacion_Tecnica_Profesional_DMVPN_Fase2_IKEv1.docx
│   └── Documentacion_Tecnica_Profesional_DMVPN_Fase2_IKEv1.pdf
│
├── images/
│   ├── 01_topologia_general.png
│   ├── 02_ping_pc_hub_a_pc_spoke1.png
│   ├── 03_r1_show_ip_interface_brief.png
│   ├── 04_r1_show_ip_nhrp.png
│   ├── 05_r3_show_crypto_isakmp_sa.png
│   ├── 06_r2_show_crypto_ipsec_sa_parte1.png
│   ├── 07_r2_show_crypto_ipsec_sa_parte2.png
│   └── 08_r1_show_running_config_tunnel0.png
│
└── configs/
    ├── ISP.cfg
    ├── PC-HUB.cfg
    ├── PC-SPOKE1.cfg
    ├── PC-SPOKE2.cfg
    ├── R1-HUB.cfg
    ├── R2-SPOKE1.cfg
    ├── R3-SPOKE2.cfg
    ├── SW-HUB.cfg
    ├── SW-SPOKE1.cfg
    └── SW-SPOKE2.cfg
```

---

## Conclusión

El laboratorio demuestra una VPN **DMVPN Fase 2 Hub and Spoke** usando **IKEv1/IPsec** y **EIGRP**. Las evidencias confirman interfaces activas, registros NHRP, negociación IKEv1, SAs IPsec activas, rutas dinámicas y conectividad entre las LANs.

La prueba más importante es la comunicación entre Spokes, porque confirma el comportamiento esperado de **DMVPN Fase 2**.
