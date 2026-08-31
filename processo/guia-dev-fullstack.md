# Guia de desenvolvimento de features: do zero até entrega

Este documento descreve, de forma prática e sequencial, como uma feature deve ser construída em um projeto fullstack, cobrindo as etapas de backend, frontend, integração, validação e entrega.

O objetivo é ajudar a equipe a planejar bem o trabalho, evitar retrabalho e garantir que cada feature seja entregue com clareza de requisitos, arquitetura, testes e qualidade.

---

## 1. Visão geral do processo

Uma feature completa normalmente passa por estas grandes fases:

1. Entendimento do problema e do objetivo de negócio
2. Definição do escopo e critérios de aceite
3. Modelagem de dados e contrato de integração
4. Desenvolvimento do backend
5. Desenvolvimento do frontend
6. Integração entre frontend e backend
7. Testes e validação
8. Revisão, ajustes e entrega

Importante: backend e frontend não devem ser desenvolvidos “de olhos fechados”. Eles precisam andar em paralelo, mas com um contrato claro.

---

## 2. Etapa 1: entendimento da feature

Antes de escrever código, a equipe precisa responder:

- Qual problema a feature resolve?
- Quem é o usuário?
- Qual ação ele precisa realizar?
- Qual é o resultado esperado?
- Quais regras de negócio existem?
- Existem casos de sucesso e casos de erro?
- Existem regras de autorização/autenticação?
- Há dependências com outros sistemas?

### Entregáveis esperados
- História de usuário ou descrição da feature
- Critérios de aceite
- Fluxo principal
- Fluxos alternativos e erros
- Definição de campos necessários
- Definição de status e regras de negócio

### Exemplo
Feature: “Criar pedido”

Perguntas:
- Quem pode criar um pedido?
- Quais campos são obrigatórios?
- O pedido pode ser criado sem itens?
- O sistema precisa validar estoque antes de confirmar?
- Qual é o status inicial do pedido?
- O backend precisa enviar evento para outro serviço?
- O frontend precisa mostrar feedback de sucesso/erro?

---

## 3. Etapa 2: definição do escopo e critérios de aceite

A feature precisa ser traduzida em requisitos claros e testáveis.

### Critérios de aceite devem responder:
- O que exatamente deve acontecer?
- Quando a ação é considerada concluída?
- Quais validações são obrigatórias?
- Como o usuário percebe sucesso ou falha?
- Quais operações precisam ser persistidas no banco?
- Quais erros devem aparecer para o usuário?

### Exemplo de critério de aceite
- Usuário autenticado consegue criar um pedido com pelo menos 1 item
- Sistema valida saldo do estoque antes de confirmar
- Caso não haja estoque, exibe mensagem de erro
- Pedido é salvo com status “PENDING”
- API retorna 201 Created no sucesso
- Frontend redireciona para a tela de detalhes do pedido

---

## 4. Etapa 3: modelagem do problema e contrato de dados

Antes do desenvolvimento, a equipe deve decidir:
- Estruturas de dados do backend
- Estruturas de dados do frontend
- Campos obrigatórios
- Formatos de datas e valores
- Enumerações de status
- Endpoints necessários
- Resposta esperada da API
- Casos de erro

### O que definir nesta etapa
- Entidades/objetos do domínio
- Campos do banco
- DTOs de entrada e saída
- Payload JSON
- Validações de campos
- Regras de negócio

### Exemplo
Pedido:
- id
- userId
- status
- totalAmount
- createdAt
- items[]

Item:
- sku
- quantity
- unitPrice

### Contrato da API
- POST /api/orders
- Body:
  ```json
  {
    "userId": "user-123",
    "items": [
      { "sku": "SKU-001", "quantity": 2 }
    ]
  }
  ```
- Resposta:
  ```json
  {
    "id": 101,
    "status": "PENDING",
    "totalAmount": 199.90,
    "createdAt": "2026-08-31T10:00:00Z"
  }
  ```

---

## 5. Etapas do backend

O backend é responsável por implementar a regra de negócio, persistência, validações e integração com outros serviços.

### 5.1. Definição da estrutura da API
- Endpoint
- Método HTTP
- Path
- Parâmetros de rota e query
- Body da requisição
- Códigos de resposta esperados

Exemplo:
- POST /api/orders
- 201 Created
- 400 Bad Request
- 401 Unauthorized
- 404 Not Found
- 409 Conflict

### 5.2. Modelagem do banco de dados
- Criação de tabela(s)
- Campos
- Relacionamentos
- Constraints
- Índices
- Observações com histórico e auditoria

Se o projeto usa migrations:
- criar migration
- validar script
- aplicar no ambiente de desenvolvimento

### 5.3. Entidades e regras de domínio
- Entidade principal
- Relacionamentos
- Validações de negócio
- Status e transições
- Regras de integridade

Exemplo:
- pedido só pode ser criado se houver itens
- quantidade deve ser maior que zero
- soma total deve ser calculada corretamente
- estoque precisa ser validado antes do envio

### 5.4. DTOs
Separar entrada e saída da API:
- Request DTO
- Response DTO
- Mapper/Converter
- Validação com Bean Validation ou regras customizadas

### 5.5. Camada de serviço
Implementar a lógica de negócio:
- criação de pedido
- validação de regras
- chamada a outros serviços
- atualização de dados
- manipulação de transações

### 5.6. Repositórios / persistência
- acesso ao banco
- consultas
- filtros
- atualização de estados
- operações transacionais

### 5.7. Integrações externas
Se a feature depende de outro serviço:
- HTTP client
- fila
- mensageria
- cache
- auth service
- gateway ou BFF

Exemplo:
- backend valida estoque no serviço de inventory
- backend publica evento de pedido criado
- backend grava log de auditoria

### 5.8. Tratamento de erro e API
- exception handler
- tipos de erro
- mensagens amigáveis
- códigos HTTP coerentes
- logs estruturados

### 5.9. Segurança
- autenticação
- autorização
- validação de roles/permissões
- proteção contra abuso
- validação de headers, tokens etc.

### 5.10. Testes do backend
Testes obrigatórios:
- unitários
- de integração
- de endpoint/API
- testes de regra de negócio
- testes de error handling

Exemplos:
- salvar pedido com itens válidos
- rejeitar pedido sem itens
- rejeitar quantidade zero
- falhar quando estoque indisponível
- retornar 400 para payload inválido

### 5.11. Documentação
- Swagger/OpenAPI
- exemplos de request/response
- cenários de erro
- campos obrigatórios

---

## 6. Etapas do frontend

O frontend é responsável por entregar a experiência do usuário, coletar dados, validar inputs e comunicar com o backend.

### 6.1. Entendimento da tela ou fluxo
- Onde a feature aparece?
- É uma tela nova ou atualização de uma existente?
- Quem usa?
- Quais campos são exibidos?
- Qual é o fluxo do usuário?

### 6.2. Definição da interface
- layout
- campos
- botões
- estados de loading
- mensagens de sucesso/erro
- acessibilidade
- responsividade

### 6.3. Definição da estrutura de dados da tela
- dados vindos do backend
- dados do formulário
- estados locais
- estados de carregamento
- estados vazios
- erros de validação

### 6.4. Criação do fluxo da UI
- abrir tela
- carregar dados
- editar formulário
- validar campos
- enviar requisição
- tratar sucesso
- tratar erro
- redirecionar ou atualizar lista

### 6.5. Comunicação com o backend
- API client/service
- chamada HTTP
- efeitos de loading
- payload montado corretamente
- tratamento de respostas e erros

### 6.6. Validação do frontend
- campos obrigatórios
- tipos válidos
- regras de negócio do formulário
- impedir ações inválidas antes do envio

### 6.7. Feedback para o usuário
- loading spinner
- disabled buttons
- toast/message
- mensagens de erro específicas
- confirmação de sucesso

### 6.8. Estados da tela
Tratar todos os estados:
- inicial
- carregando
- vazio
- sucesso
- erro
- sem permissão

### 6.9. Testes do frontend
- testes de renderização
- testes de interação
- testes de validação de formulário
- testes de chamadas de API
- testes de casos de erro

---

## 7. Integração frontend + backend

Depois de finalizar cada camada separadamente, a equipe precisa validar o fluxo completo.

### Checklist de integração
- Frontend monta payload correto
- Backend aceita e valida corretamente
- Endpoints estão acessíveis
- Autenticação/autorização estão funcionando
- Respostas pagam corretamente no frontend
- Mensagens de erro ficam legíveis
- Loading e feedback estão funcionando
- Dados persistem corretamente no banco
- Fluxo de sucesso e falha foram testados

---

## 8. Fluxo completo de desenvolvimento de uma feature

Abaixo está uma sequência prática recomendada:

### Fase 1 — Planejamento
- Entender o problema
- Definir escopo
- Escrever critérios de aceite
- Definir contrato de API
- Validar regras com equipe
- Identificar dependências

### Fase 2 — Backend
- Criar/alterar migration
- Modelar banco e entidades
- Criar DTOs
- Implementar regras de negócio
- Criar endpoints
- Tratar erros e segurança
- Fazer testes de backend

### Fase 3 — Frontend
- Criar/editar tela ou componente
- Montar formulário
- Validar inputs
- Integrar com API
- Tratar loading, sucesso e erro
- Fazer testes de UI

### Fase 4 — Integração
- Rodar backend e frontend localmente
- Validar end-to-end
- Corrigir inconsistências de payload
- Corrigir fluxos de erro
- Ajustar UX

### Fase 5 — QA / entrega
- Validar critérios de aceite
- Testar casos negativos
- Validar responsividade e acessibilidade
- Revisar logs, autenticação e segurança
- Fazer deploy ou compartilhar build

---

## 9. Checklist geral para qualquer feature

Use este checklist como referência:

### Requisitos
- [ ] Entendimento do problema claro
- [ ] Critérios de aceite definidos
- [ ] Fluxo principal validado
- [ ] Casos de erro definidos
- [ ] Dependências mapeadas

### Backend
- [ ] Banco/migration definida
- [ ] Entidades e regras de negócio criadas
- [ ] DTOs implementados
- [ ] Endpoints criados
- [ ] Validações e segurança aplicadas
- [ ] Tratamento de erro implementado
- [ ] Testes do backend executados

### Frontend
- [ ] Tela/componente criado
- [ ] Campos e validações definidas
- [ ] Estados de loading/error/sucesso implementados
- [ ] Integração com API feita
- [ ] UX e acessibilidade revisadas
- [ ] Testes do frontend executados

### Integração
- [ ] Backend e frontend funcionando juntos
- [ ] Dados corretos enviados/recebidos
- [ ] Fluxo principal validado
- [ ] Fluxos de falha validados
- [ ] Build e execução local confirmados

---

## 10. Exemplo prático: feature “Criação de pedido”

### Backend
1. Definir endpoint POST /api/orders
2. Validar autenticação do usuário
3. Validar payload
4. Validar itens e quantidade
5. Validar estoque no serviço de inventory
6. Calcular total do pedido
7. Salvar pedido e itens
8. Registrar eventos/auditoria
9. Retornar resposta com status e dados

### Frontend
1. Criar tela de pedido
2. Carregar catálogo ou itens da compra
3. Permitir adicionar/remover itens
4. Validar campos obrigatórios
5. Enviar request ao backend
6. Mostrar spinner e mensagem de sucesso
7. Redirecionar para detalhes do pedido

### Fluxo final
- Usuário clica em “Criar pedido”
- Frontend envia dados
- Backend valida, calcula e persiste
- Frontend recebe resposta
- Tela mostra sucesso e atualiza visualmente

---

## 11. Boas práticas para o time

- Começar pelo problema, não pela tela
- Definir contrato antes de codar
- Desenvolver backend e frontend em paralelo, com alinhamento frequente
- Garantir testes de regra de negócio
- Expor erros claros ao usuário
- Não deixar feature “quase pronta”; validar o fluxo completo
- Registrar decisões de arquitetura e regras de negócio

---

## 12. Conclusão

Uma feature completa não é apenas “tela + endpoint”. Ela exige:
- clareza de negócio
- contrato de dados
- modelagem de domínio
- persistência
- regra de negócio
- API segura e documentada
- frontend amigável e validado
- integração entre sistemas
- testes e QA

