# Laravel 13 + Livewire 4 — Template Base

Template Docker pronto para uso como ponto de partida em projetos Laravel 13 com Livewire 4, PostgreSQL e Redis.

---

## Stack

| Tecnologia | Versão |
|---|---|
| PHP | 8.4 |
| Laravel | 13 |
| Livewire | 4 |
| PostgreSQL | 16 |
| Redis | 7 |
| Nginx | 1.25 |

---

## Estrutura de pastas

```
meu-projeto/
├── docker/
│   ├── nginx/
│   │   └── nginx.conf
│   └── php/
│       └── php.ini
├── src/                  # Código-fonte do Laravel
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Dockerfile
└── README.md
```

---

## Instalação local — passo a passo

### 1. Usar este template ou clonar

No GitHub, clique em **"Use this template"** para criar seu repositório a partir deste, ou clone diretamente:

```bash
git clone https://github.com/SEU_USUARIO/laravel-livewire-template.git meu-projeto
cd meu-projeto
```

### 2. Instalar dependências do Laravel

Se o repositório já possui `src/` com o Laravel:
```bash
docker run --rm -v "${PWD}/src:/app" composer install
```

Para uma instalação do zero (sem `src/`):
```bash
docker run --rm -v "${PWD}/src:/app" composer create-project laravel/laravel:^13 .
```

### 3. Copiar e configurar o ambiente


Para o docker-compose (raiz do projeto)
```bash
copy .env.example .env          # Windows
```

Para o Laravel (dentro de src/)
```bash
copy .env.example src\.env      # Windows
```

Edite `src/.env` e ajuste pelo menos:
- `APP_NAME` — nome da sua aplicação
- `APP_SLUG` — slug usado nos nomes dos containers (sem espaços)
- `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`

Edite `.env` (raiz) com as mesmas credenciais de banco definidas acima.

### 4. Subir os containers

```bash
docker compose up -d --build
```

### 5. Gerar a chave da aplicação

```bash
docker compose exec app php artisan key:generate
```

### 6. Corrigir permissões

```bash
docker compose exec app chmod -R 775 storage bootstrap/cache
docker compose exec app chown -R www-data:www-data storage bootstrap/cache
```

### 7. Rodar as migrations

```bash
docker compose exec app php artisan migrate
```

### 8. Publicar assets do Livewire

```bash
docker compose exec app php artisan livewire:publish --assets
```

### 9. Criar o link de storage

```bash
docker compose exec app php artisan storage:link
```

### 10. Instalar dependências frontend e compilar assets

```bash
docker compose exec app npm install
docker compose exec app npm run build
```

### 11. Acessar o sistema

Abra o navegador em: [http://localhost:8080](http://localhost:8080)

---

## Comandos úteis do dia a dia

```bash
# Subir os containers
docker compose up -d

# Parar os containers
docker compose down

# Ver logs da aplicação
docker compose logs -f app

# Acessar o container da aplicação
docker compose exec app bash

# Rodar artisan
docker compose exec app php artisan COMANDO

# Rodar migrations
docker compose exec app php artisan migrate

# Reverter última migration
docker compose exec app php artisan migrate:rollback

# Limpar todos os caches
docker compose exec app php artisan optimize:clear

# Recompilar assets frontend
docker compose exec app npm run build

# Ver filas em execução
docker compose logs -f queue

# Acessar PostgreSQL via psql
docker compose exec postgres psql -U laravel_user -d laravel
```

---

## Variáveis de ambiente principais

| Variável | Valor padrão | Descrição |
|---|---|---|
| `APP_NAME` | `Minha Aplicação` | Nome da aplicação |
| `APP_SLUG` | `meu_projeto` | Slug usado nos nomes dos containers |
| `APP_URL` | `http://localhost:8080` | URL local |
| `APP_DEBUG` | `true` | Ativar em dev, **false em produção** |
| `DB_HOST` | `postgres` | Nome do container do banco |
| `DB_DATABASE` | `laravel` | Nome do banco |
| `DB_USERNAME` | `laravel_user` | Usuário do banco |
| `DB_PASSWORD` | `secret` | Senha — **alterar em produção** |
| `REDIS_HOST` | `redis` | Nome do container Redis |
| `CACHE_STORE` | `redis` | Driver de cache |
| `SESSION_DRIVER` | `redis` | Driver de sessão |
| `QUEUE_CONNECTION` | `redis` | Driver de filas |

---

## Deploy em VPS (Hostinger / Ubuntu 24.04 com Docker)

```bash
# 1. Acessar a VPS via SSH
ssh root@SEU_IP_VPS

# 2. Clonar o repositório
git clone https://github.com/SEU_USUARIO/meu-projeto.git /var/www/meu-projeto
cd /var/www/meu-projeto

# 3. Instalar dependências do Laravel
docker run --rm -v "$(pwd)/src:/app" composer install --no-dev --optimize-autoloader

# 4. Configurar os arquivos de ambiente
cp .env.example .env
cp .env.example src/.env
nano src/.env
# Alterar obrigatoriamente:
# APP_ENV=production
# APP_DEBUG=false
# APP_URL=https://seudominio.com.br
# DB_PASSWORD=senha_forte_aqui
# Replicar as mesmas credenciais no .env da raiz

# 5. Subir os containers
docker compose up -d --build

# 6. Gerar chave e configurar a aplicação
docker compose exec app php artisan key:generate
docker compose exec app chmod -R 775 storage bootstrap/cache
docker compose exec app chown -R www-data:www-data storage bootstrap/cache

# 7. Rodar migrations e seeders
docker compose exec app php artisan migrate --force
# docker compose exec app php artisan db:seed  # se aplicável

# 8. Otimizar para produção
docker compose exec app php artisan livewire:publish --assets
docker compose exec app php artisan optimize
docker compose exec app php artisan storage:link

# 9. Compilar assets
docker compose exec app npm install
docker compose exec app npm run build
```

### Atualizar em produção

```bash
cd /var/www/meu-projeto
git pull origin main
docker compose exec app composer install --no-dev --optimize-autoloader
docker compose exec app php artisan migrate --force
docker compose exec app php artisan optimize:clear
docker compose exec app php artisan optimize
docker compose exec app npm run build
```

### SSL/HTTPS

Use **Certbot** diretamente na VPS ou **Nginx Proxy Manager** para gerenciar certificados SSL.

---

## Troubleshooting

### Container não sobe
```bash
docker compose down -v
docker compose up -d --build
```

### Erro de permissão no storage
```bash
docker compose exec app chmod -R 775 storage bootstrap/cache
docker compose exec app chown -R www-data:www-data storage bootstrap/cache
```

### Livewire JS não carrega (404)
```bash
docker compose exec app php artisan livewire:publish --assets
docker compose exec app php artisan optimize:clear
```

### Banco não conecta
Verifique se o container do postgres está em execução:
```bash
docker compose ps
```
Confirme que `DB_HOST=postgres`, `DB_USERNAME`, `DB_PASSWORD` e `DB_DATABASE` no `src/.env` estão corretos e idênticos às variáveis no `.env` da raiz.

### Cache travado
```bash
docker compose exec app php artisan optimize:clear
```

---

## Versionamento sugerido

```bash
git add .
git commit -m "feat: descrição da funcionalidade"
git push origin main
```

Branches sugeridas:
- `main` — produção estável
- `develop` — desenvolvimento ativo
- `feature/nome-da-feature` — funcionalidades em desenvolvimento
