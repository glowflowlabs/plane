# Configuração do Dokploy para Plane

Este documento explica como configurar o roteamento no Dokploy para acessar os serviços do Plane.

## Portas Internas dos Containers

Os containers expõem as seguintes portas **internamente** na rede Docker (não expostas no host):

- **web** (Frontend): porta interna `3000`
- **admin** (Admin Panel): porta interna `3000` (serve em `/god-mode/` dentro do container)
- **api** (Backend API): porta interna `8000`
- **space** (Space App): porta interna `3000`
- **live** (Live App): porta interna `3000`

**Importante**: As portas não estão expostas no host para evitar conflitos. O Dokploy acessa os containers através da rede interna do Docker usando os nomes dos containers.

## Configuração no Dokploy

O Dokploy gerencia o roteamento através da rede interna do Docker Compose. Configure o reverse proxy do Dokploy usando os **nomes dos containers** e **portas internas**:

### Roteamento Principal (Web)
```
Container Name: web
Internal Port: 3000
Path: /
```

### Roteamento Admin
```
Container Name: admin
Internal Port: 3000
Path: /god-mode
```

**Importante**: O admin serve os arquivos em `/god-mode/` dentro do container. Configure o Dokploy para rotear `/god-mode` para o container `admin` na porta interna `3000`.

### Roteamento API
```
Container Name: api
Internal Port: 8000
Path: /api
```

Ou configure um subdomínio:
```
Subdomain: api
Container Name: api
Internal Port: 8000
```

### Roteamento Space
```
Container Name: space
Internal Port: 3000
Path: /spaces
```

### Roteamento Live
```
Container Name: live
Internal Port: 3000
Path: /live
```

## Como Configurar no Dokploy

No Dokploy, ao configurar o roteamento do reverse proxy:

### Configuração Básica

1. **Hostname/Target**: Use o **nome exato do container** (ex: `web`, `admin`, `api`)
2. **Porta**: Use a **porta interna** do container (ex: `3000`, `8000`)
3. **Rede**: Os containers estão na rede `plane-network` (bridge)

### Exemplo de Configuração no Dokploy

#### Para o serviço Web (Frontend):
```
Hostname: web
Port: 3000
Path: /
```

#### Para o serviço Admin:
```
Hostname: admin
Port: 3000
Path: /god-mode
```

#### Para o serviço API:
```
Hostname: api
Port: 8000
Path: /api
```

**Importante**: 
- Use o nome do container **exatamente como está** no docker-compose.yml
- Não use `localhost` ou `127.0.0.1` - use o nome do container
- O Dokploy acessará os containers através da rede Docker interna

## Variáveis de Ambiente Necessárias

Certifique-se de configurar todas as variáveis de ambiente no Dokploy:

### Banco de Dados
- `POSTGRES_USER`
- `POSTGRES_DB`
- `POSTGRES_PASSWORD`

### RabbitMQ
- `RABBITMQ_USER`
- `RABBITMQ_PASSWORD`
- `RABBITMQ_VHOST`

### MinIO/S3
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_S3_BUCKET_NAME` (opcional, padrão: `uploads`)

### API Django
Consulte `apps/api/.env.example` para todas as variáveis necessárias da API Django, incluindo:
- `SECRET_KEY`
- `DATABASE_URL` ou configurações individuais do banco
- `REDIS_URL`
- E outras variáveis específicas do Plane

## Troubleshooting

### Bad Gateway (502)

Se você está recebendo "Bad Gateway", verifique:

1. **Nome do container está correto?**
   - No Dokploy, use exatamente: `web`, `admin`, `api`, `space`, ou `live`
   - Não use `localhost`, `127.0.0.1`, ou o nome do projeto

2. **Porta está correta?**
   - Web/Admin/Space/Live: porta `3000`
   - API: porta `8000`

3. **Container está rodando?**
   ```bash
   docker ps | grep -E "web|admin|api"
   ```
   Você deve ver os containers `web`, `admin`, e `api` na lista

4. **Container está saudável?**
   ```bash
   docker logs web
   docker logs admin
   docker logs api
   ```
   Verifique se há erros nos logs

5. **Teste de conectividade interna:**
   ```bash
   # Teste se o container web responde
   docker exec web curl -I http://localhost:3000
   
   # Teste se o container admin responde
   docker exec admin curl -I http://localhost:3000
   
   # Teste se o container api responde
   docker exec api curl -I http://localhost:8000
   ```

6. **Rede Docker:**
   - Os containers estão na rede `plane-network`
   - O Dokploy deve conseguir acessar essa rede
   - Verifique: `docker network ls | grep plane`

### Erro 404 no Admin

O admin está configurado para servir em `/god-mode/`. Configure o Dokploy para rotear `/god-mode` para o container `admin` na porta interna `3000`.

**Configuração correta no Dokploy:**
```
Hostname: admin
Port: 3000
Path: /god-mode
```

### Erro 404 no Web

Verifique se:
1. O container `web` está rodando: `docker ps | grep web`
2. O reverse proxy do Dokploy está configurado para rotear para `web:3000`
3. O nome do container está correto no Dokploy (deve ser exatamente `web`)

**Configuração correta no Dokploy:**
```
Hostname: web
Port: 3000
Path: /
```

### API não responde / Container API reiniciando

Se o container `api` está em estado "Restarting", verifique os logs:

```bash
docker logs api
# ou para logs em tempo real
docker logs -f api
```

**Problemas comuns e soluções:**

1. **Falta de variáveis de ambiente essenciais:**
   - `SECRET_KEY` - Obrigatório para Django
   - `DATABASE_URL` ou configurações individuais do banco
   - `REDIS_URL` - Obrigatório para cache
   - `AWS_ACCESS_KEY_ID` e `AWS_SECRET_ACCESS_KEY` - Para MinIO/S3

2. **Banco de dados não acessível:**
   - Verifique se o container `plane-db` está rodando
   - Verifique as variáveis `POSTGRES_USER`, `POSTGRES_DB`, `POSTGRES_PASSWORD`
   - Teste a conexão: `docker exec api python manage.py check --database default`

3. **Migrações não executadas:**
   - Execute o container `migrator` primeiro:
   ```bash
   docker compose run --rm migrator
   ```

4. **Redis não acessível:**
   - Verifique se o container `plane-redis` está rodando
   - Configure `REDIS_URL` corretamente (ex: `redis://plane-redis:6379/0`)

5. **Variáveis de ambiente mínimas necessárias:**
   ```bash
   # Django
   SECRET_KEY=<sua-chave-secreta>
   DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@plane-db:5432/${POSTGRES_DB}
   REDIS_URL=redis://plane-redis:6379/0
   
   # MinIO/S3
   AWS_ACCESS_KEY_ID=<sua-chave>
   AWS_SECRET_ACCESS_KEY=<seu-secret>
   AWS_S3_BUCKET_NAME=uploads
   AWS_S3_ENDPOINT_URL=http://plane-minio:9000
   AWS_S3_REGION_NAME=us-east-1
   
   # Gunicorn
   GUNICORN_WORKERS=4
   PORT=8000
   ```

**Configuração correta no Dokploy:**
```
Hostname: api
Port: 8000
Path: /api
```

**Para verificar se a API está funcionando após corrigir:**
```bash
# Verifique se o container está rodando (não mais "Restarting")
docker ps | grep api

# Teste a API internamente
docker exec api curl -I http://localhost:8000/api/health
```

## Verificação dos Containers

Para verificar se todos os containers estão rodando:

```bash
docker ps
```

Você deve ver:
- `web`
- `admin`
- `api`
- `space`
- `live`
- `bgworker`
- `beatworker`
- `plane-db`
- `plane-redis`
- `plane-mq`
- `plane-minio`

