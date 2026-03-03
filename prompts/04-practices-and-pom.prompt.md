# Prompt 4: Boas práticas Playwright e Page Object Model (POM)

**Responsabilidade:** Regras de locators, AAA, resiliência e POM (estrutura, getters, fixture, labels visíveis, main heading com fallback).

**Pressuposto:** Projeto com scaffolding e ambientes já configurados (prompts 01–03).

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

**Próximo passo:** use o prompt **`05-test-coverage.prompt.md`** para definir o que os testes devem cobrir.
