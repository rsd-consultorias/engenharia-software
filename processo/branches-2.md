# Processo de Deploy com Artefato Congelado em UAT

Este processo garante que o pacote (artifact) testado em UAT seja exatamente o mesmo que será implantado em produção. Além disso, descreve como os hotfixes são resolvidos seguindo uma abordagem similar, para manter a consistência entre os ambientes.

---

## 1. Estrutura de Branches

- **Branch `dev`:**  
  - **Ambiente:** Desenvolvimento.  
  - **Uso:** Todas as novas features e correções iniciam a partir dessa branch.  
  - **Prática:** Desenvolvedores criam branches de feature/bugfix a partir de `dev`.

- **Branch `uat`:**  
  - **Ambiente:** Homologação.  
  - **Uso:** Realiza a validação dos pacotes construídos a partir de `dev`.  
  - **Prática:** Ao mergear de `dev` para `uat`, o pipeline gera um artifact que é testado e, uma vez aprovado, é congelado para promoção.

- **Branch `prod`:**  
  - **Ambiente:** Produção.  
  - **Uso:** Recebe **exatamente** o mesmo artifact que foi testado e aprovado em UAT, garantindo que nada seja recompilado ou alterado no caminho.  
  - **Prática:** A promoção para produção é feita via merge request que utiliza o artifact congelado de UAT.

---

## 2. Processo de Build e Deploy

### 2.1 Desenvolvimento e Build em UAT

1. **Desenvolvimento e Integração em `dev`:**  
   - Desenvolvedores trabalham em branches de feature/bugfix originadas de `dev`.  
   - Após revisões e testes locais, as mudanças são mergeadas em `dev`.

2. **Promoção para UAT:**  
   - Um merge request (MR) é aberto para fundir `dev` em `uat`.  
   - Ao fazer o merge na branch `uat`, o pipeline de CI/CD é disparado para:
     - **Buildar o Artifact:** Gera o pacote que será utilizado para deploy.
     - **Testar o Artifact:** Executa testes automatizados (unitários, integração e aceitação) no ambiente UAT.
   - **Congelamento do Artifact:** Assim que o pacote passa nos testes de UAT, ele é marcado como “congelado” (armazenado e versionado) para ser utilizado na promoção para produção.

### 2.2 Promoção do Artifact para Produção

1. **Merge Request para Produção:**  
   - Com o artifact congelado e validado em UAT, um MR é aberto para promover `uat` em `prod`.
   
2. **Deploy do Mesmo Artifact:**  
   - O pipeline de CI/CD não realiza uma nova build em `prod`; ele simplesmente pega o artifact congelado e o implanta em produção.
   - **Benefício:** Isso garante que o mesmo pacote testado em UAT seja exatamente o que está rodando em produção.

3. **Validação em Produção:**  
   - Monitoramento e eventuais testes de fumaça podem ser realizados em produção para confirmar que o deploy foi realizado sem incidentes.

---

## 3. Processo de Hotfixes

### 3.1 Fluxo de Hotfix

1. **Identificação e Criação da Branch de Hotfix:**
   - Ao identificar um bug crítico em produção, o time de sustentação cria uma branch de hotfix a partir da branch `prod`.  
     - **Exemplo de nomenclatura:** `hotfix/descricao-do-problema`

2. **Implementação do Hotfix:**
   - O hotfix é desenvolvido com prioridade e submetido a commits rápidos e controlados.
   - **Geração do Artifact:** O pipeline gera um novo artifact especificamente para o hotfix.

3. **Teste do Hotfix em UAT:**
   - Antes de promover para produção, o artifact do hotfix é implantado em UAT:
     - Um merge request é aberto para integrar as mudanças da branch de hotfix na branch `uat`.  
     - O mesmo processo de build e teste é aplicado: o artifact é gerado, testado e, se aprovado, congelado.

4. **Promoção do Hotfix para Produção:**
   - Assim como no fluxo principal, um MR é aberto para promover as mudanças (já testadas e congeladas em UAT) para a branch `prod`.  
   - O pipeline pega o artifact do hotfix testado e o implanta em produção.

5. **Propagação para Outras Branches:**
   - Após o deploy do hotfix, o código corrigido é mesclado de volta em `dev` (e, se necessário, em `uat`) para manter todos os ambientes sincronizados.

---

## 4. Coordenação e Automação com CI/CD

- **CI/CD Automatizado:**  
  - Cada merge request dispara um pipeline com etapas de build e teste.  
  - O artifact é versionado e armazenado num repositório de artefatos (ex.: Nexus, Artifactory), garantindo imutabilidade entre UAT e produção.

- **Artifact Promotion:**  
  - O pipeline que implanta em produção utiliza o artifact congelado de UAT, eliminando diferenças entre o que foi testado e o que é implantado.

- **Monitoramento e Feedback:**  
  - Instrumente os ambientes com monitoramento e alertas para garantir que tanto o artifact principal quanto os hotfixes operem conforme esperado.

---

## 5. Fluxo Resumido de Exemplo

1. **Desenvolvimento e Merge em `dev`:**
   - Branches de feature/bugfix são criadas a partir de `dev` e, após revisão, são mergeadas em `dev`.

2. **Promoção para UAT:**
   - Merge request de `dev` para `uat`.  
   - O pipeline gera e testa um artifact em UAT.  
   - O artifact aprovado é congelado.

3. **Promoção para Produção:**
   - Merge request de `uat` para `prod`.  
   - O pipeline utiliza o mesmo artifact congelado e o implanta em produção.

4. **Hotfix:**
   - Ao detectar um problema crítico, cria-se uma branch `hotfix` a partir de `prod`.  
   - Desenvolve-se o fix e o pipeline gera um novo artifact para o hotfix.  
   - Após teste e congelamento em UAT, o MR promove o artifact do hotfix para produção.  
   - O hotfix é mesclado de volta em `dev` e `uat`.

---

## 6. Considerações Finais

- **Integridade do Artifact:**  
  - Usar o mesmo pacote testado em UAT para produção elimina discrepâncias decorrentes de novas builds ou alterações.
  
- **Processo Ágil de Hotfix:**  
  - O fluxo de hotfix permite a rápida correção de problemas críticos enquanto garante que a correção seja devidamente testada antes do deploy.
  
- **Automação e Monitoramento:**  
  - Uma robusta infraestrutura de CI/CD e um repositório de artifacts garantem a consistência e a segurança entre os ambientes.

Essa abordagem assegura que o artifact validado em UAT seja promovido para produção sem alterações. Os hotfixes seguem o mesmo padrão, garantindo consistência e agilidade na resolução de falhas.
