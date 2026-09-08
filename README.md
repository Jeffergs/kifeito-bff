# Kifeito BFF

O **Kifeito BFF (Backend for Frontend)** é o ponto de entrada do frontend para as funcionalidades da aplicação.

O BFF é responsável por:

- Expor uma API adequada às necessidades do frontend;
- Encaminhar requisições aos microsserviços responsáveis pelo domínio;
- Adaptar contratos entre frontend e microsserviços quando necessário;
- Agregar informações provenientes de diferentes serviços para composição de telas;
- Centralizar o acesso do frontend aos serviços internos.

O BFF **não é responsável pelas regras de negócio dos domínios de User, Tasks ou Notification**.

---

## 📋 Índice

1. [🎯 Responsabilidade](#-responsabilidade)
2. [🔐 Autenticação](#-autenticação)
3. [👤 Usuário](#-usuário)
4. [📋 Tarefas](#-tarefas)
5. [📊 Dashboard](#-dashboard)
6. [🔗 Comunicação](#-comunicação)
7. [🛠️ Tecnologias](#️-tecnologias)
8. [📁 Estrutura](#-estrutura)
9. [⚙️ Configuração](#️-configuração)
10. [🐳 Docker](#-docker)
11. [🧪 Testes](#-testes)
12. [📄 Licença](#-licença)

---

# 🎯 Responsabilidade

O serviço possui uma responsabilidade específica:

> **Disponibilizar uma API adequada ao frontend e intermediar o acesso aos microsserviços internos do Kifeito.**

O BFF centraliza o acesso do frontend aos serviços internos, evitando que o frontend precise conhecer diretamente os endpoints e contratos de cada microsserviço.

| Responsabilidade | Serviço |
|---|---|
| API utilizada pelo frontend | BFF |
| Adaptação de contratos | BFF |
| Encaminhamento de requisições | BFF |
| Agregação de dados para telas | BFF |
| Regras de negócio de usuário | User |
| Autenticação | User |
| Regras de negócio de tarefas | Tasks |
| Persistência das tarefas | Tasks |
| Lembretes e notificações | Notification |

O BFF não deve duplicar as regras de negócio dos microsserviços.

⬆️ [Voltar ao índice](#-índice)

---

# 🔐 Autenticação

O BFF disponibiliza os endpoints necessários para o frontend realizar autenticação.

| Operação | Método | Endpoint |
|---|---|---|
| Cadastro | POST | `/api/auth/register` |
| Login | POST | `/api/auth/login` |

As operações são encaminhadas ao serviço **User**, responsável pela autenticação e pelo gerenciamento das credenciais.

### Fluxo de cadastro

```text
Frontend
  ↓
BFF
  ↓
User
  ↓
Validação
  ↓
Criação da conta
  ↓
Resposta
  ↓
BFF
  ↓
Frontend
```

### Fluxo de login

```text
Frontend
  ↓
BFF
  ↓
User
  ↓
Autenticação
  ↓
Geração do JWT
  ↓
BFF
  ↓
Frontend
```

O BFF não é responsável por validar diretamente as credenciais do usuário.

O serviço User permanece responsável pela autenticação.

⬆️ [Voltar ao índice](#-índice)

---

# 👤 Usuário

O BFF disponibiliza as operações relacionadas à própria conta do usuário.

| Operação | Método | Endpoint |
|---|---|---|
| Consultar perfil | GET | `/api/user` |
| Atualizar perfil | PUT | `/api/user` |
| Excluir conta | DELETE | `/api/user` |

As operações são encaminhadas ao serviço **User**, que permanece responsável pelo domínio e pelas regras relacionadas à conta.

O BFF apenas disponibiliza o contrato adequado ao frontend.

### Fluxo

```text
Frontend
  ↓
BFF
  ↓
User
  ↓
Regra de negócio
  ↓
Resposta
  ↓
BFF
  ↓
Frontend
```

O `userId` não é fornecido pelo frontend para determinar a conta.

A identificação do usuário autenticado é obtida a partir do contexto de autenticação.

⬆️ [Voltar ao índice](#-índice)

---

# 📋 Tarefas

O BFF disponibiliza ao frontend as operações de gerenciamento das tarefas.

| Operação | Método | Endpoint |
|---|---|---|
| Listar | GET | `/api/tasks` |
| Buscar | GET | `/api/tasks/{id}` |
| Criar | POST | `/api/tasks` |
| Atualizar | PUT | `/api/tasks/{id}` |
| Concluir | PATCH | `/api/tasks/{id}/complete` |
| Reabrir | PATCH | `/api/tasks/{id}/reopen` |
| Cancelar | PATCH | `/api/tasks/{id}/cancel` |
| Reativar | PATCH | `/api/tasks/{id}/reactivate` |
| Excluir | DELETE | `/api/tasks/{id}` |

As operações são encaminhadas ao serviço **Tasks**, que é responsável pelo domínio, pelas regras de negócio e pela persistência das tarefas.

### Fluxo

```text
Frontend
  ↓
BFF
  ↓
Tasks
  ↓
Regra de negócio
  ↓
Persistência
  ↓
Resposta
  ↓
BFF
  ↓
Frontend
```

O `userId` não é fornecido pelo frontend.

A identificação do usuário autenticado é obtida a partir do contexto de autenticação e encaminhada ao serviço responsável pela operação.

O BFF não controla o ciclo de vida da tarefa.

⬆️ [Voltar ao índice](#-índice)

---

# 📊 Dashboard

O BFF disponibiliza um endpoint específico para fornecer ao frontend os dados necessários para a composição do Dashboard.

**GET /api/dashboard**

O Dashboard **não representa um domínio ou entidade própria**.

Ele representa uma visão da aplicação destinada ao frontend.

O BFF pode consultar um ou mais microsserviços, agregar os dados necessários e retornar uma resposta adequada à tela.

### Fluxo

```text
Frontend
  ↓
GET /api/dashboard
  ↓
BFF
  ├──→ User
  │
  └──→ Tasks
         ↓
       dados das tarefas
  ↓
Agregação
  ↓
DashboardResponse
  ↓
Frontend
```

### Exemplo de informações

```text
┌─────────────────────────────────────────────┐
│                                             │
│  Olá, Jefferson                             │
│                                             │
│  Total        Pendentes        Concluídas   │
│    20              8                10      │
│                                             │
│  Próximas tarefas                           │
│  ─────────────────────────────────────────  │
│  Estudar Spring       04/09 19:00           │
│  Fazer exercício      05/09 07:00           │
│                                             │
│  Tarefas atrasadas                          │
│  ─────────────────────────────────────────  │
│  Revisar Java         02/09 20:00           │
│                                             │
└─────────────────────────────────────────────┘
```

Essas informações são uma composição para apresentação no frontend.

O BFF não cria um novo domínio de tarefas para armazenar esses dados.

### Responsabilidade

O BFF é responsável por:

- Consultar os serviços necessários;
- Combinar as informações;
- Adaptar os dados para o formato esperado pelo frontend;
- Retornar uma resposta única para composição do Dashboard.

As regras de negócio continuam pertencendo aos microsserviços responsáveis pelos respectivos domínios.

⬆️ [Voltar ao índice](#-índice)

---

# 🔗 Comunicação

O BFF realiza comunicação síncrona com os microsserviços internos.

```text
┌──────────────┐
│   Frontend   │
└──────┬───────┘
       │
       │ HTTP
       ▼
┌──────────────┐
│     BFF      │
└──────┬───────┘
       │
       ├───────────────┐
       │               │
       ▼               ▼
┌──────────────┐ ┌──────────────┐
│     User     │ │    Tasks     │
└──────────────┘ └──────────────┘
```

### User

Responsável pelas operações relacionadas à identidade, autenticação e conta do usuário.

### Tasks

Responsável pelas operações relacionadas às tarefas.

### Notification

O BFF não se comunica diretamente com o Notification na V1.

O Notification recebe eventos do Tasks através do RabbitMQ.

```text
Tasks
  │
  │ eventos
  ▼
RabbitMQ
  │
  ▼
Notification
```

O BFF não participa desse fluxo.

⬆️ [Voltar ao índice](#-índice)

---

# 🛠️ Tecnologias

| Tecnologia | Utilização |
|---|---|
| Java 17 | Linguagem |
| Spring Boot | Framework |
| Spring Web | Desenvolvimento da API REST |
| Spring Cloud OpenFeign | Comunicação com microsserviços |
| Bean Validation | Validação dos dados de entrada |
| Lombok | Redução de código repetitivo |
| Maven | Build |
| Docker | Containerização |
| GitHub Actions | CI |

⬆️ [Voltar ao índice](#-índice)

---

# 📁 Estrutura

Estrutura inicial:

```text
kifeito-bff
│
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com.jefferson.bff
│   │   │       │
│   │   │       ├── controller
│   │   │       │   ├── AuthController.java
│   │   │       │   ├── UserController.java
│   │   │       │   ├── TaskController.java
│   │   │       │   └── DashboardController.java
│   │   │       │
│   │   │       ├── service
│   │   │       │   ├── AuthService.java
│   │   │       │   ├── UserService.java
│   │   │       │   ├── TaskService.java
│   │   │       │   └── DashboardService.java
│   │   │       │
│   │   │       ├── client
│   │   │       │   ├── UserClient.java
│   │   │       │   └── TaskClient.java
│   │   │       │
│   │   │       ├── dto
│   │   │       │   ├── auth
│   │   │       │   ├── user
│   │   │       │   ├── task
│   │   │       │   └── dashboard
│   │   │       │
│   │   │       ├── exception
│   │   │       │   ├── ApiException.java
│   │   │       │   └── GlobalExceptionHandler.java
│   │   │       │
│   │   │       ├── config
│   │   │       │   └── FeignConfig.java
│   │   │       │
│   │   │       └── BffApplication.java
│   │   │
│   │   └── resources
│   │       └── application.yaml
│
├── .github
│   └── workflows
│       └── pull-request.yml
│
├── .gitignore
├── Dockerfile
├── build.gradle
├── gradlew
├── gradlew.bat
└── README.md
```

⬆️ [Voltar ao índice](#-índice)

---

# ⚙️ Configuração

Os endereços dos microsserviços são fornecidos por variáveis de ambiente.

```text
USER_SERVICE_URL
TASKS_SERVICE_URL
```

As configurações específicas de ambiente não devem ser armazenadas diretamente no código-fonte.

> O arquivo `.env` não deve ser versionado.

⬆️ [Voltar ao índice](#-índice)

---

# 🐳 Docker

O Kifeito BFF possui seu próprio `Dockerfile` e pode ser executado junto aos demais serviços através do Docker Compose.

O Docker garante um ambiente de execução padronizado, facilita a configuração e permite executar os serviços de forma isolada e reproduzível.

⬆️ [Voltar ao índice](#-índice)

---

# 🧪 Testes

O serviço terá testes unitários para as regras de negócio e testes de integração para suas principais integrações.

- **Testes unitários:** adaptação de contratos, encaminhamento de requisições e composição dos dados do Dashboard.
- **Testes de integração:** User, Tasks e chamadas HTTP entre os serviços.

⬆️ [Voltar ao índice](#-índice)

---

# 📄 Licença

O Kifeito está sendo desenvolvido inicialmente para uso próprio e para um grupo limitado de usuários.

Apesar do uso inicial restrito, o projeto está sendo desenvolvido com arquitetura, práticas e estrutura voltadas para um produto comercial, podendo futuramente ser disponibilizado de forma mais ampla.

O código-fonte, a aplicação, a identidade visual, a documentação e demais componentes do projeto são de propriedade do próprio autor.

A utilização, cópia, modificação, distribuição ou comercialização de qualquer parte do projeto depende de autorização expressa do detentor dos direitos.

⬆️ [Voltar ao índice](#-índice)
