# DMVPN Hub and Spoke Fase 2 con IKEv1 y EIGRP

<p align="center">
  <img src="https://img.shields.io/badge/GNS3-Lab-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/VPN-DMVPN-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Fase-2-success?style=for-the-badge" />
  <img src="https://img.shields.io/badge/IKE-IKEv1-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/IPsec-AES--256-critical?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Routing-EIGRP-purple?style=for-the-badge" />
</p>

<p align="center">
  <b>VPN Hub and Spoke punto a multipunto usando DMVPN Fase 2, IKEv1/IPsec y EIGRP</b>
</p>

---

## Datos del proyecto

| Campo | Detalle |
|---|---|
| **Autor** | Michael David Robles Fermín |
| **Matrícula** | 2025-0845 |
| **Asignatura** | Seguridad de Redes |
| **Práctica** | DMVPN 1 Hub y 2 Spokes |
| **Tipo de VPN** | DMVPN Hub and Spoke punto a multipunto |
| **Fase DMVPN** | Fase 2 |
| **Seguridad** | IKEv1/IPsec |
| **Enrutamiento dinámico** | EIGRP |
| **Video demostrativo** | https://youtu.be/W1ePrYkyxNs |
| **Repositorio** | https://github.com/iClexi/DMVP-Hub-Spoke-IKEv1-Phase-2 |

---

## Documentación técnica profesional

La documentación técnica profesional completa se encuentra en la carpeta [`docs/`](docs/).

| Archivo | Descripción |
|---|---|
| [`docs/Documentacion Tecnica Profesional.pdf`](docs/Documentacion%20Tecnica%20Profesional.pdf) | Documento técnico profesional en PDF con objetivo, topología, parámetros, evidencias, explicación y configuraciones utilizadas. |

> El README funciona como una versión resumida y visual de la documentación técnica. La documentación completa está en el PDF indicado arriba.

---

## Índice

- [Descripción general](#descripción-general)
- [Objetivo de la práctica](#objetivo-de-la-práctica)
- [Topología](#topología)
- [Direccionamiento IP](#direccionamiento-ip)
- [VLANs utilizadas](#vlans-utilizadas)
- [Parámetros de la VPN](#parámetros-de-la-vpn)
- [Explicación técnica](#explicación-técnica)
- [Archivos de configuración](#archivos-de-configuración)
- [Evidencias](#evidencias)
- [Comandos de verificación](#comandos-de-verificación)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Conclusión](#conclusión)

---

## Descripción general

Este laboratorio implementa una VPN **DMVPN Fase 2** con diseño **Hub and Spoke**. La topología está formada por un router central, dos routers remotos, un router ISP y una LAN detrás de cada peer DMVPN.

El router **R1-HUB** funciona como el Hub de la VPN. Los routers **R2-SPOKE1** y **R3-SPOKE2** funcionan como Spokes. El router **ISP** representa la red pública o Internet simulado dentro de GNS3.

La comunicación entre las LANs privadas se realiza mediante una nube **DMVPN** protegida con **IKEv1/IPsec**. El ISP solo conecta las IP WAN de los routers, pero no participa en el cifrado ni tiene configuradas las redes privadas internas.

---

## Objetivo de la práctica

Configurar y demostrar una VPN **Hub and Spoke punto a multipunto DMVPN Fase 2 con IKEv1 y enrutamiento dinámico**.

La práctica demuestra:

- Conectividad WAN entre el Hub y los Spokes a través del ISP.
- Túnel DMVPN usando **GRE multipunto**.
- Registro de Spokes en el Hub mediante **NHRP**.
- Seguridad mediante **IKEv1/IPsec**.
- Anuncio automático de redes mediante **EIGRP**.
- Comunicación entre Hub y Spokes.
- Comunicación directa **Spoke-to-Spoke**, característica clave de DMVPN Fase 2.

---

## Topología

<p align="center">
  <img src="images/01_topologia_general.png" alt="Topología general DMVPN Hub and Spoke" width="900">
</p>

<p align="center">
  <b>Figura 1.</b> Topología DMVPN con 1 Hub, 2 Spokes, router ISP, switches IOSvL2 y VPCS.
</p>

### Dispositivos utilizados

| Dispositivo | Rol |
|---|---|
| **R1-HUB** | Router central de la nube DMVPN |
| **R2-SPOKE1** | Primer router remoto |
| **R3-SPOKE2** | Segundo router remoto |
| **ISP** | Router que simula la red pública |
| **SW-HUB** | Switch IOSvL2 de la LAN del Hub |
| **SW-SPOKE1** | Switch IOSvL2 de la LAN del Spoke 1 |
| **SW-SPOKE2** | Switch IOSvL2 de la LAN del Spoke 2 |
| **PC-HUB** | VPCS de la LAN del Hub |
| **PC-SPOKE1** | VPCS de la LAN del Spoke 1 |
| **PC-SPOKE2** | VPCS de la LAN del Spoke 2 |

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

### Enlaces WAN

| Equipo | Interfaz | Dirección IP | Conectado hacia |
|---|---:|---:|---|
| ISP | Gi0/0 | 20.25.8.45/30 | R1-HUB |
| R1-HUB | Gi0/0 | 20.25.8.46/30 | ISP |
| ISP | Gi0/1 | 20.25.8.49/30 | R2-SPOKE1 |
| R2-SPOKE1 | Gi0/0 | 20.25.8.50/30 | ISP |
| ISP | Gi0/2 | 20.25.8.53/30 | R3-SPOKE2 |
| R3-SPOKE2 | Gi0/0 | 20.25.8.54/30 | ISP |

### Redes LAN

| Sitio | Gateway | PC | Red |
|---|---:|---:|---:|
| Hub | 192.168.45.1 | 192.168.45.10 | 192.168.45.0/24 |
| Spoke 1 | 192.168.84.1 | 192.168.84.10 | 192.168.84.0/24 |
| Spoke 2 | 192.168.58.1 | 192.168.58.10 | 192.168.58.0/24 |

### Red del túnel DMVPN

| Router | Interfaz | IP de túnel |
|---|---:|---:|
| R1-HUB | Tunnel0 | 172.16.45.1/24 |
| R2-SPOKE1 | Tunnel0 | 172.16.45.2/24 |
| R3-SPOKE2 | Tunnel0 | 172.16.45.3/24 |

---

## VLANs utilizadas

Los switches IOSvL2 se usaron para representar cada red local. Cada sitio tiene una VLAN propia.

| Switch | VLAN | Nombre | Uso |
|---|---:|---|---|
| SW-HUB | 45 | LAN_HUB | LAN detrás de R1-HUB |
| SW-SPOKE1 | 84 | LAN_SPOKE1 | LAN detrás de R2-SPOKE1 |
| SW-SPOKE2 | 58 | LAN_SPOKE2 | LAN detrás de R3-SPOKE2 |

---

## Parámetros de la VPN

| Parámetro | Valor |
|---|---|
| Tipo de VPN | DMVPN |
| Diseño | Hub and Spoke |
| Fase | Fase 2 |
| Modelo | Punto a multipunto |
| Router Hub | R1-HUB |
| Routers Spokes | R2-SPOKE1 y R3-SPOKE2 |
| Protocolo de seguridad | IKEv1/IPsec |
| Clave precompartida | ITLA20250845 |
| Cifrado IKEv1 | AES 256 |
| Hash IKEv1 | SHA |
| Diffie-Hellman | Grupo 5 |
| Transform-set | TS-DMVPN-IKEV1 |
| Perfil IPsec | PROF-DMVPN-IKEV1 |
| Modo IPsec | Transport |
| Túnel | GRE multipunto |
| NHRP Authentication | DMVPN45 |
| NHRP Network-ID | 45 |
| Enrutamiento dinámico | EIGRP |
| Proceso EIGRP | 45 |

---

## Explicación técnica

### Hub and Spoke

El diseño **Hub and Spoke** tiene un punto central, llamado Hub, y varios routers remotos, llamados Spokes. En este caso, **R1-HUB** funciona como centro de la nube DMVPN, mientras **R2-SPOKE1** y **R3-SPOKE2** se registran contra él.

Este diseño es común cuando una sede principal necesita conectarse con varias sucursales.

---

### Punto a multipunto

La VPN es punto a multipunto porque se utiliza una sola interfaz lógica `Tunnel0` con GRE multipunto. Esto evita crear un túnel individual para cada par de routers.

El comando principal es:

```cisco
tunnel mode gre multipoint
```

Con esto, la misma interfaz de túnel puede participar en la comunicación con múltiples peers.

---

### DMVPN Fase 2

En **DMVPN Fase 2**, los Spokes pueden comunicarse directamente entre ellos. El Hub ayuda en el registro y en la resolución inicial, pero el tráfico entre R2 y R3 puede viajar de forma directa.

En el Hub se configuraron estos comandos:

```cisco
no ip split-horizon eigrp 45
no ip next-hop-self eigrp 45
```

Estos comandos permiten que:

- R1 anuncie a un Spoke las rutas aprendidas desde otro Spoke.
- El next-hop se mantenga como el Spoke real.
- R2 pueda llegar a la LAN de R3 vía `172.16.45.3`.
- R3 pueda llegar a la LAN de R2 vía `172.16.45.2`.

Ese comportamiento es lo que diferencia la Fase 2 de un modelo donde todo el tráfico tendría que pasar por el Hub.

---

### IKEv1/IPsec

IKEv1 negocia la seguridad inicial entre los routers. IPsec protege el tráfico real que pasa por el túnel GRE.

Configuración base usada:

```cisco
crypto isakmp policy 10
 encr aes 256
 hash sha
 authentication pre-share
 group 5
 lifetime 86400

crypto isakmp key ITLA20250845 address 0.0.0.0 0.0.0.0

crypto ipsec transform-set TS-DMVPN-IKEV1 esp-aes 256 esp-sha-hmac
 mode transport

crypto ipsec profile PROF-DMVPN-IKEV1
 set transform-set TS-DMVPN-IKEV1
```

El perfil IPsec se aplica al túnel con:

```cisco
tunnel protection ipsec profile PROF-DMVPN-IKEV1
```

---

### NHRP

NHRP permite que los routers relacionen una IP de túnel con una IP WAN real.

En el Hub:

```cisco
ip nhrp map multicast dynamic
ip nhrp network-id 45
```

En los Spokes:

```cisco
ip nhrp map 172.16.45.1 20.25.8.46
ip nhrp map multicast 20.25.8.46
ip nhrp nhs 172.16.45.1
```

Esto permite que R2 y R3 sepan que el Hub tiene IP de túnel `172.16.45.1` y dirección WAN `20.25.8.46`.

---

### EIGRP

EIGRP se usa para anunciar dinámicamente las redes LAN por medio de la VPN.

```cisco
router eigrp 45
 no auto-summary
 network 172.16.45.0 0.0.0.255
 network 192.168.X.0 0.0.0.255
 passive-interface GigabitEthernet0/1
```

La red `172.16.45.0/24` permite formar vecindades sobre el túnel. La red LAN se anuncia para que las demás sedes puedan alcanzarla.

---

## Archivos de configuración

Todas las configuraciones están en la carpeta [`configs/`](configs/) y usan extensión `.cfg`.

| Dispositivo | Archivo | Descripción |
|---|---|---|
| R1-HUB | [`configs/R1-HUB.cfg`](configs/R1-HUB.cfg) | Hub DMVPN con IKEv1, IPsec, NHRP dinámico, GRE multipunto y EIGRP |
| R2-SPOKE1 | [`configs/R2-SPOKE1.cfg`](configs/R2-SPOKE1.cfg) | Spoke 1 con NHRP hacia el Hub, IKEv1, IPsec y EIGRP |
| R3-SPOKE2 | [`configs/R3-SPOKE2.cfg`](configs/R3-SPOKE2.cfg) | Spoke 2 con NHRP hacia el Hub, IKEv1, IPsec y EIGRP |
| ISP | [`configs/ISP.cfg`](configs/ISP.cfg) | Router ISP con enlaces WAN |
| SW-HUB | [`configs/SW-HUB.cfg`](configs/SW-HUB.cfg) | Switch IOSvL2 con VLAN 45 |
| SW-SPOKE1 | [`configs/SW-SPOKE1.cfg`](configs/SW-SPOKE1.cfg) | Switch IOSvL2 con VLAN 84 |
| SW-SPOKE2 | [`configs/SW-SPOKE2.cfg`](configs/SW-SPOKE2.cfg) | Switch IOSvL2 con VLAN 58 |
| PC-HUB | [`configs/PC-HUB.cfg`](configs/PC-HUB.cfg) | Configuración IP de PC-HUB |
| PC-SPOKE1 | [`configs/PC-SPOKE1.cfg`](configs/PC-SPOKE1.cfg) | Configuración IP de PC-SPOKE1 |
| PC-SPOKE2 | [`configs/PC-SPOKE2.cfg`](configs/PC-SPOKE2.cfg) | Configuración IP de PC-SPOKE2 |

---

## Evidencias

### 1. Topología general

<p align="center">
  <img src="images/01_topologia_general.png" alt="Topología general" width="900">
</p>

Esta imagen evidencia la topología completa en GNS3 con nombre, matrícula, Hub, dos Spokes, ISP, switches y PCs.

---

### 2. Ping desde PC-HUB hacia PC-SPOKE1

<p align="center">
  <img src="images/02_ping_pc_hub_a_pc_spoke1.png" alt="Ping PC-HUB a PC-SPOKE1" width="900">
</p>

La prueba muestra conectividad desde **PC-HUB** hacia **PC-SPOKE1** usando la IP `192.168.84.10`. Los primeros paquetes pueden fallar mientras NHRP e IPsec terminan de negociar, pero luego el tráfico responde correctamente.

---

### 3. Interfaces activas en R1-HUB

<p align="center">
  <img src="images/03_r1_show_ip_interface_brief.png" alt="show ip interface brief R1" width="900">
</p>

El comando confirma que R1 tiene activas sus interfaces WAN, LAN y `Tunnel0`.

---

### 4. Tabla NHRP en R1-HUB

<p align="center">
  <img src="images/04_r1_show_ip_nhrp.png" alt="show ip nhrp R1" width="900">
</p>

R1 muestra a los Spokes registrados dinámicamente:

- `172.16.45.2` asociado a `20.25.8.50`.
- `172.16.45.3` asociado a `20.25.8.54`.

---

### 5. Estado IKEv1 en R3-SPOKE2

<p align="center">
  <img src="images/05_r3_show_crypto_isakmp_sa.png" alt="show crypto isakmp sa R3" width="900">
</p>

El estado `QM_IDLE` indica que IKEv1 negoció correctamente y que las asociaciones de seguridad están activas.

---

### 6. Estado IPsec en R2-SPOKE1

<p align="center">
  <img src="images/06_r2_show_crypto_ipsec_sa_parte1.png" alt="show crypto ipsec sa parte 1" width="900">
</p>

La salida muestra paquetes encapsulados y desencapsulados, confirmando que IPsec protege tráfico por `Tunnel0`.

---

### 7. Detalle de SAs IPsec

<p align="center">
  <img src="images/07_r2_show_crypto_ipsec_sa_parte2.png" alt="show crypto ipsec sa parte 2" width="900">
</p>

Se observan SAs entrantes y salientes en estado activo, con transform-set ESP AES/SHA.

---

### 8. Configuración de Tunnel0 en R1-HUB

<p align="center">
  <img src="images/08_r1_show_running_config_tunnel0.png" alt="show running-config interface Tunnel0" width="900">
</p>

La evidencia muestra los comandos principales de DMVPN Fase 2: NHRP, GRE multipunto, ajustes MTU/MSS, comandos especiales de EIGRP y protección IPsec.

---

## Comandos de verificación

### Interfaces

```cisco
show ip interface brief
```

### DMVPN

```cisco
show dmvpn
show dmvpn detail
```

### NHRP

```cisco
show ip nhrp
```

### IKEv1

```cisco
show crypto isakmp sa
```

Estado esperado:

```text
QM_IDLE
```

### IPsec

```cisco
show crypto ipsec sa
```

Validar que aumenten:

```text
#pkts encaps
#pkts decaps
```

### EIGRP

```cisco
show ip eigrp neighbors
show ip route eigrp
```

### Comprobación de Fase 2

En R2:

```cisco
show ip route 192.168.58.0
```

Debe aparecer vía:

```text
172.16.45.3
```

En R3:

```cisco
show ip route 192.168.84.0
```

Debe aparecer vía:

```text
172.16.45.2
```

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
│   └── Documentacion Tecnica Profesional.pdf
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

El laboratorio demuestra una VPN **DMVPN Hub and Spoke Fase 2** usando **IKEv1/IPsec** y **EIGRP**. Las evidencias confirman que las interfaces están activas, los Spokes se registran mediante NHRP, IKEv1 queda en estado `QM_IDLE`, IPsec protege tráfico, EIGRP aprende rutas dinámicas y existe conectividad entre las LANs.

La prueba más importante es la comunicación entre Spokes, porque confirma el comportamiento esperado de **DMVPN Fase 2**.
