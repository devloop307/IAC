# Infraestructura con Terraform + Docker

Este repositorio define, mediante **Terraform** y el **provider de Docker**, una infraestructura local compuesta por:

* **Nginx**: múltiples contenedores balanceables por puerto (usando `count` y un **puerto base**).
* **PostgreSQL**: base de datos en la red de **persistencia**.
* **Redis**: caché/cola en la red de **persistencia**.
* **Grafana Enterprise**: observabilidad/visualización en la red de **monitoreo** y de **aplicación**.

Las piezas se conectan a través de **tres redes Docker** separadas para aislar capas:

* `app_net` → capa de aplicación (Nginx, Grafana).
* `persistence_net` → capa de datos (PostgreSQL, Redis).
* `monitor_net` → capa de observabilidad (Grafana).

> **Objetivo**: que cualquier persona pueda clonar el repo, provisionar todo en su máquina y acceder a los servicios sin pasos manuales adicionales.

---

## 🔧 Requisitos previos

1. **Terraform**

   * Instala Terraform (>= 1.5 recomendado). Verifica con:

     ```bash
     terraform -version
     ```
2. **Docker Engine** (no es necesario Docker Desktop)

   * Verifica que Docker esté instalado:

     ```bash
     docker -v
     ```
   * Asegura que el servicio esté corriendo:

     * **Linux systemd**:

       ```bash
       sudo systemctl status docker    # debe mostrar active (running)
       # si no está activo:
       sudo systemctl start docker
       ```
     * **Permisos sin sudo (opcional)**:

       ```bash
       sudo usermod -aG docker $USER
       # cierra sesión y vuelve a entrar para aplicar el grupo
       ```

> Si usas **WSL**: habilita el servicio Docker del host (Docker Desktop o dockerd en Linux) y comparte el daemon con WSL.

---

## 🗂️ Estructura del proyecto

```text
.
├── .gitignore                # Ignora .terraform/, .tfstate y *.tfvars (sensibles)
├── .terraform.lock.hcl       # Lock de versiones de providers (reproducibilidad)
├── main.tf                   # Configuración del provider Docker, backends, etc.
├── networks.tf               # Redes Docker: app_net, persistence_net, monitor_net
├── nginx.tf                  # Contenedores Nginx (count) y mapeo de puertos desde base
├── grafana.tf                # Contenedor Grafana (app_net + monitor_net)
├── postgre.tf                # Contenedor PostgreSQL (env: usuario/password/db)
├── redis.tf                  # Contenedor Redis (persistence_net)
├── variables.tf              # Variables de entrada (puertos, credenciales, etc.)
└── README.md                 # Este documento
```

**Resumen de cada archivo**

* **`main.tf`**: declara el `provider "docker"` y cualquier configuración global.
* **`networks.tf`**: crea redes `bridge` separadas por dominio (app/datos/monitoreo).
* **`nginx.tf`**: levanta *N* contenedores Nginx, exponiendo puertos consecutivos a partir de `nginx_base_port`.
* **`grafana.tf`**: ejecuta Grafana Enterprise, conectado a `app_net` y `monitor_net`.
* **`postgre.tf`**: instancia PostgreSQL en `persistence_net` con usuario/contraseña/db vía variables de entorno.
* **`redis.tf`**: instancia Redis en `persistence_net`.
* **`variables.tf`**: centraliza variables (conteo y puertos de Nginx, credenciales de PostgreSQL, etc.).

---

## ⚙️ Variables configurables

Define estos valores en `terraform.tfvars` (no se versiona) o pásalos por CLI con `-var`.

```hcl
# Escalado horizontal de Nginx
nginx_container_count = 3          # número de contenedores Nginx
nginx_base_port       = 3000       # primer puerto publicado (luego 3001, 3002, ...)

# Credenciales de PostgreSQL
postgres_user     = "appuser"
postgres_password = "changeme"
postgres_db       = "appdb"

# (Opcional) Imagenes/tag
# grafana_image  = "grafana/grafana-enterprise:9.4.7"
# nginx_image    = "nginx:alpine"
# postgres_image = "postgres:16-alpine"
# redis_image    = "redis:7-alpine"
```

**Cómo funcionan los puertos de Nginx**

* El contenedor `i` (empezando en 0) publica `nginx_base_port + i`.
* Ejemplo con `nginx_container_count=3` y `nginx_base_port=3000` → expone **3000, 3001, 3002**.

> Consejo: evita choques de puertos (ej. si otro servicio usa 3000, elige otra base como 8080).

---

## 🚀 Pasos para desplegar

1. **Clonar el repositorio**

```bash
git clone <URL_DE_TU_REPO>
cd <DIRECTORIO_DEL_REPO>
```

2. **(Opcional) Crear `terraform.tfvars`** con tus valores (ver sección de variables)

3. **Inicializar Terraform** (descarga providers y prepara el directorio)

```bash
terraform init
```

4. **Validar la configuración**

```bash
terraform validate
```

5. **Planear la ejecución** (muestra qué se va a crear)

```bash
terraform plan
# o sobreescribir variables al vuelo
# terraform plan -var="nginx_container_count=2" -var="nginx_base_port=8080"
```

6. **Aplicar cambios**

```bash
terraform apply -auto-approve
# o con variables en línea
# terraform apply -auto-approve \
#   -var="nginx_container_count=3" -var="nginx_base_port=3000"
```

7. **Verificar contenedores**

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
```

---

## 🌐 Acceso a los servicios

| Servicio       | URL / Host                                                                         | Puerto                           | Notas                                                          |
| -------------- | ---------------------------------------------------------------------------------- | -------------------------------- | -------------------------------------------------------------- |
| **Nginx**      | [http://localhost:\`{nginx\_base\_port](http://localhost:`{nginx_base_port) + i}\` | `nginx_base_port`, `+1`, `+2`, … | Con `count=3` y base `3000` → 3000, 3001, 3002                 |
| **Grafana**    | [http://localhost:3000](http://localhost:3000)                                     | 3000                             | Cambia en `grafana.tf` si deseas otro puerto                   |
| **PostgreSQL** | localhost                                                                          | 5432                             | Variables: `postgres_user`, `postgres_password`, `postgres_db` |
| **Redis**      | localhost                                                                          | 6379                             | Sin auth por defecto salvo que la definas                      |

> Para probar PostgreSQL: `psql -h localhost -p 5432 -U <user> -d <db>`
>
> Para probar Redis: `redis-cli -h localhost -p 6379 PING` (debe responder `PONG`).

---

## 🛠️ Comandos útiles

Formatear y chequear:

```bash
terraform fmt -recursive
terraform validate
```

Estado y recursos:

```bash
terraform state list
terraform show
```

Escalar Nginx (p. ej., de 2 a 4 instancias):

```bash
terraform apply -auto-approve -var="nginx_container_count=4"
```

Destruir la infraestructura:

```bash
terraform destroy -auto-approve
```

Logs y diagnóstico con Docker:

```bash
docker logs <nombre_contenedor>
docker exec -it <nombre_contenedor> sh
```

### (Opcional) Workspaces de Terraform

Si tus recursos usan `terraform.workspace` en los nombres (p. ej., `name = "grafana-${terraform.workspace}"`), puedes aislar entornos:

```bash
terraform workspace list
terraform workspace new dev
terraform workspace select dev
terraform apply -auto-approve
```

> Cada workspace mantiene su propio estado; útil para *dev/qa/prod* locales.

---

## ✅ Buenas prácticas

* **`/.terraform/` y archivos de estado** (`*.tfstate`, `*.tfstate.backup`) **no deben versionarse**. Mantén tu `.gitignore` actualizado.
* **`terraform.tfvars`** puede contener secretos (contraseñas). **No lo subas** al repositorio.
* **`.terraform.lock.hcl`**:

  * **Recomendado** versionarlo para reproducibilidad de providers.
  * En ejercicios académicos/prototipos, puedes omitirlo si cambias de máquina/proveedor con frecuencia.
* Usa `terraform fmt` y `terraform validate` en tus PRs.
* Documenta los puertos que expones y evita colisiones.

---

## 🧯 Problemas comunes y soluciones rápidas

1. **`Cannot connect to the Docker daemon`**

   * Asegura que Docker está corriendo: `sudo systemctl start docker`.
   * En Linux, añade tu usuario al grupo `docker`: `sudo usermod -aG docker $USER` y reinicia sesión.
   * En WSL, valida que el daemon del host está accesible.

2. **Conflicto de puertos** (p. ej., 3000 ya en uso)

   * Cambia `nginx_base_port` (ej. 8080) o ajusta el puerto de Grafana en `grafana.tf`.
   * Verifica qué usa el puerto: `sudo lsof -i :3000` (Linux/macOS) o `netstat -ano` (Windows).

3. **`Error: Reference to undeclared input variable`**

   * Te falta declarar la variable en `variables.tf` **o** definirla en `terraform.tfvars`.

4. **`Failed to install provider` / problemas con plugins**

   * Ejecuta `terraform init -upgrade` para actualizar índices y providers.

5. **Contenedores sin iniciar o reiniciándose**

   * Revisa `docker logs <contenedor>` (p. ej., Postgres puede fallar si faltan variables o hay volúmenes previos).
   * Elimina volúmenes antiguos si corresponde (`docker volume ls` / `docker volume rm`).

6. **WSL + puertos localhost**

   * Asegura que los puertos están expuestos hacia Windows si el daemon corre en WSL/VM. Prueba desde dentro y fuera de WSL (`curl http://localhost:3000`).

---

## 📌 Notas finales

* Este stack está pensado para **desarrollo local**. Para producción, añade:

  * Persistencia de datos en volúmenes/bind mounts gestionados por Terraform.
  * Autenticación/contraseñas seguras vía variables de entorno o `tfvars` encriptados.
  * Reverse proxy / TLS (Traefik, Nginx con certificados) si expones servicios fuera del host.

¡Listo! Con esto deberías poder levantar Nginx, PostgreSQL, Redis y Grafana con un par de comandos y comenzar a iterar sobre tu aplicación y dashboards.
