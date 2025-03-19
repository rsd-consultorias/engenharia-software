# Melhores Práticas para Nomeação em Java e C#

## Classes
- Use **nomes substantivos** que reflitam o papel da classe.
  - Exemplo: `InvoiceProcessor`, `UserAccount`, `OrderManager`.
- Utilize **CamelCase**, iniciando com letra maiúscula.
  - Exemplo: `CustomerService`, `PaymentGateway`.
- Não inclua detalhes de implementação no nome.
  - **Evitar**: `HashBasedListManager`. Prefira: `ListManager`.

---

## Funções e Métodos
- Use **verbos ou frases verbais** para descrever o que o método faz.
  - Exemplo: `calculateTotal()`, `fetchDataFromServer()`, `validateInput()`.
- **CamelCase**, começando com letra minúscula.
  - Exemplo: `processOrder()`, `sendEmailNotification()`.
- Nomeie métodos de forma específica e clara.
  - Exemplo: `getUserById()` ao invés de `getData()`.

---

## Variáveis
- Use **nomes descritivos** que expliquem o propósito da variável.
  - Exemplo: `userAge`, `totalPrice`, `orderList`.
- Utilize **CamelCase**, começando com letra minúscula.
  - Exemplo: `currentIndex`, `errorCount`.
- Evite abreviações exceto quando muito óbvias.
  - **Evitar**: `usr` ao invés de `user`.

---

## Parâmetros
- Nomeie parâmetros de acordo com seu uso na função/método.
  - Exemplo: `username`, `productId`, `orderQuantity`.
- Evite generalizações, como `obj`, `val`, ou `data`, salvo em casos genéricos.
  - **Evitar**: `arg1`, `obj1`.

---

## Constantes
- Use **nomes em UPPER_SNAKE_CASE** para constantes.
  - Exemplo: `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`, `PI`.
- Nomeie de maneira descritiva para indicar o propósito.
  - Exemplo: `MAX_USERS_PER_SESSION` ao invés de `MAX_USERS`.
- Prefira adicionar um prefixo para contexto se necessário.
  - Exemplo: `DB_DEFAULT_PORT` para uma constante relacionada ao banco de dados.

---

## Dicas Gerais
- Sempre **priorize clareza sobre brevidade**. Um nome mais longo, mas claro, é preferível a um nome curto e confuso.
- Use o inglês para consistência, especialmente em projetos internacionais.
- Evite palavras reservadas da linguagem e caracteres especiais.
- Siga o **Guia de Estilo** do seu time ou projeto.
