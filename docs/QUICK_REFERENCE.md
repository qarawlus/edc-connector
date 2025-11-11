# EDC Framework Quick Reference

## What is EDC?

**EDC (Eclipse Dataspace Components) is a FRAMEWORK** for building dataspace connectors. It's NOT a standalone application.

- **Use the framework**: Build custom connectors for your needs
- **Use implementations**: Deploy [Tractus-X EDC](https://github.com/eclipse-tractusx/tractusx-edc) or similar
- **Study samples**: Learn from [EDC Samples](https://github.com/eclipse-edc/Samples)

## Framework Structure

```
spi/              → Interfaces (what you can extend)
core/             → Implementations (how it works)
extensions/       → Tech-specific add-ons (storage, protocols, etc.)
data-protocols/   → Protocol implementations (DSP)
```

## Key Concepts

| Concept | Description |
|---------|-------------|
| **SPI** | Service Provider Interface - extension points you can implement |
| **Extension** | Plugin module that adds functionality |
| **Control Plane** | Manages contracts, negotiations, orchestration |
| **Data Plane** | Executes actual data transfers |
| **Asset** | Data resource you want to share |
| **Policy** | Access control rules |
| **Contract** | Agreement for data access |
| **Transfer Process** | Workflow that moves data |

## Common Commands

```bash
# Build the framework
./gradlew build

# Run tests
./gradlew test

# Build specific module
./gradlew :core:control-plane:control-plane-core:build

# Run tests for specific module
./gradlew :spi:common:core-spi:test

# Check code style
./gradlew checkstyleMain checkstyleTest
```

## Creating an Extension

1. **Define dependency** in `build.gradle.kts`:
```kotlin
dependencies {
    api(project(":spi:common:core-spi"))
    implementation(project(":core:common:util"))
}
```

2. **Create extension class**:
```java
@Extension(value = "My Extension")
public class MyExtension implements ServiceExtension {
    
    @Inject
    private SomeDependency dependency;
    
    @Override
    public void initialize(ServiceExtensionContext context) {
        var service = new MyServiceImpl(dependency);
        context.registerService(MyService.class, service);
    }
}
```

3. **Register in META-INF/services**:
```
# File: src/main/resources/META-INF/services/org.eclipse.edc.spi.system.ServiceExtension
com.example.MyExtension
```

## Building a Connector

### Minimal Connector Build File

```kotlin
plugins {
    `java-library`
    id("application")
}

val edcVersion = "0.x.x"

dependencies {
    // Core
    implementation("org.eclipse.edc:boot:$edcVersion")
    implementation("org.eclipse.edc:connector-core:$edcVersion")
    
    // Control Plane
    implementation("org.eclipse.edc:control-plane-core:$edcVersion")
    implementation("org.eclipse.edc:management-api:$edcVersion")
    
    // Data Plane
    implementation("org.eclipse.edc:data-plane-core:$edcVersion")
    implementation("org.eclipse.edc:data-plane-http:$edcVersion")
    
    // Protocol
    implementation("org.eclipse.edc:dsp:$edcVersion")
    
    // Storage (choose based on needs)
    implementation("org.eclipse.edc:asset-index-sql:$edcVersion")
}

application {
    mainClass.set("org.eclipse.edc.boot.system.runtime.BaseRuntime")
}
```

## Configuration Basics

```properties
# Identity
edc.participant.id=my-connector

# Management API
web.http.port=8181
web.http.path=/api
edc.api.auth.key=your-secret-key

# DSP Protocol
web.http.protocol.port=8282
web.http.protocol.path=/api/v1/dsp

# Data Plane
edc.dataplane.token.validation.endpoint=http://localhost:8183/control/token

# Logging
edc.logging.level=INFO
```

## Key Extension Points (SPI)

| Interface | Purpose | Location |
|-----------|---------|----------|
| `AssetIndex` | Asset storage | `spi/control-plane/asset-spi` |
| `ContractNegotiationStore` | Contract storage | `spi/control-plane/contract-spi` |
| `TransferProcessStore` | Transfer storage | `spi/control-plane/transfer-spi` |
| `Vault` | Secrets management | `spi/common/core-spi` |
| `IdentityService` | Authentication | `spi/common/core-spi` |
| `DataSource` | Data reading | `spi/data-plane/data-plane-spi` |
| `DataSink` | Data writing | `spi/data-plane/data-plane-spi` |

## State Machine States

### Transfer Process
```
INITIAL → PROVISIONING → PROVISIONED → REQUESTING → REQUESTED →
STARTING → STARTED → COMPLETING → COMPLETED
```

### Contract Negotiation
```
REQUESTING → REQUESTED → OFFERING → OFFERED → 
ACCEPTING → ACCEPTED → AGREEING → AGREED
```

## Debugging Quick Tips

```bash
# Enable debug logging
export EDC_LOGGING_LEVEL=DEBUG

# Enable specific package logging
export EDC_LOGGING_ORG_ECLIPSE_EDC_CONNECTOR_TRANSFER=TRACE

# Start with remote debugging
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 \
     -jar your-connector.jar

# Attach debugger on port 5005
```

### Common Breakpoint Locations
- `ServiceExtension.initialize()` - Extension loading
- `TransferProcessManagerImpl.processTransfers()` - Transfer state changes
- `ContractNegotiationManagerImpl.processNegotiations()` - Contract state changes
- `PolicyEngineImpl.evaluate()` - Policy evaluation

## API Examples

### Create Asset
```bash
curl -X POST http://localhost:8181/management/v3/assets \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: your-key" \
  -d '{
    "@context": {"@vocab": "https://w3id.org/edc/v0.0.1/ns/"},
    "@id": "asset-1",
    "properties": {"name": "My Asset"},
    "dataAddress": {"type": "HttpData", "baseUrl": "https://api.example.com"}
  }'
```

### Query Catalog
```bash
curl -X POST http://localhost:8181/management/v3/catalog/request \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: your-key" \
  -d '{
    "@context": {"@vocab": "https://w3id.org/edc/v0.0.1/ns/"},
    "counterPartyAddress": "http://provider:8282/api/v1/dsp",
    "protocol": "dataspace-protocol-http"
  }'
```

## Getting Help

- **Documentation**: [Full Docs Index](README.md)
- **Discord**: https://discord.gg/n4sD9qtjMQ
- **GitHub Issues**: https://github.com/eclipse-edc/Connector/issues
- **Samples**: https://github.com/eclipse-edc/Samples

## Documentation Quick Links

| Document | Purpose |
|----------|---------|
| [Getting Started](GETTING_STARTED.md) | Learn the framework basics |
| [Architecture](ARCHITECTURE.md) | Understand the design |
| [Developer Guide](DEVELOPER_GUIDE.md) | Set up development |
| [Debugging](DEBUGGING.md) | Troubleshoot issues |
| [Component Relationships](COMPONENT_RELATIONSHIPS.md) | Deep dive into internals |

## Production Implementations

- **[Tractus-X EDC](https://github.com/eclipse-tractusx/tractusx-edc)** - For Catena-X/Tractus-X
- Build your own using this framework
- See samples for reference implementations

---

**Remember**: EDC is a **framework**. You build connectors with it, you don't run it directly.
