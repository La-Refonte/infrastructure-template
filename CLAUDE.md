# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Infrastructure Template** - a minimal, ready-to-deploy professional infrastructure template for client projects using nginx, Docker, and modular architecture. This template provides:

- **Clean Base**: No pre-configured services, add only what you need
- **Nginx Reverse Proxy**: SSL termination and security headers
- **Docker Compose**: Empty template ready for your services
- **Examples Included**: Full examples in `examples/` directory

## Architecture

Template infrastructure for client deployment with modular services:

```
├── services/                           # Client-specific services (to be added)
│   └── [client-project]/              # Add client services here
│       ├── backend/                    # Backend API + Database
│       ├── frontend/                   # Frontend application
│       └── docs/                       # Project documentation
├── nginx/                             # Nginx configuration
│   ├── sites-available/              # Your configurations (empty)
│   ├── ssl/                          # SSL configuration
│   └── conf.d/                       # Security headers, rate limiting
├── scripts/                          # Utility scripts
│   ├── deploy-nginx.sh              # Nginx deployment
│   ├── backup-docker-volumes.sh     # Volume backup
│   └── restore-docker-volume.sh     # Volume restore
├── examples/                         # Configuration examples
│   ├── docker-compose.examples.yml  # Service examples
│   └── nginx-templates/             # Nginx templates
├── docker-compose.yml               # Service orchestration template
├── .env.example                     # Environment variables template
└── CLAUDE.md                        # Development guide
```

## Common Commands

### Client Setup (First Time)
```bash
# 1. Copy environment template
cp .env.example .env

# 2. Configure client-specific variables
nano .env  # Edit with client domain, passwords, API keys

# 3. Copy needed services from examples
# See examples/docker-compose.examples.yml

# 4. Add client services as submodules
git submodule add https://github.com/client/project.git services/client-project

# 5. Deploy nginx configuration
./scripts/deploy-nginx.sh
```

### Docker Services Management
```bash
# Start all configured services
docker compose up -d

# Stop all services
docker compose down

# Restart specific service
docker compose restart n8n
docker compose restart [client-service-name]

# Rebuild after code changes
docker compose up -d --build [service-name]

# View logs
docker compose logs -f n8n
docker compose logs -f [client-service]

# Check resource usage
docker stats
```

### Adding Client Services

#### Option 1: Ajouter submodule client existant
```bash
# 1. Ajouter le projet client comme submodule
git submodule add https://github.com/client/project-repo.git services/client-project

# 2. Initialiser le submodule
git submodule update --init --recursive

# 3. Adapter docker-compose.yml pour ce projet
# 4. Créer la configuration nginx
cp nginx/sites-available/template.conf nginx/sites-available/client.conf

# 5. Déployer
./scripts/deploy-nginx.sh
docker compose up -d --build
```

#### Option 2: Créer nouveau service client
```bash
# 1. Créer structure service
mkdir -p services/client-project/{backend,frontend}

# 2. Développer le service
# 3. En faire un repo Git séparé si besoin
# 4. Optionnellement l'ajouter comme submodule

# 5. Adapter docker-compose.yml et déployer
```

### N8N Workflows Secure Backup
```bash
# Backup all N8N projects to private GitHub repositories
./scripts/n8n-workflows-backup.sh

# Backup specific project only
./scripts/n8n-workflows-backup.sh nom-du-projet

# Backup with custom commit message
./scripts/n8n-workflows-backup.sh nom-du-projet "Custom backup message"

# Requirements:
# - Create .env file with GITHUB_TOKEN=ghp_xxxxx
# - Token needs access to La-Refonte organization
# - Workflows must be named: [PROJECT] Workflow name
# - Creates private repos: n8n-{project-name} in La-Refonte org
```

### Nginx Configuration Deployment
```bash
# Full deployment with automatic backup
./scripts/deploy-nginx.sh

# Test configuration only
./scripts/deploy-nginx.sh test

# Deploy frontends only (detects HTML static vs React/Vue builds automatically)
./scripts/deploy-nginx.sh frontend

# Deploy specific frontend
./scripts/deploy-nginx.sh frontend dashboard

# Rollback to backup
./scripts/deploy-nginx.sh rollback /root/nginx-backups/20250108_143022
```

### Frontend Development
```bash
# Client frontend development workflow
cd services/client-project/frontend
npm install
npm run dev      # Development server (varies by framework)

# Deploy frontend changes
./scripts/deploy-nginx.sh frontend

# The deploy script automatically detects:
# - HTML/CSS/JS static files → copies directly from services/project/frontend/
# - React/Vue projects with dist/ or build/ → copies build output

# For React/Vue projects:
cd services/client-project/frontend
npm run build    # Generate dist/ or build/
cd ../../..
./scripts/deploy-nginx.sh  # Auto-detects and deploys build output

# Deploy specific frontend only
./scripts/deploy-nginx.sh frontend client-project
```

### Service Monitoring
```bash
# Check Docker services status
docker compose ps

# View nginx logs by service
tail -f /var/log/nginx/n8n.access.log
tail -f /var/log/nginx/cercledesvoyages.access.log

# System resource usage
docker system df
docker stats
```

## Key Service Ports (Internal)

**Standard Template Ports:**
- **5678**: N8N Interface (standard for all clients)

**Client-Configurable Ports:**
- **3001**: Primary backend API (adapt per client)
- **3000**: Secondary services (scraping, etc.)
- **8080**: Frontend development servers

All services bind to `127.0.0.1` only and are accessible via nginx reverse proxy with SSL termination.

## Frontend Architecture Template

### Recommended Client Frontend Structure
```
services/client-project/frontend/
├── src/                               # Source code
├── public/                            # Static assets
├── package.json                       # Dependencies and scripts
├── vite.config.js / webpack.config.js # Build configuration
└── dist/ or build/                    # Build output (generated)
```

### Supported Frontend Types
1. **Static HTML/CSS/JS**: Direct deployment from `frontend/` folder
2. **React/Vue/Svelte**: Build-based deployment from `dist/` or `build/`
3. **Node.js SSR**: Custom Docker service with build process

**Deploy script automatically detects:**
- Static files → copies `services/project/frontend/` to `/var/www/`
- Build projects → copies `services/project/frontend/dist/` or `build/` to `/var/www/`

## Security Features

- Let's Encrypt SSL certificates for all domains
- HSTS enabled on all sites
- Security headers (X-Content-Type-Options, X-XSS-Protection, etc.)
- **CORS properly configured** with domain whitelist (not wildcard)
- Rate limiting (API: 30 req/min, General: 60 req/min)
- All ports bound to `127.0.0.1` only
- VNC access requires basic authentication

## Business Architecture

This template is designed for **client deployment scalability**:

- **Template-based approach**: Copy template, customize per client
- **Environment-driven configuration**: All client-specific settings in `.env`
- **Modular services**: Add only needed services to `docker-compose.yml`
- **Portable deployment**: Works on any server with Docker

### Client Deployment Workflow:
```bash
# 1. Copy template for new client
cp -r larefonte-infrastructure-template client-name-infrastructure
cd client-name-infrastructure

# 2. Configure for client
cp .env.example .env
nano .env  # Set client domain, credentials

# 3. Add client services (submodules dynamiques)
git submodule add https://github.com/client/backend-repo.git services/client-backend
git submodule add https://github.com/client/frontend-repo.git services/client-frontend
git submodule update --init --recursive

# 4. Edit docker-compose.yml to uncomment needed services

# 5. Deploy
./scripts/deploy-nginx.sh
docker compose up -d
```

## Development Workflow

### Client Project Development
1. Modify client code in `services/client-project/`
2. Test locally if necessary
3. For frontend changes: `./scripts/deploy-nginx.sh frontend`
4. For backend changes: `docker compose up -d --build client-service`

### Adding New Client Service
1. Add as submodule: `git submodule add https://github.com/client/service-repo.git services/new-service`
2. Initialize: `git submodule update --init --recursive`
3. Add service definition to `docker-compose.yml`
4. Create nginx config: `cp nginx/sites-available/template.conf nginx/sites-available/new-service.conf`
5. Deploy: `./scripts/deploy-nginx.sh`

### Template Maintenance
1. Update base template when needed
2. Test with sample client configuration
3. Update documentation for new features
4. Version template for client deployments

## Volume Management

### Template Volumes
- `n8n_data`: N8N workflows and configurations (standard for all clients)
- Client volumes: Add as needed in `docker-compose.yml`

### Client-Specific Volumes
```yaml
volumes:
  n8n_data:
    driver: local
  client_database:
    driver: local
  client_uploads:
    driver: local
```

Volumes persist data across container restarts and updates.

## SSL Certificate Management

```bash
# Check certificate expiration
sudo certbot certificates

# Manual renewal
sudo certbot renew

# Test renewal
sudo certbot renew --dry-run
```

## Troubleshooting

### Configuration Issues
```bash
# Test nginx configuration
sudo nginx -t

# Check nginx status
sudo systemctl status nginx

# Restart nginx completely
sudo systemctl restart nginx
```

### Docker Issues
```bash
# Rebuild specific service
docker compose up -d --build cercle-des-voyages-backend

# Clean up Docker resources
docker system prune

# Check container resource usage
docker stats
```

### Client Service Issues
```bash
# Rebuild specific client service
docker compose up -d --build client-service

# Reset client service completely
docker compose down client-service
docker volume rm client_service_data  # If needed
docker compose up -d --build client-service
```

## Important Notes for Template Usage

1. **Template-first approach**: Copy entire template for each client
2. **Environment-driven**: All client settings in `.env` file
3. **Frontend deploy script is intelligent**: Detects static vs build-based projects automatically
4. **Nginx deployment includes automatic backup**: Safe to deploy, can rollback if issues
5. **Standard directory names**: Use `frontend/` and `backend/` for consistency
6. **Service modularity**: Add only needed services to `docker-compose.yml`
7. **Environment configuration**: Always copy `.env.example` to `.env` and configure before first run
8. **Service isolation**: All services bind to `127.0.0.1` only, external access via nginx reverse proxy

The infrastructure includes automatic backup and rollback functionality for safe configuration updates.

## Quick Client Setup Workflow

1. **Copy template**: `cp -r template/ client-project/`
2. **Configure environment**: `cp .env.example .env && nano .env`
3. **Add client submodules**: `git submodule add https://github.com/client/repo.git services/client-service`
4. **Initialize submodules**: `git submodule update --init --recursive`
5. **Configure services**: Edit `docker-compose.yml`
6. **Deploy**: `./scripts/deploy-nginx.sh && docker compose up -d`
7. **Test and verify**: `docker compose ps` and check service logs

## Gestion des Submodules Client

### Ajout de submodules dynamiques
```bash
# Ajouter un projet client existant
git submodule add https://github.com/client/backend-api.git services/client-backend
git submodule add https://github.com/client/react-frontend.git services/client-frontend

# Initialiser tous les submodules
git submodule update --init --recursive
```

### Mise à jour des submodules client
```bash
# Mettre à jour tous les submodules
git submodule update --remote --merge

# Ou manuellement un submodule spécifique
cd services/client-backend
git pull origin main
cd ../..
git add services/client-backend
git commit -m "Update client backend"
```

### Important pour les clients
- **Template clean** : Le template n'a aucun submodule fixe
- **Submodules dynamiques** : Ajoutés uniquement selon besoins client
- **Flexibilité totale** : Chaque client peut avoir ses propres repos
- **Indépendance** : Chaque déploiement client est autonome