# Roteiro de vídeo — Tech Challenge Fase 2 (FCG)

**Duração total:** até 20 minutos  
**Formato:** 5 partes  
**Antes de gravar:** no repo `fcg-platform`, imagens buildadas (`.\build-images.ps1`) e stack no ar (`docker compose up -d`); Swagger aberto e terminal com logs visível.

---

## Parte 1 — Apresentação e arquitetura (3 min)

**O que mostrar:** slide simples ou diagrama; depois pasta `microservices/` no VS Code.

**Fala sugerida:**

> Olá. Somos o grupo [NOME] e vamos apresentar a Fase 2 do FIAP Cloud Games.
>
> Na Fase 1 tínhamos um monólito. Agora evoluímos para **quatro microsserviços** em .NET 8, comunicando de forma **assíncrona** com **RabbitMQ** e **MassTransit**.
>
> **Users API** — cadastro e login com JWT.  
> **Catalog API** — jogos e início da compra.  
> **Payments API** — simula o pagamento.  
> **Notifications API** — simula e-mails no log.
>
> Os eventos principais são: `UserCreatedEvent`, `OrderPlacedEvent` e `PaymentProcessedEvent`.

**Encerrar a parte:** mostrar rapidamente a pasta `FCG.Messaging.Contracts` com os três eventos.

---

## Parte 2 — Repositórios e estrutura (3 min)

**O que mostrar:** pastas `Users.Api`, `Catalog.Api`, `Payments.Api`, `Notifications.Api`, `platform/`; um `Dockerfile` e uma pasta `k8s/` de qualquer serviço.

**Fala sugerida:**

> Cada microsserviço tem **repositório próprio** na entrega. Aqui no projeto, eles ficam em `microservices/`, com solução em `FCG.Microservices.sln`.
>
> Em cada API temos: código .NET 8, **Dockerfile multi-stage**, pasta **`k8s/`** com Deployment, Service, ConfigMap e Secret, e **README** com variáveis de ambiente.
>
> A orquestração local fica em **`platform/`**, com o `docker-compose.yml` que sobe RabbitMQ e as quatro APIs.

**Encerrar a parte:** abrir `platform/docker-compose.yml` e apontar os cinco serviços (rabbitmq + 4 APIs).

---

## Parte 3 — Docker Compose e ambiente rodando (4 min)

**O que mostrar:** terminal com `docker compose ps` ou Docker Desktop; Swagger Users (5101) e Catalog (5102); opcional RabbitMQ Management (15672).

**Fala sugerida:**

> Como cada microsserviço está em um repositório separado, o compose da orquestração não builda o código: ele consome as **imagens** `fcg/*:latest`. Primeiro buildamos as quatro imagens, cada uma a partir do seu repositório — é o que o `build-images.ps1` faz — e então subimos tudo com um único `docker compose up`.
>
> Aqui estão os containers: RabbitMQ, Mailpit e as quatro APIs nas portas **5101 a 5104**.
>
> Users API na **5101** — autenticação.  
> Catalog API na **5102** — jogos e biblioteca.  
> Payments na **5103** e Notifications na **5104** — processamento assíncrono.

**Encerrar a parte:** abrir Swagger do Users e do Catalog; confirmar que respondem.

---

## Parte 4 — Fluxos de cadastro e compra (7 min)

**O que mostrar:** Swagger ou Postman; logs de `notifications-api`, `payments-api` e `catalog-api` (`docker compose logs -f`).

### 4.1 Cadastro (~2 min)

**Ações:**
1. `POST /api/auth/register` — nome, e-mail, senha forte (ex.: `Senha@123`)
2. Mostrar log: `[EMAIL simulado] Boas-vindas`

**Fala sugerida:**

> No cadastro, o **Users API** grava o usuário e publica o **`UserCreatedEvent`**. O **Notifications API** consome e registra o e-mail de boas-vindas no log.

### 4.2 Compra (~5 min)

**Ações:**
1. Login admin: `admin@fcg.local` / `Admin@123`
2. `POST /api/games` no Catalog — criar jogo (anotar `gameId`)
3. Login do usuário criado
4. `POST /api/users/me/library/games/{gameId}` — resposta **202** com `orderId`
5. Log do **payments-api** — pagamento simulado (Approved)
6. `GET /api/users/me/library` — jogo na biblioteca
7. Log do **notifications-api** — confirmação de compra

**Fala sugerida:**

> A compra não é síncrona. O Catalog publica **`OrderPlacedEvent`** com userId, gameId e preço.
>
> O Payments consome, simula o pagamento e publica **`PaymentProcessedEvent`**. Se aprovado, o Catalog adiciona o jogo à biblioteca e o Notifications envia a confirmação no log.

**Encerrar a parte:** resumir a cadeia em uma frase: *Register → evento → Notifications; Compra → Payments → Catalog + Notifications.*

---

## Parte 5 — Kubernetes e encerramento (3 min)

**O que mostrar:** pasta `k8s/` na **raiz** do repo `fcg-users-api` (Deployment, Service, ConfigMap, Secret); terminal com `.\deploy-k8s.ps1` (ou `kubectl apply`) e `kubectl get pods`.

**Fala sugerida:**

> Para Kubernetes, cada repositório tem os manifestos na pasta **`k8s/`** da raiz: Deployment, Service, ConfigMap e Secret, conforme o enunciado. O repo de orquestração traz a infraestrutura — RabbitMQ e Mailpit.
>
> Aplicamos tudo com `kubectl apply`. Os pods se comunicam pelo nome do Service, por exemplo `rabbitmq` e `payments-api`.
>
> Aqui estão todos os pods em **Running**.
>
> Para encerrar: documentação nos READMEs, checklist em `ENTREGA_CHECKLIST_FASE2.md`, repositórios no GitLab/GitHub e relatório na data da entrega. Obrigado!

**Encerrar a parte:** tela com links dos repositórios e nome do grupo.

---

## Controle de tempo

| Parte | Conteúdo              | Tempo |
|-------|------------------------|-------|
| 1     | Apresentação e arquitetura | 3 min |
| 2     | Repositórios e estrutura   | 3 min |
| 3     | Docker Compose             | 4 min |
| 4     | Fluxos cadastro + compra   | 7 min |
| 5     | Kubernetes + encerramento  | 3 min |
| **Total** |                        | **20 min** |

---

## Dicas rápidas

- Grave a **Parte 4** com a stack já estável; teste os fluxos uma vez antes.
- Se a compra demorar, enquanto espera explique os eventos — não fique em silêncio.
- Se passar de 20 min, encurte a Parte 2 (mostrar só um repo + compose) ou a Parte 5 (só `kubectl get pods`, sem explicar cada YAML).
- Mantenha credenciais de demo visíveis só se for ambiente local.

---

Arquivo relacionado: [`ENTREGA_CHECKLIST_FASE2.md`](../ENTREGA_CHECKLIST_FASE2.md)
