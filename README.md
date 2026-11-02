# Laboratorio SOC casero

Laboratorio SOC desplegado sobre **KVM** en un único portátil: **Wazuh 4.14** como SIEM, **OPNsense** como cortafuegos entre tres redes aisladas y **Suricata** como IDS de red. Dos víctimas monitorizadas (Windows 10 con Sysmon y Ubuntu Server) y una máquina atacante (Kali) en una red separada, de modo que todo el tráfico ofensivo atraviesa el cortafuegos.

📄 **Informe completo (PDF):** en la sección [Releases](../../releases/latest) · fuente LaTeX en [`informe/`](informe/)

## TL;DR

| # | Escenario | MITRE ATT&CK | Fuente | Detectado |
|---|---|---|---|---|
| 1 | Fuerza bruta SSH contra Windows | [T1110.001](https://attack.mitre.org/techniques/T1110/001/) | Wazuh (registro de seguridad) | ✅ |
| 2 | Respuesta HTTP con `uid=0(root)` | — | Suricata → Wazuh | ✅ |
| 3 | Escaneo de puertos desde Kali | [T1046](https://attack.mitre.org/techniques/T1046/) | Suricata | ⏳ |

**Hallazgos principales**

- La política de bloqueo de cuenta de Windows corta la fuerza bruta, pero puede usarse para denegar el servicio a un usuario legítimo.
- OPNsense registra en UTC y el resto de fuentes en hora local: sin unificar la zona horaria, correlacionar Suricata y Wazuh es propenso a errores.

## Arquitectura

![Topología](img/topologia-fisica.png)

| Red | Subred | Uso |
|---|---|---|
| `default` | 192.168.122.0/24 | WAN de OPNsense (NAT del host) |
| `soc-lab` | 10.10.10.0/24 | SIEM y víctimas |
| `attack` | 10.10.20.0/24 | Atacante (Kali) |

| VM | IP | Rol |
|---|---|---|
| opnsense | 10.10.10.1 / 10.10.20.1 | Router, firewall, DHCP, Suricata |
| wazuh | 10.10.10.10 | SIEM (indexer, manager, dashboard) |
| windows10 | 10.10.10.130 | Víctima Windows + Sysmon + agente |
| ubuntu-victima | 10.10.10.20 | Víctima Linux + agente |
| kali | 10.10.20.10 | Atacante |

**Por qué así:** la red del atacante está separada para que su tráfico pase por OPNsense y Suricata; Suricata va dentro de OPNsense en lugar de Security Onion por falta de RAM en el host. Más detalle en el capítulo 2 del informe.

## Escenarios

Cada escenario sigue la misma estructura: **hipótesis → ejecución → telemetría en bruto → regla/consulta → cronología → respuesta → huecos de detección**.

1. **Fuerza bruta SSH (T1110.001).** Intentos fallidos desde `10.10.10.254` contra OpenSSH en Windows. Wazuh dispara 60122 (fallo de login) y 60115 (bloqueo de cuenta, nivel 9).
2. **Detección de red con Suricata.** La víctima recibe una respuesta HTTP con `uid=0(root)`; Suricata dispara el SID 2100498 y la alerta llega a Wazuh a través del agente de OPNsense (regla 86601).
3. **Escaneo de puertos desde Kali (T1046).** *En curso.*

## Stack

KVM/libvirt · OPNsense 26.7 · Suricata · Wazuh 4.14 · Sysmon (config. SwiftOnSecurity) · Windows 10 22H2 · Ubuntu Server 26.04 · Kali Linux

## Estructura del repositorio

```
.
├── README.md            ← este resumen
├── informe/             ← write-up completo en LaTeX (main.tex → main.pdf)
│   └── secciones/
├── img/                 ← capturas (compartidas por README e informe)
├── configs/             ← configuraciones para reproducir el laboratorio
└── topologiaSOC.pkt     ← topología en Packet Tracer
```

Compilar el informe:

```bash
cd informe && latexmk -pdf main.tex
```

## Referencias

- [Wazuh documentation](https://documentation.wazuh.com/current/)
- [OPNsense documentation](https://docs.opnsense.org/)
- [Suricata User Guide](https://docs.suricata.io/)
- [SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [NIST SP 800-61r2 – Computer Security Incident Handling Guide](https://doi.org/10.6028/NIST.SP.800-61r2)
