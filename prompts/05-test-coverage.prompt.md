# Prompt 5: O que os testes devem cobrir

**Responsabilidade:** Definir os cenários e comportamentos que os specs E2E devem validar (smoke, formulário, validação).

**Pressuposto:** Boas práticas e POM já aplicados (prompt 04); page object com locators estáveis e fixture disponível.

---

## 5. O que os testes devem cobrir

Gere specs que incluam:

### 5.1 Smoke / página inicial

- Título da página (ex.: "TDD Frontend Example").
- Texto principal visível (main heading) — usar locator com fallback (heading ou getByText) conforme regras do POM (prompt 04).
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

**Próximo passo:** use o prompt **`06-gotchas-and-deliverables.prompt.md`** para aplicar descobertas/gotchas e conferir a lista de entregáveis.
