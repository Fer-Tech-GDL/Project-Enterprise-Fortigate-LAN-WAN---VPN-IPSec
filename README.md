# 🏢 Enterprise Edge Security & High Availability Architecture

## 📖 Descripción General del Proyecto
Este proyecto simula el diseño, configuración y aseguramiento de la infraestructura perimetral y el núcleo de red para un entorno corporativo distribuido. El objetivo principal es garantizar la continuidad del negocio mediante Alta Disponibilidad (HA), enrutamiento dinámico resiliente y segmentación segura, aplicando los más altos estándares de la industria.

## 🛠️ Tecnologías y Herramientas Utilizadas
* **Simulador:** GNS3
* **Seguridad Perimetral:** Fortinet FortiGate (FortiOS)
* **Routing & Switching:** Cisco (OSPF, RIPv2, 802.1Q)
* **Gestión:** Contenedores Docker (Linux)
* **Endpoints:** VPCS para simulación de hosts

## 🏗️ Topología de la Red
> **Nota:** <img width="766" height="661" alt="image" src="https://github.com/user-attachments/assets/aa35c45f-f4dd-4e1d-bc9d-72a8690d9a62" />


El diseño consta de dos sedes principales interconectadas, utilizando firewalls FortiGate en el borde y switches/routers Cisco en la capa de distribución, segmentando el tráfico en múltiples VLANs para departamentos específicos (Finanzas, HR, IT, Industrial, Producción).

---

## 🔐 Gestión de la Seguridad y Continuidad
En esta fase del proyecto, el enfoque trasciende la simple interconexión, centrándose en satisfacer las necesidades críticas del negocio mediante controles de seguridad perimetral basados en las mejores prácticas. La arquitectura se fundamenta en **FortiOS**, proporcionando un despliegue robusto de políticas y minimizando riesgos operativos.

### Matriz de Políticas de Seguridad (Traffic Flow)
Las reglas de filtrado se diseñaron bajo el principio de menor privilegio:

| Origen | Destino | Servicios Permitidos | Acción | Estado NAT | Propósito |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **LAN** | **WAN** | ALL | ACCEPT | **Activado** | Salida a Internet para usuarios corporativos. |
| **DMZ** | **WAN** | ALL | ACCEPT | **Activado** | Acceso a actualizaciones para servidores. |
| **LAN** | **VLANs** | ALL | ACCEPT | **Desactivado** | Tráfico interno y enrutamiento Inter-VLAN. |
| **WEB (Mgmt)** | **WAN** | SSH / HTTPS / ICMP | ACCEPT | **Activado** | Gestión remota segura del nodo. |
| **LAN (Sede A)**| **VPN (Sede B)**| ALL | ACCEPT | **Desactivado** | Envío de tráfico seguro hacia la sucursal remota. |
| **VPN (Sede B)**| **LAN (Sede A)**| ALL | ACCEPT | **Desactivado** | Recepción de tráfico cifrado desde la sucursal remota. |

### Aislamiento de la Gestión (Out-of-Band Management)
Para garantizar la integridad del plano de control, se implementó un contenedor Docker basado en Linux dedicado exclusivamente a la administración de la interfaz web del firewall. Este entorno aislado (*Out-of-Band*) separa el tráfico de gestión del de producción, optimizando el rendimiento y blindando el acceso al equipo.

### Conectividad Segura Inter-Sedes (Site-to-Site VPN IPsec)
Para garantizar la comunicación confidencial a través de la infraestructura WAN, se desplegó un túnel **VPN IPsec Site-to-Site**. Este enlace cifra el tráfico de extremo a extremo, asegurando la integridad de los datos corporativos y permitiendo que las redes locales interactúen de forma transparente.

---

## 🔀 Análisis de Ruteo Dinámico y Alta Disponibilidad (OSPF vs RIP)
Para lograr una arquitectura resiliente, se diseñó un esquema de enrutamiento híbrido:

* **Protocolo Principal (OSPF - Estado de Enlace):** El núcleo está estructurado bajo el **Backbone (Área 0)**. OSPF es el protocolo primario gracias a su rápida convergencia frente a fallas y su capacidad de calcular las rutas más eficientes.
* **Protocolo de Respaldo (RIPv2 - Vector Distancia):** Su rol es estrictamente secundario (Failover). Al poseer una Distancia Administrativa mayor (120) a la de OSPF (110), el plano de control solo inyectará las rutas de RIP si el proceso de OSPF llega a colapsar.

**Redistribución:** Se configuró la redistribución mutua entre ambos protocolos para unificar los dominios y propagar dinámicamente las rutas por defecto hacia los firewalls.

---

## 🚀 Alta Disponibilidad Perimetral (Fortinet FGCP)
El perímetro está protegido por un clúster de firewalls en modo **Activo-Pasivo**. El estado de sincronización y las pruebas de *failover* demostraron una pérdida nula de sesiones activas al apagar el nodo Master, asegurando un tiempo de actividad continuo para los usuarios de las VLANs inferiores.
