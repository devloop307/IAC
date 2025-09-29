# 🏢 Sistema de Gestión Empresarial On-Premise con IaC

![Status](https://img.shields.io/badge/status-production--ready-success)
![Infrastructure](https://img.shields.io/badge/infrastructure-as--code-blue)
![License](https://img.shields.io/badge/license-Academic-orange)

> **Infraestructura empresarial completa automatizada con Terraform y Ansible para operaciones críticas en entornos LAN de alta demanda**

Universidad Privada Antenor Orrego | Infraestructura como Código | Grupo 5

---

## 📋 Tabla de Contenidos

- [El Problema de Negocio](#-el-problema-de-negocio)
- [Nuestra Solución](#-nuestra-solución)
- [Arquitectura](#-arquitectura)
- [Atributos de Calidad](#-atributos-de-calidad)
- [Stack Tecnológico](#-stack-tecnológico)
- [Infraestructura como Código](#-infraestructura-como-código)
- [Guía de Instalación](#-guía-de-instalación)
- [Estructura del Proyecto](#-estructura-del-proyecto)
- [Monitoreo y Observabilidad](#-monitoreo-y-observabilidad)
- [Equipo del Proyecto](#-equipo-del-proyecto)
- [Roadmap](#-roadmap)

---

## ❌ El Problema de Negocio

### Situación Actual

Las operaciones enfrentan desafíos críticos que impactan directamente la productividad y confiabilidad:

| Problema | Impacto en el Negocio |
|----------|----------------------|
| Sin infraestructura centralizada | Pérdida de ventas por inconsistencias de datos |
| Trabajo paralelo sin sincronización | Duplicidades y conflictos de inventario |

---

## ✅ Nuestra Solución

### Propuesta de Valor

Infraestructura empresarial **100% automatizada** que garantiza **seguridad** y **confiabilidad** mediante:

🎯 **Arquitectura de Tres Capas**
- **Capa de Presentación** (VLAN 10): Servicios de frontend y proxy reverso
- **Capa de Aplicación** (VLAN 20): Lógica de negocio y mensajería
- **Capa de Datos** (VLAN 30): Persistencia, caché y procesamiento

🛡️ **Seguridad por Diseño**
- Segmentación de red mediante VLANs aisladas
- Firewalling entre capas con routers virtuales
- Acceso controlado mediante NAT y reglas iptables
- Sin exposición directa a Internet de servicios críticos

🔄 **Confiabilidad Operacional**
- Base de datos transaccional PostgreSQL con ACID
- Sistema de caché Redis para alta disponibilidad
- Colas de mensajería RabbitMQ para procesamiento asíncrono
- Monitoreo continuo con Grafana y alertas automatizadas
- Logs centralizados para trazabilidad completa

---

## 🏗️ Arquitectura

### Diagrama de Infraestructura

<img width="3775" height="2174" alt="image" src="https://github.com/user-attachments/assets/291cb9c6-2a71-4d74-8430-7d83f223d582" />

### Componentes y Justificación Técnica

#### **Proxmox VE** - Capa de Virtualización
- Hypervisor bare-metal tipo 1 para máximo rendimiento
- Gestión centralizada de recursos computacionales
- Soporte nativo para VLANs y redes virtuales

#### **VLAN 10 - Presentación y Acceso**
- **Nginx (10.10.0.1:80)**: Reverse proxy y balanceador
- **Grafana (10.10.2:3000)**: Dashboards de monitoreo
- **Loki**: Agregación de logs centralizada

**Justificación:** Aísla la capa de acceso público/interno del resto de servicios críticos

#### **VLAN 20 - Aplicación**
- **RabbitMQ (10.20.0.2:5672)**: Cola de mensajes para procesamiento asíncrono
- **Routers Virtuales**: Control de tráfico entre VLANs

**Justificación:** Desacopla servicios mediante arquitectura orientada a eventos

#### **VLAN 30 - Datos**
- **Redis (10.30.0.1:6379)**: Caché de alta velocidad y sesiones
- **PostgreSQL (10.30.0.2:5432)**: Base de datos transaccional ACID
- **Cron Jobs**: Tareas programadas y mantenimiento
- **NTFS (1TB)**: Almacenamiento compartido para backups

**Justificación:** Centraliza persistencia con aislamiento de red para máxima seguridad

#### **NAT Instance** - Seguridad
- Punto único de salida controlado
- Reglas iptables para filtrado de tráfico
- Logging de conexiones salientes

### Flujo de Datos End-to-End

```
Cliente → DNS Local → Nginx (VLAN 10) 
  → Router → RabbitMQ (VLAN 20)
  → Router → PostgreSQL/Redis (VLAN 30)
  → Log Sync → Loki (VLAN 10) → Grafana
```

### Decisiones de Diseño

**¿Por qué Proxmox?**
- Control total sobre hardware sin costos de licenciamiento
- API REST completa para automatización con Terraform
- Soporte empresarial para virtualización en LAN

**¿Por qué PostgreSQL?**
- Garantías ACID para transacciones financieras
- Extensibilidad y soporte para tipos de datos complejos
- Rendimiento superior en operaciones de lectura/escritura simultáneas

**¿Por qué Redis?**
- Latencia sub-milisegundo para operaciones críticas
- Persistencia opcional con snapshots
- Soporte nativo para estructuras de datos complejas

---

## 🎖️ Atributos de Calidad

Nuestra arquitectura prioriza dos atributos de calidad fundamentales:

### 🔒 Seguridad

**Implementación:**
- Segmentación de red en 3 VLANs completamente aisladas
- Comunicación inter-VLAN únicamente mediante routers con reglas de firewall
- NAT Instance como único punto de salida a Internet
- Sin exposición directa de servicios de datos (PostgreSQL, Redis) a capas superiores
- Acceso a servicios mediante DNS interno sin resolución externa

**Métricas:**
- 0 puertos de servicios críticos expuestos a Internet
- 100% del tráfico entre VLANs inspeccionado
- Logs de acceso centralizados para auditoría

### 🛡️ Confiabilidad

**Implementación:**
- Base de datos PostgreSQL con garantías ACID para consistencia transaccional
- Redis como capa de caché para reducir latencia y carga en BD
- RabbitMQ para procesamiento asíncrono y desacoplamiento de servicios
- Sistema de logs centralizado (Loki) para trazabilidad completa
- Monitoreo activo con Grafana y alertas en tiempo real
- Almacenamiento compartido NTFS para respaldos persistentes

**Métricas:**
- Latencia <5ms en LAN pura
- Disponibilidad de datos 99.9% mediante replicación de caché
- Tiempo medio de detección de fallos <2 minutos (Grafana)
- Recuperación ante fallos de servicios mediante restart automatizado

---

## 🔧 Stack Tecnológico

### Virtualización e Infraestructura

| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| Proxmox VE | 8.x | Hypervisor y gestión de VMs |
| Linux LXC | - | Contenedores ligeros para servicios |
| VLANs | 10, 20, 30 | Segmentación de red |

### Servicios Core

| Servicio | Versión | Propósito | VLAN |
|----------|---------|-----------|------|
| PostgreSQL | 15+ | Base de datos transaccional | 30 |
| Redis | 7+ | Caché y gestión de sesiones | 30 |
| RabbitMQ | 3.12+ | Cola de mensajes | 20 |
| Nginx | 1.24+ | Reverse proxy y balanceo | 10 |

### Observabilidad

| Herramienta | Versión | Propósito | VLAN |
|-------------|---------|-----------|------|
| Grafana | 10+ | Dashboards y visualización | 10 |
| Loki | 2.9+ | Agregación de logs | 10 |
| Prometheus | (integrado) | Métricas de sistema | 10 |

### Automatización

| Herramienta | Versión | Propósito |
|-------------|---------|-----------|
| Terraform | 1.6+ | Aprovisionamiento de infraestructura |
| Ansible | 2.15+ | Gestión de configuración |
| Git | 2.40+ | Control de versiones |

---

## 🤖 Infraestructura como Código

### Filosofía: Automatización Total

**TODO el entorno** es reproducible mediante código declarativo:

```
Infraestructura física → Terraform → VMs en Proxmox
Configuración de servicios → Ansible → Servicios productivos
Monitoreo → Dashboards como código → Observabilidad
```

### Terraform - Aprovisionamiento

**Gestiona:**
- ✅ Creación de VMs en Proxmox con especificaciones exactas
- ✅ Configuración de redes virtuales (VLANs 10, 20, 30)
- ✅ Asignación de recursos (CPU, RAM, almacenamiento)
- ✅ Configuración de routers virtuales y NAT
- ✅ Volúmenes de almacenamiento compartido

**Ejemplo de uso:**
```bash
cd terraform/
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

### Ansible - Configuración y Despliegue

**Gestiona:**
- ✅ Instalación de servicios (PostgreSQL, Redis, RabbitMQ, Nginx)
- ✅ Configuración de seguridad (firewalls, usuarios, permisos)
- ✅ Despliegue de aplicaciones y dependencias
- ✅ Configuración de monitoreo (agentes, exporters)
- ✅ Inicialización de bases de datos y esquemas

**Playbooks principales:**
```bash
ansible-playbook -i inventory/production site.yml              # Despliegue completo
ansible-playbook -i inventory/production playbooks/database.yml # Solo BD
ansible-playbook -i inventory/production playbooks/monitoring.yml # Solo monitoreo
```

### Pipeline de Despliegue

```
1. Código → Git Push
2. Terraform Apply → Infraestructura provisionada
3. Ansible Playbooks → Servicios configurados
4. Health Checks → Validación automatizada
5. Grafana → Monitoreo activo
```

**Tiempo total de despliegue desde cero: ~12-15 minutos**

### Scripts de Gestión

```bash
# Despliegue completo de entorno
./deploy.sh --env production

# Destrucción segura del entorno
./destroy.sh --confirm

# Backup automatizado
./backup.sh --full

# Restauración desde backup
./restore.sh --timestamp 2025-09-28-1200
```

---

## 📦 Guía de Instalación

### Prerrequisitos

- Servidor físico con Proxmox VE 8.x instalado
- Acceso SSH al servidor Proxmox
- Git instalado en máquina de control
- Terraform 1.6+ instalado
- Ansible 2.15+ instalado

### Paso 1: Clonar el Repositorio

```bash
git clone https://github.com/grupo5-upao/infraestructura-iac.git
cd infraestructura-iac
```

### Paso 2: Configurar Variables

```bash
# Copiar archivo de variables de ejemplo
cp terraform/terraform.tfvars.example terraform/terraform.tfvars

# Editar con datos de tu entorno Proxmox
nano terraform/terraform.tfvars
```

**Variables requeridas:**
```hcl
proxmox_url = "https://tu-proxmox:8006/api2/json"
proxmox_token_id = "terraform@pam!terraform"
proxmox_token_secret = "tu-secret-token"
```

### Paso 3: Provisionar Infraestructura

```bash
cd terraform/
terraform init
terraform validate
terraform plan
terraform apply -auto-approve
```

**Salida esperada:**
```
Apply complete! Resources: 15 added, 0 changed, 0 destroyed.

Outputs:
nginx_ip = "10.10.0.1"
database_ip = "10.30.0.2"
grafana_url = "http://10.10.2:3000"
```

### Paso 4: Configurar Servicios

```bash
cd ../ansible/
# Configurar inventario con IPs provisionadas
ansible-playbook -i inventory/production site.yml
```

### Paso 5: Validar Despliegue

```bash
# Verificar conectividad de servicios
ansible -i inventory/production all -m ping

# Acceder a Grafana
# http://10.10.2:3000 (admin/admin)
```

### Despliegue en un Solo Comando

```bash
./deploy.sh --full --env production
```

---

## 📁 Estructura del Proyecto

```
infrastructure-iac/
├── terraform/                      # Infraestructura como código
│   ├── main.tf                     # Configuración principal
│   ├── variables.tf                # Variables de entrada
│   ├── outputs.tf                  # Outputs de aprovisionamiento
│   ├── providers.tf                # Providers (Proxmox)
│   ├── modules/
│   │   ├── networking/             # VLANs y routers
│   │   ├── compute/                # VMs y contenedores
│   │   └── storage/                # Volúmenes y NFS
│   └── terraform.tfvars            # Valores específicos del entorno
│
├── ansible/                        # Gestión de configuración
│   ├── site.yml                    # Playbook principal
│   ├── playbooks/
│   │   ├── database.yml            # Configuración PostgreSQL
│   │   ├── cache.yml               # Configuración Redis
│   │   ├── messaging.yml           # Configuración RabbitMQ
│   │   ├── proxy.yml               # Configuración Nginx
│   │   └── monitoring.yml          # Configuración Grafana/Loki
│   ├── roles/
│   │   ├── common/                 # Tareas comunes a todos los hosts
│   │   ├── security/               # Hardening y firewalls
│   │   ├── postgresql/             # Rol de base de datos
│   │   ├── redis/                  # Rol de caché
│   │   └── nginx/                  # Rol de proxy
│   └── inventory/
│       ├── production/             # Inventario de producción
│       └── staging/                # Inventario de pruebas
│
├── monitoring/                     # Observabilidad
│   ├── grafana-dashboards/
│   │   ├── infrastructure.json     # Dashboard de infraestructura
│   │   ├── application.json        # Dashboard de aplicación
│   │   └── security.json           # Dashboard de seguridad
│   └── prometheus/
│       └── alerts.yml              # Reglas de alertas
│
├── scripts/                        # Automatización
│   ├── deploy.sh                   # Despliegue completo
│   ├── destroy.sh                  # Destrucción de entorno
│   ├── backup.sh                   # Backup automatizado
│   └── health-check.sh             # Validación de servicios
│
├── docs/                           # Documentación
│   ├── architecture.md             # Decisiones arquitectónicas
│   ├── runbook.md                  # Procedimientos operativos
│   └── troubleshooting.md          # Solución de problemas
│
└── README.md                       # Este archivo
```

---

## 📊 Monitoreo y Observabilidad

### Dashboards de Grafana

**Dashboard de Infraestructura**
- CPU, memoria y disco de todas las VMs
- Tráfico de red por VLAN
- Estado de servicios críticos
- Alertas activas

**Dashboard de Base de Datos**
- Conexiones activas a PostgreSQL
- Queries lentas y bloqueos
- Hit rate de Redis
- Uso de memoria de caché

**Dashboard de Aplicación**
- Requests por segundo en Nginx
- Latencia de respuesta
- Mensajes en colas de RabbitMQ
- Errores y excepciones

### Métricas Clave Monitoreadas

| Métrica | Umbral de Alerta | Acción |
|---------|------------------|--------|
| CPU > 80% | 5 minutos | Email al equipo |
| Memoria > 90% | 2 minutos | Slack + Email |
| Disco > 85% | Inmediato | PagerDuty |
| PostgreSQL down | Inmediato | SMS + Email |
| Redis down | 30 segundos | Email |
| Latencia > 100ms | 1 minuto | Slack |

### Sistema de Logs

**Centralización con Loki:**
- Logs de todos los servicios agregados
- Búsqueda y filtrado en tiempo real
- Retención de 30 días
- Alertas basadas en patrones de log

**Acceso:**
```
http://10.10.2:3000/explore
Datasource: Loki
```

---

## 👥 Equipo del Proyecto

### Universidad Privada Antenor Orrego
**Curso:** Infraestructura como Código  
**Grupo:** 5

### Integrantes

- **ALCANTARA RODRIGUEZ, PIERO**  
  
- **BAUTISTA REYES, LOURDES**  
  
- **DAVALOS ALFARO, MARISELLA**  
  
- **LEYVA VALQUI, GABRIEL**  
  
- **RODRIGUEZ GONZALES, ALEJANDRO**  

---

## 🗺️ Roadmap

### ✅ Fase 1: Infraestructura Base (Completado)
- Arquitectura de 3 capas con VLANs
- Automatización con Terraform y Ansible
- Servicios core (PostgreSQL, Redis, RabbitMQ, Nginx)
- Monitoreo con Grafana

### 🚧 Fase 2: Alta Disponibilidad (Q4 2025)
- Cluster de PostgreSQL con replicación maestro-esclavo
- Redis Sentinel para failover automático
- Nginx con balanceo de carga entre múltiples backends
- Backups automatizados diarios con retención de 30 días

### 📋 Fase 3: Disaster Recovery (Q1 2026)
- Sitio secundario para recuperación ante desastres
- Replicación asíncrona de datos entre sitios
- Procedimientos automatizados de failover
- RPO < 5 minutos, RTO < 15 minutos

### 🌐 Fase 4: Multi-Sitio (Q2 2026)
- Arquitectura distribuida entre oficinas
- VPN site-to-site entre ubicaciones
- Base de datos distribuida con sincronización
- Load balancing geográfico

---

## 📄 Licencia y Contacto

### Licencia
Este proyecto es desarrollado con fines académicos bajo la Universidad Privada Antenor Orrego.

### Contacto
Para consultas sobre este proyecto:
- **Email:** grupo5.iac@upao.edu.pe
- **Repositorio:** https://github.com/grupo5-upao/infraestructura-iac

---

<p align="center">
  <strong>Desarrollado con ❤️ por el Grupo 5 - UPAO</strong><br>
  <em>Infraestructura como Código | 2025</em>
</p>
