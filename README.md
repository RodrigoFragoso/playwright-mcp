# Playwright MCP + E2E

Projeto de testes E2E com **Playwright**, **Page Object Model (POM)** e suporte a **múltiplos ambientes** (localhost, staging, production). Inclui prompts para gerar e manter a suíte a partir do zero, integração opcional com o **MCP do Playwright** no Cursor para inspecionar a aplicação, e workflow de **CI no GitHub Actions** com execução dinâmica por ambiente. A aplicação alvo de exemplo é a [vanilla-js-web-app-example](https://erickwendel.github.io/vanilla-js-web-app-example/) (TDD Frontend Example).

---

## Prompts E2E – como usar e como subir o projeto

Este projeto contém os prompts (na pasta `prompts/`) para criar e manter a suíte de testes E2E com Playwright, POM e múltiplos ambientes.

---

## Como subir o projeto

Depois de aplicar os prompts (ou em um repositório que já tenha a estrutura), use os passos abaixo na **pasta raiz do projeto** (onde está o `package.json`).

### Pré-requisitos

- **Node.js** 18 ou superior  
- **npm** (já vem com o Node)

### 1. Instalar dependências

```bash
npm install
```

### 2. Instalar os browsers do Playwright

É necessário instalar pelo menos o Chromium para rodar os testes:

```bash
npx playwright install --with-deps chromium
```

Para instalar todos os browsers (Chromium, Firefox, WebKit):

```bash
npx playwright install --with-deps
```

### 3. Rodar os testes

**Ambiente padrão (production):**

```bash
npm test
```

**Por ambiente (scripts do package.json):**

```bash
npm run test:production   # production
npm run test:staging      # staging
npm run test:local        # localhost (app deve estar rodando, ex.: :3000)
```

**URL customizada:**

```bash
E2E_BASE_URL=https://minha-url.com npm test
```

### 4. Ver o relatório após a execução

Se o Playwright estiver configurado para gerar relatório HTML:

```bash
npx playwright show-report
```

### 5. (Opcional) Rodar com Chrome do sistema no Mac

Em Mac com Apple Silicon, se o Chromium do Playwright der problema, use o Chrome instalado no sistema:

```bash
CI= npm test
```

(Isso desativa o uso do Chromium em modo CI e usa `channel: 'chrome'` quando configurado no `playwright.config.ts`.)

---

## Como usar os prompts

Há duas formas de usar este material:

### Opção A: Prompts modulares (por etapas)

Cada prompt tem uma responsabilidade e indica o próximo. Ordem recomendada:

| # | Arquivo | Responsabilidade |
|---|---------|------------------|
| 1 | `01-mcp-and-scope.prompt.md` | MCP do Playwright + escopo da aplicação |
| 2 | `02-environments.prompt.md` | Ambientes (config, baseUrl, scripts) |
| 3 | `03-scaffolding-and-ci.prompt.md` | Scaffolding do projeto + CI (GitHub Actions) dinâmico |
| 4 | `04-practices-and-pom.prompt.md` | Boas práticas Playwright + Page Object Model |
| 5 | `05-test-coverage.prompt.md` | O que os testes devem cobrir |
| 6 | `06-gotchas-and-deliverables.prompt.md` | Descobertas/gotchas + lista de entregáveis |

**Uso:** abra o **prompt 1** no Cursor, aplique as instruções; ao final, ele indica o prompt 2. Repita até o 6. O projeto é criado **na pasta atual** em que você está.

### Opção B: Prompt único (tudo de uma vez)

O arquivo **`e2e-setup-and-tests.prompt.md`** contém todo o conteúdo em um só lugar. Use-o quando quiser passar o contexto completo de uma vez para a IA. O projeto também é criado **na pasta atual**.

---

## Resumo da cadeia (modular)

```
01 (MCP + escopo) → 02 (ambientes) → 03 (scaffolding + CI) → 04 (práticas + POM) → 05 (cobertura) → 06 (gotchas + entregáveis)
```

---

## Estrutura esperada do projeto (após aplicar os prompts)

Na raiz do projeto (pasta atual):

- `package.json` – scripts `test`, `test:local`, `test:staging`, `test:production`
- `playwright.config.ts` – baseURL via `getBaseUrl()`, timeouts, projeto Chromium
- `config/environments.ts` – ambientes (localhost, staging, production) e `getBaseUrl()`
- `pages/` – page objects (ex.: `VanillaAppPage.ts`)
- `tests/` – `fixtures.ts` e specs (ex.: `app.spec.ts`)
- `.github/workflows/playwright.yml` – CI dinâmico com escolha de ambiente
- `.gitignore` – `node_modules/`, `playwright-report/`, `test-results/`

A aplicação testada é a [vanilla-js-web-app-example](https://erickwendel.github.io/vanilla-js-web-app-example/) (TDD Frontend Example); os mesmos testes podem rodar em localhost, staging ou production conforme a configuração em `config/environments.ts`.
