# Prompt 3: Scaffolding do projeto e CI (GitHub Actions)

**Responsabilidade:** Estrutura inicial do projeto (deps, config Playwright, pastas) e workflow de CI dinâmico por ambiente.

**Pressuposto:** Ambientes já configurados em `config/environments.ts` e uso de `getBaseUrl()` no POM/fixture (prompt 02).

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

Antes de escrever os testes, **navegue na aplicação** (ou consulte o HTML/estrutura) para identificar títulos, labels, placeholders, roles e textos exatos dos elementos que serão usados nos locators.

---

## 3.4 CI (GitHub Actions) dinâmico

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

**Próximo passo:** use o prompt **`04-practices-and-pom.prompt.md`** para aplicar boas práticas do Playwright e o Page Object Model.
