 # Ataque DNS Spoofing + ARP Poisoning MitM

**Nombre:** Henry Vicente Quezada | **Matrícula:** 2025-1332 | **Fecha:** 12 de Junio 2026

---

## 🎬 Video Demostrativo

https://youtu.be/TU_LINK_AQUI

---

## 1. Objetivo del Laboratorio

Demostrar cómo un atacante puede interceptar consultas DNS de las víctimas mediante ARP Poisoning (MitM) y responderlas con IPs falsas, redirigiendo el tráfico hacia un servidor web falso controlado por el atacante, y aplicar DAI + DHCP Snooping como contramedida.

---

## 2. Objetivo del Script

Combinar ARP Poisoning con sniffing de paquetes DNS para interceptar consultas hacia `itla.edu.do` y responder con la IP del atacante (`10.13.32.50`), redirigiendo a la víctima a un sitio web falso servido por Kali.

### 2.1 Parámetros Usados

| Parámetro  | Descripción                          | Ejemplo        |
| ---------- | ------------------------------------ | -------------- |
| `interfaz` | Interfaz de red del atacante         | `eth0`         |
| `gateway`  | IP del gateway                       | `10.13.10.1`   |
| `victima`  | IP de la víctima                     | `10.13.10.11`  |
| `DOMINIO`  | Dominio a falsificar (en el script)  | `itla.edu.do`  |
| `IP_FALSA` | IP falsa a devolver                  | `10.13.10.5`  |

### 2.2 Requisitos

- Sistema operativo: **Kali Linux**
- Python 3.x
- Librería Scapy: `pip install scapy`
- Permisos de root: `sudo`
- IP forwarding habilitado (el script lo activa automáticamente)
- Servidor web falso activo en puerto 80 (`python3 -m http.server`)

---

## 3. Funcionamiento del Script

1. Obtiene la MAC real de la víctima y del gateway mediante ARP
2. Habilita IP forwarding para mantener conectividad en la red
3. Lanza hilo de ARP Poisoning: envenena víctima y gateway cada 2 segundos
4. Escucha en el puerto UDP 53 (DNS) con Scapy `sniff()`
5. Al interceptar una query para `itla.edu.do`, construye respuesta DNS falsa con `IP_FALSA`
6. Al detener con `Ctrl+C`, restaura las tablas ARP originales

```
Crear y guardar el script:
bash
nano /home/kali-linux/HenryVicenteQuezada_2025-1332_dns_spoofing.py

Dar permisos de ejecución:
bash
chmod +x /home/kali-linux/HenryVicenteQuezada_2025-1332_dns_spoofing.py

Pasos de ejecución:

Paso 1 — Kali: Levantar el servidor web falso
bash
mkdir -p /tmp/sitio_falso
cat > /tmp/sitio_falso/index.html << 'EOF'
<!DOCTYPE html><html>
<head><title>ITLA - SPOOFED</title></head>
<body style="background:#cc0000;color:white;text-align:center">
  <h1>SITIO FALSO — DNS SPOOFING</h1>
  <h2>Henry Vicente Quezada | 2025-1332</h2>
</body></html>
EOF
cd /tmp/sitio_falso && sudo python3 -m http.server 80 &

Paso 2 — VPC10: Verificar DNS real antes del ataque
bash
nslookup itla.edu.do 10.13.32.1

Paso 3 — Kali: Habilitar IP Forwarding
bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

Paso 4 — Kali: Ejecutar el ataque
bash
sudo python3 /home/kali-linux/HenryVicenteQuezada_2025-1332_dns_spoofing.py eth0 10.13.32.1 10.13.32.11

Paso 5 — Kali: Abrir Wireshark para capturar tráfico DNS
bash
sudo wireshark &
Filtro: dns

Paso 6 — VPC10: Probar el DNS Spoofing
bash
nslookup itla.edu.do

Paso 7 — VPC10: Verificar que llega al sitio falso
bash
curl http://itla.edu.do

Paso 8 — SW1: Aplicar contramedida
conf t
ip dhcp snooping
ip dhcp snooping vlan 10
ip dhcp snooping vlan 20
no ip dhcp snooping information option
interface e0/0
ip dhcp snooping trust
ip arp inspection trust
exit
ip arp inspection vlan 10
ip arp inspection vlan 20
end
write memory

Paso 9 — Kali: Ejecutar el ataque de nuevo
bash
sudo python3 /home/kali-linux/HenryVicenteQuezada_2025-1332_dns_spoofing.py eth0 10.13.32.1 10.13.32.11

Paso 10 — SW1: Verificar que el ARP Poison fue bloqueado
bash
show ip arp inspection
show ip arp inspection statistics
show ip arp inspection vlan 10

🐍 Script — HenryVicenteQuezada_2025-1332_dns_spoofing.py
python
#!/usr/bin/env python3
# =============================================================
# Nombre:     Henry Vicente Quezada
# Matricula:  2025-1332
# Ataque:     DNS Spoofing + ARP Poisoning MitM
# Fecha:      2026
# =============================================================
[PEGA AQUÍ EL SCRIPT COMPLETO]

🛡️ Contramedida aplicada
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 10
SW1(config)# ip dhcp snooping vlan 20
SW1(config)# no ip dhcp snooping information option
SW1(config)# interface e0/0
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# ip arp inspection trust
SW1(config-if)# exit
SW1(config)# ip arp inspection vlan 10
SW1(config)# ip arp inspection vlan 20
SW1(config)# end
SW1# write memory
! Verificación
SW1# show ip arp inspection
SW1# show ip arp inspection statistics
SW1# show ip arp inspection vlan 10
SW1# show ip dhcp snooping binding
```

---

## 4. Documentación de la Red

### Topología

 <img width="1150" height="733" alt="image" src="https://github.com/user-attachments/assets/37f89e10-1c11-4c7b-a2a1-a35b8ac56d3b" />


## Tabla de Interfaces por Dispositivo

| Dispositivo | Interfaz | VLAN | IP | Máscara | Descripción |
|---|---|---|---|---|---|
| ISP | e0/0 | WAN | 200.13.32.1 | /30 | Enlace hacia R1 |
| R1 | e0/3 | WAN | 200.13.32.2 | /30 | Enlace hacia ISP (NAT Outside) |
| R1 | e0/0 | WAN | 10.13.32.1 | /30 | Enlace hacia R2 (OSPF) |
| R1 | e0/1 | Trunk | — | — | Trunk hacia SW1 |
| R1 | e0/1.10 | 10 | 10.13.10.1 | /24 | Gateway VLAN 10 Usuarios |
| R1 | e0/1.99 | 99 | 10.13.99.1 | /28 | Gateway Gestión SW1 |
| R1 | e0/1.999 | 999 | — | — | VLAN Nativa |
| R1 | e0/2 | Trunk | — | — | Trunk hacia SW2 |
| R1 | e0/2.20 | 20 | 10.13.20.1 | /24 | Gateway VLAN 20 Administración |
| R1 | e0/2.99 | 99 | 10.13.99.17 | /28 | Gateway Gestión SW2 |
| R1 | e0/2.999 | 999 | — | — | VLAN Nativa |
| R2 | e0/0 | WAN | 10.13.32.2 | /30 | Enlace hacia R1 (OSPF) |
| R2 | e0/1 | Trunk | — | — | Trunk hacia SW3 |
| R2 | e0/1.30 | 30 | 10.13.30.1 | /24 | Gateway VLAN 30 Servicios |
| R2 | e0/1.99 | 99 | 10.13.99.33 | /28 | Gateway Gestión SW3 |
| R2 | e0/1.999 | 999 | — | — | VLAN Nativa |
| R2 | e0/2 | Trunk | — | — | Trunk hacia SW4 |
| R2 | e0/2.40 | 40 | 10.13.40.1 | /24 | Gateway VLAN 40 Invitados |
| R2 | e0/2.99 | 99 | 10.13.99.49 | /28 | Gateway Gestión SW4 |
| R2 | e0/2.999 | 999 | — | — | VLAN Nativa |
| SW1 | VLAN 99 | 99 | 10.13.99.2 | /28 | Gestión Switch |
| SW2 | VLAN 99 | 99 | 10.13.99.18 | /28 | Gestión Switch |
| SW3 | VLAN 99 | 99 | 10.13.99.34 | /28 | Gestión Switch |
| SW4 | VLAN 99 | 99 | 10.13.99.50 | /28 | Gestión Switch |
| PC-Usuarios | eth0 | 10 | DHCP | /24 | Cliente VLAN 10 |
| Kali Linux | eth0 | 10 | 10.13.10.5 | /24 | Equipo Kali Linux |
| PC-Administración | eth0 | 20 | DHCP | /24 | Cliente VLAN 20 |
| Servidor | eth0 | 30 | DHCP | /24 | Cliente VLAN 30 |
| PC-Invitados | eth0 | 40 | DHCP | /24 | Cliente VLAN 40 |

## Pools DHCP

| Pool | Red | Gateway | Rango Disponible | Excluidos |
|---|---|---|---|---|
| VLAN10_USUARIOS | 10.13.10.0/24 | 10.13.10.1 | 10.13.10.11 - 10.13.10.254 | 10.13.10.1 - 10.13.10.10 |
| VLAN20_ADMIN | 10.13.20.0/24 | 10.13.20.1 | 10.13.20.11 - 10.13.20.254 | 10.13.20.1 - 10.13.20.10 |
| VLAN30_SERVICIOS | 10.13.30.0/24 | 10.13.30.1 | 10.13.30.11 - 10.13.30.254 | 10.13.30.1 - 10.13.30.10 |
| VLAN40_INVITADOS | 10.13.40.0/24 | 10.13.40.1 | 10.13.40.11 - 10.13.40.254 | 10.13.40.1 - 10.13.40.10 |

## Interfaces de Acceso y Troncales

| Dispositivo | Puerto | Modo | VLAN | Conectado a |
|---|---|---|---|---|
| SW1 | e0/0 | Trunk 802.1Q | 10,99,999 | R1 |
| SW1 | e0/1 | Access | 10 | PC Usuarios |
| SW1 | e0/2 | Access | 10 | Kali Linux |
| SW2 | e0/0 | Trunk 802.1Q | 20,99,999 | R1 |
| SW2 | e0/1 | Access | 20 | PC Administración |
| SW3 | e0/0 | Trunk 802.1Q | 30,99,999 | R2 |
| SW3 | e0/1 | Access | 30 | Servidor |
| SW4 | e0/0 | Trunk 802.1Q | 40,99,999 | R2 |
| SW4 | e0/1 | Access | 40 | PC Invitados |

## VLANs Implementadas

| VLAN | Nombre | Red | Gateway |
|---|---|---|---|
| 10 | Usuarios | 10.13.10.0/24 | 10.13.10.1 |
| 20 | Administración | 10.13.20.0/24 | 10.13.20.1 |
| 30 | Servicios | 10.13.30.0/24 | 10.13.30.1 |
| 40 | Invitados | 10.13.40.0/24 | 10.13.40.1 |
| 99 | Administración de Equipos | 10.13.99.0/28, 10.13.99.16/28, 10.13.99.32/28, 10.13.99.48/28 | Según segmento |
| 999 | VLAN Nativa No Utilizada | N/A | N/A |

---

## 5. Capturas de Pantalla

### Antes del ataque

![antes](PEGA_IMAGEN_AQUI)

📷 VPC10# nslookup itla.edu.do — Responde con IP real

### Script en ejecución

![script](PEGA_IMAGEN_AQUI)

📷 Kali ejecutando dns_spoofing.py — ARP Poisoning activo

### Durante el ataque

![durante](PEGA_IMAGEN_AQUI)

📷 VPC10# nslookup itla.edu.do — Responde con 10.13.32.50 (IP falsa) ✅

### Contramedida aplicada

![contramedida](PEGA_IMAGEN_AQUI)

📷 SW1# show ip arp inspection statistics — ARPs falsos descartados por DAI ✅

---

DAI valida cada paquete ARP contra la tabla de DHCP Snooping. Los ARPs falsos de Kali son descartados porque no existe un binding válido para su MAC/IP. Sin ARP Poisoning el tráfico DNS no pasa por Kali, eliminando completamente el vector de DNS Spoofing.
