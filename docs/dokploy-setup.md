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

No Dokploy, ao configurar o roteamento:

1. Use o **nome do container** (ex: `web`, `admin`, `api`) como hostname
2. Use a **porta interna** do container (ex: `3000`, `8000`)
3. O Dokploy acessará os containers através da rede interna do Docker Compose
4. Não é necessário expor portas no host - o Dokploy gerencia isso automaticamente

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

O admin está configurado para servir em `/god-mode/`. Configure o Dokploy para rotear `/god-mode` para o container `admin` na porta interna `3000`.

### Erro 404 no Web

Verifique se:
1. O container `web` está rodando: `docker ps | grep web`
2. O reverse proxy do Dokploy está configurado para rotear para `web:3000`
3. O nome do container está correto no Dokploy (deve ser exatamente `web`)

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

