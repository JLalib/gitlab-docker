# 🐙 GitLab Docker - Servidor Git Autohospedado con CI/CD

[![GitHub](https://img.shields.io/badge/GitHub-Repo-blue?logo=github)](https://github.com/JLalib/gitlab-docker)
[![Docker](https://img.shields.io/badge/Docker-Image-blue?logo=docker)](https://hub.docker.com/r/gitlab/gitlab-ce)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

## 📋 Descripción general

**GitLab en Docker** es un servidor Git completo autohospedado que proporciona control de versiones, CI/CD integrado, gestión de proyectos, usuarios y permisos, webhooks, registry Docker privado, y todas las features de GitHub Enterprise sin necesidad de suscripción, completamente bajo tu control en Docker. Ideal para equipos de desarrollo y homelabs profesionales.

**Propuesta clave:** Git server enterprise-grade self-hosted. Repositorios privados/públicos ilimitados. CI/CD pipelines integradas (.gitlab-ci.yml). Gestión usuarios multi-tenant. Webhooks, API REST completa. Docker registry privado integrado. SSH + HTTPS Git access. Backup automático.

## ✨ Características principales

- **Git repositories** - Repositorios privados/públicos ilimitados. SSH + HTTPS access. Full Git features.
- **CI/CD integrado** - Pipelines nativas .gitlab-ci.yml. Runners auto-escalables. Artifacts, caches.
- **Container registry** - Docker registry privado integrado. Push/pull imágenes. Integración CI/CD.
- **Gestión usuarios** - Multi-tenant. Roles (Admin, Maintainer, Developer). LDAP/OAuth opcional.
- **Merge Requests** - Code review integrado. Discussions, aprobaciones. Auto-merge.
- **Issues + Epics** - Tracking tareas. Epics (subissues). Labels, asignaciones, milestones.
- **Webhooks + API** - API REST completa. Webhooks para eventos. Integraciones externas.
- **Backup automático** - Backup scheduled. Restore fácil. Disaster recovery.

## 📋 Requisitos del sistema

- **Docker & Docker Compose v2+**
- **8 GB - 16 GB RAM mínimo** (GitLab consume recursos)
- **50 GB espacio disco mínimo** (repositorios, CI/CD artifacts)
- **/mnt/media/gitlab** directorio accesible (o configurar ruta)
- **Puertos TCP:** 80, 443 (HTTP/HTTPS), 22 (SSH, opcional)
- **Dominio válido externo** (NO localhost) - ej: gitlab.tudominio.com
- **CPU:** 4+ cores (GitLab es pesado, 6+ recomendado)
- **Acceso internet** (para webhooks, OAuth, notificaciones)
- **PostgreSQL o SQLite** (integrado en contenedor)

> ⚠️ **IMPORTANTE:** GitLab REQUIERE un dominio externo válido (no localhost). HTTPS altamente recomendado. Consume muchos recursos (RAM, CPU, disco). Resource-heavy: GitLab es bastante pesado. Mínimo 8GB RAM. En Raspberry Pi puede no funcionar bien. Mejor en servidor dedicado o VPS.

## 🐳 Instalación

### Estructura de directorios

GitLab necesita carpetas para config, logs y datos:

```
/mnt/media/gitlab/
├── config/  # Configuración GitLab
├── logs/    # Logs (aplicación)
└── data/    # Datos (repositorios, DB, etc)
```

### Paso 1: Preparar estructura directorios

```bash
# Crear carpetas GitLab
sudo mkdir -p /mnt/media/gitlab/{config,logs,data}

# Permisos
sudo chown -R 1000:1000 /mnt/media/gitlab
sudo chmod -R 755 /mnt/media/gitlab
```

### Paso 2: Crear docker-compose.yml

```bash
mkdir -p ~/gitlab && cd ~/gitlab
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    hostname: gitlab.example.com
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        # Configuración externa (CAMBIAR a tu dominio)
        external_url 'http://gitlab.example.com'
        # Si tienes HTTPS (recomendado)
        # external_url 'https://gitlab.example.com'
        # nginx['ssl_certificate'] = '/etc/gitlab/ssl/gitlab.crt'
        # nginx['ssl_certificate_key'] = '/etc/gitlab/ssl/gitlab.key'
        # PostgreSQL (usar SQLite integrado es más simple)
        # gitlab_rails['db_adapter'] = 'postgresql'
        # SSH Port (si quieres cambiar de 22)
        # gitlab_rails['gitlab_shell_ssh_port'] = 2424
    ports:
      - "80:80"
      - "443:443"
      - "22:22"
    volumes:
      - /mnt/media/gitlab/config:/etc/gitlab
      - /mnt/media/gitlab/logs:/var/log/gitlab
      - /mnt/media/gitlab/data:/var/opt/gitlab
    shm_size: '256m'
EOF
```

### Paso 3: Configurar dominio y HTTPS

> ⚠️ **IMPORTANTE:** Cambia `gitlab.example.com` en `docker-compose.yml` a tu dominio real (ej: `gitlab.miempresa.com`). Sin esto GitLab no funcionará correctamente.

### Paso 4: Iniciar GitLab

```bash
docker compose up -d
# GitLab tarda 2-3 MINUTOS en iniciar completamente
docker compose logs -f
# Espera a ver "gitlab Reconfigured!" en logs
```

### Paso 5: Obtener contraseña root inicial

```bash
docker compose exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
# O si no funciona, intenta:
docker compose exec -it gitlab cat /etc/gitlab/initial_root_password
```

### Acceder a GitLab

🐙 **GitLab Web UI:** `http://gitlab.example.com` (o `https://` si configuraste SSL)

**Credenciales iniciales**
- Usuario: `root`
- Contraseña: (obtener con comando arriba)

> ⚠️ **IMPORTANTE:** Cambia la contraseña root después del primer login! Settings → Change password.

💡 **SSH access:** `ssh://git@gitlab.example.com/user/project.git` (reemplaza dominio y ruta)

## ⚙️ Configuración

1. **Dominio externo** - Edita `external_url` en `docker-compose.yml` con tu dominio real
2. **HTTPS/SSL** - Descomenta líneas SSL y coloca certificados en `/mnt/media/gitlab/config/ssl/`
3. **Puerto SSH** - Si el puerto 22 está ocupado, cambia `gitlab_rails['gitlab_shell_ssh_port']` y mapea en `ports`
4. **PostgreSQL externo** - Descomenta y configura `gitlab_rails['db_adapter'] = 'postgresql'` con credenciales
5. **Recursos** - Ajusta `shm_size` si necesitas más memoria compartida (mínimo 256m)
6. **Timezone** - Añade `gitlab_rails['time_zone'] = 'Europe/Madrid'` en GITLAB_OMNIBUS_CONFIG
7. **Email/SMTP** - Configura `gitlab_rails['smtp_*']` para notificaciones
8. **Backup schedule** - Configura `gitlab_rails['backup_keep_time'] = 604800` (7 días)

## 🚀 Primeros pasos

1. **Login y cambiar contraseña**
   - Abre `http://gitlab.example.com`
   - Login: usuario `root`, contraseña (obtenida arriba)
   - Click avatar (esquina superior derecha) → Settings → Account → Change password
   - Establece contraseña nueva segura

2. **Crear proyecto nuevo**
   - Click "New project" (botón azul)
   - Opción: "Create blank project"
   - Project name: (ej: "mi-primer-proyecto")
   - Project slug: auto-generado
   - Visibility: Private (privado) o Public (público)
   - Click "Create project"

3. **Clonar proyecto y hacer commit**
   ```bash
   # Clonar proyecto (HTTPS)
   git clone http://gitlab.example.com/root/mi-primer-proyecto.git
   cd mi-primer-proyecto
   
   # O SSH (si tienes SSH configurado)
   git clone ssh://git@gitlab.example.com/root/mi-primer-proyecto.git
   
   # Hacer cambios y commit
   echo "Hello GitLab" > README.md
   git add .
   git commit -m "Initial commit"
   git push -u origin main
   ```

4. **Crear usuario nuevo**
   - Admin area (icono admin en sidebar)
   - Users → New user
   - Fill: name, username, email
   - Set password, click Create user

5. **Agregar usuario a proyecto**
   - Project → Members → Add members
   - Selecciona usuario, rol (Developer, Maintainer, etc)
   - Click "Add to project"

6. **Configurar CI/CD (pipelines)**
   - En proyecto, crear archivo `.gitlab-ci.yml`:
   ```yaml
   stages:
     - build
     - test
   
   build:
     stage: build
     script:
       - echo "Building..."
       - ls -la
   
   test:
     stage: test
     script:
       - echo "Testing..."
       - echo "Test passed!"
   ```
   - Commit y push → CI/CD pipeline ejecuta automático

7. **Usar Container Registry**
   ```bash
   # Login al registry GitLab
   docker login gitlab.example.com
   # Username: usuario GitLab, Password: access token
   
   # Build y push imagen
   docker build -t gitlab.example.com/root/mi-imagen:latest .
   docker push gitlab.example.com/root/mi-imagen:latest
   
   # Pull desde registry
   docker pull gitlab.example.com/root/mi-imagen:latest
   ```

## 💡 Casos de uso

- **Equipos de desarrollo:** Git server profesional. CI/CD integrado. Control privacidad.
- **DevOps:** Container registry privado. CI/CD pipelines. Artifacts.
- **Homelabs:** Repositorios privados sin GitHub/GitLab.com. Backup local.
- **Educación:** Servidor Git para estudiantes. Privado y controlado.
- **Compliance:** Datos completamente on-premise. Sin cloud externo.

## 🔒 Acceso remoto seguro

Para acceso remoto seguro se recomienda:

- **Reverse Proxy** (Nginx Proxy Manager, Traefik, Caddy) con Let's Encrypt automático
- **VPN** (WireGuard, Tailscale) para acceso solo a red privada
- **Authelia/Keycloak** para SSO y 2FA adicional
- **Fail2ban** / **CrowdSec** para protección contra fuerza bruta
- **Firewall** restringiendo puertos solo a IPs necesarias

## 🛠️ Gestión y mantenimiento

### Ver estado
```bash
docker compose ps
```

### Ver logs
```bash
docker compose logs -f gitlab
```

### Detener GitLab
```bash
docker compose down
```

### Actualizar a versión nueva
```bash
docker compose pull
docker compose up -d
```

### Backup (muy importante!)
```bash
docker compose exec -it gitlab gitlab-backup create
# Encuentra el archivo backup en /mnt/media/gitlab/data/backups/
ls -lh /mnt/media/gitlab/data/backups/
```

### Restore desde backup
```bash
docker compose down
docker compose exec -it gitlab gitlab-backup restore BACKUP=timestamp_of_backup
```

### Cambiar puerto SSH
```yaml
# En docker-compose.yml, modifica GITLAB_OMNIBUS_CONFIG:
gitlab_rails['gitlab_shell_ssh_port'] = 2424
# Y en ports:
- "2424:22"
```
```bash
docker compose up -d
```

### GitLab Community Edition vs Enterprise Edition

- **Community Edition (CE):** Gratuita, open source. Features completas Git/CI/CD. Perfect para homelabs y pequeños equipos.
- **Enterprise Edition (EE):** Pago (suscripción GitLab). Features premium: LDAP sync, Geo replication, advanced security scanning, etc.

> Para homelab, usa **CE** (imagen: `gitlab/gitlab-ce:latest`). Es completamente funcional y gratuita.

## 📝 Licencia

Este proyecto está bajo licencia MIT. Ver [LICENSE](LICENSE) para más detalles.

GitLab Community Edition es software open source bajo licencia MIT/Expat.

---

> 📖 **Guía completa:** [Cómo instalar GitLab en Docker - Servidor Git autohospedado](https://genbyte.blogspot.com/2026/09/como-instalar-gitlab-en-docker-servidor.html)