# DMVPN Hub and Spoke IKEv1 Phase 2

---

## Información del proyecto

**Autor:** Michael David Robles Fermín  
**Matrícula:** 2025-0845 / 20250845  
**Asignatura:** Seguridad de Redes  
**Tipo de VPN:** DMVPN Hub and Spoke Fase 2  
**IKE:** IKEv1  
**Enrutamiento dinámico:** EIGRP 45  
**Repositorio:** https://github.com/iClexi/DMVP-Hub-Spoke-IKEv1-Phase-2  
**Video demostrativo:** https://youtu.be/W1ePrYkyxNs  
**Documentación técnica profesional:** `docs/Documentacion_Tecnica_Profesional_DMVPN_Fase2_IKEv1.pdf`

---

## Vista general de la topología

La práctica fue desarrollada en GNS3 usando una topología **Hub and Spoke punto a multipunto**. R1 funciona como **Hub**, R2 y R3 funcionan como **Spokes**, y el router ISP simula la red pública por donde viaja la VPN.

```text
PC-HUB --- SW-HUB --- R1-HUB --- ISP --- R2-SPOKE1 --- SW-SPOKE1 --- PC-SPOKE1
                                                             +--- R3-SPOKE2 --- SW-SPOKE2 --- PC-SPOKE2
```

![Topología general](images/01_topologia_general.png)

---

## Objetivo del laboratorio

Configurar y demostrar una VPN **DMVPN Fase 2 con IKEv1** y enrutamiento dinámico. El objetivo principal es que las LANs detrás de R1, R2 y R3 puedan comunicarse de forma segura a través de un ISP que no conoce las redes privadas.

La práctica demuestra:

- Topología Hub and Spoke con 1 Hub y 2 Spokes.
- GRE multipunto usando `Tunnel0`.
- Registro y resolución de peers mediante NHRP.
- Protección del túnel mediante IPsec con IKEv1.
- Aprendizaje dinámico de rutas mediante EIGRP.
- Comunicación directa Spoke-to-Spoke, característica de DMVPN Fase 2.

---

## Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Función |
|---|---:|---:|---|
| R1-HUB | Gi0/0 | 20.25.8.46/30 | WAN hacia ISP |
| R1-HUB | Gi0/1 | 192.168.45.1/24 | Gateway LAN Hub |
| R1-HUB | Tunnel0 | 172.16.45.1/24 | Hub DMVPN |
| R2-SPOKE1 | Gi0/0 | 20.25.8.50/30 | WAN hacia ISP |
| R2-SPOKE1 | Gi0/1 | 192.168.84.1/24 | Gateway LAN Spoke 1 |
| R2-SPOKE1 | Tunnel0 | 172.16.45.2/24 | Spoke 1 DMVPN |
| R3-SPOKE2 | Gi0/0 | 20.25.8.54/30 | WAN hacia ISP |
| R3-SPOKE2 | Gi0/1 | 192.168.58.1/24 | Gateway LAN Spoke 2 |
| R3-SPOKE2 | Tunnel0 | 172.16.45.3/24 | Spoke 2 DMVPN |
| PC-HUB | e0 | 192.168.45.10/24 | Host LAN Hub |
| PC-SPOKE1 | e0 | 192.168.84.10/24 | Host LAN Spoke 1 |
| PC-SPOKE2 | e0 | 192.168.58.10/24 | Host LAN Spoke 2 |

---

## VLANs utilizadas

| Switch | VLAN | Nombre | Uso |
|---|---:|---|---|
| SW-HUB | 45 | LAN_HUB | Conecta PC-HUB con R1-HUB |
| SW-SPOKE1 | 84 | LAN_SPOKE1 | Conecta PC-SPOKE1 con R2-SPOKE1 |
| SW-SPOKE2 | 58 | LAN_SPOKE2 | Conecta PC-SPOKE2 con R3-SPOKE2 |

---

## Parámetros principales de la VPN

| Parámetro | Valor |
|---|---|
| Tipo de VPN | DMVPN Hub and Spoke Fase 2 |
| IKE | IKEv1 / ISAKMP |
| Clave precompartida | `ITLA20250845` |
| Cifrado IKEv1 | AES 256 |
| Hash IKEv1 | SHA |
| Grupo Diffie-Hellman | Grupo 5 |
| IPsec transform-set | `TS-DMVPN-IKEV1` |
| Perfil IPsec | `PROF-DMVPN-IKEV1` |
| Modo IPsec | Transport |
| Túnel | GRE multipunto |
| NHRP network-id | 45 |
| Autenticación NHRP | `DMVPN45` |
| Enrutamiento dinámico | EIGRP 45 |

---

## Scripts de configuración

Los scripts completos están en la carpeta `scripts/`:

- `scripts/R1-HUB.txt`
- `scripts/R2-SPOKE1.txt`
- `scripts/R3-SPOKE2.txt`
- `scripts/ISP.txt`
- `scripts/SW-HUB.txt`
- `scripts/SW-SPOKE1.txt`
- `scripts/SW-SPOKE2.txt`
- `scripts/PC-HUB.txt`
- `scripts/PC-SPOKE1.txt`
- `scripts/PC-SPOKE2.txt`

---

## Comandos de verificación

```cisco
show ip interface brief
show dmvpn
show ip nhrp
show ip eigrp neighbors
show ip route eigrp
show crypto isakmp sa
show crypto ipsec sa
show running-config interface Tunnel0
```

Pruebas principales desde las PCs:

```bash
PC-HUB> ping 192.168.84.10
PC-HUB> ping 192.168.58.10
PC-SPOKE1> ping 192.168.58.10
PC-SPOKE2> ping 192.168.84.10
```

---

## Evidencias principales

La carpeta `images/` contiene las capturas usadas en la documentación profesional:

1. Topología general.
2. Ping desde PC-HUB hacia PC-SPOKE1.
3. Interfaces activas en R1-HUB.
4. Registros NHRP en R1-HUB.
5. Estado IKEv1 en R3-SPOKE2.
6. Estado IPsec en R2-SPOKE1.
7. Continuación de estado IPsec en R2-SPOKE1.
8. Configuración de Tunnel0 en R1-HUB.

---

## Resultado

La VPN quedó funcionando correctamente. Las PCs ubicadas en diferentes LANs se comunican entre sí mediante la nube DMVPN. Las rutas fueron aprendidas dinámicamente por EIGRP, los Spokes se registraron por NHRP y el tráfico fue protegido por IPsec con IKEv1.
