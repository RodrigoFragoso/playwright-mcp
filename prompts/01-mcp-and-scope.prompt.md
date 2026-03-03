# Prompt 1: MCP do Playwright e escopo da aplicação

**Responsabilidade:** Definir como usar o MCP para inspecionar a aplicação antes de escrever testes e fixar o escopo do projeto E2E.

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

- **Onde criar o projeto:** todos os arquivos e pastas devem ser criados **na pasta atual** em que o usuário está (diretório de trabalho atual). Não criar subpastas como “projeto” ou “playwright-mcp” — o projeto vive na pasta atual.
- **URL da aplicação (production):** `https://erickwendel.github.io/vanilla-js-web-app-example/`
- **Objetivo:** Configurar o Playwright (somente test runner, sem outros frameworks), criar a pasta de testes e escrever specs que cubram smoke, formulário (submit + validação) e listagem. Os mesmos testes devem poder rodar em **localhost**, **staging** ou **production** (ver prompt de ambientes).

---

**Próximo passo:** use o prompt **`02-environments.prompt.md`** para configurar os ambientes (config, baseUrl, scripts por ambiente).
