# WebDock

> A fully containerized infrastructure built with Docker and Docker Compose — featuring NGINX, WordPress, and MariaDB orchestrated inside a personal virtual machine. A 42 School project inspired by **Inception**.

---

## 🏗️ Architecture

```
                        ┌─────────────────────────────┐
                        │        Virtual Machine       │
                        │                              │
   HTTPS :443  ──────►  │  ┌────────┐   ┌──────────┐  │
                        │  │ NGINX  │──►│WordPress │  │
                        │  │(TLSv1.3│   │ php-fpm  │  │
                        │  └────────┘   └────┬─────┘  │
                        │                    │         │
                        │              ┌─────▼──────┐  │
                        │              │  MariaDB   │  │
                        │              └────────────┘  │
                        └─────────────────────────────┘
```

Each service runs in its own dedicated container, built from a custom Dockerfile. No pre-built images from DockerHub are used (except Alpine/Debian base images).

---

## 📁 Project Structure

```
WebDock/
├── Makefile
└── srcs/
    ├── docker-compose.yml
    ├── .env                        # Environment variables (not committed)
    ├── ExampleEnvFile              # Template for .env
    └── requirements/
        ├── nginx/
        │   ├── Dockerfile
        │   └── conf/               # NGINX config (TLSv1.3, SSL certs)
        ├── wordpress/
        │   ├── Dockerfile
        │   └── conf/               # php-fpm config + wp-config setup
        ├── mariadb/
        │   ├── Dockerfile
        │   └── conf/               # DB init scripts
        └── bonus/
            └── website/            # Static website (JS/CSS/HTML)
                ├── Dockerfile
                └── ...
```

---

## ⚙️ Services

| Container | Role | Port |
|-----------|------|------|
| **NGINX** | Reverse proxy with TLSv1.3 — the only entry point | `443` |
| **WordPress** | CMS running with php-fpm (no Apache) | internal |
| **MariaDB** | Database for WordPress | internal |
| **Static site** *(bonus)* | Custom JS/CSS/HTML site | internal |

Rules enforced by the project:
- Containers must restart automatically on crash
- No passwords in Dockerfiles — all secrets go in `.env`
- Volumes persist WordPress files and the MariaDB database
- Containers communicate over a dedicated Docker network

---

## 🚀 Getting Started

### Prerequisites

- Docker & Docker Compose
- Make
- A virtual machine running Linux (Debian/Alpine recommended)

### Setup

```bash
git clone https://github.com/KamenHero/WebDock.git
cd WebDock
```

Copy the environment file template and fill in your values:

```bash
cp srcs/ExampleEnvFile srcs/.env
nano srcs/.env
```

Edit the `login` variable in the `Makefile` to match your 42 login:

```makefile
LOGIN = your_login
```

### Build & Run

```bash
make
```

This will build all Docker images and spin up the containers via `docker-compose`.

### Access

| URL | Service |
|-----|---------|
| `https://localhost` | WordPress site |
| `https://your_login.42.fr` | WordPress site (via domain) |

> ⚠️ You may need to add `your_login.42.fr` to your `/etc/hosts` file pointing to `127.0.0.1`.

---

## 🛠️ Makefile Rules

| Rule | Description |
|------|-------------|
| `make` | Build images and start all containers |
| `make down` | Stop and remove containers |
| `make clean` | Remove containers, images, and volumes |
| `make re` | Full rebuild from scratch |
| `make logs` | Show live container logs |

---

## 🔐 Environment Variables

The `.env` file (based on `ExampleEnvFile`) must define:

```env
# Domain
DOMAIN_NAME=your_login.42.fr

# MariaDB
DB_NAME=wordpress
DB_USER=wp_user
DB_PASSWORD=your_db_password
DB_ROOT_PASSWORD=your_root_password

# WordPress
WP_ADMIN_USER=admin
WP_ADMIN_PASSWORD=your_wp_password
WP_ADMIN_EMAIL=your@email.com
```


---

## 🧠 Key Concepts

**Docker** packages applications and their dependencies into isolated containers, ensuring consistent behavior across environments.

**Docker Compose** orchestrates multiple containers, defining how they build, network, and share volumes — all from a single `docker-compose.yml`.

**NGINX as a reverse proxy** means all external traffic hits NGINX first. It terminates TLS and forwards requests to WordPress internally, keeping the other services unexposed.

**php-fpm** runs PHP processes for WordPress separately from NGINX — a more efficient and production-realistic setup than Apache.

---

## 📚 References

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Reference](https://docs.docker.com/compose/)
- [NGINX Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)
- [WordPress CLI](https://wp-cli.org/)
- [Alpine Linux](https://alpinelinux.org/)

---
