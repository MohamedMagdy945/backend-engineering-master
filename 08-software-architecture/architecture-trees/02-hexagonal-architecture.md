# Hexagonal Architecture (Ports & Adapters) — Project Tree & Structure

Hexagonal Architecture isolates the **Application Core** behind symmetrical **Ports**, connected to the outside world through interchangeable **Adapters**.

---

## 01 — Architectural Concept & Symmetrical Boundaries

```text
       DRIVING (PRIMARY) SIDE                           DRIVEN (SECONDARY) SIDE
 [External Triggers Drive Core]                     [Core Drives External Systems]

┌───────────────────────────────┐                  ┌───────────────────────────────┐
│     Inbound Adapters          │                  │     Outbound Adapters         │
│  ├── REST API Controllers     │                  │  ├── PostgreSQL Database      │
│  ├── CLI Command Runners      │                  │  ├── Stripe Payment Gateway   │
│  ├── Kafka / RabbitMQ Queues  │                  │  ├── SendGrid Email System    │
│  └── gRPC Service Handlers    │                  │  └── Redis Cache Store        │
└───────────────┬───────────────┘                  └───────────────▲───────────────┘
                │                                                  │
                ▼                                                  │
       ┌─────────────────┐                                ┌─────────────────┐
       │  Driving Ports  │                                │  Driven Ports   │
       │    (Inbound)    │                                │   (Outbound)    │
       └────────┬────────┘                                └────────▲────────┘
                │                                                  │
                └────────────────►┌───────────────┐◄───────────────┘
                                  │  APPLICATION  │
                                  │     CORE      │
                                  │ (Domain Model │
                                  │  & Use Cases) │
                                  └───────────────┘
```

---

## 02 — Comprehensive Project Tree

```text
BankingSystem/
│
├── src/
│   ├── BankingSystem.Core/                             # The Hexagon (Zero external dependencies)
│   │   ├── Domain/
│   │   │   ├── Entities/
│   │   │   │   ├── Account.cs
│   │   │   │   ├── Transaction.cs
│   │   │   │   ├── Customer.cs
│   │   │   │   └── Card.cs
│   │   │   ├── ValueObjects/
│   │   │   │   ├── Money.cs
│   │   │   │   ├── Iban.cs
│   │   │   │   ├── Currency.cs
│   │   │   │   └── CardNumber.cs
│   │   │   ├── Enums/
│   │   │   │   ├── AccountStatus.cs
│   │   │   │   └── TransactionType.cs
│   │   │   ├── Exceptions/
│   │   │   │   ├── InsufficientBalanceException.cs
│   │   │   │   └── AccountLockedException.cs
│   │   │   └── Events/
│   │   │       ├── FundsTransferredEvent.cs
│   │   │       └── CardBlockedEvent.cs
│   │   │
│   │   ├── Ports/
│   │   │   ├── Inbound/                                # Driving Ports (What outside can ask the Core to do)
│   │   │   │   ├── ITransferFundsUseCase.cs
│   │   │   │   ├── IDepositFundsUseCase.cs
│   │   │   │   ├── IWithdrawFundsUseCase.cs
│   │   │   │   ├── IBlockCardUseCase.cs
│   │   │   │   └── IGetAccountBalanceQuery.cs
│   │   │   │
│   │   │   └── Outbound/                               # Driven Ports (What the Core needs from the outside)
│   │   │       ├── IAccountPersistencePort.cs
│   │   │       ├── ITransactionPersistencePort.cs
│   │   │       ├── INotificationPort.cs
│   │   │       ├── IFraudDetectionPort.cs
│   │   │       └── IAuditLogPort.cs
│   │   │
│   │   ├── DTOs/
│   │   │   ├── Inbound/
│   │   │   │   ├── TransferFundsCommandDto.cs
│   │   │   │   └── DepositFundsCommandDto.cs
│   │   │   └── Outbound/
│   │   │       ├── AccountBalanceResponseDto.cs
│   │   │       └── TransferReceiptDto.cs
│   │   │
│   │   └── UseCases/                                   # Implementations of Inbound Driving Ports
│   │       ├── TransferFundsUseCase.cs
│   │       ├── DepositFundsUseCase.cs
│   │       ├── WithdrawFundsUseCase.cs
│   │       └── BlockCardUseCase.cs
│   │
│   ├── BankingSystem.Adapters.Inbound/                 # Driving Adapters (Receive inputs & call Inbound Ports)
│   │   ├── RestApi/
│   │   │   ├── Controllers/
│   │   │   │   ├── AccountsController.cs
│   │   │   │   └── CardsController.cs
│   │   │   ├── Requests/
│   │   │   │   └── TransferFundsHttpRequest.cs
│   │   │   └── Filters/
│   │   │       └── ApiExceptionFilter.cs
│   │   │
│   │   ├── Cli/
│   │   │   ├── Commands/
│   │   │   │   └── DailyInterestCalculationCliCommand.cs
│   │   │   └── Program.cs
│   │   │
│   │   ├── MessageConsumers/
│   │   │   ├── RabbitMq/
│   │   │   │   └── IncomingWireTransferConsumer.cs
│   │   │   └── Kafka/
│   │   │       └── CardAuthorizationConsumer.cs
│   │   │
│   │   └── Grpc/
│   │       ├── Protos/
│   │       │   └── accounts.proto
│   │       └── Services/
│   │           └── AccountGrpcService.cs
│   │
│   ├── BankingSystem.Adapters.Outbound/                # Driven Adapters (Implement Outbound Driven Ports)
│   │   ├── Persistence.Postgres/
│   │   │   ├── Configurations/
│   │   │   │   ├── AccountEntityConfiguration.cs
│   │   │   │   └── TransactionEntityConfiguration.cs
│   │   │   ├── Repositories/
│   │   │   │   ├── PostgresAccountPersistenceAdapter.cs     # Implements IAccountPersistencePort
│   │   │   │   └── PostgresTransactionPersistenceAdapter.cs # Implements ITransactionPersistencePort
│   │   │   ├── BankingDbContext.cs
│   │   │   └── Migrations/
│   │   │
│   │   ├── Notifications.Smtp/
│   │   │   └── SmtpEmailNotificationAdapter.cs              # Implements INotificationPort
│   │   │
│   │   ├── FraudDetection.ExternalService/
│   │   │   ├── FraudServiceClient.cs
│   │   │   └── FraudDetectionServiceAdapter.cs              # Implements IFraudDetectionPort
│   │   │
│   │   └── Audit.Elasticsearch/
│   │       └── ElasticsearchAuditLogAdapter.cs              # Implements IAuditLogPort
│   │
│   └── BankingSystem.Host/                             # Composition Root (Wires ports to adapters)
│       ├── DependencyInjection/
│       │   ├── CoreServiceRegistration.cs
│       │   ├── InboundAdapterRegistration.cs
│       │   └── OutboundAdapterRegistration.cs
│       ├── appsettings.json
│       └── Program.cs
│
└── tests/
    ├── BankingSystem.Core.UnitTests/                   # 100% in-memory business tests (no database, no HTTP)
    │   ├── Domain/
    │   │   └── AccountTests.cs
    │   └── UseCases/
    │       └── TransferFundsUseCaseTests.cs
    │
    ├── BankingSystem.Adapters.Inbound.Tests/
    │   └── RestApi/
    │       └── AccountsControllerTests.cs
    │
    └── BankingSystem.Adapters.Outbound.Tests/
        └── Persistence/
            └── PostgresAccountPersistenceAdapterTests.cs
```

---

## 03 — Folder Responsibilities & Boundary Matrix

| Directory / Layer | Role | Responsibility | Allowed References |
|---|---|---|---|
| `Core/Domain/` | Core Model | Pure entities, value objects, and domain rules. | None |
| `Core/Ports/Inbound/` | Driving Contract | Interfaces declaring use cases callable by outside actors. | `Core/Domain`, `Core/DTOs` |
| `Core/Ports/Outbound/` | Driven Contract | Interfaces declaring external capabilities needed by core. | `Core/Domain` |
| `Core/UseCases/` | Core Workflows | Implements inbound ports; orchestrates domain & outbound ports. | `Core/Domain`, `Core/Ports` |
| `Adapters.Inbound/RestApi/` | Driving Adapter | Translates HTTP JSON payloads into Inbound Port executions. | `Core/Ports/Inbound` |
| `Adapters.Inbound/Cli/` | Driving Adapter | Translates console commands into Inbound Port executions. | `Core/Ports/Inbound` |
| `Adapters.Inbound/Consumers/`| Driving Adapter | Translates queue messages into Inbound Port executions. | `Core/Ports/Inbound` |
| `Adapters.Outbound/Persistence/` | Driven Adapter | Implements `IAccountPersistencePort` using PostgreSQL / ORM. | `Core/Ports/Outbound`, `Core/Domain` |
| `Adapters.Outbound/Gateways/` | Driven Adapter | Implements driven ports using external third-party SDKs. | `Core/Ports/Outbound`, `Core/Domain` |
| `Host/` | Bootstrap | Assembles all adapters to ports at application startup. | All projects |

---

## 04 — Core Rules of Hexagonal Architecture

1. **The Core Knows Nothing About Adapters**: `BankingSystem.Core` has zero package references to EF Core, ASP.NET, RabbitMQ, or Stripe.
2. **Ports Belong Inside The Core**: Both Inbound Ports and Outbound Ports are declared in the Core project.
3. **Inbound Adapters Call Inbound Ports**: The API controller does not talk to the database adapter directly; it talks to a use case.
4. **Outbound Adapters Implement Outbound Ports**: The persistence adapter implements the port defined by the core.
5. **Adapters Are Swappable**: Changing PostgreSQL to MongoDB only requires replacing the Outbound Persistence Adapter without touching `Core`.
