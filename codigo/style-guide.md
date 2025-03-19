# Guia de Estilo de Código para Java

## 1. Organização Geral
- Utilize **4 espaços para indentação** (não use tabs).
- O comprimento máximo de linhas deve ser **80-120 caracteres**.
- Inclua sempre uma **linha em branco** entre métodos.

## 2. Convenções de Nomeação
- Classes: **CamelCase** com a primeira letra maiúscula.
  - Exemplo: `CustomerService`.
- Métodos: **camelCase** com a primeira letra minúscula.
  - Exemplo: `calculateTotalPrice()`.
- Variáveis: **camelCase** com nomes descritivos.
  - Exemplo: `userId`.
- Constantes: **UPPER_SNAKE_CASE**.
  - Exemplo: `MAX_RETRIES`.

## 3. Espaços e Quebras
- Utilize um espaço após `if`, `for` e `while`.
  - **Correto**: `if (condition) {`.
  - **Errado**: `if(condition){`.
- Abra as chaves `{` na mesma linha que a declaração do bloco.
  - **Correto**:
    ```java
    if (condition) {
        // código
    }
    ```
  - **Errado**:
    ```java
    if (condition)
    {
        // código
    }
    ```

## 4. Comentários
- Use **Javadoc** para documentar classes e métodos públicos.
  - Exemplo:
    ```java
    /**
     * Calcula o preço total dos itens.
     * @param items Lista de itens.
     * @return Preço total.
     */
    public double calculateTotal(List<Item> items) { ... }
    ```

## 5. Boas Práticas
- Sempre inicialize variáveis.
- Evite **métodos longos**. Separe a lógica em métodos menores e reutilizáveis.
- Prefira **interfaces** para expor contratos e **abstrações** ao invés de classes concretas.

---

# Guia de Estilo de Código para C#

## 1. Organização Geral
- Utilize **4 espaços para indentação** (não use tabs).
- O comprimento máximo de linhas deve ser **120 caracteres**.
- Inclua sempre uma **linha em branco** entre métodos.

## 2. Convenções de Nomeação
- Classes: **PascalCase** (primeira letra de cada palavra maiúscula).
  - Exemplo: `CustomerService`.
- Métodos: **PascalCase**.
  - Exemplo: `CalculateTotalPrice()`.
- Variáveis e parâmetros: **camelCase**.
  - Exemplo: `userId`.
- Constantes: **PascalCase** ou **UPPER_SNAKE_CASE**.
  - Exemplo: `DefaultTimeout`, `MAX_RETRIES`.

## 3. Espaços e Quebras
- Use **espaços entre palavras-chave** e parênteses.
  - **Correto**: `if (condition) {`.
  - **Errado**: `if(condition){`.
- Chaves `{` devem ser abertas na **linha seguinte** (padrão Allman).
  - **Correto**:
    ```csharp
    if (condition)
    {
        // código
    }
    ```
  - **Errado**:
    ```csharp
    if (condition) {
        // código
    }
    ```

## 4. Comentários
- Utilize **XML Documentation Comments** para classes e métodos públicos.
  - Exemplo:
    ```csharp
    /// <summary>
    /// Calcula o preço total dos itens.
    /// </summary>
    /// <param name="items">Lista de itens.</param>
    /// <returns>Preço total.</returns>
    public double CalculateTotal(List<Item> items) { ... }
    ```

## 5. Boas Práticas
- Utilize propriedades (`Properties`) ao invés de campos públicos.
- Prefira **expressões lambda** para métodos simples.
- Sempre use `async/await` para métodos assíncronos ao invés de `Task.Wait()` ou `Result`.
