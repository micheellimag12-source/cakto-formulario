# Implementação no Framer — Formulário Cakto

Esse formulário é uma **página completa em HTML/CSS/JS standalone**, não um embed. Tem 3 caminhos pra colocar isso em produção dentro do ecossistema Framer. Recomendo o **Caminho A**.

---

## Arquivos da pasta `/cakto-formulario/`

| Arquivo | Função | Obrigatório |
|---|---|---|
| `index.html` | A página | ✅ |
| `logo.svg` | Wordmark da Cakto | ✅ |
| `symbol.svg` | Símbolo (favicon, traveler, toasts, loader) | ✅ |
| `og.png` | OG card 1200×630 pra preview em WhatsApp/redes | ✅ |
| `og-card.html` | Template do OG card | ❌ (só pra re-gerar) |
| `cakto-formulario-figma.html` | Versão self-contained pra import no Figma | ❌ (não vai pra produção) |

Os 4 obrigatórios precisam ficar no **mesmo diretório** servido em `cakto.me/formulario/` pra o `og:image` resolver.

---

## Caminho A — Hospedar fora do Framer + path redirect *(recomendado)*

> **Por quê:** Framer não foi feito pra servir HTML cru com canvas, Web Audio, partículas e backdrop-filter. Hospedar externamente garante 100% do comportamento e zero conflito com a estilização do Framer.

### A.1. Suba os 4 arquivos num host estático

Opções equivalentes (escolha uma):

- **Vercel**: `vercel --prod` na pasta → você ganha algo tipo `https://cakto-formulario.vercel.app`
- **Cloudflare Pages**: conecta o repo GitHub → deploy automático
- **Netlify Drop**: drag-and-drop em `app.netlify.com/drop`
- **GitHub Pages**: gh-pages branch ou Actions
- **AWS S3 + CloudFront**: pra escala maior

Qualquer um serve. Você precisa do URL final, ex.: `https://cakto-formulario.vercel.app`.

### A.2. No Framer, configure path redirect

1. Abra o projeto Framer da `cakto.me`
2. No painel de páginas, **clique no botão `+` → "External URL"** (não crie uma página Framer normal)
3. **Path**: `/formulario`
4. **External URL**: cole o URL do passo A.1
5. **Redirect type**: `301 Permanent` se for definitivo, `302 Temporary` enquanto testa
6. Publica o site Framer

O resultado: `cakto.me/formulario` agora redireciona pro HTML que você subiu. O navegador mostra a URL final no address bar (`vercel.app` ou seja onde for).

### A.3. *(Opcional)* Cloaking via subdomínio + CNAME

Se quiser que a URL apareça como `cakto.me/formulario` (sem o redirect aparente no address bar):

1. Crie um subdomínio Cakto, ex.: `form.cakto.me`, apontando CNAME pro Vercel/CF/etc.
2. No Vercel/CF, configure o domínio custom `form.cakto.me` no projeto
3. No Framer, use **rewrite** em vez de redirect (se disponível no plano) apontando `/formulario` → `https://form.cakto.me`

A URL pública vira `cakto.me/formulario` mantendo o domínio principal, mas o conteúdo serve direto do host externo.

---

## Caminho B — Custom Code dentro da página Framer

> **Use se:** você não pode hospedar fora ou quer manter tudo dentro do Framer.
> **Limitações:** Framer injeta seu próprio `<html>`, `<head>` e `<body>`. Você não controla 100% da estrutura. Algumas coisas (font-loading order, scroll behavior, viewport) podem conflitar.

### B.1. Crie a página

1. Painel de páginas → `+ Page`
2. Path: `/formulario`
3. Deixa o canvas **completamente vazio** (nenhum frame Framer)
4. Page Settings (⚙️ no canto) → **Custom Code**

### B.2. Cole o conteúdo em 2 lugares

**Em "Start of `<head>` tag":**
Cole **tudo entre `<head>` e `</head>`** do `index.html`, **menos** o `<title>` e o `<meta name="description">` (esses são gerenciados pelo Framer no painel SEO).

**Em "End of `<body>` tag":**
Cole **tudo entre `<body>` e `</body>`** do `index.html`. Isso inclui:
- A estrutura `<canvas>`, `<div class="sun">`, etc.
- A `<div class="page">` com header, main, footer
- A `<div class="toast-stack">`
- Todos os `<script>` (sistema de partículas, lógica do form, áudio)

### B.3. Suba os assets

Framer aceita upload de arquivos via **Assets panel** (canvas → ícone de imagem → upload):

- `logo.svg` → copie a URL gerada pelo Framer → substitua TODAS as ocorrências de `src="logo.svg"` por `src="<URL-Framer>"`
- `symbol.svg` → mesma coisa pra TODAS as ocorrências de `src="symbol.svg"` e `href="symbol.svg"` (inclui CSS `background-image` se houver, e o JS dentro da função `showSaleToast` que cria `<img src="symbol.svg">`)
- `og.png` → upload pro Framer, copie URL, troque a `og:image` meta

> Alternativa mais limpa: use a **versão self-contained** (`cakto-formulario-figma.html`) que já tem os SVGs em base64 inline. Você só cola o conteúdo das tags HTML — sem upload de SVGs.

### B.4. Ajustes finos no Framer

Framer adiciona estilos globais que podem afetar seu CSS. Se vir conflitos:

- Page Settings → **Custom Code** → "Start of `<head>`": adicione no fim:
  ```html
  <style>
    /* Reset Framer wrappers que possam estar interferindo */
    body > div[data-framer-root] { all: revert; }
    [data-framer-name] { font-family: inherit !important; }
  </style>
  ```
- Em "Page Settings → Layout": desliga "Locked Aspect Ratio" e mantém auto-height
- Em "Page Settings → SEO": preencha title, description, OG image manualmente (Framer sobrescreve as meta tags)

---

## Caminho C — Iframe full-screen *(não recomendado)*

Você marcou que **não quer embed**, mas pra completar o quadro: dá pra criar uma página Framer normal com um único Code Component que renderiza um iframe ocupando 100vw × 100dvh apontando pro URL externo. Resultado funcional, mas tem custo de scroll duplo, SEO ruim e analytics quebrados. Pula esse.

---

## Checklist antes de publicar

- [ ] Os 4 arquivos (`index.html`, `logo.svg`, `symbol.svg`, `og.png`) estão no mesmo diretório do host
- [ ] `og:image` aponta pra URL absoluta acessível (`https://cakto.me/formulario/og.png` ou equivalente)
- [ ] `canonical` URL bate com onde a página realmente vive
- [ ] Teste o WhatsApp preview: cole o URL final num grupo de teste OU use [opengraph.xyz](https://www.opengraph.xyz/) → confere se a imagem aparece
- [ ] Teste mobile real (não só DevTools): câmera, teclado numérico no campo WhatsApp, máscara funcionando, som de notificação tocando no primeiro clique
- [ ] Teste com `prefers-reduced-motion` ativado (Mac: System Settings → Accessibility → Display → Reduce motion) — animações de loop param, partículas somem, áudio fica mudo
- [ ] Confere os 4 passos + loader + success — `?step=1`, `?step=2`, `?step=3`, `?step=4`, `?step=loader`, `?step=success` na URL
- [ ] Submit do form: por enquanto faz `console.log` — antes de publicar, conecta o endpoint real (webhook Cakto ou Framer Forms API). O ponto de integração tá em `function submit()` no JS, perto da linha "`Demo only — connect your backend here`"

---

## Conectando o backend

Hoje o submit só faz `console.log`. Quando você tiver o endpoint:

```js
// Em vez de:
console.log('[Cakto formulário] Submission:', data);

// Faça:
fetch('https://api.cakto.me/leads/formulario', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(data)
}).catch(err => console.error('Submit error:', err));
```

O objeto `data` já inclui `whatsapp_e164` (`+5511999...`) pronto pra integrações de WhatsApp/CRM, além de todos os campos do formulário e `submitted_at` em ISO 8601.

---

## Por que não usei o sistema de Forms nativo do Framer

O Framer Forms é ótimo pra formulários simples (1 step, sem lógica custom). Pra esse projeto não funcionaria porque:

- **Máscara de telefone em tempo real** — Framer Forms não suporta máscaras
- **Validação custom de DDD ANATEL + bloqueio de números genéricos** — não dá pra plugar lógica JS no validator deles
- **Multi-step com progresso animado** — Framer Forms é monolítico
- **Sand-swirl, Web Audio, canvas de partículas, toast de venda iOS-style** — todos dependem de JS custom e não cabem no editor visual

Por isso o caminho é HTML standalone hospedado.
