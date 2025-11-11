# EDC Connector Architecture

## Overview

The Eclipse Dataspace Components (EDC) Connector is a framework for building sovereign, interoperable dataspaces using the Dataspace Protocol (DSP) and Decentralized Claims Protocol (DCP). This document provides a comprehensive overview of the connector's architecture, components, and design principles.

## High-Level Architecture

The EDC Connector follows a modular, extensible architecture organized into several key layers:

```
┌─────────────────────────────────────────────────────────┐
│                    Extensions Layer                      │
│  (Technology-specific implementations: SQL, HTTP, etc.)  │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                   Core Layer                             │
│  (Essential connector functionality and services)        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                    SPI Layer                             │
│  (Service Provider Interfaces - extension points)        │
└─────────────────────────────────────────────────────────┘
```

## Core Architectural Principles

### 1. **Modularity**
The connector is designed as a collection of independent, composable modules that can be combined to create custom connector configurations.

### 2. **Extensibility**
The SPI (Service Provider Interface) layer defines clear extension points, allowing users to customize behavior without modifying core code.

### 3. **Technology Neutrality**
Core functionality is independent of specific technologies. Technology-specific implementations reside in the extensions layer.

### 4. **Separation of Concerns**
The architecture separates different responsibilities:
- **Control Plane**: Manages contract negotiations, policies, and orchestration
- **Data Plane**: Handles actual data transfer
- **Data Plane Selector**: Routes data transfer requests to appropriate data plane instances

## Major Components

### Control Plane

The Control Plane is responsible for:
- **Contract Negotiation**: Negotiating data access contracts between providers and consumers
- **Transfer Process Management**: Orchestrating data transfer workflows
- **Catalog Management**: Publishing and querying available data offerings
- **Policy Evaluation**: Enforcing access policies and usage constraints
- **Asset Management**: Managing metadata about available data assets

Key modules:
- `control-plane-core`: Core control plane services
- `control-plane-contract`: Contract negotiation logic
- `control-plane-transfer`: Transfer process management
- `control-plane-catalog`: Catalog service implementation

### Data Plane

The Data Plane is responsible for:
- **Data Transfer**: Moving data from source to destination
- **Protocol Support**: Supporting various transfer protocols (HTTP, Kafka, S3, etc.)
- **Authentication**: Handling authentication for data access
- **Streaming**: Managing data streaming operations

Key modules:
- `data-plane-core`: Core data plane services
- `data-plane-util`: Utility functions for data plane operations
- Extensions for specific protocols (HTTP, Kafka, etc.)

### Data Plane Selector

The Data Plane Selector:
- Routes transfer requests to appropriate data plane instances
- Manages data plane instance registration and discovery
- Supports distributed data plane architectures

### SPI (Service Provider Interface) Layer

The SPI layer defines interfaces for:
- **Storage**: Asset stores, contract stores, transfer process stores
- **Security**: Vaults, key management, authentication
- **Policy**: Policy evaluation engines, constraint functions
- **Transfer**: Data source/sink implementations
- **Protocols**: Communication protocol implementations

Key SPI categories:
- `common/spi`: Common interfaces used across the connector
- `control-plane/spi`: Control plane-specific interfaces
- `data-plane/spi`: Data plane-specific interfaces

### Extensions Layer

Extensions provide concrete implementations of SPI interfaces for specific technologies:
- **Storage Extensions**: SQL-based stores (PostgreSQL, etc.)
- **Vault Extensions**: HashiCorp Vault, Azure Key Vault
- **Transfer Extensions**: HTTP, Kafka, cloud storage (S3, Azure Blob)
- **API Extensions**: Management API, observability APIs

## Data Flow

### Contract Negotiation Flow

1. **Consumer** sends catalog request to **Provider's Control Plane**
2. **Provider** returns available offerings with policies
3. **Consumer** initiates contract negotiation
4. **Control Planes** exchange offers/counter-offers until agreement
5. Contract agreement is stored by both parties

### Data Transfer Flow

1. **Consumer** initiates transfer request with contract agreement reference
2. **Provider's Control Plane** validates contract and policies
3. **Provider's Control Plane** requests **Data Plane Selector** to select data plane
4. **Data Plane** is provisioned and configured
5. **Data Plane** transfers data from source to destination
6. Transfer completion is reported back to **Control Plane**

## Protocol Support

### Dataspace Protocol (DSP)

The connector implements the Dataspace Protocol for interoperable dataspace communication:
- Catalog Protocol: Discovering data offerings
- Contract Negotiation Protocol: Negotiating access contracts
- Transfer Process Protocol: Initiating and managing transfers

Implementation location: `data-protocols/dsp/`

### Decentralized Claims Protocol (DCP)

The connector supports the Decentralized Claims Protocol for decentralized identity and authorization:
- Verifiable Credentials
- Decentralized Identifiers (DIDs)
- Trust framework integration

## Runtime and Bootstrapping

### Runtime Initialization

The connector uses a dependency injection framework to:
1. Load extensions from the classpath
2. Resolve dependencies between services
3. Initialize services in correct order
4. Start the runtime

Key modules:
- `boot`: Bootstrap mechanisms
- `runtime-core`: Core runtime functionality

### Configuration

Configuration is managed through:
- Environment variables
- Configuration files
- Extension-specific configuration

## Deployment Patterns

### Single Instance

```
┌─────────────────────────┐
│   EDC Connector         │
│  ┌─────────────────┐    │
│  │  Control Plane  │    │
│  └─────────────────┘    │
│  ┌─────────────────┐    │
│  │   Data Plane    │    │
│  └─────────────────┘    │
└─────────────────────────┘
```

All components run in a single process.

### Distributed Type 2

```
┌─────────────────┐         ┌─────────────────┐
│ Control Plane   │ ←────→  │   Data Plane    │
│    Instance     │         │    Instance     │
└─────────────────┘         └─────────────────┘
```

Control Plane and Data Plane run as separate processes, potentially on different infrastructure.

### Cluster

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Control Plane   │    │ Control Plane   │    │ Control Plane   │
│   Instance 1    │    │   Instance 2    │    │   Instance 3    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         ↓                      ↓                      ↓
    ┌────────────────────────────────────────────────────┐
    │            Shared State Store (SQL)                │
    └────────────────────────────────────────────────────┘
```

Multiple instances share state for high availability and scalability.

For detailed deployment patterns, see [Management Domains](developer/management-domains/management-domains.md).

## Security Architecture

### Identity and Authentication

- Support for various identity systems (OAuth2, DIDs, X.509)
- Participant identity verification
- Token-based authentication

### Authorization

- Policy-based access control
- Contract-based authorization
- Fine-grained usage policies

### Secrets Management

- Secure storage of credentials and keys
- Integration with vault systems (HashiCorp Vault, cloud providers)
- Key rotation support

## Extension Development

### Creating a Custom Extension

1. **Define SPI Interface** (if needed)
2. **Implement the Interface**
3. **Create Extension Class** with `@Extension` annotation
4. **Register Services** in the service context
5. **Package and Deploy**

Example extension structure:
```java
@Extension(value = "My Custom Extension")
public class MyCustomExtension implements ServiceExtension {
    
    @Inject
    private ServiceDependency dependency;
    
    @Override
    public void initialize(ServiceExtensionContext context) {
        // Initialize your extension
        context.registerService(MyService.class, new MyServiceImpl());
    }
}
```

## Testing Strategy

### Unit Testing
- Test individual components in isolation
- Mock dependencies using test utilities

### Integration Testing
- Test component interactions
- Use in-memory implementations for storage

### System Testing
- End-to-end testing of complete flows
- Multi-connector scenarios

Test utilities location: `core/common/junit/`

## Observability

### Logging
- SLF4J-based logging throughout the connector
- Configurable log levels

### Metrics
- Micrometer-based metrics
- Extensible metric providers

### Tracing
- OpenTelemetry support
- Distributed tracing capabilities

## Further Reading

- [Developer Decision Records](developer/decision-records/README.md) - Architectural decisions and rationale
- [Management Domains](developer/management-domains/management-domains.md) - Deployment topologies
- [Developer Guide](DEVELOPER_GUIDE.md) - Getting started with development
- [Contributing Guidelines](https://github.com/eclipse-edc/eclipse-edc.github.io/blob/main/CONTRIBUTING.md)
