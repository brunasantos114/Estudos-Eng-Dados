# Explicação Docker Compose 

## **O QUE É DOCKER COMPOSE?**

Docker Compose é uma ferramenta que permite definir e rodar múltiplos containers Docker como um sistema único. Em vez de executar cada container manualmente, você descreve todos em um arquivo YAML e executa tudo com um comando.

**Analogia**: Se Docker é como uma caixa (container) isolada, Docker Compose é como uma fábrica que gerencia várias caixas funcionando juntas.

---

## Estrutura Básica

```yaml
version: '3.9'

services:
  # Seu aplicação web
  app:
    image: myapp:1.0
    ports:
      - "3000:3000"
    environment:
      DB_HOST: postgres
      DB_PASSWORD: ${DB_PASSWORD}
    volumes:
      - app_data:/app/data
    restart: unless-stopped

  # Banco de dados
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data

# Armazenamento de dados
volumes:
  app_data:
  db_data:
```

---

## Conceitos-Chave

| Conceito | O que é | Exemplo |
|----------|---------|---------|
| **image** | Imagem pronta do Docker Hub | `postgres:15` |
| **ports** | Porta visível do lado de fora | `"3000:3000"` |
| **environment** | Variáveis de configuração | `DB_PASSWORD` |
| **volumes** | Armazenamento persistente | `db_data:/data` |
| **restart** | O que fazer se cair | `unless-stopped` |

---

## Comandos Principais

```bash
# Começar tudo
docker-compose up -d

# Ver status
docker-compose ps

# Ver logs
docker-compose logs -f

# Parar tudo
docker-compose down

# Validar sintaxe
docker-compose config
```

---

## Segurança (IMPORTANTE!)

### ❌ NÃO faça isto:
```yaml
environment:
  DB_PASSWORD: "123456"
```

### ✅ Faça isto:
```yaml
# docker-compose.yml
environment:
  DB_PASSWORD: ${DB_PASSWORD}

# .env (adicione no .gitignore!)
DB_PASSWORD=minha_senha_secreta
```

**Por quê?** Porque senhas em código versionado = RISCO GIGANTE!

---

## Setup Rápido

### 1. Copiar template
```bash
cp .env.example .env
```

### 2. Editar valores
```bash
# Abra .env e mude as senhas e valores
DB_PASSWORD=senhacomplexa123
JWT_SECRET=outrasecreta456
```

### 3. Iniciar
```bash
docker-compose up -d
```

### 4. Verificar
```bash
docker-compose ps
# Tudo deve estar "Up"
```

---

## Checklist Pré-Produção

- [ ] `.env` criado e NÃO versionado
- [ ] Senhas complexas configuradas
- [ ] Todos os serviços rodando (`docker-compose ps`)
- [ ] Health checks passando
- [ ] Volumes configurados
- [ ] Logs aparecendo (`docker-compose logs`)

---

## Troubleshooting Rápido

**Container não inicia?**
```bash
docker-compose logs app
# Ver qual é o erro
```

**Espaço em disco cheio?**
```bash
docker system prune -a  # Remove imagens não usadas
```

**Não consegue conectar DB?**
```bash
docker-compose ps
# Verificar se DB está "Up"
docker-compose logs postgres
```

---

## Arquivo .env (exemplo)

```bash
# Aplicação
APP_VERSION=1.0.0
NODE_ENV=production

# Banco de dados
DB_USER=appuser
DB_PASSWORD=senhaforte123
DB_NAME=myapp

# Segredos
JWT_SECRET=sua_chave_secreta_muito_complexa
```

---

## Estrutura de Pastas Recomendada

```
seu-projeto/
├── docker-compose.yml      # Sua configuração
├── .env                    # Variáveis (NÃO versionar!)
├── .env.example            # Template (sim, versionar!)
├── .gitignore              # Ignore .env
├── app/                    # Seu código
└── backups/                # Backups do DB
```

---

## Restart Policies (qual usar?)

- `restart: unless-stopped` ← **Use isto em produção!**
  - Reinicia se cair, mas respeita `docker-compose stop`
  
- `restart: always`
  - Reinicia sempre (até ao rebootar servidor)
  
- `restart: on-failure`
  - Só reinicia se houver erro

---

## Health Checks (serviço saudável?)

```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s      # A cada 30 segundos
      timeout: 10s       # Timeout de 10 segundos
      retries: 3         # 3 tentativas antes de marcar como "down"
```

Isso permite ao Docker saber se seu app está realmente funcionando.

---

## Logs - Onde Estão?

```bash
# Ver logs em tempo real
docker-compose logs -f

# Apenas de um serviço
docker-compose logs -f app

# Últimas 50 linhas
docker-compose logs --tail=50
```

Os logs costumam ficar em: `/var/lib/docker/containers/`

---

## Backup Básico

```bash
# Backup do banco de dados
docker-compose exec postgres pg_dump -U appuser myapp > backup.sql

# Restaurar
docker-compose exec -T postgres psql -U appuser myapp < backup.sql
```

---

## Exemplo Real (API + DB)

```yaml
version: '3.9'

services:
  api:
    image: myregistry.azurecr.io/api:${APP_VERSION}
    ports:
      - "3000:3000"
    environment:
      DB_HOST: postgres
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped
    
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  db_data:
```

---

## Dúvidas Comuns

**P: E se o servidor cair?**
R: Se usar `restart: unless-stopped`, Docker reinicia tudo automaticamente quando o servidor voltar.

**P: Como atualizar a versão?**
R: Mude `APP_VERSION` no `.env`, depois execute:
```bash
docker-compose pull
docker-compose up -d
```

**P: Onde os dados ficam salvos?**
R: Em volumes Docker, geralmente em `/var/lib/docker/volumes/`

**P: Como fazer backup?**
R: Use `docker-compose exec` para fazer dump do banco, ou backup dos volumes.

---

## Links Úteis

- [Docker Compose Docs](https://docs.docker.com/compose/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)

---

**É isto! Comece com isto e leia a documentação completa conforme precisar.** 


