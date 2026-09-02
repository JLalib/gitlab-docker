# 🐙 GitLab Docker - Servidor Git Autohospedado con CI/CD

[![GitHub Stars](https://img.shields.io/github/stars/genbyte/gitlab-docker?style=social)](https://github.com/genbyte/gitlab-docker)
[![Docker Pulls](https://img.shields.io/docker/pulls/gitlab/gitlab-ce?label=Docker%20Pulls&logo=docker)](https://hub.docker.com/r/gitlab/gitlab-ce)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Blog Post](https://img.shields.io/badge/📖_Blog_Post-Genbyte-orange)](https://genbyte.blogspot.com/2026/09/como-instalar-gitlab-en-docker-servidor.html)

---

## 📋 Descripción general

**GitLab Docker** es una configuración lista para producción que despliega **GitLab Community Edition** (CE) en contenedores Docker. Proporciona un servidor Git completo autohospedado con **CI/CD integrado**, registry Docker privado, gestión de usuarios y proyectos, webhooks, API REST, y backup automático — todo bajo tu control total, sin suscripciones externas.

> 🎯 **Alternativa a GitHub Enterprise** autohospedada, gratuita (Community Edition) y production-ready. Ideal para equipos de desarrollo, homelabs profesionales y entornos que requieren privacidad total de datos.

---

## ✨ Características principales

- 🔐 **Repositorios Git ilimitados** — Privados/públicos con acceso SSH + HTTPS
- ⚙️ **CI/CD nativo** — Pipelines con `.gitlab-ci.yml`, runners auto-escalables, artifacts y caches
- 📦 **Container Registry integrado** — Push/pull imágenes Docker privadas vinculadas a CI/CD
- 👥 **Gestión multi-tenant** — Usuarios, grupos, roles (Admin, Maintainer, Developer), LDAP/OAuth opcional
- 🔀 **Merge Requests completos** — Code review, discussions, aprobaciones, auto-merge
- 📋 **Issues + Epics** — Tracking de tareas, subissues, labels, asignaciones, milestones
- 🌐 **Webhooks + API REST** — Integraciones externas y automatización completa
- 💾 **Backup automático** — Scheduled backups con restore fácil (disaster recovery)
- 📖 **Wiki integrado** — Documentación por proyecto
- 📦 **Package Registry** — npm, Maven, PyPI, Conan, etc.
- 🔒 **100% On-premise** — Zero cloud, privacidad total, sin suscripción GitHub Enterprise

---

## 📋 Requisitos del sistema

- ✅ **Docker & Docker Compose v2+**
- 🧠 **RAM: 8 GB mínimo** (16 GB recomendado) — GitLab consume muchos recursos
- 💾 **Disco: 50 GB mínimo** (repositorios, CI/CD artifacts, registry, DB)
- 📁 **Directorio accesible:** `/mnt/media/gitlab` (o ruta personalizada)
- 🌐 **Puertos TCP:** `80` (HTTP), `443` (HTTPS), `22` (SSH Git — opcional)
- 🌍 **Dominio válido externo** — **OBLIGATORIO** (NO `localhost`), ej: `gitlab.tudominio.com`
- ⚡ **CPU: 4+ cores** (6+ recomendado)
- 🔗 **Acceso a internet** — Para webhooks, OAuth, notificaciones, Let's Encrypt
- 🐘 **PostgreSQL o SQLite** — Integrado en contenedor (SQLite por defecto)

> ⚠️ **IMPORTANTE:** GitLab **REQUIERE** un dominio externo válido. HTTPS altamente recomendado. En Raspberry Pi puede no funcionar bien — mejor servidor dedicado o VPS.

---

## 🐳 Instalación

### Paso 1: Preparar estructura de directorios

```bash
# Crear carpetas GitLab
sudo mkdir -p /mnt/media/gitlab/{config,logs,data}

# Permisos (UID/GID 1000 para contenedor GitLab)
sudo chown -R 1000:1000 /mnt/media/gitlab
sudo chmod -R 755 /mnt/media/gitlab
```

### Paso 2: Crear `docker-compose.yml`

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

> ⚠️ **IMPORTANTE:** Edita `docker-compose.yml` y cambia `gitlab.example.com` por tu dominio real (ej: `gitlab.miempresa.com`). Sin esto GitLab no funcionará correctamente.

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

---

## ⚙️ Configuración

1. **Dominio obligatorio** — Modifica `external_url` en `GITLAB_OMNIBUS_CONFIG` con tu dominio real
2. **HTTPS (recomendado)** — Descomenta líneas SSL y coloca certificados en `/mnt/media/gitlab/config/ssl/`
3. **Puerto SSH** — Si el 22 está ocupado, cambia `gitlab_shell_ssh_port` y mapea puerto en `ports:`
4. **PostgreSQL externo (opcional)** — Descomenta y configura `db_adapter`, `db_host`, `db_username`, `db_password`, `db_database`
5. **Recursos** — Ajusta `shm_size` si tienes problemas de memoria compartida (mínimo 256m)
6. **Backup automático** — Configura cron en host para `gitlab-backup create` periódico

---

## 🚀 Primeros pasos

1. **Login y cambiar contraseña**
   - Abre `http://gitlab.example.com` (o `https://` si configuraste SSL)
   - Usuario: `root` | Contraseña: (obtenida en Paso 5)
   - Avatar (esquina sup. der.) → **Settings** → **Account** → **Change password**
   - Establece contraseña nueva segura

2. **Crear proyecto nuevo**
   - Click **"New project"** (botón azul) → **"Create blank project"**
   - Project name: `mi-primer-proyecto` | Visibility: **Private** (recomendado)
   - Click **"Create project"**

3. **Clonar proyecto y hacer commit**
   ```bash
   # HTTPS
   git clone http://gitlab.example.com/root/mi-primer-proyecto.git
   cd mi-primer-proyecto
   
   # O SSH (si configurado)
   git clone ssh://git@gitlab.example.com/root/mi-primer-proyecto.git
   
   # Commit inicial
   echo "Hello GitLab" > README.md
   git add . && git commit -m "Initial commit"
   git push -u origin main
   ```

4. **Crear usuario nuevo**
   - **Admin Area** (icono llave inglesa en sidebar) → **Users** → **New user**
   - Rellena: name, username, email, password → **Create user**

5. **Agregar usuario a proyecto**
   - Proyecto → **Members** → **Add members**
   - Selecciona usuario, rol (**Developer**, **Maintainer**, etc.) → **Add to project**

6. **Configurar CI/CD (pipelines)**
   - En proyecto, crea archivo `.gitlab-ci.yml`:
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
   - Commit y push → Pipeline se ejecuta automático en **CI/CD → Pipelines**

7. **Usar Container Registry**
   ```bash
   # Login (usuario GitLab + Personal Access Token con scope registry)
   docker login gitlab.example.com
   
   # Build y push
   docker build -t gitlab.example.com/root/mi-imagen:latest .
   docker push gitlab.example.com/root/mi-imagen:latest
   
   # Pull
   docker pull gitlab.example.com/root/mi-imagen:latest
   ```

---

## 💡 Casos de uso

- 👨‍💻 **Equipos de desarrollo** — Git server profesional con CI/CD integrado, control total de privacidad
- 🚀 **DevOps / Platform Engineering** — Container registry privado, pipelines, artifacts, deployment tracking
- 🏠 **Homelabs** — Repositorios privados sin depender de GitHub/GitLab.com, backup local completo
- 🎓 **Educación** — Servidor Git para estudiantes, entorno controlado y privado
- 📋 **Compliance / Regulado** — Datos 100% on-premise, sin cloud externo, auditoría completa

---

## 🔒 Acceso remoto seguro

> **Recomendado:** No expongas puertos directamente a internet. Usa:

- **Tailscale / WireGuard / ZeroTier** — VPN mesh para acceso solo a tu red privada
- **Cloudflare Tunnel** — `cloudflared` tunnel sin abrir puertos, con WAF y DDoS protection
- **Nginx Proxy Manager / Traefik** — Reverse proxy local con Let's Encrypt automático
- **Authelia / Authentik** — SSO + 2FA delante de GitLab

```yaml
# Ejemplo: Cloudflare Tunnel (docker-compose.yml adicional)
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    command: tunnel run --token ${TUNNEL_TOKEN}
    network_mode: "host"
    restart: unless-stopped
```

---

## 🛠️ Gestión y mantenimiento

| Acción | Comando |
|--------|---------|
| **Ver estado** | `docker compose ps` |
| **Ver logs** | `docker compose logs -f gitlab` |
| **Detener** | `docker compose down` |
| **Actualizar versión** | `docker compose pull && docker compose up -d` |
| **Backup manual** | `docker compose exec -it gitlab gitlab-backup create` |
| **Listar backups** | `ls -lh /mnt/media/gitlab/data/backups/` |
| **Restaurar backup** | `docker compose down && docker compose exec -it gitlab gitlab-backup restore BACKUP=timestamp_of_backup` |
| **Cambiar puerto SSH** | Edita `gitlab_shell_ssh_port` en compose + mapea `"2424:22"` → `docker compose up -d` |

> 💡 **Backup automático recomendado:** Añade cron en host: `0 2 * * * docker exec gitlab gitlab-backup create CRON=1`

---

## 📝 Licencia

Este proyecto está bajo licencia **MIT** — ver archivo [LICENSE](LICENSE) para detalles.

La imagen Docker `gitlab/gitlab-ce` usa **GitLab Community Edition** (licencia MIT/Expat).
GitLab Enterprise Edition requiere suscripción comercial.

---

> 📖 **Guía completa en el blog:** [Cómo instalar GitLab en Docker - Servidor Git autohospedado](https://genbyte.blogspot.com/2026/09/como-instalar-gitlab-en-docker-servidor.html)
>
> 🐙 **Genbyte** — Automatización, Homelab, DevOps, Linux
> [YouTube](https://youtube.com/@genbyte) · [GitHub](https://github.com/genbyte) · [Blog](https://genbyte.blogspot.com) · [Newsletter](https://genbyte.blogspot.com/newsletter) · [Ko-fi](https://ko-fi.com/genbyte)