# Getting Started with EDC Connector

## Overview

This guide will help you get started with the Eclipse Dataspace Components (EDC) Connector framework. You'll learn how the framework is structured, how to build connector implementations, and understand its capabilities.

## What is EDC Connector?

**EDC Connector is a framework**, not a standalone application. It provides the building blocks for creating sovereign, interoperable dataspace connectors. 

Organizations use this framework to build custom connector implementations that:

- **Share data securely** with fine-grained access control
- **Negotiate contracts** automatically based on policies
- **Transfer data** between different systems and clouds
- **Maintain sovereignty** over their data assets
- **Participate in dataspaces** using standard protocols (DSP, DCP)

### Framework vs. Implementation

- **This repository (eclipse-edc/Connector)**: The **framework** - provides core components, SPIs, and extensions
- **Production implementations**: Organizations build their own connectors using this framework
  - Example: [Tractus-X EDC Connector](https://github.com/eclipse-tractusx/tractusx-edc) - A complete implementation for the Catena-X/Tractus-X ecosystem

## Quick Start

### Prerequisites

- Java 17 or higher ([Download](https://adoptium.net/))
- Gradle (included via wrapper)
- Basic understanding of REST APIs

### Option 1: Explore the Framework

1. **Clone the framework repository**:
   ```bash
   git clone https://github.com/eclipse-edc/Connector.git
   cd Connector
   ```

2. **Build the framework**:
   ```bash
   ./gradlew build
   ```

3. **Explore the structure**:
   ```bash
   # View the SPI (extension points)
   ls -la spi/
   
   # View core implementations
   ls -la core/
   
   # View available extensions
   ls -la extensions/
   ```

### Option 2: Use a Production Implementation

For a ready-to-run connector, use an existing implementation:

**Tractus-X EDC Connector** (recommended for getting started):

```bash
# Clone the Tractus-X EDC implementation
git clone https://github.com/eclipse-tractusx/tractusx-edc.git
cd tractusx-edc

# Follow their quickstart guide
# See: https://github.com/eclipse-tractusx/tractusx-edc
```

### Option 3: Build Your Own Connector

Create a custom connector implementation using this framework:

1. **Create a new Gradle project**
2. **Add EDC dependencies** (see [Building Custom Connectors](#building-custom-connectors))
3. **Select extensions** you need
4. **Create a runtime launcher**
5. **Configure and run**

See the [Developer Guide](DEVELOPER_GUIDE.md#creating-extensions) for detailed instructions.

## Understanding the Framework

### Key Concepts

#### Assets
Digital resources you want to share (files, APIs, database tables, etc.)

#### Policies
Rules that define who can access assets and under what conditions

#### Contract Definitions
Templates that link assets with policies

#### Contract Agreements
Negotiated agreements between data providers and consumers

#### Transfer Processes
Workflows that orchestrate actual data transfers

### Framework Components

The EDC framework exposes several key components:

- **Management API**: For managing assets, policies, contracts, and transfers
- **DSP API**: For dataspace protocol communication between connectors  
- **Control API**: For control plane operations
- **Public API**: For accessing transferred data

The actual ports and endpoints are configured in your connector implementation.

## Building Custom Connectors

### Basic Connector Structure

A minimal connector implementation needs:

1. **Gradle build configuration** with EDC framework dependencies
2. **Extension selection** - choose which modules to include
3. **Runtime class** - bootstraps the connector
4. **Configuration** - environment-specific settings

### Example Build Configuration

Create a `build.gradle.kts`:

```kotlin
plugins {
    `java-library`
    id("application")
}

dependencies {
    // Core dependencies
    implementation("org.eclipse.edc:boot:VERSION")
    implementation("org.eclipse.edc:connector-core:VERSION")
    
    // Control Plane
    implementation("org.eclipse.edc:control-plane-core:VERSION")
    implementation("org.eclipse.edc:management-api:VERSION")
    
    // Data Plane
    implementation("org.eclipse.edc:data-plane-core:VERSION")
    implementation("org.eclipse.edc:data-plane-http:VERSION")
    
    // Protocol
    implementation("org.eclipse.edc:dsp:VERSION")
    
    // Storage (choose one)
    implementation("org.eclipse.edc:asset-index-sql:VERSION")
    // or use in-memory for testing
}

application {
    mainClass.set("org.eclipse.edc.boot.system.runtime.BaseRuntime")
}
```

### Creating a Runtime

The simplest runtime uses the provided `BaseRuntime`:

```java
package com.example.connector;

import org.eclipse.edc.boot.system.runtime.BaseRuntime;

public class MyConnectorRuntime {
    public static void main(String[] args) {
        BaseRuntime.main(args);
    }
}
```

For more control, extend `BaseRuntime` or implement your own runtime initialization.

### Selecting Extensions

Extensions provide specific capabilities. Common extensions include:

**Storage:**
- `asset-index-sql` - SQL-based asset storage
- `contract-negotiation-store-sql` - SQL contract storage
- `transfer-process-store-sql` - SQL transfer storage

**Security:**
- `vault-hashicorp` - HashiCorp Vault integration
- `vault-azure` - Azure Key Vault integration
- `oauth2-core` - OAuth2 authentication

**Data Transfer:**
- `data-plane-http` - HTTP data transfer
- `data-plane-kafka` - Kafka data transfer  
- `data-plane-azure` - Azure Storage integration
- `data-plane-aws` - AWS S3 integration

**APIs:**
- `management-api` - REST management API
- `observability-api` - Health and metrics endpoints

See the [extensions/](../extensions/) directory for all available extensions.

## Working with Sample Implementations

The [EDC Samples repository](https://github.com/eclipse-edc/Samples) provides complete working examples:

```bash
git clone https://github.com/eclipse-edc/Samples.git
cd Samples

# Explore basic samples
cd basic/basic-01-basic-connector
./gradlew build

# Run the sample connector
java -jar build/libs/basic-connector.jar
```

## Using Production Implementations

### Tractus-X EDC Connector

A complete, production-ready implementation:

```bash
git clone https://github.com/eclipse-tractusx/tractusx-edc.git
cd tractusx-edc

# Follow the Tractus-X specific documentation
# for building and running
```

Features:
- Pre-configured for Catena-X/Tractus-X ecosystem
- Production-grade security
- Complete data plane implementations
- Comprehensive documentation

Visit: [Tractus-X EDC Documentation](https://github.com/eclipse-tractusx/tractusx-edc)

## Basic Operations

Once you have a running connector (either from your implementation or from samples), you can interact with it via REST APIs.

### 1. Create an Asset

An asset represents data you want to share. Example using a connector's Management API:

```bash
curl -X POST http://localhost:8181/management/v3/assets \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: your-api-key" \
  -d '{
    "@context": {
      "@vocab": "https://w3id.org/edc/v0.0.1/ns/"
    },
    "@id": "asset-1",
    "properties": {
      "name": "My First Asset",
      "description": "A sample data asset",
      "contenttype": "application/json"
    },
    "dataAddress": {
      "type": "HttpData",
      "baseUrl": "https://api.example.com/data"
    }
  }'
```

**Note:** The actual endpoint URL and authentication depend on your connector implementation.

### 2. Create a Policy

Policies define access rules for assets.

```bash
curl -X POST http://localhost:8181/management/v3/policydefinitions \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: your-api-key" \
  -d '{
    "@context": {
      "@vocab": "https://w3id.org/edc/v0.0.1/ns/"
    },
    "@id": "policy-1",
    "policy": {
      "permissions": [{
        "action": "use",
        "constraints": []
      }]
    }
  }'
```

### 3. Create a Contract Definition

Contract definitions link assets with policies.

```bash
curl -X POST http://localhost:8181/management/v3/contractdefinitions \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: your-api-key" \
  -d '{
    "@context": {
      "@vocab": "https://w3id.org/edc/v0.0.1/ns/"
    },
    "@id": "contract-def-1",
    "accessPolicyId": "policy-1",
    "contractPolicyId": "policy-1",
    "assetsSelector": {
      "operandLeft": "https://w3id.org/edc/v0.0.1/ns/id",
      "operator": "=",
      "operandRight": "asset-1"
    }
  }'
```

### 4. Query the Catalog

As a consumer, query a provider's catalog:

```bash
curl -X POST http://localhost:8181/management/v3/catalog/request \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: your-api-key" \
  -d '{
    "@context": {
      "@vocab": "https://w3id.org/edc/v0.0.1/ns/"
    },
    "counterPartyAddress": "http://provider-connector:8282/api/v1/dsp",
    "protocol": "dataspace-protocol-http"
  }'
```

### 5. Initiate Contract Negotiation

Start negotiating a contract for an asset:

```bash
curl -X POST http://localhost:8181/management/v3/contractnegotiations \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: your-api-key" \
  -d '{
    "@context": {
      "@vocab": "https://w3id.org/edc/v0.0.1/ns/"
    },
    "counterPartyAddress": "http://provider-connector:8282/api/v1/dsp",
    "protocol": "dataspace-protocol-http",
    "policy": {
      "@type": "Offer",
      "offerId": "offer-from-catalog",
      "assetsSelector": {
        "operandLeft": "https://w3id.org/edc/v0.0.1/ns/id",
        "operator": "=",
        "operandRight": "asset-1"
      }
    }
  }'
```

### 6. Initiate Transfer

Once you have a contract agreement, initiate a transfer:

```bash
curl -X POST http://localhost:8181/management/v3/transferprocesses \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: your-api-key" \
  -d '{
    "@context": {
      "@vocab": "https://w3id.org/edc/v0.0.1/ns/"
    },
    "counterPartyAddress": "http://provider-connector:8282/api/v1/dsp",
    "contractId": "contract-agreement-id",
    "assetId": "asset-1",
    "protocol": "dataspace-protocol-http",
    "transferType": "HttpData-PULL",
    "dataDestination": {
      "type": "HttpProxy"
    }
  }'
```

## Configuration

Connector implementations typically support configuration through properties files or environment variables. The exact configuration options depend on which extensions are included.

### Common Configuration Patterns

Most connectors built with EDC framework support:

```properties
# Connector identity
edc.participant.id=my-connector

# API configuration
web.http.port=8181
web.http.path=/api
edc.api.auth.key=your-api-key

# DSP endpoint  
web.http.protocol.port=8282
web.http.protocol.path=/api/v1/dsp

# Data plane endpoint
edc.dataplane.token.validation.endpoint=http://localhost:8183/control/token
```

**Note:** Actual configuration keys may vary by implementation. Consult your connector implementation's documentation.

### Environment Variables

Configuration can also use environment variables:

```bash
export EDC_PARTICIPANT_ID=my-connector
export WEB_HTTP_PORT=8181
export WEB_HTTP_PATH=/api
export EDC_API_AUTH_KEY=your-api-key
export WEB_HTTP_PROTOCOL_PORT=8282
export WEB_HTTP_PROTOCOL_PATH=/api/v1/dsp
```

## Testing with Multiple Connectors

To test dataspace interactions, you need at least two connector instances: a provider and a consumer.

### Provider Setup (Example)

Using a sample or implementation:

```bash
export EDC_PARTICIPANT_ID=provider
export WEB_HTTP_PORT=8181
export WEB_HTTP_PROTOCOL_PORT=8282
# Start your connector implementation
```

Then create assets, policies, and contract definitions (see Basic Operations above).

### Consumer Setup (Example)

Using a separate instance:

```bash
export EDC_PARTICIPANT_ID=consumer  
export WEB_HTTP_PORT=9181
export WEB_HTTP_PROTOCOL_PORT=9282
# Start your connector implementation
```

Then query catalog, negotiate contract, initiate transfer (see Basic Operations above).

**Important:** The exact startup commands depend on your connector implementation (sample, Tractus-X, or custom).

## Common Use Cases

### Use Case 1: File Sharing

Share files from a cloud storage bucket:

```json
{
  "@id": "file-asset",
  "dataAddress": {
    "type": "AzureStorage",
    "container": "my-container",
    "account": "mystorageaccount",
    "blobName": "myfile.csv"
  }
}
```

### Use Case 2: API Access

Share access to a REST API:

```json
{
  "@id": "api-asset",
  "dataAddress": {
    "type": "HttpData",
    "baseUrl": "https://api.example.com/v1/data",
    "authKey": "stored-in-vault",
    "authCode": "api-key-reference"
  }
}
```

### Use Case 3: Database Query

Share specific database queries:

```json
{
  "@id": "database-asset",
  "dataAddress": {
    "type": "Sql",
    "jdbcUrl": "jdbc:postgresql://db.example.com:5432/mydb",
    "query": "SELECT * FROM public.data WHERE id = ?"
  }
}
```

## Security Considerations

### API Authentication

Always protect Management APIs in production:

```properties
edc.api.auth.key=<strong-random-key>
```

Use the key in requests:
```bash
curl -H "X-Api-Key: <strong-random-key>" ...
```

### Secrets Management

Never hardcode credentials. Use vault extensions:

```properties
# HashiCorp Vault
edc.vault.hashicorp.url=http://vault:8200
edc.vault.hashicorp.token=s.xxxxxx

# Azure Key Vault  
edc.vault.azure.name=my-keyvault
```

### Identity and Trust

Configure participant identity (implementation-specific):

```properties
# DID-based identity
edc.iam.did.web.use.https=true
edc.iam.issuer.id=did:web:example.com:participant

# OAuth2 identity
edc.oauth.token.url=https://auth.example.com/token
edc.oauth.client.id=my-client-id
```

## Monitoring and Troubleshooting

### Health Checks

Most implementations expose health endpoints:

```bash
# Overall health
curl http://localhost:8181/api/check/health

# Liveness probe
curl http://localhost:8181/api/check/liveness

# Readiness probe  
curl http://localhost:8181/api/check/readiness
```

**Note:** Actual endpoint paths depend on your implementation's configuration.

### Logging

Enable detailed logging (implementation-specific):

```properties
edc.logging.level=DEBUG
```

### Common Issues

**Issue: Framework classes not found**
- Ensure all required dependencies are included in your build
- Check classpath and verify framework version compatibility

**Issue: Extensions not loading**
- Verify extension is on classpath
- Check META-INF/services configuration
- Review logs for dependency injection errors

**Issue: "Connection refused"**
- Verify connector is running and configured ports are correct
- Check firewall and network settings

For more troubleshooting help, see [DEBUGGING.md](DEBUGGING.md).

## Next Steps

### For Framework Users

- **Build Extensions**: Learn to [create custom extensions](DEVELOPER_GUIDE.md#creating-extensions)
- **Understand Architecture**: Read the [Architecture Guide](ARCHITECTURE.md)
- **Explore Samples**: Try the [EDC Samples](https://github.com/eclipse-edc/Samples)
- **Study Implementations**: Review [Tractus-X EDC](https://github.com/eclipse-tractusx/tractusx-edc)

### For Connector Operators

- **Production Deployment**: Use a production-ready implementation like Tractus-X EDC
- **Advanced Configuration**: Consult your implementation's specific documentation
- **Integration**: Connect with your existing systems and data sources
- **Policy Development**: Create sophisticated access policies for your use cases

### Learning Resources

- **Architecture**: Understand the [framework architecture](ARCHITECTURE.md)
- **Development**: Read the [Developer Guide](DEVELOPER_GUIDE.md)
- **Components**: Learn about [component relationships](COMPONENT_RELATIONSHIPS.md)
- **Specifications**: Review the [Dataspace Protocol](https://docs.internationaldataspaces.org/ids-knowledgebase/dataspace-protocol)

### Community

- **Discord**: [Join the community](https://discord.gg/n4sD9qtjMQ)
- **GitHub**: [Contribute to the framework](https://github.com/eclipse-edc/Connector)
- **Documentation**: [Full documentation](https://eclipse-edc.github.io)

## Resources

- [EDC Framework Website](https://eclipse-edc.github.io)
- [GitHub Repository](https://github.com/eclipse-edc/Connector)
- [Sample Projects](https://github.com/eclipse-edc/Samples)
- [Tractus-X EDC Implementation](https://github.com/eclipse-tractusx/tractusx-edc)
- [Decision Records](developer/decision-records/)
- [Contributing Guidelines](https://github.com/eclipse-edc/eclipse-edc.github.io/blob/main/CONTRIBUTING.md)

---

**Remember**: This is a **framework** for building connectors, not a standalone application. For production use, either build your own connector implementation or use an existing one like Tractus-X EDC Connector.
