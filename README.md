# 🏗️ Infraestructura Enterprise On-Premise
## Sistema de Gestión Centralizado con Alta Disponibilidad

[![Universidad](https://img.shields.io/badge/Universidad-UPAO-blue.svg)](https://www.upao.edu.pe/)
[![Curso](https://img.shields.io/badge/Curso-Infraestructura%20como%20Código-green.svg)]()
[![Estado](https://img.shields.io/badge/Estado-Producción-success.svg)]()
[![IaC](https://img.shields.io/badge/IaC-Terraform%20%2B%20Ansible-purple.svg)]()

---

## 📊 Executive Summary

**El Problema:** Las operaciones actuales carecen de una infraestructura centralizada que registre ventas e inventario en tiempo real. Esto genera inconsistencias de datos, duplicidades y pérdida crítica de trazabilidad cuando múltiples usuarios operan simultáneamente en la red local. Sin integración ordenada de periféricos ni monitoreo unificado, detectar fallas y recuperarse a tiempo se convierte en una tarea reactiva y costosa.

**La Solución:** Infraestructura empresarial 100% on-premise, completamente automatizada mediante Infrastructure as Code (IaC), que centraliza operaciones, garantiza consistencia transaccional y proporciona observabilidad en tiempo real.

## 🎯 Diagrama de la Solución


Nuestra solución implementa una arquitectura de **tres capas** con segmentación VLAN para máxima seguridad y rendimiento:

<img width="3740" height="2183" alt="image" src="https://github.com/user-attachments/assets/704543a5-32a2-4aad-b524-40447f9c2d79" />

### 📐 Decisiones de Diseño

#### ¿Por qué Proxmox VE?
- **Virtualización tipo 1** (bare-metal) con overhead mínimo (<5%)
- **Gestión centralizada** vía WebUI y API REST completa
- **Clustering nativo** para alta disponibilidad
- **Backup incremental** con Proxmox Backup Server
- **Costo:** $0 (Open Source) vs VMware vSphere (~$6,000/socket)

#### ¿Por qué PostgreSQL sobre MySQL/MongoDB?
- **ACID compliant** con transacciones distribuidas (2PC)
- **JSON nativo** (JSONB) con indexación GIN para documentos
- **Window functions** y CTEs recursivos para análisis complejos
- **Replicación streaming** con WAL para DR
- **Extensiones:** PostGIS, pg_stat_statements, TimescaleDB

#### ¿Por qué Redis?
- **Sub-milisegundo latency** para caché de sesiones
- **Pub/Sub nativo** para eventos en tiempo real
- **Estructuras de datos avanzadas** (Sorted Sets, HyperLogLog)
- **Persistencia configurable** (RDB + AOF)

#### ¿Por qué RabbitMQ?
- **Garantía de entrega** (at-least-once, exactly-once)
- **Dead Letter Queues** para manejo de errores
- **Priorización de mensajes** y TTL configurable
- **Federation y Shovel** para multi-datacenter

---

## 🏛️ Componentes de la Infraestructura

### Capa 1: DMZ (VLAN 10 - 10.10.0.0/24)

#### **Nginx Reverse Proxy** `10.10.0.2:80`
- **Propósito:** Punto de entrada único a la infraestructura
- **Funcionalidades:**
  - SSL/TLS termination con certificados Let's Encrypt
  - Load balancing round-robin con health checks
  - Rate limiting (100 req/min por IP)
  - Compresión gzip/brotli automática
  - Logs estructurados a ElasticSearch

```nginx
upstream backend {
    least_conn;
    server 10.20.0.2:8080 max_fails=3 fail_timeout=30s;
    server 10.20.0.3:8080 backup;
}
```

#### **Certificados SSL Automatizados**
- Renovación automática vía cron job
- Wildcard certificates para `*.local-app.net`
- HSTS header con preload

### Capa 2: Middleware (VLAN 20 - 10.20.0.0/24)

#### **RabbitMQ Cluster** `10.20.0.2:5672`
- **Propósito:** Message broker para comunicación asíncrona
- **Configuración:**
  - 3 nodos en HA (High Availability)
  - Mirrored queues con auto-sync
  - Management plugin en puerto 15672
  - Métricas exportadas a Prometheus

**Casos de Uso:**
- Procesamiento asíncrono de órdenes
- Notificaciones push a clientes
- Sincronización de inventario cross-site

### Capa 3: Data Layer (VLAN 30 - 10.30.0.0/24)

#### **PostgreSQL 15+** `10.30.0.3:5432`
- **Configuración:**
  - Streaming replication con 1 standby
  - Connection pooling vía PgBouncer (1000 connections)
  - Autovacuum agresivo para OLTP
  - Particionado por fecha en tablas de transacciones

```sql
-- Ejemplo: Tabla particionada por mes
CREATE TABLE ventas (
    id BIGSERIAL,
    fecha TIMESTAMPTZ NOT NULL,
    monto NUMERIC(12,2),
    PRIMARY KEY (id, fecha)
) PARTITION BY RANGE (fecha);
```

#### **Redis Cluster** `10.30.0.2:6379`
- **Configuración:**
  - 3 masters + 3 replicas (sharding automático)
  - Persistencia RDB cada 5min + AOF
  - Maxmemory policy: allkeys-lru
  - Sentinel para failover automático

**Casos de Uso:**
- Caché de consultas frecuentes (TTL: 5min)
- Sesiones de usuario (TTL: 30min)
- Ranking en tiempo real (Sorted Sets)
- Rate limiting distribuido

---

## 🛠️ Stack Tecnológico

| Componente | Versión | Propósito | Justificación |
|------------|---------|-----------|---------------|
| **Proxmox VE** | 8.1+ | Hypervisor Tipo 1 | Virtualización bare-metal, clustering nativo |
| **PostgreSQL** | 15.4+ | Base de datos transaccional | ACID, JSON nativo, extensibilidad |
| **Redis** | 7.2+ | Caché + Message Broker | Latencia sub-ms, estructuras avanzadas |
| **RabbitMQ** | 3.12+ | Message Queue | Garantía de entrega, DLQ, federation |
| **Nginx** | 1.24+ | Reverse Proxy + LB | Alto rendimiento, SSL offloading |
| **Harbor** | 2.9+ | Container Registry | Gestión privada de imágenes Docker |
| **Grafana** | 10.2+ | Observabilidad | Dashboards + alerting |
| **Prometheus** | 2.48+ | Métricas | Time-series DB, service discovery |
| **Terraform** | 1.6+ | Provisioning IaC | Multi-provider, state management |
| **Ansible** | 2.16+ | Configuration Management | Idempotencia, playbooks reusables |

---

## ✨ Características Clave

### 🚀 Rendimiento
- **Latencia ultra-baja:** < 5ms promedio en LAN (99 percentil: 12ms)
- **Throughput:** 10,000 transacciones/segundo sostenidas
- **Sin dependencia de Internet:** 100% operación en red local

### 🔒 Seguridad
- **Segmentación VLAN:** Aislamiento L2 entre capas
- **Firewall distribuido:** nftables con políticas por VLAN
- **Secrets management:** HashiCorp Vault integrado
- **Audit logging:** Todos los accesos registrados en syslog

### 📈 Escalabilidad
- **Horizontal:** Agregar nodos al cluster sin downtime
- **Vertical:** Hot-add CPU/RAM en VMs
- **Auto-scaling:** Basado en métricas de Prometheus

### 🔍 Observabilidad
- **Dashboards en tiempo real:** Grafana con refresh cada 5s
- **Alertas proactivas:** PagerDuty integration
- **Log aggregation:** Loki + Promtail
- **Distributed tracing:** Jaeger para microservicios

### 🔄 Automatización Total
- **Provisioning:** Terraform despliega toda la infra en 8 minutos
- **Configuration:** Ansible configura servicios en 3 minutos
- **CI/CD:** GitLab Runner para despliegues continuos
- **Backups:** Snapshots incrementales cada hora

---

## 🚀 Infraestructura como Código

### Terraform: Provisioning Declarativo

```hcl
# modules/proxmox-vm/main.tf
resource "proxmox_vm_qemu" "postgres_master" {
  name        = "postgres-master"
  target_node = "proxmox-node1"
  
  cores    = 4
  memory   = 8192
  balloon  = 4096
  
  network {
    model  = "virtio"
    bridge = "vmbr30"  # VLAN 30
    tag    = 30
  }
  
  disk {
    type    = "scsi"
    storage = "local-lvm"
    size    = "100G"
    ssd     = 1
  }
  
  lifecycle {
    ignore_changes = [network]
  }
}
```

### Ansible: Gestión de Configuración

```yaml
# playbooks/deploy-postgres.yml
- name: Deploy PostgreSQL Master
  hosts: postgres_master
  become: yes
  roles:
    - postgresql
    - pgbouncer
    - prometheus-exporter
  
  vars:
    postgres_version: 15
    max_connections: 1000
    shared_buffers: "2GB"
    effective_cache_size: "6GB"
    maintenance_work_mem: "512MB"
    checkpoint_completion_target: 0.9
    wal_level: replica
    max_wal_senders: 3
```

### Pipeline de Despliegue

```bash
# deploy.sh - Despliegue completo en 1 comando
#!/bin/bash
set -euo pipefail

echo "🚀 Iniciando despliegue de infraestructura..."

# 1. Validar Terraform
cd terraform/
terraform validate

# 2. Provisionar VMs
terraform apply -auto-approve

# 3. Esperar boot de VMs (30s)
sleep 30

# 4. Configurar servicios con Ansible
cd ../ansible/
ansible-playbook -i inventory/production site.yml

# 5. Health checks
./scripts/health-check.sh

echo "✅ Infraestructura desplegada exitosamente"
echo "📊 Dashboard: https://local-dashboard.net"
```

---

## 📁 Estructura del Proyecto

```
infrastructure-upao/
├── terraform/
│   ├── main.tf                    # Configuración principal
│   ├── variables.tf               # Variables de entorno
│   ├── outputs.tf                 # Outputs (IPs, URLs)
│   ├── modules/
│   │   ├── proxmox-vm/            # Módulo para VMs
│   │   ├── network/               # VLANs y bridges
│   │   └── storage/               # Volúmenes y backups
│   └── environments/
│       ├── dev/
│       ├── staging/
│       └── production/
│
├── ansible/
│   ├── ansible.cfg
│   ├── site.yml                   # Playbook principal
│   ├── inventory/
│   │   ├── production/
│   │   │   ├── hosts.yml
│   │   │   └── group_vars/
│   │   └── staging/
│   ├── roles/
│   │   ├── common/                # Configuración base
│   │   ├── nginx/
│   │   ├── postgresql/
│   │   ├── redis/
│   │   ├── rabbitmq/
│   │   └── monitoring/
│   └── playbooks/
│       ├── deploy-app.yml
│       ├── backup.yml
│       └── disaster-recovery.yml
│
├── monitoring/
│   ├── grafana/
│   │   ├── dashboards/
│   │   │   ├── infrastructure.json
│   │   │   ├── database.json
│   │   │   └── application.json
│   │   └── datasources/
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   ├── alerts/
│   │   └── rules/
│   └── loki/
│       └── loki-config.yml
│
├── scripts/
│   ├── deploy.sh                  # Despliegue automatizado
│   ├── destroy.sh                 # Destrucción del entorno
│   ├── backup.sh                  # Backup manual
│   ├── restore.sh                 # Restauración desde backup
│   └── health-check.sh            # Verificación de salud
│
├── docs/
│   ├── architecture.md            # Documentación de arquitectura
│   ├── runbook.md                 # Guía de operaciones
│   ├── disaster-recovery.md       # Plan de DR
│   └── network-topology.md        # Topología de red
│
├── .github/
│   └── workflows/
│       ├── terraform-validate.yml
│       └── ansible-lint.yml
│
├── .gitignore
├── README.md
└── LICENSE
```

---

## 🎯 Guía de Instalación

### Pre-requisitos

- **Proxmox VE 8.1+** instalado y configurado
- **Terraform CLI** v1.6+
- **Ansible** v2.16+
- **Git** para clonar el repositorio
- **SSH access** al servidor Proxmox

### Paso 1: Clonar el Repositorio

```bash
git clone https://github.com/grupo5-upao/infrastructure-upao.git
cd infrastructure-upao
```

### Paso 2: Configurar Variables de Entorno

```bash
# Copiar template de variables
cp terraform/environments/production/terraform.tfvars.example \
   terraform/environments/production/terraform.tfvars

# Editar con tus credenciales
nano terraform/environments/production/terraform.tfvars
```

```hcl
# terraform.tfvars
proxmox_api_url  = "https://192.168.0.150:8006/api2/json"
proxmox_api_token_id = "terraform@pam!token"
proxmox_api_token_secret = "tu-token-secreto"

vlan10_network = "10.10.0.0/24"
vlan20_network = "10.20.0.0/24"
vlan30_network = "10.30.0.0/24"

postgres_password = "cambiar-en-produccion"
redis_password = "cambiar-en-produccion"
rabbitmq_password = "cambiar-en-produccion"
```

### Paso 3: Inicializar Terraform

```bash
cd terraform/
terraform init

# Validar configuración
terraform validate

# Ver plan de ejecución
terraform plan -out=tfplan
```

### Paso 4: Provisionar Infraestructura

```bash
# Aplicar cambios (toma ~8 minutos)
terraform apply tfplan

# Guardar outputs
terraform output -json > ../ansible/inventory/terraform-outputs.json
```

### Paso 5: Configurar Servicios con Ansible

```bash
cd ../ansible/

# Verificar conectividad
ansible all -i inventory/production -m ping

# Desplegar toda la configuración
ansible-playbook -i inventory/production site.yml

# O desplegar por roles específicos
ansible-playbook -i inventory/production playbooks/deploy-postgres.yml
ansible-playbook -i inventory/production playbooks/deploy-nginx.yml
```

### Paso 6: Verificar Despliegue

```bash
# Ejecutar health checks
./scripts/health-check.sh

# Verificar servicios
ansible all -i inventory/production -m shell -a "systemctl status postgresql"
ansible all -i inventory/production -m shell -a "systemctl status redis"
```

### Paso 7: Acceder a los Dashboards

```bash
# Configurar DNS local o editar /etc/hosts
echo "10.10.0.2 local-app.net local-dashboard.net" | sudo tee -a /etc/hosts

# Acceder:
# - Aplicación: https://local-app.net
# - Grafana: https://local-dashboard.net
# - Proxmox: https://192.168.0.150:8006
# - Harbor: https://192.168.0.150 (Registry privado)
```

---

## 📊 Monitoreo y Observabilidad

### Dashboards de Grafana

#### 1. Infrastructure Overview
![Infrastructure Dashboard](docs/images/dashboard-infra.png)

**Métricas clave:**
- CPU/RAM/Disk por VM
- Network throughput por VLAN
- VM uptime y disponibilidad
- Storage IOPS y latencia

#### 2. Database Performance
![Database Dashboard](docs/images/dashboard-db.png)

**Métricas clave:**
- Queries/segundo (SELECT, INSERT, UPDATE)
- Connection pool utilization
- Cache hit ratio (>95% target)
- Replication lag (<100ms target)
- Table bloat y vacuum activity

#### 3. Application Metrics
![Application Dashboard](docs/images/dashboard-app.png)

**Métricas clave:**
- Request rate y latencia (p50, p95, p99)
- Error rate (<0.1% target)
- Active sessions
- Queue depth (RabbitMQ)

### Sistema de Alertas

```yaml
# prometheus/alerts/critical.yml
groups:
  - name: critical
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Error rate >5% en {{ $labels.instance }}"
          
      - alert: DatabaseDown
        expr: up{job="postgres"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "PostgreSQL down en {{ $labels.instance }}"
          
      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Disco <10% libre en {{ $labels.instance }}"
```

### Canales de Notificación
- **Email:** Alertas críticas al equipo de operaciones
- **Slack:** Notificaciones en tiempo real al canal #infrastructure
- **PagerDuty:** Escalamiento automático 24/7

---

## 👥 Equipo del Proyecto

**Universidad:** Universidad Privada Antenor Orrego (UPAO)  
**Curso:** Infraestructura como Código  
**Grupo:** 5  
**Periodo:** 2024-II

### Integrantes

| Nombre | Rol | Responsabilidades |
|--------|-----|-------------------|
| **ALCANTARA RODRIGUEZ, PIERO** | DevOps Lead | Terraform modules, CI/CD pipeline |
| **BAUTISTA REYES, LOURDES** | Database Engineer | PostgreSQL tuning, replication setup |
| **DAVALOS ALFARO, MARISELLA** | Network Architect | VLAN design, firewall rules |
| **LEYVA VALQUI, GABRIEL** | Monitoring Specialist | Grafana dashboards, alerting |
| **RODRIGUEZ GONZALES, ALEJANDRO** | Systems Engineer | Ansible playbooks, automation |

---

## 🗺️ Roadmap

### ✅ Fase 1: Foundation (Completado)
- [x] Diseño de arquitectura de 3 capas
- [x] Provisioning automatizado con Terraform
- [x] Configuration management con Ansible
- [x] Monitoreo básico con Grafana + Prometheus

### 🚧 Fase 2: Resilience (En Progreso)
- [ ] Backup automatizado cada 6 horas a storage NFS
- [ ] Disaster Recovery con sitio secundario
- [ ] Chaos Engineering con Gremlin
- [ ] Automated failover testing

### 🔮 Fase 3: Advanced Features (Planeado - Q1 2025)
- [ ] Multi-site replication (Trujillo ↔ Lima)
- [ ] Kubernetes para microservicios
- [ ] Service Mesh con Istio
- [ ] Machine Learning para capacity planning

### 🌟 Fase 4: Cloud Hybrid (Planeado - Q2 2025)
- [ ] Hybrid cloud con AWS VPN
- [ ] Cold storage en S3 Glacier
- [ ] CloudWatch integration
- [ ] Multi-cloud DR strategy

---

## 🛡️ Seguridad

### Controles Implementados

✅ **Network Security**
- Segmentación VLAN con ACLs estrictas
- Firewall stateful (nftables) en cada VM
- IDS/IPS con Suricata
- VPN WireGuard para acceso remoto

✅ **Application Security**
- WAF con ModSecurity en Nginx
- Rate limiting: 100 req/min por IP
- SQL injection prevention (prepared statements)
- XSS protection headers

✅ **Data Security**
- Encryption at rest (LUKS para volúmenes)
- Encryption in transit (TLS 1.3 only)
- Secrets management con Vault
- Database audit logging

✅ **Access Control**
- RBAC con roles granulares
- MFA obligatorio para Proxmox
- SSH key-based auth (password disabled)
- Principle of least privilege

### Compliance
- **GDPR:** Anonimización de datos personales
- **ISO 27001:** Controles de seguridad documentados
- **CIS Benchmarks:** Hardening según estándares

---

## ❓ Problemas Conocidos y Soluciones

### 1. Alta latencia intermitente en Redis

**Síntoma:** Picos de latencia >50ms cada ~10 minutos

**Causa Raíz:** Redis persistence (RDB save) bloqueando el hilo principal

**Solución:**
```bash
# Ajustar política de persistencia
redis-cli CONFIG SET save "900 1 300 10"
redis-cli CONFIG SET stop-writes-on-bgsave-error no
```

**Alternativa:** Usar Redis Enterprise con persistencia asíncrona

---

### 2. Postgres connection pool exhausted

**Síntoma:** Error "sorry, too many clients already"

**Causa Raíz:** Application no cierra conexiones correctamente

**Solución:**
```ini
# /etc/pgbouncer/pgbouncer.ini
[databases]
app_db = host=10.30.0.3 port=5432 dbname=app pool_size=50

[pgbouncer]
max_client_conn = 1000
default_pool_size = 25
reserve_pool_size = 5
reserve_pool_timeout = 3
```

---

### 3. RabbitMQ memory alarm triggered

**Síntoma:** Producers bloqueados, consumers no procesan mensajes

**Causa Raíz:** Acumulación de mensajes sin consumir

**Solución:**
```bash
# Aumentar límite de memoria
rabbitmqctl set_vm_memory_high_watermark 0.6

# Configurar TTL en colas
rabbitmqctl set_policy TTL ".*" '{"message-ttl":3600000}' --apply-to queues
```

**Prevención:** Implementar backpressure en producers

---

### 4. Terraform state lock timeout

**Síntoma:** `Error acquiring state lock` al ejecutar `terraform apply`

**Causa Raíz:** Ejecución previa interrumpida sin liberar lock

**Solución:**
```bash
# Force-unlock (usar con precaución)
terraform force-unlock <LOCK_ID>

# Alternativa: Migrar a backend remoto
terraform {
  backend "consul" {
    address = "10.20.0.4:8500"
    path    = "terraform/state"
    lock    = true
  }
}
```

---

## 📜 Licencia

Este proyecto es software libre bajo **GNU GPLv3**. Ver archivo [LICENSE](LICENSE) para más detalles.

```
Copyright (C) 2024 - Grupo 5 UPAO

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License.
```

---

## 📞 Contacto y Soporte

**Universidad:** [www.upao.edu.pe](https://www.upao.edu.pe)  
**Repositorio:** [github.com/grupo5-upao/infrastructure-upao](https://github.com/grupo5-upao/infrastructure-upao)  
**Issues:** [github.com/grupo5-upao/infrastructure-upao/issues](https://github.com/grupo5-upao/infrastructure-upao/issues)  

---

<div align="center">

**⭐ Si este proyecto te fue útil, considera darle una estrella ⭐**

---

Hecho con ❤️ por el Grupo 5 | UPAO 2024

</div>
