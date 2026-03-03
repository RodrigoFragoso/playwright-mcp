# Playwright E2E: setup completo e geração de testes

Use este prompt para criar do zero um projeto Playwright que testa a aplicação e gerar todos os testes E2E seguindo boas práticas. Inclui suporte a múltiplos ambientes (localhost, staging, production) e todas as correções necessárias para a suíte rodar de forma estável.

---

## Prompts modulares (opcional)

Este conteúdo também está **dividido em prompts encadeados**, cada um com uma responsabilidade. Use-os se preferir executar por etapas (o prompt 1 indica o 2, o 2 indica o 3, etc.):

| Ordem | Arquivo | Responsabilidade |
|-------|---------|------------------|
| 1 | `01-mcp-and-scope.prompt.md` | MCP do Playwright + escopo |
| 2 | `02-environments.prompt.md` | Ambientes (config, baseUrl, scripts) |
| 3 | `03-scaffolding-and-ci.prompt.md` | Scaffolding + CI dinâmico |
| 4 | `04-practices-and-pom.prompt.md` | Boas práticas + POM |
| 5 | `05-test-coverage.prompt.md` | O que os testes devem cobrir |
| 6 | `06-gotchas-and-deliverables.prompt.md` | Gotchas + entregáveis |

Ver **`README.md`** nesta pasta para a descrição do fluxo e como encadear os prompts.

---

## 0. Como usar o MCP do Playwright (recomendado antes de escrever testes)

O **Playwright MCP** (Model Context Protocol) permite que a IA navegue na aplicação real, leia a árvore de acessibilidade e interaja com a página. Use-o para inspecionar a aplicação **antes** de escrever os page objects e os testes.

### Quando usar o MCP

- **Antes de implementar:** navegar até a URL da aplicação, fazer um **snapshot** da página (acessibilidade) para ver roles, nomes e estrutura dos elementos.
- **Para validar locators:** conferir se os textos, placeholders e roles usados no POM existem na página (ex.: "Please type a title for the image", botão "Submit", headings nível 4).
- **Para gerar ou corrigir testes:** com o snapshot, a IA pode sugerir locators estáveis (getByRole, getByLabel, getByText) e fluxos (preencher form, clicar, assertar).

### Como usar no Cursor

1. **Configurar o MCP** no Cursor (veja o README do projeto, seção "Como usar o MCP do Playwright").
2. No chat, pedir para **navegar** até a URL da aplicação (ex.: `https://erickwendel.github.io/vanilla-js-web-app-example/`).
3. Pedir um **snapshot** da página (ferramenta de acessibilidade do MCP) para listar elementos interativos, headings, textos e roles.
4. Com base no snapshot, escrever o page object e os testes usando os nomes/roles reais.

Se o MCP não estiver disponível, inspecionar a aplicação via HTML (fetch da URL ou abrir no navegador) e extrair os textos e estrutura manualmente.

---

## 1. Escopo da aplicação

- **Onde criar o projeto:** todos os arquivos e pastas (package.json, config/, pages/, tests/, .github/workflows/) devem ser criados **na pasta atual** em que o usuário está — ou seja, o projeto é criado no diretório de trabalho atual, sem criar subpastas tipo “projeto” ou “playwright-mcp” dentro dele.
- **URL da aplicação (production):** `https://erickwendel.github.io/vanilla-js-web-app-example/`
- **Objetivo:** Configurar o Playwright (somente test runner, sem outros frameworks), criar a pasta de testes e escrever specs que cubram smoke, formulário (submit + validação) e listagem. Os mesmos testes devem poder rodar em **localhost**, **staging** ou **production** (ver seção Ambientes).

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

## 3. Setup do projeto (scaffolding)

**Local:** criar todo o scaffolding **na pasta atual** (diretório de trabalho do usuário). Não criar um subdiretório para o projeto — package.json, config/, pages/, tests/, .github/ ficam na raiz do projeto atual.

Implemente o seguinte:

1. **Dependência:** apenas `@playwright/test` como devDependency. **Scripts no `package.json`:** `"test": "playwright test"` e, para cada ambiente, `"test:local"`, `"test:staging"`, `"test:production"` definindo `E2E_ENV` antes do comando (ex.: `"test:local": "E2E_ENV=localhost playwright test"`).
2. **Configuração (`playwright.config.ts`):**
   - `baseURL` = `getBaseUrl()` (da config de ambientes).
   - Timeout de teste e de ação razoáveis (ex.: 15s teste, 10s ação).
   - Um único projeto: Chromium. Em CI usar o Chromium instalado pelo Playwright; localmente pode usar `channel: 'chrome'` para usar o Chrome do sistema (evitar problema de arquitetura em Mac ARM).
   - **workers: 1** — em alguns hosts (ex.: GitHub Pages), muitas requisições paralelas podem resultar em "Site not found"; rodar com um worker evita isso.
   - **userAgent** real (ex.: Chrome desktop) — alguns provedores servem conteúdo diferente para headless; definir um userAgent de navegador real reduz bloqueios.
3. **Estrutura:** pastas `config/` (environments), `pages/` (page objects) e `tests/` (specs + `fixtures.ts`); pelo menos um spec (ex.: `tests/app.spec.ts`) que use o page object via fixture.
4. **CI:** workflow dinâmico conforme a seção **3.4** abaixo.

Antes de escrever os testes, **navegue na aplicação** (ou consulte o HTML/estrutura) para identificar títulos, labels, placeholders, roles e textos exatos dos elementos que serão usados nos locators.

### 3.4 CI (GitHub Actions) dinâmico

O workflow em `.github/workflows/playwright.yml` deve permitir **escolher o ambiente** de execução:

- **Gatilhos:** `push`/`pull_request` em `main`/`master` **e** `workflow_dispatch` (execução manual).
- **Input no `workflow_dispatch`:** um input `environment` do tipo `choice` com opções `localhost`, `staging`, `production` e default `production`. Descrição clara (ex.: "Ambiente E2E (localhost, staging, production)").
- **Variável `E2E_ENV` no workflow:** definir no nível do workflow (ou do job) de forma que:
  - Em **push/PR:** `E2E_ENV` seja `production`.
  - Em **workflow_dispatch:** `E2E_ENV` seja o valor escolhido em `inputs.environment`.
  - Exemplo em YAML: `E2E_ENV: ${{ github.event_name == 'workflow_dispatch' && inputs.environment || 'production' }}`.
- **Job de testes:** passar `E2E_ENV` para o ambiente do step que roda os testes (ex.: `env: E2E_ENV: ${{ env.E2E_ENV }}` no step "Run Playwright tests"), para que `npm test` use o ambiente correto via `getBaseUrl()`.
- **Nome do step:** incluir o ambiente no nome (ex.: "Run Playwright tests (${{ env.E2E_ENV }})") para ficar claro nos logs.
- **Artifact em falha:** fazer upload do relatório HTML quando falhar; o **nome do artifact** deve incluir o ambiente (ex.: `playwright-report-${{ env.E2E_ENV }}`) para não sobrescrever relatórios de runs com ambientes diferentes.

Steps do job: checkout, setup Node (com cache npm), install deps (`npm ci`), install Chromium (`npx playwright install --with-deps chromium`), run tests (`npm test` com `E2E_ENV` definido), upload do report em caso de falha. **Não usar `working-directory`** em nenhum step — o workflow roda na raiz do repositório (pasta atual do projeto); paths são relativos à raiz (ex.: `package-lock.json`, `playwright-report/`). Documentar no README que, na execução manual (Actions → nome do workflow → Run workflow), o usuário pode selecionar o ambiente.

---

## 4. Boas práticas obrigatórias (Playwright)

Siga estas regras em todos os testes e na organização do código.

### 4.1 Ordem de preferência dos locators

Use os locators nesta ordem (do mais recomendado ao menos recomendado):

1. **`getByRole`** – preferir sempre que o elemento tiver papel de acessibilidade (button, heading, textbox, etc.). Ex.: `getByRole('button', { name: /Submit/i })`, `getByRole('heading', { level: 4 })`.
2. **`getByLabel`** – para inputs associados a labels.
3. **`getByPlaceholder`** – quando não houver label estável mas o placeholder for confiável.
4. **`getByText`** – para textos visíveis estáticos (mensagens, títulos). Evitar para conteúdo dinâmico ou que mude com i18n.
5. **`getByTestId`** – apenas quando não houver opção semântica (evitar se o app não definir data-testid).
6. **Evitar:** seletores CSS/XPath puros; `page.$()`; locators que dependam de estrutura de DOM frágil.

### 4.2 Padrão AAA (Arrange, Act, Assert)

Estruture cada teste em três blocos claros:

- **Arrange:** preparar o estado (ex.: `vanillaAppPage.goto('/')`, preencher campos).
- **Act:** executar a ação do usuário (ex.: clicar em Submit).
- **Assert:** verificar o resultado (ex.: `expect(...).toBeVisible()`, `toHaveCount()`, `toHaveValue('')`).

Use comentários `// Arrange`, `// Act`, `// Assert` apenas quando ajudar a leitura; o código deve ser autoexplicativo.

### 4.3 Separação de responsabilidades

- **Setup global por suíte:** usar `test.beforeEach` para navegação comum (ex.: `vanillaAppPage.goto('/')`) e, se necessário, skip quando a página retornar 404 ou "Site not found" (verificar `response?.ok` e título da página).
- **Um conceito por `test.describe`:** agrupar por feature (ex.: "Vanilla JS Web App", "Form submission", "Form validation").
- **Um comportamento por `test()`:** cada teste deve validar um único fluxo ou regra; evitar testes longos com vários comportamentos.
- **Evitar duplicação:** não repetir `goto` em todo teste se o `beforeEach` já navegar; reutilizar locators por placeholder/role quando forem os mesmos.

### 4.4 Resiliência e tempo

- Preferir `waitUntil: 'domcontentloaded'` (ou `'load'`) no `goto` quando fizer sentido.
- Não depender de timeouts fixos (ex.: `page.waitForTimeout`) para conteúdo; usar assertions com timeout padrão do Playwright (ex.: `expect(locator).toBeVisible()`).
- Se a aplicação às vezes retornar "Site not found" ou 404, no `beforeEach` verificar título ou status e chamar `test.skip()` com mensagem clara em vez de falhar.

### 4.5 Page Object Model (POM)

Use **Page Object Model** para organizar locators e ações: um lugar único para seletores e fluxos, testes mais legíveis e manutenção centralizada.

#### Estrutura de pastas

- **`config/`** – configuração de ambientes (`getBaseUrl()`, etc.).
- **`pages/`** – uma classe por tela ou contexto relevante (ex.: `pages/VanillaAppPage.ts`).
- **`tests/`** – specs que usam os page objects via **fixture**, sem instanciar manualmente.

#### Regras do Page Object

1. **Construtor com baseUrl**  
   O page object recebe `(page: Page, baseUrl: string)`. O `baseUrl` vem do fixture (que chama `getBaseUrl()`). Não hardcodar URL da aplicação dentro do POM.

2. **Locators como getters (lazy)**  
   Expor locators como **getters** que retornam `Locator`, não como propriedades atribuídas no construtor. Assim os locators são resolvidos no momento do uso (melhor para conteúdo dinâmico).

   ```ts
   get titleLabel(): Locator {
     return this.page.locator('label').filter({ hasText: /Please type a title for the image/i });
   }
   get submitButton(): Locator {
     return this.page.getByRole('button', { name: /submit/i });
   }
   ```

3. **Labels do formulário: só elementos visíveis**  
   Se a página tiver mensagens de validação em elementos ocultos (ex.: `div.invalid-feedback` com o mesmo texto do label), **não** usar `getByText(...)` para o label — isso pode resolver para o elemento oculto e o teste falhar (expected visible, received hidden). Usar **`locator('label').filter({ hasText: /.../ })`** para garantir que o locator aponte para o `<label>` visível.

4. **Título principal (main heading)**  
   Se o título da página puder ser um `<h1>` ou outro elemento (div/span), usar fallback para não depender só de role:  
   `getByRole('heading', { name: /TDD Frontend Example/i }).or(this.page.getByText(/TDD Frontend Example/i).first())`.

5. **Ordem dos locators dentro do POM**  
   Mesma ordem recomendada do Playwright: `getByRole` > `getByLabel` > `getByPlaceholder` > `getByText` > `getByTestId`. Evitar CSS/XPath frágeis.

6. **Navegação encapsulada**  
   O page object deve ter um método `goto(path?)` que monta a URL com `this.baseUrl + path` e chama `this.page.goto(url, { waitUntil: 'domcontentloaded' })`. Pode retornar a `Response` para o teste checar status (ex.: 404). Opcionalmente um método `isAvailable()` que verifica título (ex.: "Site not found") para decidir skip.

7. **Ações, não assertions**  
   Métodos do page object devem representar **ações do usuário** (preencher, clicar, navegar). As **assertions** (`expect(...).toBeVisible()`, `toHaveCount()`, etc.) ficam nos testes. O page object pode expor helpers como `getCardCount()` que retornam dados para o teste assertar.

8. **Não expor `page`**  
   Manter `page` como dependência interna (constructor). Os testes que precisarem de `page` (ex.: `expect(page).toHaveTitle()`) recebem `page` pelo fixture; o restante usa apenas o page object.

9. **Fixture para o Page Object**  
   Criar `tests/fixtures.ts` com `test.extend<{ vanillaAppPage: VanillaAppPage }>({ vanillaAppPage: async ({ page }, use) => { await use(new VanillaAppPage(page, getBaseUrl())); } })`. Exportar `test` e `expect`. Nos specs, importar `test` e `expect` de `./fixtures` e usar o page object pelo parâmetro do teste (ex.: `async ({ vanillaAppPage }) => { ... }`), **sem** fazer `new VanillaAppPage(page)` em cada teste.

10. **Um page object por tela/contexto**  
    Uma classe por página ou por bloco lógico (ex.: formulário + listagem na mesma tela). Nomear de forma clara (ex.: `VanillaAppPage`, `LoginPage`).

#### Exemplo de uso nos testes

- No `beforeEach`: chamar `vanillaAppPage.goto('/')`, checar `response?.ok` e título (ex.: "Site not found") e, se necessário, `test.skip(...)`.
- Nos testes: usar apenas `vanillaAppPage.*` para locators e ações; assertions com `expect(vanillaAppPage.titleLabel).toBeVisible()`, etc.

---

## 5. O que os testes devem cobrir

Gere specs que incluam:

### 5.1 Smoke / página inicial

- Título da página (ex.: "TDD Frontend Example").
- Texto principal visível (main heading) — usar locator com fallback (heading ou getByText) conforme item 4.5.4.
- Presença dos **labels** visíveis do formulário (ex.: "Please type a title for the image", "Please type a valid URL") — usar `locator('label').filter({ hasText: ... })` para não pegar `.invalid-feedback`.
- Inputs e botão de submit visíveis (getByLabel ou getByRole conforme a página).
- Cards iniciais visíveis (ex.: headings nível 4 com "AI Alien", "Predator Night Vision", "ET Bilu").

### 5.2 Submissão do formulário

- Com título e URL válidos preenchidos, ao clicar em Submit:
  - Um novo card com o título informado aparece na listagem.
  - O número total de cards (ex.: headings nível 4) aumenta em 1.
  - Os campos do formulário são limpos (valor vazio).

### 5.3 Validação do formulário

- Submit com formulário vazio: a listagem não deve ganhar novos cards (contagem de cards permanece a mesma).
- Submit com apenas o título preenchido: nenhum card novo com esse título; contagem inalterada.
- Submit com apenas a URL preenchida: contagem inalterada.

Use locators estáveis (role, label, text em `<label>`) e padrão AAA; agrupe em `describe` por tema (smoke, form submission, form validation).

---

## 6. Descobertas e gotchas (para deixar a suíte estável)

- **GitHub Pages / "Site not found":** em alguns contextos (headless, muitos workers), o site pode retornar "Site not found". Mitigações: `workers: 1`, `userAgent` de navegador real no `playwright.config.ts`, e no `beforeEach` verificar o título da página e dar `test.skip()` se contiver "not found".
- **Navegação:** o page object deve usar a URL construída com `baseUrl` (injetada), não depender só do `baseURL` do Playwright para o `goto`, para evitar problemas de resolução em certos hosts.
- **Main heading:** se `getByRole('heading', { name: /.../ })` não encontrar (ex.: título em outro elemento), usar fallback com `getByText(/.../).first()`.
- **Labels vs. invalid-feedback:** textos como "Please type a title for the image" podem existir no `<label>` e também em um `div.invalid-feedback` oculto. `getByText(...)` pode resolver para o div e o teste falhar (expected visible). Sempre usar `locator('label').filter({ hasText: ... })` para os labels do formulário.
- **CI:** usar `channel: 'chrome'` apenas quando **não** for CI (`...(process.env.CI ? {} : { channel: 'chrome' })`), para que em CI o Playwright use o Chromium instalado por `npx playwright install`.

---

## 7. Entregáveis esperados

Ao final, o projeto deve ter:

- `package.json` com script `test`, scripts por ambiente (`test:local`, `test:staging`, `test:production`) e devDependency `@playwright/test`.
- **`config/environments.ts`** – ambientes (localhost, staging, production), `getBaseUrl()`, suporte a `E2E_ENV` e `E2E_BASE_URL`.
- `playwright.config.ts` com `baseURL: getBaseUrl()`, timeouts, workers (ex.: 1 quando necessário), userAgent quando necessário, e projeto Chromium (com fallback para Chrome no ambiente local quando não for CI).
- **`pages/`** – page object(s) com construtor `(page, baseUrl)`, locators em getters (labels via `locator('label').filter(...)`, main heading com fallback), `goto(path)` usando `baseUrl`, `isAvailable()` e ações reutilizáveis (sem assertions). Ex.: `pages/VanillaAppPage.ts`.
- **`tests/fixtures.ts`** – `test.extend()` que injeta o page object passando `getBaseUrl()`; exportar `test` e `expect`.
- `tests/app.spec.ts` (ou nome equivalente) que importa `test`/`expect` de `./fixtures`, usa o page object via fixture, segue AAA e POM (sem `new PageObject(page)` nos testes).
- **`.github/workflows/playwright.yml`** – CI dinâmico: `workflow_dispatch` com input `environment` (choice: localhost, staging, production); em push/PR usar `E2E_ENV=production`; passar `E2E_ENV` ao step que roda `npm test`; artifact do relatório com nome que inclua o ambiente (ex.: `playwright-report-${{ env.E2E_ENV }}`). Ver seção 3.4.
- `.gitignore` com `node_modules/`, `playwright-report/`, `test-results/`.
- **README** com instruções para rodar por ambiente (scripts `test:local`, `test:staging`, `test:production` e variáveis `E2E_ENV`/`E2E_BASE_URL`), e com explicação do CI dinâmico (push/PR = production; execução manual = escolha do ambiente).

O agente que executar este prompt deve primeiro inspecionar a aplicação (navegando ou lendo o HTML) para garantir que os textos, roles e placeholders usados nos page objects correspondam ao que realmente existe na página.
