# 📦 Sistema de Inventario Distribuido - Bravo SAC

## 📋 Contexto y Problemática

### ¿Qué es Bravo SAC?

**Bravo SAC** es una empresa distribuidora de consumos y bienes que enfrenta desafíos críticos en su operación diaria:

#### Problemas Identificados

- **Inconsistencias en el inventario**: Actualizaciones desincronizadas que generan discrepancias en los registros
- **Falta de trazabilidad**: Sin auditoría clara de transacciones
- **Seguridad deficiente**: Ausencia de controles robustos para proteger información sensible
- **Pérdidas económicas**: Pedidos erróneos debido a datos incorrectos

### Solución Propuesta: Infraestructura como Código (IaC)

Implementación de una arquitectura basada en **Linux Containers (LXC)** gestionada con **Terraform** y automatizada con **Ansible**.

#### Beneficios Clave

✅ **Reproducibilidad**: Despliegue automatizado de entornos idénticos  
✅ **Versionamiento**: Control de cambios mediante Git  
✅ **Eficiencia de costos**: Ejecución local sin dependencia de nube pública  
✅ **Aislamiento**: Segregación por VLANs y contenedores  
✅ **Confiabilidad**: Sistema de colas para garantizar consistencia de datos

---

## 🏗️ Arquitectura de la Solución

### Vista General

La solución implementa una **arquitectura multinivel segmentada por VLANs**, donde cada capa tiene responsabilidades específicas y aislamiento de red.

```
Internet → Home Router → Proxmox Server → VLANs → Contenedores LXC
```

### Segmentación por VLANs

#### **VLAN 10 - Capa de Presentación** (`10.10.0.0/24`)

| Componente | IP | Función |
|------------|------------|---------|
| **Nginx Proxy** | `10.10.0.1` | Proxy inverso, terminación SSL|
| **NAT Gateway** | `10.10.0.2` | Gateway para salida a internet |
| **Grafana** | `10.10.0.3` | Dashboard de monitoreo y visualización |
| **Log Sync** | `10.10.0.4` | Centralización de logs para auditoría |

**Propósito**: Punto de entrada seguro y servicios de observabilidad.

#### **VLAN 20 - Capa de Aplicación** (`10.20.0.0/24`)

| Componente | IP | Función |
|------------|------------|---------|
| **RabbitMQ** | `10.20.0.1:8080` | Sistema de colas de mensajes |

**Propósito**: Lógica de negocio y procesamiento asíncrono confiable.

#### **VLAN 30 - Capa de Persistencia** (`10.30.0.0/22`)

| Componente | IP | Función |
|------------|------------|---------|
| **PostgreSQL** | `10.30.0.3` | Base de datos relacional principal |
| **cron-job** | `10.30.0.2:3412` | Gestor de trabajos programados |
| **Redis** | `10.30.0.1:3375` | Sistema de caché en memoria |
| **NTFS Storage** | - | Almacenamiento persistente de 1TB |

**Propósito**: Almacenamiento y persistencia de datos críticos.

#### **VLAN 40 - Red de Gestión** (`10.40.0.0/24`)

| Componente | IP | Función |
|------------|------------|---------|
| **NAT Instance** | `10.40.0.1` | Control de acceso a internet |
| **iptables/nftables** | - | Firewall y reglas de seguridad |

**Propósito**: Aislamiento y control de acceso seguro a internet.

### Flujo de Datos y Seguridad

```
Cliente → DNS (local-app.net) → Nginx Proxy (10.10.0.1)
                                      ↓
                                RabbitMQ (VLAN 20)
                                      ↓
                            Procesamiento Asíncrono
                                      ↓
                                PostgreSQL (VLAN 30)
```

**Principios de Seguridad Aplicados**:
- Defensa en profundidad (múltiples capas)
- Principio de mínimo privilegio
- Segregación de red
- Cifrado en tránsito

---

## 🗂️ Estructura del Proyecto

```
bravo-sac-infrastructure/
├── app/                      # Código de aplicación (futuro)
├── config/                   # Configuración de Ansible
│   ├── ansible.cfg           # Configuración principal de Ansible
│   ├── inventory.ini         # Inventario de hosts
│   └── group_vars/           # Variables por grupo
│       ├── all.yml           # Variables globales
│       └── via_proxy.yml     # Configuración de proxy SSH
│
├── iac/                      # Infraestructura como Código (Terraform)
│   ├── main.tf               # Configuración del provider Proxmox
│   ├── variables.tf          # Definición de variables
│   ├── credenciales.auto.tfvars  # Credenciales (NO versionar)
│   ├── vlans.tf              # Definición de VLANs
│   ├── proxy.tf              # Contenedor Nginx Proxy
│   ├── natgateway.tf         # Contenedor NAT Gateway
│   ├── grafana.tf            # Contenedor Grafana
│   └── logsync.tf            # Contenedor Log Sync
│
├── docs/                     # Documentación
│   ├── diagrams/             # Diagramas de arquitectura
│   └── guides/               # Guías de configuración
│
├── .gitignore                # Archivos excluidos de Git
└── README.md                 # Este documento
```

---
### Software

| Herramienta | Versión | Propósito |
|-------------|---------|-----------|
| **Terraform** | ≥ 1.5.0 | Provisión de infraestructura |
| **Ansible** | ≥ 2.14 | Automatización de configuración |
| **Proxmox VE** | ≥ 7.0 | Virtualización |
| **SSH** | OpenSSH 8+ | Acceso remoto |

### Templates Requeridos

En Proxmox, debe existir:
```bash
local:vztmpl/devuan-5.0-standard_5.0_amd64.tar.gz
```

Descargar template:
```bash
pveam download local devuan-5.0-standard_5.0_amd64.tar.gz
```

### Claves SSH

Generar par de claves para Ansible:
```bash
ssh-keygen -t ed25519 -f ~/.ssh/ansible -C "ansible@bravo-sac"
```

---

## 🚀 Instalación y Despliegue

### Paso 1: Clonar el Repositorio

```bash
git clone https://github.com/tu-organizacion/bravo-sac-infrastructure.git
cd bravo-sac-infrastructure
```

### Paso 2: Configurar Credenciales

Crear archivo `iac/credenciales.auto.tfvars`:

```hcl
proxmox_api_url   = "https://192.168.0.201:8006/"
proxmox_api_token = "root@pam!terraform=TU_TOKEN_AQUI"
ansible_pub_key   = "ssh-ed25519 AAAAC3Nza... ansible@bravo-sac"
```

**⚠️ IMPORTANTE**: Este archivo contiene credenciales sensibles. Ya está incluido en `.gitignore`.

### Paso 3: Crear Token de API en Proxmox

```bash
# En el servidor Proxmox
pveum user token add root@pam terraform --privsep=0
```

### Paso 4: Inicializar Terraform

```bash
cd iac/
terraform init
```

### Paso 5: Validar Configuración

```bash
terraform validate
terraform plan
```

### Paso 6: Desplegar Infraestructura

```bash
terraform apply
```

Confirmar con `yes` cuando se solicite.

**Tiempo estimado**: 5-10 minutos

### Paso 7: Verificar Contenedores

```bash
# En el servidor Proxmox
pct list
```

Salida esperada:
```
VMID       Status     Lock         Name
701        running                 proxy
702        running                 grafana
703        running                 logsynch
704        running                 nat-gateway
```

### Paso 8: Configurar Ansible

Editar `config/group_vars/all.yml`:

```yaml
ansible_user: root
ansible_ssh_private_key_file: ~/.ssh/ansible
ansible_ssh_common_args: '-o ProxyJump=root@192.168.0.222'
```

### Paso 9: Ejecutar Playbooks de Ansible

```bash
cd ../config/
ansible-playbook -i inventory.ini playbooks/initial_setup.yml
```

---

## 🌐 Configuración de Red

### Topología de Red

```
192.168.0.0/24 (Red LAN)
    ├── 192.168.0.1    → Home Router (Gateway)
    ├── 192.168.0.201  → Proxmox Host
    ├── 192.168.0.220  → NAT Gateway (eth0)
    └── 192.168.0.222  → Proxy (eth0)

10.10.0.0/24 (VLAN 10)
    ├── 10.10.0.1      → Proxy (eth1)
    ├── 10.10.0.2      → NAT Gateway (eth1)
    ├── 10.10.0.3      → Grafana
    └── 10.10.0.4      → Log Sync

10.40.0.0/24 (VLAN 40)
    └── 10.40.0.1      → NAT Gateway (eth2)
```

### Configuración del NAT Gateway

El contenedor `nat-gateway` actúa como router entre las VLANs y la red pública.

**Configuración de iptables** (dentro del contenedor):

```bash
# Habilitar IP forwarding
sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf

# Reglas NAT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Persistir reglas
iptables-save > /etc/iptables/rules.v4
```

### Configuración del Proxy Inverso (Nginx)

El contenedor `proxy` expone servicios de forma segura.

**Ejemplo de configuración** (`/etc/nginx/sites-available/default`):

```nginx
server {
    listen 80;
    server_name local-app.net;

    location / {
        proxy_pass http://10.20.0.2:3472;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /rabbitmq/ {
        proxy_pass http://10.20.0.1:8080/;
    }

    location /grafana/ {
        proxy_pass http://10.10.0.3:3000/;
    }
}
```

### DNS Local

Agregar entradas en el router o en `/etc/hosts` de los clientes:

```
192.168.0.222  local-app.net
192.168.0.222  local-dashboard.net
```

---

## 📦 Gestión de Contenedores

### Comandos Útiles de Proxmox

```bash
# Listar contenedores
pct list

# Iniciar contenedor
pct start 701

# Detener contenedor
pct stop 701

# Reiniciar contenedor
pct reboot 701

# Acceder a la consola
pct enter 701

# Ver logs
pct exec 701 -- journalctl -u nginx -f

# Snapshot (respaldo)
pct snapshot 701 backup-$(date +%Y%m%d)

# Restaurar snapshot
pct rollback 701 backup-20250101
```

### Acceso SSH

```bash
# Via proxy jump
ssh -J root@192.168.0.222 root@10.10.0.3

# Directo al proxy
ssh root@192.168.0.222
```

### Actualizar Contenedores

```bash
# Desde Ansible
ansible-playbook -i config/inventory.ini playbooks/update_all.yml

# Manual en cada contenedor
pct enter 701
apt update && apt upgrade -y
```

---

## 🔒 Seguridad

### Firewall de Proxmox Host

```bash
# Habilitar firewall en el nodo
pvesh set /nodes/proxmox/firewall/options -enable 1

# Permitir SSH
pvesh create /cluster/firewall/rules --type in --action ACCEPT --proto tcp --dport 22

# Permitir API de Proxmox
pvesh create /cluster/firewall/rules --type in --action ACCEPT --proto tcp --dport 8006

# Aplicar cambios
pvesh set /nodes/proxmox/firewall/options -enable 1
```

### Hardening de Contenedores

#### Deshabilitar Root Login por SSH

```bash
# En cada contenedor
sed -i 's/PermitRootLogin yes/PermitRootLogin without-password/' /etc/ssh/sshd_config
systemctl restart sshd
```

#### Configurar Fail2Ban

```bash
apt install fail2ban -y
systemctl enable fail2ban
systemctl start fail2ban
```

#### Limitar Acceso con iptables

```bash
# Ejemplo: Solo permitir SSH desde la red 192.168.0.0/24
iptables -A INPUT -p tcp --dport 22 -s 192.168.0.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
```

### Rotación de Credenciales

**Cambiar token de Proxmox**:
```bash
pveum user token remove root@pam terraform
pveum user token add root@pam terraform --privsep=0
```

Actualizar `iac/credenciales.auto.tfvars` y ejecutar:
```bash
terraform apply -var-file=credenciales.auto.tfvars
```

---

## 📊 Monitoreo y Logs

### Grafana Dashboard

Acceder a: `http://192.168.0.222/grafana/`

**Credenciales predeterminadas**:
- Usuario: `admin`
- Contraseña: `admin` (cambiar al primer acceso)

### Centralización de Logs

El contenedor `logsynch` (10.10.0.4) recibe logs de todos los servicios.

**Enviar logs con rsyslog**:

```bash
# En cada contenedor (ejemplo: proxy)
echo "*.* @@10.10.0.4:514" >> /etc/rsyslog.conf
systemctl restart rsyslog
```

### Consultar Logs

```bash
# Logs del sistema en logsynch
pct enter 703
tail -f /var/log/syslog

# Logs de Nginx
pct exec 701 -- tail -f /var/log/nginx/access.log

# Logs de RabbitMQ
# (Desde la interfaz web o archivos)
```

---

## 🔧 Troubleshooting

### Problema: Contenedor no inicia

**Síntomas**: `pct list` muestra status `stopped`

**Solución**:
```bash
# Ver logs del contenedor
pct status 701 --verbose
journalctl -xe | grep pve

# Intentar iniciar manualmente
pct start 701

# Si falla por falta de recursos
pvesh get /nodes/proxmox/status
```

### Problema: No hay conectividad entre VLANs

**Síntomas**: Ping entre contenedores falla

**Diagnóstico**:
```bash
# Verificar interfaces en Proxmox
ip a | grep vmbr0

# Verificar configuración de VLANs
cat /etc/network/interfaces

# Verificar forwarding en NAT Gateway
pct exec 704 -- sysctl net.ipv4.ip_forward
```

**Solución**:
```bash
# Reiniciar networking en Proxmox
systemctl restart networking

# Verificar reglas de iptables en NAT Gateway
pct exec 704 -- iptables -L -n -v
```

### Problema: Terraform falla al aplicar

**Síntomas**: Error de autenticación o timeout

**Solución**:
```bash
# Verificar conectividad
curl -k https://192.168.0.201:8006/

# Verificar token
pveum user token list root@pam

# Regenerar token si es necesario
pveum user token add root@pam terraform --privsep=0

# Limpiar estado de Terraform
terraform refresh
terraform plan
```

### Problema: SSH no funciona via ProxyJump

**Síntomas**: `ssh: connect to host 10.10.0.3 port 22: No route to host`

**Solución**:
```bash
# Verificar que proxy esté corriendo
pct status 701

# Verificar conectividad desde el proxy
pct exec 701 -- ping -c 3 10.10.0.3

# Verificar configuración SSH
cat ~/.ssh/config

# Debe contener:
Host proxy
  HostName 192.168.0.222
  User root
  IdentityFile ~/.ssh/ansible

Host grafana
  HostName 10.10.0.3
  User root
  ProxyJump proxy
  IdentityFile ~/.ssh/ansible
```

---

## 🛠️ Mantenimiento

### Respaldos

#### Backup de Configuración

```bash
# Backup de Terraform state
cd iac/
terraform state pull > terraform.tfstate.backup-$(date +%Y%m%d)

# Backup de configuración Ansible
tar -czf ansible-config-$(date +%Y%m%d).tar.gz config/
```

#### Backup de Contenedores

```bash
# Snapshot de contenedor
pct snapshot 701 "backup-$(date +%Y%m%d-%H%M)"

# Backup completo (dump)
vzdump 701 --dumpdir /var/lib/vz/dump --mode snapshot

# Backup automatizado (agregar a cron)
0 2 * * 0 vzdump 701 702 703 704 --dumpdir /var/lib/vz/dump --mode snapshot
```

### Actualizaciones

#### Actualizar Templates de LXC

```bash
pveam update
pveam available --section system
pveam download local devuan-5.0-standard_5.0_amd64.tar.gz
```

#### Actualizar Proxmox

```bash
apt update
apt dist-upgrade
pve7to8 --check  # Si migras de v7 a v8
```

#### Actualizar Terraform Provider

```bash
cd iac/
terraform init -upgrade
terraform plan
terraform apply
```

### Escalado

#### Añadir nuevo contenedor

1. Crear archivo `iac/nuevo_servicio.tf`:

```hcl
resource "proxmox_virtual_environment_container" "nuevo_servicio" {
  vm_id     = 705
  node_name = "proxmox"
  
  initialization {
    hostname = "nuevo-servicio"
    
    ip_config {
      ipv4 {
        address = "10.20.0.5/24"
        gateway = "10.10.0.2"
      }
    }
    
    user_account {
      keys = [var.ansible_pub_key]
    }
  }
  
  operating_system {
    type             = "alpine"
    template_file_id = "local:vztmpl/devuan-5.0-standard_5.0_amd64.tar.gz"
  }
  
  cpu { cores = 2 }
  memory { dedicated = 1024 }
  
  network_interface {
    name    = "eth0"
    bridge  = "vmbr0"
    vlan_id = 20
  }
  
  disk {
    datastore_id = "local-lvm"
    size         = 16
  }
  
  unprivileged = false
}
```

2. Aplicar cambios:
```bash
terraform apply
```

3. Añadir al inventario de Ansible:
```ini
[nuevo_servicio_group]
nuevo_servicio ansible_host=10.20.0.5
```

---

## 📄 Licencia

Este proyecto es propiedad de **Bravo SAC** y está protegido bajo licencia propietaria.

---

**Última actualización**: Octubre 2025  
**Versión**: 1.0.0  
**Autor**: Equipo de DevOps - Bravo SAC
