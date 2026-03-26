# 🏫 Infraestructura Híbrida y Segura para Entorno Escolar (Proyecto Work & Play)

## 📝 Descripción del Proyecto
Este proyecto documenta la planificación, despliegue y auditoría de un servidor basado en Ubuntu Server virtualizado, diseñado para proveer almacenamiento seguro (NAS) y servicios recreativos aislados para un entorno de laboratorio escolar (11 equipos en topología de estrella). 

El objetivo principal es implementar una arquitectura gestionada mediante contenedores (Docker), asegurando el control de acceso, la monitorización de recursos y el cumplimiento de estándares básicos de seguridad de la información (inspirado en ISO 27001 e ISO 9001).

## 🛠️ Stack Tecnológico
* **Sistema Operativo:** Ubuntu Server (Virtualizado).
* **Almacenamiento en Red:** Samba (SMB/CIFS) con gestión de permisos estrictos.
* **Orquestación:** Docker & Docker Compose.
* **Gestión y Monitoreo:** Portainer, Node Exporter (Prometheus-ready).
* **Seguridad y Redes:** Firewall UFW, Fail2Ban, WireGuard (VPN), Caddy (Proxy Inverso).

## 🗄️ Arquitectura de Almacenamiento
Se diseñó un esquema de discos virtuales separados para proteger la integridad de los datos de los usuarios frente a fallos del sistema o saturación por servicios:
* **Disco 1 (30 GB):** Sistema Raíz `/`, Memoria Swap (2 GB) y volúmenes de contenedores `/srv/juegos`.
* **Disco 2 (200 GB):** Disco dedicado exclusivamente para datos de usuarios, montado con `ext4` en `/srv/archivos_compartidos`.

## 🛡️ Auditoría y Seguridad DevSecOps
La infraestructura fue diseñada bajo el principio de "Denegar por defecto" e incluye las siguientes capas de seguridad:
1. **Acceso Remoto Seguro:** Despliegue de un túnel VPN con `WireGuard` (Puerto 51820/UDP) para la administración remota cifrada.
2. **Aislamiento de Red:** Los contenedores comparten la red del host solo cuando es estrictamente necesario, administrados por interfaces internas de Docker.
3. **Validación (Penetration Testing):** Se realizó una auditoría básica de puertos utilizando `nmap` (`nmap -sV -p 22,8081,9000...`) para verificar la correcta configuración del firewall UFW, confirmando que servicios internos como Samba no están expuestos al exterior.

### 💡 Lecciones Aprendidas (Troubleshooting)
* **Limitaciones de Kernel con Fail2Ban en Contenedores:** Inicialmente se proyectó desplegar `Fail2Ban` containerizado para proteger el puerto 22. Durante las pruebas, se diagnosticaron conflictos de compatibilidad entre los módulos del Kernel del Host y la capacidad del contenedor (`NET_ADMIN`) para inyectar reglas dinámicas en el enrutamiento. Para no comprometer la estabilidad del sistema, se optó por descartarlo y depender estrictamente del Firewall UFW y la tunelización por WireGuard.

## 📂 Estructura del Repositorio
En este repositorio se encuentran los archivos de configuración utilizados para el despliegue de la infraestructura como código:

- `/admin-tools/docker-compose.yml`: Stack de administración (Portainer, WireGuard, Node Exporter, Caddy, Fail2Ban).
- `/gaming/docker-compose.yml`: Stack de servidores recreativos aislados (Minecraft PaperMC con límites estrictos de RAM).
- `/network/50-cloud-init.yaml`: Configuración de IP estática y DNS usando Netplan.
- `/samba/smb.conf`: Configuración de la carpeta compartida y permisos.

## 📊 Monitoreo y Rendimiento
El servidor exporta métricas en tiempo real a través de `Node Exporter` en el puerto 9100, permitiendo la visualización de carga de CPU, uso de RAM, latencia de lectura/escritura de discos (I/O) y tráfico de red, previniendo la saturación del sistema host por parte de los contenedores recreativos.
