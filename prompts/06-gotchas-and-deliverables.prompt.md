# Prompt 6: Descobertas/gotchas e entregáveis

**Responsabilidade:** Aplicar mitigações para problemas conhecidos e validar que todos os entregáveis do fluxo E2E estão presentes.

**Pressuposto:** Testes escritos conforme cobertura do prompt 05; projeto com ambientes, scaffolding, CI e POM dos prompts anteriores.

---

## 6. Descobertas e gotchas (para deixar a suíte estável)

- **GitHub Pages / "Site not found":** em alguns contextos (headless, muitos workers), o site pode retornar "Site not found". Mitigações: `workers: 1`, `userAgent` de navegador real no `playwright.config.ts`, e no `beforeEach` verificar o título da página e dar `test.skip()` se contiver "not found".
- **Navegação:** o page object deve usar a URL construída com `baseUrl` (injetada), não depender só do `baseURL` do Playwright para o `goto`, para evitar problemas de resolução em certos hosts.
- **Main heading:** se `getByRole('heading', { name: /.../ })` não encontrar (ex.: título em outro elemento), usar fallback com `getByText(/.../).first()`.
- **Labels vs. invalid-feedback:** textos como "Please type a title for the image" podem existir no `<label>` e também em um `div.invalid-feedback` oculto. `getByText(...)` pode resolver para o div e o teste falhar (expected visible). Sempre usar `locator('label').filter({ hasText: ... })` para os labels do formulário.
- **CI:** usar `channel: 'chrome'` apenas quando **não** for CI (`...(process.env.CI ? {} : { channel: 'chrome' })`), para que em CI o Playwright use o Chromium instalado por `npx playwright install`.

---

## 7. Entregáveis esperados

Todos os itens abaixo ficam **na pasta atual do projeto** (raiz do repositório). Ao final do fluxo (prompts 01–06), o projeto deve ter:

- `package.json` com script `test`, scripts por ambiente (`test:local`, `test:staging`, `test:production`) e devDependency `@playwright/test`.
- **`config/environments.ts`** – ambientes (localhost, staging, production), `getBaseUrl()`, suporte a `E2E_ENV` e `E2E_BASE_URL`.
- `playwright.config.ts` com `baseURL: getBaseUrl()`, timeouts, workers (ex.: 1 quando necessário), userAgent quando necessário, e projeto Chromium (com fallback para Chrome no ambiente local quando não for CI).
- **`pages/`** – page object(s) com construtor `(page, baseUrl)`, locators em getters (labels via `locator('label').filter(...)`, main heading com fallback), `goto(path)` usando `baseUrl`, `isAvailable()` e ações reutilizáveis (sem assertions). Ex.: `pages/VanillaAppPage.ts`.
- **`tests/fixtures.ts`** – `test.extend()` que injeta o page object passando `getBaseUrl()`; exportar `test` e `expect`.
- `tests/app.spec.ts` (ou nome equivalente) que importa `test`/`expect` de `./fixtures`, usa o page object via fixture, segue AAA e POM (sem `new PageObject(page)` nos testes).
- **`.github/workflows/playwright.yml`** – CI dinâmico: `workflow_dispatch` com input `environment` (choice: localhost, staging, production); em push/PR usar `E2E_ENV=production`; passar `E2E_ENV` ao step que roda `npm test`; artifact do relatório com nome que inclua o ambiente (ex.: `playwright-report-${{ env.E2E_ENV }}`). Ver prompt 03.
- `.gitignore` com `node_modules/`, `playwright-report/`, `test-results/`.
- **README** com instruções para rodar por ambiente (scripts `test:local`, `test:staging`, `test:production` e variáveis `E2E_ENV`/`E2E_BASE_URL`), e com explicação do CI dinâmico (push/PR = production; execução manual = escolha do ambiente).

O agente que executar este prompt deve conferir a lista acima e, se algo estiver faltando ou inconsistente com os gotchas, ajustar. Antes de escrever ou alterar page objects, inspecionar a aplicação (navegando ou lendo o HTML) para garantir que os textos, roles e placeholders usados nos locators correspondam ao que realmente existe na página.

---

**Fim do fluxo.** Para recomeçar do zero, use **`01-mcp-and-scope.prompt.md`** e siga a cadeia até o prompt 06.
