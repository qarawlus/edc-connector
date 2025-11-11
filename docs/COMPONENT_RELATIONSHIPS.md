# Component Relationships and Interactions

## Overview

This document describes how the major components of the EDC Connector interact with each other, the dependencies between them, and the communication patterns used throughout the system.

## Component Hierarchy

### Dependency Layers

The connector follows a strict dependency hierarchy to maintain modularity and extensibility:

```
┌────────────────────────────────────────┐
│           Extensions                   │  ← Technology-specific implementations
├────────────────────────────────────────┤
│        Data Protocols (DSP)            │  ← Protocol implementations
├────────────────────────────────────────┤
│         Core Implementations           │  ← Business logic
├────────────────────────────────────────┤
│      SPI (Interfaces)                  │  ← Contracts/Extension points
├────────────────────────────────────────┤
│        Common Utilities                │  ← Shared utilities
└────────────────────────────────────────┘
```

**Key Principles:**
- **Lower layers cannot depend on higher layers**
- **SPI defines contracts, Core provides implementations**
- **Extensions provide technology-specific implementations of SPI interfaces**

## Core Component Interactions

### Control Plane Components

```
┌─────────────────────┐
│  Management API     │ ← REST API for management operations
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐     ┌──────────────────────┐
│ Contract Negotiation│────→│  Catalog Service     │
│    Manager          │     └──────────────────────┘
└──────────┬──────────┘              ↓
           │                  ┌──────────────────────┐
           │                  │   Asset Index        │
           │                  └──────────────────────┘
           ↓                         
┌─────────────────────┐     ┌──────────────────────┐
│ Transfer Process    │────→│  Policy Engine       │
│    Manager          │     └──────────────────────┘
└──────────┬──────────┘              
           │                  
           ↓                  
┌─────────────────────┐     ┌──────────────────────┐
│ Data Plane Selector │────→│  Data Plane Manager  │
└─────────────────────┘     └──────────────────────┘
```

### Data Flow Through Components

#### Catalog Request Flow

1. **Consumer** → Management API → `CatalogService`
2. `CatalogService` → DSP Protocol Layer → **Provider**
3. **Provider** DSP Endpoint → `CatalogRequestHandler`
4. `CatalogRequestHandler` → `AssetIndex` (query assets)
5. `CatalogRequestHandler` → `ContractDefinitionStore` (get policies)
6. `CatalogRequestHandler` → Response with offerings
7. Response → DSP Protocol Layer → **Consumer**

#### Contract Negotiation Flow

1. **Consumer** → Management API → `ContractNegotiationManager`
2. `ContractNegotiationManager` → State Machine (INITIAL → REQUESTING)
3. State Machine → DSP Protocol → **Provider** (send offer)
4. **Provider** DSP → `ContractNegotiationManager`
5. `ContractNegotiationManager` → `PolicyEngine.evaluate()` (validate policy)
6. If valid → State Machine (REQUESTING → REQUESTED → OFFERED → ACCEPTED)
7. `ContractNegotiationManager` → `ContractStore` (store agreement)
8. DSP Protocol → **Consumer** (send agreement)
9. **Consumer** → Store agreement → State Machine (AGREED)

#### Transfer Process Flow

1. **Consumer** → Management API → `TransferProcessManager`
2. `TransferProcessManager` → State Machine (INITIAL → PROVISIONING)
3. State Machine → `ProvisionManager` (provision resources if needed)
4. State Machine → PROVISIONED → REQUESTING
5. `TransferProcessManager` → DSP Protocol → **Provider** (transfer request)
6. **Provider** DSP → `TransferProcessManager`
7. `TransferProcessManager` → `ContractStore` (validate contract)
8. `TransferProcessManager` → `PolicyEngine` (validate policies)
9. `TransferProcessManager` → `DataPlaneSelector` (select data plane)
10. `DataPlaneSelector` → `DataPlaneManager` (initiate transfer)
11. `DataPlaneManager` → `DataSource` (read data)
12. `DataPlaneManager` → `DataSink` (write data)
13. Transfer completion → Callback to `TransferProcessManager`
14. State Machine → COMPLETED

## Inter-Component Communication Patterns

### 1. Dependency Injection

Components declare dependencies using `@Inject`:

```java
@Extension(value = "Transfer Manager Extension")
public class TransferProcessManagerExtension implements ServiceExtension {
    
    @Inject
    private PolicyEngine policyEngine;
    
    @Inject
    private ContractStore contractStore;
    
    @Inject
    private DataPlaneSelector dataPlaneSelector;
    
    // Use injected dependencies
}
```

**Resolution Order:**
1. Runtime scans classpath for `ServiceExtension` implementations
2. Builds dependency graph
3. Resolves dependencies in topological order
4. Initializes services

### 2. Service Registry

Components register and retrieve services:

```java
// Register a service
context.registerService(TransferProcessManager.class, transferProcessManager);

// Retrieve a service
var monitor = context.getMonitor();
var config = context.getConfig();
```

### 3. Event Bus

Components communicate asynchronously via events:

```java
// Publish event
eventRouter.publish(TransferProcessStarted.Builder.newInstance()
    .transferProcessId(transferProcess.getId())
    .build());

// Subscribe to events
eventRouter.register(TransferProcessEvent.class, event -> {
    // Handle event
});
```

**Common Events:**
- `ContractNegotiationEvent` (negotiation state changes)
- `TransferProcessEvent` (transfer state changes)
- `AssetEvent` (asset lifecycle events)

### 4. State Machines

Many components use state machines for workflow orchestration:

```java
// State machine processor
@Override
public boolean process(TransferProcess transferProcess) {
    return switch (transferProcess.getState()) {
        case INITIAL -> transitionToProvisioning(transferProcess);
        case PROVISIONING -> processProvisioning(transferProcess);
        case PROVISIONED -> transitionToRequesting(transferProcess);
        // ... more states
    };
}
```

**Key State Machines:**
- `ContractNegotiationManager` - Contract negotiation workflow
- `TransferProcessManager` - Transfer process workflow
- `PolicyMonitorManager` - Policy monitoring workflow

### 5. Command Pattern

Operations are encapsulated as commands:

```java
// Create command
var command = new TransferCommand(transferProcessId, TransferCommand.Type.START);

// Process command
commandProcessor.process(command);
```

## SPI → Core → Extension Flow

### Example: Asset Storage

**1. SPI Definition** (`spi/control-plane/asset-spi/`)

```java
public interface AssetIndex {
    Asset findById(String assetId);
    Collection<Asset> queryAssets(QuerySpec querySpec);
    void create(Asset asset);
    Asset deleteById(String assetId);
}
```

**2. Core Implementation** (`core/control-plane/control-plane-core/`)

```java
public class InMemoryAssetIndex implements AssetIndex {
    private final Map<String, Asset> assets = new ConcurrentHashMap<>();
    
    @Override
    public Asset findById(String assetId) {
        return assets.get(assetId);
    }
    
    // ... other methods
}
```

**3. Extension Implementation** (`extensions/control-plane/store/sql/asset-index-sql/`)

```java
public class SqlAssetIndex implements AssetIndex {
    private final DataSource dataSource;
    
    @Override
    public Asset findById(String assetId) {
        // SQL query to database
        return queryDatabase("SELECT * FROM assets WHERE id = ?", assetId);
    }
    
    // ... other methods
}
```

**4. Extension Registration**

```java
@Extension(value = "SQL Asset Index")
public class SqlAssetIndexExtension implements ServiceExtension {
    
    @Inject
    private DataSource dataSource;
    
    @Override
    public void initialize(ServiceExtensionContext context) {
        var assetIndex = new SqlAssetIndex(dataSource);
        context.registerService(AssetIndex.class, assetIndex);
    }
}
```

**Resolution:** The runtime uses the last registered implementation, so SQL implementation overrides in-memory.

## Data Plane Architecture

### Data Plane Components

```
┌──────────────────┐
│  Control Plane   │
└────────┬─────────┘
         │ 1. Request data plane
         ↓
┌──────────────────┐
│ Data Plane       │
│   Selector       │
└────────┬─────────┘
         │ 2. Select instance
         ↓
┌──────────────────┐
│  Data Plane      │
│   Manager        │
└────────┬─────────┘
         │ 3. Initiate transfer
         ↓
┌──────────────────┐     ┌──────────────────┐
│   Data Source    │────→│    Data Sink     │
│  (HTTP/S3/...)   │     │  (HTTP/S3/...)   │
└──────────────────┘     └──────────────────┘
```

### Data Plane Selection Process

1. **Transfer Request** arrives at Control Plane
2. Control Plane → `DataPlaneSelector.select(transferRequest)`
3. `DataPlaneSelector` queries registered data planes:
   - Check capabilities (can handle source/destination types)
   - Check availability/health
   - Apply selection strategy (round-robin, least-loaded, etc.)
4. Returns `DataPlaneInstance`
5. Control Plane → `DataPlaneManager.initiate(transferRequest, dataPlaneInstance)`

### Data Transfer Execution

1. `DataPlaneManager` receives transfer request
2. Creates `DataFlowRequest` from transfer details
3. Validates access token
4. `DataFlowController.initiateFlow(dataFlowRequest)`
5. `DataFlowController` → `DataSource.openPartStream()` (open source)
6. `DataFlowController` → `DataSink.transfer(stream)` (stream to sink)
7. Monitor transfer progress
8. On completion/error → Send callback to Control Plane

## Protocol Layer Integration

### DSP Protocol Components

```
┌─────────────────────────────────┐
│      REST Controller            │ ← Jetty/Jersey HTTP layer
└───────────┬─────────────────────┘
            │
            ↓
┌─────────────────────────────────┐
│   DSP Request Handlers          │ ← Protocol message handlers
│  - CatalogRequestHandler        │
│  - NegotiationRequestHandler    │
│  - TransferRequestHandler       │
└───────────┬─────────────────────┘
            │
            ↓
┌─────────────────────────────────┐
│   Message Transformers          │ ← JSON-LD transformation
│  - JSON → Domain Object         │
│  - Domain Object → JSON         │
└───────────┬─────────────────────┘
            │
            ↓
┌─────────────────────────────────┐
│   Core Services                 │ ← Business logic
│  - CatalogService               │
│  - ContractNegotiationManager   │
│  - TransferProcessManager       │
└─────────────────────────────────┘
```

### Message Flow

**Incoming DSP Message:**
1. HTTP request → Jetty server
2. Jersey REST controller → Route to handler
3. Handler → Extract JSON-LD payload
4. Transformer → Convert JSON-LD to domain object
5. Domain object → Core service method
6. Core service → Process and generate response
7. Response domain object → Transformer
8. JSON-LD response → HTTP response

**Outgoing DSP Message:**
1. Core service → Create domain object
2. Transformer → Convert to JSON-LD
3. HTTP client → Send to remote endpoint
4. Receive response → Transform back to domain object
5. Domain object → Core service

## Extension Points and Customization

### Common Extension Points

| SPI Interface | Purpose | Common Implementations |
|---------------|---------|----------------------|
| `AssetIndex` | Asset storage | In-memory, SQL, NoSQL |
| `ContractNegotiationStore` | Contract storage | In-memory, SQL |
| `TransferProcessStore` | Transfer process storage | In-memory, SQL |
| `Vault` | Secret storage | HashiCorp Vault, Azure Key Vault |
| `IdentityService` | Identity/authentication | OAuth2, DID, X.509 |
| `PolicyEngine` | Policy evaluation | Default rule-based engine |
| `DataSource` | Data reading | HTTP, S3, Azure Blob, Kafka |
| `DataSink` | Data writing | HTTP, S3, Azure Blob, Kafka |

### Extension Loading Order

1. **Boot Extensions** - System initialization
2. **Core Extensions** - Essential services
3. **Common Extensions** - Shared functionality
4. **Protocol Extensions** - Communication protocols
5. **Storage Extensions** - Persistence implementations
6. **API Extensions** - REST APIs
7. **Domain Extensions** - Business logic extensions

## Thread and Concurrency Model

### Thread Pools

1. **State Machine Threads** - Process state machine iterations
   - Configurable batch size and wait time
   - Processes entities in batches

2. **HTTP Server Threads** - Handle incoming HTTP requests
   - Jetty thread pool (configurable)

3. **Data Transfer Threads** - Execute data transfers
   - Transfer-specific thread pool

### Synchronization

- **State Machines** - Lease-based concurrency control
- **Stores** - Optimistic locking with version numbers
- **Event Bus** - Asynchronous, non-blocking

## Transaction Boundaries

### Transaction Management

```
┌──────────────────────────────────┐
│    Transaction Boundary          │
│                                  │
│  ┌────────────────────────────┐ │
│  │  Database Operations       │ │
│  │  - Read entity             │ │
│  │  - Update entity           │ │
│  │  - Write entity            │ │
│  └────────────────────────────┘ │
│                                  │
│  Commit or Rollback              │
└──────────────────────────────────┘
```

**Transaction Contexts:**
- Each state machine iteration runs in a transaction
- API operations that modify state use transactions
- Data plane transfers may use separate transactions

## Observability Integration

### Monitoring Points

```
Application Code
       ↓
┌──────────────────┐
│   Monitor API    │ ← Logging abstraction
└────────┬─────────┘
         │
    ┌────┴─────────────┬──────────────┐
    ↓                  ↓              ↓
┌─────────┐    ┌───────────┐   ┌──────────┐
│ Logging │    │  Metrics  │   │ Tracing  │
│ (SLF4J) │    │(Micrometer)│   │ (OpenTel)│
└─────────┘    └───────────┘   └──────────┘
```

Components use `Monitor` interface for:
- Structured logging
- Metric collection
- Distributed tracing

## Security Flow

### Authentication and Authorization

```
Request → API Gateway → Auth Filter
                           ↓
                    Validate Token
                           ↓
                    Extract Identity
                           ↓
                    Create ParticipantAgent
                           ↓
                    Policy Evaluation
                           ↓
                    Grant/Deny Access
```

**Security Components:**
- `IdentityService` - Validates participant identity
- `ParticipantAgentService` - Creates authenticated agent
- `PolicyEngine` - Evaluates access policies
- `Vault` - Stores and retrieves secrets

## Further Reading

- [Architecture Overview](ARCHITECTURE.md) - High-level architecture
- [Developer Guide](DEVELOPER_GUIDE.md) - Development instructions
- [Decision Records](developer/decision-records/) - Design decisions
- [SPI Javadoc](https://eclipse-edc.github.io/docs) - API documentation
