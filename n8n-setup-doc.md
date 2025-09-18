# 🚀 Documentação de Setup do n8n com Docker + PostgreSQL

Este documento descreve o processo completo de configuração do **n8n** em ambiente **self-hosted** usando **Docker Compose** e **PostgreSQL**.  
Objetivo: rodar o n8n localmente (Windows/Linux/macOS) com persistência de dados e possibilidade de deploy futuro.

---

## 📂 Estrutura de Pastas

No Windows, criamos a pasta:

```
D:\n8n
```

Dentro dela, manteremos os arquivos de configuração e os volumes de dados do Docker.

---

## ⚙️ Arquivos de Configuração

### `.env`

Crie o arquivo `D:\n8n\.env` com o seguinte conteúdo (ajuste as variáveis):

```env
# n8n
N8N_PORT=5678
N8N_PROTOCOL=http
N8N_HOST=localhost
TZ=America/Sao_Paulo

# URL pública (caso use ngrok/cloudflared)
WEBHOOK_URL=

# chave de criptografia para credenciais (substitua por uma forte e guarde em local seguro)
N8N_ENCRYPTION_KEY=minha-chave-super-secreta-123456

# banco PostgreSQL
POSTGRES_DB=n8n
POSTGRES_USER=n8n
POSTGRES_PASSWORD=minha-senha-forte
```

---

### `docker-compose.yml`

Crie o arquivo `D:\n8n\docker-compose.yml` com o conteúdo:

```yaml
services:
  postgres:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - ./postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 3s
      retries: 10

  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    ports:
      - "${N8N_PORT}:5678"
    environment:
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=5678
      - N8N_PROTOCOL=${N8N_PROTOCOL}
      - WEBHOOK_URL=${WEBHOOK_URL}
      - NODE_ENV=production
      - TZ=${TZ}
      # banco
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      # segurança
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
    volumes:
      - ./data:/home/node/.n8n
```

---

## ▶️ Inicializando os Containers

No **PowerShell** ou terminal, dentro da pasta `D:\n8n`, execute:

```powershell
docker compose up -d
```

Isso irá:

- Criar o container `postgres`.
- Criar o container `n8n`.
- Mapear as pastas locais para persistência (`data` e `postgres`).

---

## 🌐 Acesso ao n8n

Após subir os containers, abra no navegador:

```
http://localhost:5678
```

Na primeira vez, será solicitado criar usuário e senha de administrador.

---

## 🛠️ Conectando ao Banco via pgAdmin

Se desejar acessar o banco diretamente:

- **Host**: `localhost`  
- **Port**: `5432`  
- **User**: `n8n`  
- **Password**: `minha-senha-forte`  
- **Database**: `n8n`  

---

## 🔄 Atualizar o n8n

Para atualizar a versão do n8n:

```bash
docker compose pull
docker compose up -d
```

---

## ⏹️ Parar e Logs

Parar os containers:

```bash
docker compose stop
```

Ver os logs do n8n:

```bash
docker compose logs -f n8n
```

---

## 🧪 Expondo Webhooks para Internet

Para receber webhooks externos (ex.: WhatsApp Evolution API):

- **ngrok**:
  ```bash
  ngrok http 5678
  ```
- **cloudflared**:
  ```bash
  cloudflared tunnel --url http://localhost:5678
  ```

Copie a URL gerada e configure no `.env`:

```env
WEBHOOK_URL=https://seu-endereco-publico
```

Depois reinicie os containers:

```bash
docker compose up -d
```

---

## 🚨 Problemas Comuns

- **Porta 5678 em uso** → Alterar `N8N_PORT` no `.env`.  
- **Webhook não dispara** → Configurar `WEBHOOK_URL` corretamente.  
- **Permissões em Linux** → Rodar `sudo chown -R 1000:1000 ./data ./postgres`.  
- **Perda de workflows** → Sempre manter backup da pasta `data/`.  

---

## 📌 Observações

- Esse setup é **quase produção**, pronto para evoluir para deploy em servidores (VPS, Cloud, Kubernetes).  
- Para testes rápidos, pode ser usado um setup mais simples com **SQLite** apenas.  
- Esse README serve como **documentação base** do projeto.

---

✍️ Autor: *Matheus Barbosa Meloni*  
📅 Data: 18/09/2025
