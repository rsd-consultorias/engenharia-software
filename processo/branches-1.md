# Processo de Commit, Branches e Merge Request para Ambientes de Dev, UAT e Prod

Este documento descreve de forma detalhada o fluxo de commits, branches e merge requests, permitindo o trabalho colaborativo de múltiplos times em um único projeto e oferecendo suporte à equipe de sustentação para a aplicação de hotfixes. Cada ambiente (Dev, UAT e Prod) é atualizado via merge requests que promovem as mudanças conforme a branch correspondente.

---

## 1. Estrutura de Branches

- **Branch `dev`:**  
  - **Ambiente:** Desenvolvimento.  
  - **Uso:** Todas as implementações e correções iniciam a partir desta branch.  
  - **Prática:** Desenvolvedores criam branches de feature ou bugfix a partir de `dev`.

- **Branch `uat`:**  
  - **Ambiente:** Homologação.  
  - **Uso:** Após a integração e validação na branch `dev`, as mudanças são promovidas para a branch `uat` por meio de uma merge request (MR).  
  - **Prática:** O ambiente UAT simula a produção e é utilizado para testes de aceitação.

- **Branch `prod`:**  
  - **Ambiente:** Produção.  
  - **Uso:** Recebe somente código revisado e testado, através de uma MR final, garantindo deploys seguros.  
  - **Prática:** A branch é protegida e só recebe alterações por meio de MRs aprovadas.

---

## 2. Processo de Commits

- **Mensagens Padronizadas:**  
  Utilize convenções como o *Conventional Commits* para manter mensagens claras e consistentes, por exemplo:  
  - `feat: implementar nova funcionalidade de reserva`  
  - `fix: corrigir bug no fluxo de autenticação`  
  - `refactor: reorganização do código de integração`

- **Commits Pequenos e Frequentes:**  
  Realize commits focados em tarefas específicas para facilitar a revisão, o rastreamento das mudanças e eventuais reversões.

- **Associação a Issues ou Tickets:**  
  Sempre associe o commit a um ticket ou issue (por exemplo, `#123`), garantindo melhor rastreabilidade e comunicação entre os times.

---

## 3. Processo de Merge Requests (MRs)

- **Criação de Feature/Bugfix Branches:**  
  Cada nova funcionalidade ou correção é desenvolvida a partir da branch `dev`. Após concluir a tarefa e testar localmente, o desenvolvedor abre uma MR para integrar sua alteração na branch `dev`.

- **Etapas de Revisão:**  
  - **Revisão de Código:**  
    Cada MR deve ser revisada por pelo menos um membro da equipe, garantindo que o código siga os padrões estabelecidos e evitando conflitos.  
  - **Testes Automatizados:**  
    Configure um pipeline de CI/CD para executar testes unitários, de integração e testes de regressão (quando aplicável) em cada MR.

- **Fluxo de Promoção dos Ambientes:**  
  - **Promoção para UAT:**  
    Quando a branch `dev` acumula um conjunto consistente de mudanças testadas, abre-se uma MR para promover estes códigos para a branch `uat`.  
    Neste ambiente, são executados os testes de aceitação e homologação.  
  - **Promoção para Prod:**  
    Após validação completa no ambiente UAT, uma MR final é criada para mesclar a branch `uat` na branch `prod`.  
    Dessa forma, o deploy na produção ocorre apenas com código revisado e testado.

- **MRs Automatizadas e Sincronizadas:**  
  A aprovação de cada MR aciona automaticamente o pipeline de CI/CD que realiza o deploy na branch e ambiente correspondente.

---

## 4. Processo de Hotfixes

- **Criação de Branch de Hotfix:**  
  Em casos de emergência ou erros críticos em produção, o time de sustentação cria uma branch de hotfix a partir da branch `prod` (por exemplo, `hotfix/identificador` ou `hotfix/nome-correcao`).

- **Implementação e Testes:**  
  O hotfix é desenvolvido e testado com prioridade, passando por uma revisão rápida e testes automatizados para garantir sua eficácia.

- **Propagação das Correções:**  
  Após o merge e o deploy do hotfix na branch `prod`, é essencial propagar a correção para as branches `dev` e `uat` através de MRs específicas, mantendo a consistência dos ambientes.

---

## 5. Coordenação entre Múltiplos Times

- **Branches de Feature por Time:**  
  Cada time deve criar e gerenciar suas branches de feature a partir da `dev`.

- **Comunicação e Sincronização:**  
  Utilize ferramentas de comunicação (como Slack ou Microsoft Teams) e sistemas de rastreamento de issues (como JIRA ou Trello) para acompanhar o status dos merges, resolver conflitos e gerenciar dependências.

- **Políticas de Revisão e Aprovação:**  
  Defina regras de revisão de código que exijam um número mínimo de revisores e garantam que os testes automatizados sejam executados antes do merge.

- **Integração Contínua e Deploy:**  
  Configure um pipeline central de CI/CD que, uma vez detectado um merge aprovado, realize automaticamente o deploy do código no ambiente correspondente, permitindo feedback rápido sobre a qualidade das mudanças.

---

## 6. Fluxo Resumido de Exemplo

1. **Desenvolvimento:**  
   - Um desenvolvedor cria uma branch de feature (por exemplo, `feature/nova-funcionalidade`) a partir da `dev`.  
   - Realiza commits pequenos e frequentes com mensagens padronizadas.  
   - Após testar localmente, abre uma MR para fundir a feature na branch `dev`.

2. **Integração e Testes:**  
   - A MR passa por uma revisão de código e, após aprovação, o pipeline de CI/CD realiza testes e efetua o deploy no ambiente de desenvolvimento.

3. **Promoção para UAT:**  
   - Com as mudanças validadas em `dev`, é aberta uma MR para promover o código para a branch `uat`.  
   - São realizados testes de aceitação e homologação no ambiente UAT.

4. **Promoção para Prod:**  
   - Com a homologação completa, abre-se uma MR final para mesclar a branch `uat` na `prod`.  
   - O deploy em produção ocorre automaticamente após a aprovação e execução dos testes.

5. **Hotfix:**  
   - Se um erro crítico for identificado em produção, o time de sustentação cria uma branch de hotfix (ex.: `hotfix/erro-crítico`) a partir da `prod`, realiza a correção e abre uma MR para merge imediato em `prod`.  
   - A correção de hotfix é, então, propagada para as branches `dev` e `uat` por meio de MRs específicas.

---

## 7. Considerações Finais

- **Automação:**  
  O uso intenso de CI/CD é crucial para validar todas as etapas do processo, garantindo a execução sistemática dos testes e a segurança dos deploys.

- **Revisão de Código e Qualidade:**  
  Estabeleça diretrizes e templates para as MRs, assegurando que a qualidade do código se mantenha elevada mesmo com a colaboração de múltiplos times.

- **Documentação e Comunicação:**  
  Mantenha a documentação do procedimento atualizada e promova reuniões periódicas para discutir melhorias e resolver eventuais gargalos.
