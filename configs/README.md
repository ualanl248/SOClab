# Configuraciones

Ficheros de configuración del laboratorio, para que sea reproducible.

| Ruta | Qué es |
|---|---|
| `libvirt/attack.xml` | Red virtual aislada del atacante (10.10.20.0/24) |
| `libvirt/soc-lab.xml` | *Pendiente:* `virsh -c qemu:///system net-dumpxml soc-lab > libvirt/soc-lab.xml` |
| `wazuh/local_rules.xml` | Reglas propias de Wazuh |
| `wazuh/ossec-agent-windows.conf` | *Pendiente:* fragmento del `ossec.conf` del agente Windows |
| `suricata/` | *Pendiente:* exportación de la configuración de Suricata/OPNsense |

> Antes de subir nada: revisa que no haya contraseñas, claves de registro de agentes ni IPs públicas.
