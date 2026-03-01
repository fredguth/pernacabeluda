# Perna - Verificacao de Idade com Credenciais Verificaveis

# Perna - Age Verification with Verifiable Credentials

Prova de conceito de verificacao de idade usando o protocolo OpenID4VP (Verifiable Presentations). Um usuario apresenta uma credencial digital (ECACredential) que comprova ser maior de 18 anos, sem revelar sua identidade.

Proof of concept for age verification using the OpenID4VP (Verifiable Presentations) protocol. A user presents a digital credential (ECACredential) proving they are over 18, without revealing their identity.

## Pre-requisitos / Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (para o backend / for the backend)
- [Caddy](https://caddyserver.com/) (servidor de arquivos / file server)
- [Node.js](https://nodejs.org/) 18+

## Estrutura / Structure

```
perna/
  eca-context.jsonld          # JSON-LD context da credencial ECA / ECA credential JSON-LD context
  Caddyfile                   # Caddy: serve eca-context.jsonld na porta 5500 / on port 5500
  vanillajs/                  # Implementacao vanilla JS / Vanilla JS implementation
  react/                      # Implementacao React / React implementation
  scripts/                    # Scripts de simulacao de wallet / Wallet simulation scripts
```

## Backend

O backend (`verify-service`) roda em Docker e expoe a API na porta 8080.

The backend (`verify-service`) runs in Docker and exposes the API on port 8080.

```bash
cd inji-verify/docker-compose && docker compose up -d
```

## Caddy (obrigatorio / required)

O backend precisa buscar o `eca-context.jsonld` via HTTP para validar as assinaturas das credenciais. De dentro do Docker, ele acessa `http://host.docker.internal:5500/eca-context.jsonld`. O Caddy serve esse arquivo.

The backend needs to fetch `eca-context.jsonld` over HTTP to validate credential signatures. From inside Docker, it accesses `http://host.docker.internal:5500/eca-context.jsonld`. Caddy serves this file.

```bash
cd perna && caddy run
```

Isso e necessario tanto para a versao vanilla JS quanto para a versao React.

This is required for both the vanilla JS and the React versions.

## Implementacoes / Implementations

Ha duas implementacoes do mesmo webapp:

There are two implementations of the same webapp:

- **[vanillajs/](vanillajs/)** - HTML + JavaScript puro, usando a API REST diretamente / Plain HTML + JavaScript, using the REST API directly
- **[react/](react/)** - React 18 + TypeScript, usando o SDK `@mosip/react-inji-verify-sdk` / React 18 + TypeScript, using the `@mosip/react-inji-verify-sdk` SDK

Veja o README de cada uma para instrucoes especificas. / See each README for specific instructions.

## Scripts de simulacao / Simulation scripts

Como nao temos uma wallet real, usamos scripts que simulam o comportamento de uma wallet digital submetendo uma Verifiable Presentation ao backend.

Since we don't have a real wallet, we use scripts that simulate a digital wallet submitting a Verifiable Presentation to the backend.

### Uso / Usage

1. Abra o webapp (vanillajs ou react) e clique em "Verificar". / Open the webapp (vanillajs or react) and click "Verificar".
2. Copie o `requestId` do console do navegador (F12). O formato e `req_xxx`. / Copy the `requestId` from the browser console (F12). The format is `req_xxx`.
3. Rode o script: / Run the script:

```bash
cd perna/scripts

# Instalar dependencias (primeira vez) / Install dependencies (first time)
npm install

# Happy path: isOver18=true, assinatura valida → /conteudo_adulto/
# Happy path: isOver18=true, valid signature → /conteudo_adulto/
node simulate-wallet-happy.mjs req_xxx

# Unhappy path: isOver18=false → /nao/
node simulate-wallet-unhappy.mjs req_xxx
```

### Variaveis de ambiente / Environment variables

| Variavel / Variable | Default | Descricao / Description |
|---|---|---|
| `VERIFY_SERVICE_URL` | `http://localhost:8080/v1/verify` | URL do backend / Backend URL |
| `CONTEXT_PORT` | `5500` | Porta do Caddy servindo `eca-context.jsonld` / Caddy port serving `eca-context.jsonld` |

## Fluxo de verificacao / Verification flow

```
Usuario/User          Webapp              Backend (Docker)        Wallet (script)
     |                   |                      |                      |
     |-- Clica Verificar |                      |                      |
     |                   |-- POST /vp-request -->|                      |
     |                   |<-- requestId, QR -----|                      |
     |                   |                      |                      |
     |                   |-- GET /status ------->|  (polling)           |
     |                   |                      |                      |
     |                   |                      |<-- POST /vp-submission|
     |                   |                      |    (VP + credential)  |
     |                   |                      |                      |
     |                   |<-- VP_SUBMITTED ------|                      |
     |                   |-- GET /vp-result ---->|                      |
     |                   |<-- vcStatus, vc ------|                      |
     |                   |                      |                      |
     |<-- Redirect ------|                      |                      |
     |  /conteudo_adulto/ (SUCCESS + isOver18)                        |
     |  /nao/              (caso contrario / otherwise)                |
```
