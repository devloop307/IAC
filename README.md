# Infraestructura con Terraform + Docker

## 👥 Grupo e Integrantes

**Grupo:** 5

**Integrantes:**
- ALCANTARA RODRIGUEZ, PIERO
- BAUTISTA REYES, LOURDES
- DAVALOS ALFARO, MARISELLA
- LEYVA VALQUI,GABRIEL
- RODRIGUEZ GONZALES, ALEJANDRO

## 📋 Descripción

Este repositorio define una infraestructura local con Terraform y Docker compuesta por:

* **Nginx**: múltiples contenedores balanceables por puerto
* **PostgreSQL**: base de datos en red de persistencia
* **Redis**: caché en red de persistencia (sin volúmenes)
* **Grafana**: monitoreo conectado a múltiples redes.

## 🐳Imágenes Docker Utilizadas

| Servicio | Imagen | Justificación |
|----------|--------|---------------|
| **Nginx** | nginx:stable-alpine3.21-perl | Versión estable con soporte Perl para funcionalidades. |
| **PostgreSQL** | postgres:15-alpine | Versión LTS con Alpine para menor consumo de recursos. |
| **Redis** | redis:7.4.1-alpine | Versión estable y ligera, Alpine reduce el tamaño de imagen. |
| **Grafana** | grafana/grafana-enterprise:9.4.7 | Versión Enterprise con funcionalidades adicionales de monitoreo. |

## 🚀 Instrucciones de Uso

### 1. Clonar e inicializar
```bash
git clone git@github.com:picantitoDev/terraform-semana-02.git
cd terraform-semana-02
terraform init
```
### 2. Crear y moverse al ambiente de desarrollo:

```bash
# Listar workspaces existentes
terraform workspace list

# Crear workspace dev
terraform workspace new dev

# Moverse al workspace dev
terraform workspace select dev
```

### 3. Comandos a utilizar
```bash

# Ver plan de ejecución
terraform plan

# Aplicar infraestructura
terraform apply

# Verificar contenedores
docker ps

# Destruir infraestructura
terraform destroy
```

## 🔌 Justificación de Puertos

| Servicio | Puerto | Justificación |
|----------|--------|---------------|
| **Nginx** | 8080, 8081, 8082+ | Usamos el puerto 8080 en Nginx por ser la alternativa estándar al 80 y evitar conflictos.. |
| **PostgreSQL** | 5432 | Puerto estándar de PostgreSQL. |
| **Redis** | 6379 | Puerto por defecto de Redis. |
| **Grafana** | 3000 | Puerto por defecto de Grafana. |

## 💾 Consideraciones de Storage

**Redis sin volúmenes:** No se utilizan volúmenes en Redis porque se especificó en clase que es únicamente para caching, por lo que no importa perder datos en el reinicio del contenedor.

## 🔧 Problemas Durante el Desarrollo

### 1. Configuración de Redes
- **Problema:** Contenedores no se comunicaban entre capas
- **Solución:** Definir redes separadas (`app_net`, `persistence_net`, `monitor_net`)

## 🌐 Acceso a Servicios

- **Nginx:** http://localhost:8080, 8081, 8082...
- **Grafana:** http://localhost:3000 (admin/admin)
- **PostgreSQL:** localhost:5432
- **Redis:** localhost:6379