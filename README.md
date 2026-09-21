# 🐙 GitLab Docker - Servidor Git Autohospedado con CI/CD

[![GitHub Stars](https://img.shields.io/github/stars/genbyte/gitlab-docker?style=social)](https://github.com/genbyte/gitlab-docker)
[![GitHub Forks](https://img.shields.io/github/forks/genbyte/gitlab-docker?style=social)](https://github.com/genbyte/gitlab-docker)
[![Docker Pulls](https://img.shields.io/docker/pulls/gitlab/gitlab-ce?label=Docker%20Pulls&logo=docker)](https://hub.docker.com/r/gitlab/gitlab-ce)
[![License](https://img.shields.io/github/license/genbyte/gitlab-docker)](LICENSE)
[![Blog Post](https://img.shields.io/badge/📖_Blog_Post-Genbyte-blue)](https://genbyte.blogspot.com/2026/09/como-instalar-gitlab-en-docker-servidor.html)

---

## 📋 Descripción general

**GitLab Docker** es una implementación lista para producción de **GitLab Community Edition (CE)** en contenedores Docker. Proporciona un servidor Git completo autohospedado con **CI/CD integrado**, proyectos privados/públicos ilimitados, gestión de usuarios y permisos, webhooks, **registry Docker privado**, backup automático y todas las características de GitHub Enterprise sin suscripción — completamente bajo tu control en tu homelab o servidor.

> 🎯 **Propuesta clave**: Servidor Git enterprise-grade self-hosted. Repositorios ilimitados. CI/CD nativo (.gitlab-ci.yml). Container registry integrado. Zero cloud, 100% privacidad. Community Edition gratuita y production-ready.

---

## ✨ Características principales

- 🔐 **Repositorios Git** — Privados/públicos ilimitados, acceso SSH + HTTPS, funcionalidad Git completa
- ⚙️ **CI/CD Integrado** — Pipelines nativas con `.gitlab-ci.yml`, runners auto-escalables, artifacts y caches
- 📦 **Container Registry** — Docker registry privado integrado, push/pull imágenes, integración CI/CD nativa
- 👥 **Gestión Multi-tenant** — Usuarios, grupos, roles (Admin, Maintainer, Developer), LDAP/OAuth opcional
- 🔄 **Merge Requests** — Code review integrado, discussions, aprobaciones, auto-merge
- 📋 **Issues + Epics** — Tracking de tareas, epics (sub-issues), labels, asignaciones, milestones
- 🌐 **Webhooks + API** — API REST completa, webhooks para eventos, integraciones externas
- 💾 **Backup Automático** — Backup programado, restore fácil, disaster recovery
- 📚 **Wiki Integrado** — Documentación por proyecto
- 📦 **Package Registry** — NPM, Maven, PyPI, Conan, etc.

---

## 📋 Requisitos del sistema

- ✅ **Docker & Docker Compose v2+**
- 🧠 **RAM**: 8 GB mínimo (16 GB recomendado) — *GitLab consume muchos recursos*
- 💾 **Disco**: 50 GB mínimo (repositorios, CI/CD artifacts, registry)
- 📁 **Directorio accesible**: `/mnt/media/gitlab` (o ruta personalizada)
- 🌐 **Puertos TCP**: 80 (HTTP), 443 (HTTPS), 22 (SSH Git — opcional)
- 🔗 **Dominio válido externo** — **OBLIGATORIO** (NO localhost), ej: `gitlab.tudominio.com`
- ⚡ **CPU**: 4+ cores (6+ recomendado)
- 🌍 **Acceso a Internet** — Para webhooks, OAuth, notificaciones, Let's Encrypt
- 🐘 **PostgreSQL o SQLite** — Integrado en contenedor (SQLite por defecto)

> ⚠️ **IMPORTANTE**: GitLab **REQUIERE** un dominio externo válido (no localhost). HTTPS altamente recomendado. En Raspberry Pi puede no funcionar bien — mejor en servidor dedicado o VPS.

---

## 🐳 Instalación

### Paso 1: Preparar estructura de directorios

```bash
# Crear carpetas GitLab
sudo mkdir -p /mnt/media/gitlab/{config,logs,data}

# Permisos (UID/GID 1000 para el contenedor)
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

> ⚠️ **IMPORTANTE**: Cambia `gitlab.example.com` en `docker-compose.yml` a tu dominio real (ej: `gitlab.miempresa.com`). Sin esto GitLab no funcionará correctamente.

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

# O si no funciona:
docker compose exec -it gitlab cat /etc/gitlab/initial_root_password
```

### Acceder a GitLab

| Acceso | URL |
|--------|-----|
| 🌐 **Web UI** | `http://gitlab.example.com` (o `https://` si configuraste SSL) |
| 👤 **Usuario** | `root` |
| 🔑 **Contraseña** | (obtenida con comando arriba) |

> ⚠️ **IMPORTANTE**: Cambia la contraseña root después del primer login!  
> Settings → Change password.

💡 **SSH access**: `ssh://git@gitlab.example.com/user/project.git` (reemplaza dominio y ruta)

---

## ⚙️ Configuración

1. **Dominio externo** — Edita `external_url` en `docker-compose.yml` (sección `GITLAB_OMNIBUS_CONFIG`)
2. **HTTPS/SSL** — Descomenta líneas SSL y coloca certificados en `/mnt/media/gitlab/config/ssl/`
3. **Puerto SSH** — Cambia `gitlab_rails['gitlab_shell_ssh_port']` y mapea en `ports:`
4. **PostgreSQL externo** — Configura `gitlab_rails['db_adapter'] = 'postgresql'` + credenciales
5. **Recursos** — Ajusta `shm_size` si ves errores de memoria compartida
6. **Timezone** — Añade `gitlab_rails['time_zone'] = 'Europe/Madrid'` en configuración
7. **Email/SMTP** — Configura `gitlab_rails['smtp_*']` para notificaciones
8. **Backup schedule** — Programa cron en host o usa GitLab backup integrado

---

## 🚀 Primeros pasos

1. **Login y cambiar contraseña**
   - Abre `http://gitlab.example.com`
   - Login: usuario `root`, contraseña (obtenida arriba)
   - Click avatar (esquina superior derecha) → Settings → Account → Change password
   - Establece contraseña nueva segura

2. **Crear proyecto nuevo**
   - Click "New project" (botón azul)
   - Opción: "Create blank project"
   - Project name: (ej: `mi-primer-proyecto`)
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
   - Admin area (icono admin en sidebar) → Users → New user
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

---

## 💡 Casos de uso

- 👨‍💻 **Equipos de desarrollo** — Git server profesional, CI/CD integrado, control total de privacidad
- 🚀 **DevOps** — Container registry privado, CI/CD pipelines, artifacts, deployment automático
- 🏠 **Homelabs** — Repositorios privados sin GitHub/GitLab.com, backup local, zero cloud
- 🎓 **Educación** — Servidor Git para estudiantes, privado y controlado
- 🏛️ **Compliance** — Datos completamente on-premise, sin cloud externo, auditoría completa

---

## 🔒 Acceso remoto seguro

### Opción A: Reverse Proxy (Nginx Proxy Manager / Traefik / Caddy)
- Termina SSL en el proxy
- `external_url 'https://gitlab.tudominio.com'` en GitLab
- Proxy pasa tráfico HTTP al puerto 80 del contenedor

### Opción B: Let's Encrypt integrado (GitLab Omnibus)
```yaml
environment:
  GITLAB_OMNIBUS_CONFIG: |
    external_url 'https://gitlab.example.com'
    letsencrypt['enable'] = true
    letsencrypt['contact_emails'] = ['admin@example.com']
```
> Requiere puerto 80 accesible desde Internet para validación HTTP-01

### Opción C: Tailscale / WireGuard / VPN
- Acceso solo via VPN, sin exposición pública
- Ideal para homelabs privados

---

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

### Backup (¡Muy importante!)
```bash
# Crear backup
docker compose exec -it gitlab gitlab-backup create

# Encuentra el archivo backup
ls -lh /mnt/media/gitlab/data/backups/
```

### Restore desde backup
```bash
docker compose down
docker compose exec -it gitlab gitlab-backup restore BACKUP=timestamp_of_backup
docker compose up -d
```

### Cambiar puerto SSH
```yaml
# En docker-compose.yml, modifica GITLAB_OMNIBUS_CONFIG:
gitlab_rails['gitlab_shell_ssh_port'] = 2424

# Y en ports:
ports:
  - "2424:22"
```
```bash
docker compose up -d
```

---

## 📝 Licencia

Este proyecto está bajo licencia **MIT** — ver archivo [LICENSE](LICENSE) para detalles.

GitLab Community Edition es software open source bajo licencia **MIT Expat**.
GitLab Enterprise Edition requiere suscripción comercial.

---

> 📖 **Guía completa en el blog**: [Cómo instalar GitLab en Docker - Servidor Git autohospedado](https://genbyte.blogspot.com/2026/09/como-instalar-gitlab-en-docker-servidor.html)  
> 🐙 **Imagen oficial**: [gitlab/gitlab-ce en Docker Hub](https://hub.docker.com/r/gitlab/gitlab-ce)  
> 📚 **Docs oficiales**: [GitLab Docker Installation](https://docs.gitlab.com/ee/install/docker.html)

---

**¿Te fue útil?** ⭐ Dale una estrella al repo y comparte.  
**¿Preguntas?** Abre un [Issue](https://github.com/genbyte/gitlab-docker/issues) o comenta en el blog.