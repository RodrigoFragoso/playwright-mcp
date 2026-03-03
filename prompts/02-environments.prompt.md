# Prompt 2: Ambientes (localhost, staging, production)

**Responsabilidade:** Configurar múltiplos ambientes para a mesma suíte de testes e integrar baseUrl no projeto.

**Pressuposto:** O escopo da aplicação e o uso do MCP já foram definidos (prompt 01).

---

## 2. Ambientes (localhost, staging, production)

A mesma suíte de testes deve rodar em ambientes diferentes sem alterar código. Implementar:

### 2.1 Configuração de ambientes

- **Arquivo:** `config/environments.ts`
- **Conteúdo:** objeto `environments` com chaves `localhost`, `staging`, `production`. **Por padrão (ou em início de projeto)** todos podem apontar para a **mesma URL** usando uma constante (ex.: `const APP_URL = 'https://...';` e `environments = { localhost: APP_URL, staging: APP_URL, production: APP_URL }`). Assim a suíte roda em qualquer “ambiente” sem mudar código; quando houver deploys reais por ambiente, basta alterar as URLs em um só lugar (ex.: `localhost: 'http://localhost:3000'`, `staging: 'https://staging.example.com'`, `production: APP_URL`).
- **Função `getBaseUrl()`:** lê `process.env.E2E_ENV` para escolher o ambiente; se `process.env.E2E_BASE_URL` estiver definido, usa essa URL e ignora `E2E_ENV`. Retorna a URL sem barra final para consistência.
- **Default:** quando `E2E_ENV` não estiver definido, usar `production`.

### 2.2 Uso do baseUrl no projeto

- **Page Object:** o construtor do page object deve receber `(page, baseUrl: string)`. O método `goto(path)` monta a URL como `baseUrl + path` (tratando barra final). Não hardcodar a URL da aplicação dentro do POM.
- **Fixture:** em `tests/fixtures.ts`, ao instanciar o page object, chamar `getBaseUrl()` e passar como segundo argumento: `new VanillaAppPage(page, getBaseUrl())`.
- **playwright.config.ts:** importar `getBaseUrl` de `./config/environments` e usar `baseURL: getBaseUrl()` em `use` e no projeto (ex.: Chromium), para que relatórios e links usem a URL correta.

### 2.3 Como executar por ambiente

Criar no `package.json` os scripts por ambiente para facilitar a execução:

```json
"scripts": {
  "test": "playwright test",
  "test:local": "E2E_ENV=localhost playwright test",
  "test:staging": "E2E_ENV=staging playwright test",
  "test:production": "E2E_ENV=production playwright test"
}
```

Assim o time pode rodar:

- **Production (default):** `npm test` ou `npm run test:production`
- **Staging:** `npm run test:staging`
- **Localhost:** `npm run test:local` (garantir que a app está rodando na porta configurada, ex.: 3000)

Para **URL customizada** (CI ou qualquer host): `E2E_BASE_URL=https://minha-url.com npm test` — sobrescreve o ambiente.

Documentar esses comandos no README do projeto. Em Windows (cmd/PowerShell), definir a variável antes do comando (ex.: `set E2E_ENV=localhost && npm test`) ou usar um pacote como `cross-env` nos scripts se precisar de compatibilidade.

---

**Próximo passo:** use o prompt **`03-scaffolding-and-ci.prompt.md`** para criar o scaffolding do projeto e o workflow de CI dinâmico.
