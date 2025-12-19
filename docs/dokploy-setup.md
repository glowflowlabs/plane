# Configuração do Dokploy para Plane

Este documento explica como configurar o roteamento no Dokploy para acessar os serviços do Plane.

## Portas dos Serviços

Os containers expõem as seguintes portas:

- **web** (Frontend): `3000` → Acessível em `http://seu-dominio.com` ou `http://seu-dominio.com:3000`
- **admin** (Admin Panel): `3001` → Acessível em `http://seu-dominio.com:3001` ou `http://seu-dominio.com/god-mode`
- **api** (Backend API): `8000` → Acessível em `http://seu-dominio.com:8000` ou `http://seu-dominio.com/api`
- **space** (Space App): `3002` → Acessível em `http://seu-dominio.com:3002` ou `http://seu-dominio.com/spaces`
- **live** (Live App): `3003` → Acessível em `http://seu-dominio.com:3003` ou `http://seu-dominio.com/live`

## Configuração no Dokploy

### Opção 1: Usar Portas Diretas (Mais Simples)

No Dokploy, configure os serviços para acessar diretamente pelas portas:

1. **Web (Frontend Principal)**
   - Container: `web`
   - Porta: `3000`
   - URL: `http://seu-dominio.com:3000`

2. **Admin Panel**
   - Container: `admin`
   - Porta: `3001`
   - URL: `http://seu-dominio.com:3001`
   - **Nota**: O admin está configurado para servir em `/god-mode/` dentro do container

3. **API**
   - Container: `api`
   - Porta: `8000`
   - URL: `http://seu-dominio.com:8000` ou configure um subdomínio como `api.seu-dominio.com`

### Opção 2: Configurar Reverse Proxy no Dokploy (Recomendado)

Configure o reverse proxy do Dokploy para rotear corretamente:

#### Roteamento Principal (Web)
```
Path: /
Target: web:3000
```

#### Roteamento Admin
```
Path: /god-mode
Target: admin:3000
```

**Importante**: O admin serve os arquivos em `/god-mode/` dentro do container, então você precisa:
- Acessar via `http://seu-dominio.com/god-mode/` OU
- Configurar o Dokploy para rotear `/god-mode` para o container `admin` na porta `3000`

#### Roteamento API
```
Path: /api
Target: api:8000
```

Ou configure um subdomínio:
```
Subdomain: api
Target: api:8000
```

#### Roteamento Space
```
Path: /spaces
Target: space:3000
```

#### Roteamento Live
```
Path: /live
Target: live:3000
```

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

### Erro 404 no Admin

O admin está configurado para servir em `/god-mode/`. Se você está acessando diretamente pela porta `3001`, tente:
- `http://seu-dominio.com:3001/god-mode/`

Ou configure o reverse proxy do Dokploy para rotear `/god-mode` para `admin:3000`.

### Erro 404 no Web

Verifique se:
1. O container `web` está rodando: `docker ps | grep web`
2. A porta 3000 está acessível
3. O reverse proxy está configurado corretamente

### API não responde

Verifique se:
1. O container `api` está rodando
2. As migrações foram executadas (container `migrator` deve ter rodado)
3. As variáveis de ambiente do banco de dados estão corretas
4. O banco de dados está acessível pelo container `api`

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

