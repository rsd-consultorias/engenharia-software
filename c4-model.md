# O que é o C4 Model?

O **C4 Model** é uma abordagem para a criação de diagramas de arquitetura de software que visa facilitar a compreensão de sistemas complexos, utilizando quatro níveis de abstração. Criado por Simon Brown, o nome "C4" vem dos quatro níveis que ele propõe: **Context**, **Container**, **Component** e **Code**.

---

## Níveis do C4 Model

### 1. **Context (Contexto)**
- **Foco:** Visão geral do sistema.
- **Objetivo:** Mostrar como o sistema se encaixa no ambiente em que opera. Inclui os usuários e outros sistemas externos com os quais ele interage.
- **Pergunta respondida:** *"Quem usa o sistema e como ele se conecta a outros sistemas?"*
- **Exemplo:** Um diagrama indicando que o "Sistema A" é usado por um usuário final e se comunica com o "Sistema B".

---

### 2. **Container (Contêiner)**
- **Foco:** A arquitetura dentro do sistema.
- **Objetivo:** Dividir o sistema em "contêineres" (como aplicações web, APIs, bancos de dados, etc.), mostrando como eles interagem.
- **Pergunta respondida:** *"O que compõe o sistema e onde estão os limites de execução?"*
- **Exemplo:** Um diagrama que mostra um aplicativo web, um serviço backend e um banco de dados como partes do sistema.

---

### 3. **Component (Componente)**
- **Foco:** O que há dentro de cada contêiner.
- **Objetivo:** Representar os componentes principais que compõem cada contêiner e como eles interagem entre si.
- **Pergunta respondida:** *"Quais são os blocos de construção no interior de cada contêiner?"*
- **Exemplo:** Mostrar que o contêiner do backend é composto de um módulo de autenticação, um módulo de pagamento e um módulo de notificação.

---

### 4. **Code (Código)**
- **Foco:** O nível mais baixo de detalhe.
- **Objetivo:** Visualizar a implementação técnica de um componente específico em termos de classes, métodos ou funções.
- **Pergunta respondida:** *"Como um componente é implementado?"*
- **Exemplo:** Diagramar uma classe específica dentro do módulo de autenticação.

---

## Benefícios do C4 Model
- **Clareza e Simplicidade:** Ajuda equipes técnicas e não-técnicas a entender sistemas complexos em diferentes níveis de detalhe.
- **Escalabilidade:** Funciona bem para projetos de software pequenos ou grandes.
- **Colaboração:** Facilita a comunicação entre desenvolvedores, arquitetos e stakeholders.
