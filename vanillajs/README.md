# Perna Vanilla JS

Implementacao em HTML + JavaScript puro da verificacao de idade. Usa a API REST do `verify-service` diretamente (sem SDK).

Plain HTML + JavaScript implementation of age verification. Uses the `verify-service` REST API directly (no SDK).

## Como rodar / How to run

Necessita de 2 terminais. / Requires 2 terminals.

**Terminal 1** - Caddy (serve o webapp na porta 5500, faz proxy da API, e serve o `eca-context.jsonld`):

Caddy (serves the webapp on port 5500, proxies the API, and serves `eca-context.jsonld`):

```bash
cd perna/vanillajs && caddy run
```

Abra o navegador em `http://localhost:5500`. / Open browser at `http://localhost:5500`.

**Terminal 2** - Script de simulacao / Simulation script (ver / see [perna/README.md](../README.md#scripts-de-simulacao--simulation-scripts)):

```bash
cd perna/scripts && node simulate-wallet-happy.mjs req_xxx
```

## Como funciona / How it works

1. `startVerification()` faz POST em `/v1/verify/vp-request` com a `presentationDefinition` / POSTs to `/v1/verify/vp-request` with the `presentationDefinition`
2. Constroi uma URL `openid4vp://authorize?...` e gera um QR code / Builds an `openid4vp://authorize?...` URL and generates a QR code
3. `pollStatus()` faz polling em `/v1/verify/vp-request/{requestId}/status` / polls `/v1/verify/vp-request/{requestId}/status`
4. Quando o status e `VP_SUBMITTED`, busca o resultado em `/v1/verify/vp-result/{transactionId}` / When status is `VP_SUBMITTED`, fetches result from `/v1/verify/vp-result/{transactionId}`
5. Verifica `verificationStatus === "SUCCESS"` e `credentialSubject.isOver18 === true` / Checks `verificationStatus === "SUCCESS"` and `credentialSubject.isOver18 === true`
6. Redireciona para `/conteudo_adulto/` ou `/nao/` / Redirects to `/conteudo_adulto/` or `/nao/`

## Arquivos / Files

| Arquivo / File | Descricao / Description |
|---|---|
| `index.html` | Pagina principal com toda a logica JS / Main page with all JS logic |
| `styles.css` | Estilos (fundo preto, fonte monospace) / Styles (black bg, monospace font) |
| `Caddyfile` | Caddy: file server + reverse proxy + eca-context.jsonld |
