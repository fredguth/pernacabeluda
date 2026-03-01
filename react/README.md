# Perna React

Implementacao em React 18 + TypeScript da verificacao de idade. Usa o componente `<OpenID4VPVerification>` do SDK `@mosip/react-inji-verify-sdk`, que cuida de todo o fluxo (VP request, QR code, polling, resultado).

React 18 + TypeScript implementation of age verification. Uses the `<OpenID4VPVerification>` component from the `@mosip/react-inji-verify-sdk` SDK, which handles the entire flow (VP request, QR code, polling, result).

## Como rodar / How to run

Necessita de 2 terminais. / Requires 2 terminals.

**Terminal 1** - Caddy (serve o `eca-context.jsonld` na porta 5500 para o backend Docker / serves `eca-context.jsonld` on port 5500 for the Docker backend):

```bash
cd perna && caddy run
```

**Terminal 2** - Vite dev server:

```bash
cd perna/react
npm install   # ou / or: bun install
npm run dev   # ou / or: bun run dev
```

Abra o navegador em `http://localhost:5173`. / Open browser at `http://localhost:5173`.

**Testar / Test** (ver / see [perna/README.md](../README.md#scripts-de-simulacao--simulation-scripts)):

```bash
cd perna/scripts && node simulate-wallet-happy.mjs req_xxx
```

## Como funciona / How it works

O SDK faz todo o trabalho pesado. O codigo do app e minimo:

The SDK does all the heavy lifting. The app code is minimal:

1. Usuario clica "Verificar" → monta o componente `<OpenID4VPVerification>` / User clicks "Verificar" → mounts the `<OpenID4VPVerification>` component
2. O SDK cria a VP request, gera o QR code e faz polling automaticamente / The SDK creates the VP request, generates the QR code, and polls automatically
3. Callback `onVPProcessed` recebe o resultado e verifica `vcStatus` e `isOver18` / `onVPProcessed` callback receives the result and checks `vcStatus` and `isOver18`
4. Redireciona para `/conteudo_erotico/` ou `/nao/` / Redirects to `/conteudo_erotico/` or `/nao/`

### Interceptor de fetch

O `requestId` necessario para rodar os scripts de simulacao e logado no console do navegador via um interceptor de `fetch` (o SDK nao expoe esse valor diretamente).

The `requestId` needed to run the simulation scripts is logged in the browser console via a `fetch` interceptor (the SDK does not expose this value directly).

### Proxy do Vite

O Vite dev server faz proxy de `/v1/verify/*` para `localhost:8080`, evitando problemas de CORS sem precisar do Caddy para a API.

The Vite dev server proxies `/v1/verify/*` to `localhost:8080`, avoiding CORS issues without needing Caddy for the API.

### Paginas estaticas

As paginas de resultado (`/nao/`, `/conteudo_erotico/`) sao HTMLs estaticos em `public/`, servidos por um plugin Vite customizado que intercepta essas rotas antes do SPA fallback.

The result pages (`/nao/`, `/conteudo_erotico/`) are static HTMLs in `public/`, served by a custom Vite plugin that intercepts these routes before the SPA fallback.

## Arquivos / Files

| Arquivo / File | Descricao / Description |
|---|---|
| `src/App.tsx` | Componente principal / Main component |
| `src/main.tsx` | Entry point React 18 |
| `src/index.css` | Estilos (fundo preto, fonte monospace) / Styles (black bg, monospace font) |
| `vite.config.ts` | Proxy API + plugin para paginas estaticas / API proxy + static pages plugin |
| `public/nao/` | Pagina de rejeicao / Rejection page |
| `public/conteudo_erotico/` | Pagina de conteudo adulto / Adult content page |
