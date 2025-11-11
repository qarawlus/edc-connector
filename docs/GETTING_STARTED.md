# Getting Started with EDC Connector

## Overview

This guide will help you get started with the Eclipse Dataspace Components (EDC) Connector. You'll learn how to run a basic connector instance, understand its capabilities, and perform common operations.

## What is EDC Connector?

The EDC Connector is a framework for building sovereign, interoperable dataspaces. It enables organizations to:

- **Share data securely** with fine-grained access control
- **Negotiate contracts** automatically based on policies
- **Transfer data** between different systems and clouds
- **Maintain sovereignty** over their data assets
- **Participate in dataspaces** using standard protocols (DSP, DCP)

## Quick Start

### Prerequisites

- Java 17 or higher ([Download](https://adoptium.net/))
- Basic understanding of REST APIs
- (Optional) Docker for containerized deployment

### Option 1: Using Pre-built Distribution

1. **Download the latest release**:
   ```bash
   # Replace VERSION with the latest version
   wget https://github.com/eclipse-edc/Connector/releases/download/vVERSION/connector-VERSION.zip
   unzip connector-VERSION.zip
   cd connector-VERSION
   ```

2. **Start the connector**:
   ```bash
   java -jar lib/connector.jar
   ```

3. **Verify it's running**:
   ```bash
   curl http://localhost:8181/api/check/health
   ```

### Option 2: Building from Source

1. **Clone the repository**:
   ```bash
   git clone https://github.com/eclipse-edc/Connector.git
   cd Connector
   ```

2. **Build the connector**:
   ```bash
   ./gradlew :launchers:generic:build
   ```

3. **Run the connector**:
   ```bash
   cd launchers/generic/build/distributions
   unzip generic.zip
   cd generic
   java -jar lib/generic.jar
   ```

### Option 3: Using Docker

```bash
# Pull the image
docker pull edc/connector:latest

# Run the connector
docker run -p 8181:8181 -p 8282:8282 edc/connector:latest
```

## Understanding the Connector

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

### Connector Endpoints

A running connector typically exposes several API endpoints:

- **Management API** (port 8181): For managing assets, policies, contracts, and transfers
- **DSP API** (port 8282): For dataspace protocol communication with other connectors
- **Control API** (port 8183): For control plane operations
- **Public API** (varies): For accessing transferred data

## Basic Operations

### 1. Create an Asset

An asset represents data you want to share.

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

### Basic Configuration

Create a `configuration.properties` file:

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

Start with configuration:
```bash
java -Dedc.fs.config=configuration.properties -jar connector.jar
```

### Environment Variables

Alternatively, use environment variables:

```bash
export EDC_PARTICIPANT_ID=my-connector
export WEB_HTTP_PORT=8181
export WEB_HTTP_PATH=/api
export EDC_API_AUTH_KEY=your-api-key
export WEB_HTTP_PROTOCOL_PORT=8282
export WEB_HTTP_PROTOCOL_PATH=/api/v1/dsp

java -jar connector.jar
```

### Common Configuration Options

| Property | Description | Default |
|----------|-------------|---------|
| `edc.participant.id` | Unique identifier for the connector | Required |
| `web.http.port` | Management API port | 8181 |
| `web.http.protocol.port` | DSP protocol port | 8282 |
| `edc.api.auth.key` | API authentication key | none |
| `edc.dataplane.selector.url` | Data plane selector URL | none |
| `edc.vault.hashicorp.url` | HashiCorp Vault URL | none |

## Two-Connector Scenario

To test the connector, you typically need at least two instances: a provider and a consumer.

### Provider Setup

1. **Start provider connector**:
   ```bash
   export EDC_PARTICIPANT_ID=provider
   export WEB_HTTP_PORT=8181
   export WEB_HTTP_PROTOCOL_PORT=8282
   java -jar connector.jar
   ```

2. **Create an asset, policy, and contract definition** (see Basic Operations above)

### Consumer Setup

1. **Start consumer connector** (different ports):
   ```bash
   export EDC_PARTICIPANT_ID=consumer
   export WEB_HTTP_PORT=9181
   export WEB_HTTP_PROTOCOL_PORT=9282
   java -jar connector.jar
   ```

2. **Query catalog, negotiate contract, initiate transfer** (see Basic Operations above)

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

Always protect your Management API:

```properties
edc.api.auth.key=<strong-random-key>
```

Use the key in requests:
```bash
curl -H "X-Api-Key: <strong-random-key>" ...
```

### Secrets Management

Never hardcode credentials. Use a vault:

```properties
# HashiCorp Vault
edc.vault.hashicorp.url=http://vault:8200
edc.vault.hashicorp.token=s.xxxxxx

# Azure Key Vault
edc.vault.azure.name=my-keyvault
```

### Identity and Trust

Configure participant identity:

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

```bash
# Overall health
curl http://localhost:8181/api/check/health

# Liveness probe
curl http://localhost:8181/api/check/liveness

# Readiness probe
curl http://localhost:8181/api/check/readiness
```

### Logging

Enable detailed logging:

```properties
edc.logging.level=DEBUG
```

View logs:
```bash
# If running in foreground, logs appear in console
# If using systemd:
journalctl -u edc-connector -f
```

### Common Issues

**Issue: "Connection refused"**
- Check connector is running: `ps aux | grep java`
- Verify ports are open: `netstat -an | grep 8181`

**Issue: "Authentication failed"**
- Verify API key: `echo $EDC_API_AUTH_KEY`
- Check X-Api-Key header in requests

**Issue: "Transfer not starting"**
- Check contract agreement exists
- Verify data plane is configured
- Review connector logs for errors

For more troubleshooting help, see [DEBUGGING.md](DEBUGGING.md).

## Next Steps

### Learn More

- **Architecture**: Understand the [system architecture](ARCHITECTURE.md)
- **Development**: Read the [Developer Guide](DEVELOPER_GUIDE.md)
- **Examples**: Explore [sample implementations](https://github.com/eclipse-edc/Samples)
- **Specifications**: Review the [Dataspace Protocol](https://docs.internationaldataspaces.org/ids-knowledgebase/dataspace-protocol)

### Advanced Topics

- **Distributed Deployment**: Deploy separate control and data planes
- **Custom Extensions**: Build connectors with custom functionality
- **Policy Development**: Create complex access policies
- **Integration**: Integrate with your existing systems

### Community

- **Discord**: [Join the community](https://discord.gg/n4sD9qtjMQ)
- **GitHub**: [Contribute to the project](https://github.com/eclipse-edc/Connector)
- **Documentation**: [Full documentation](https://eclipse-edc.github.io)

## Resources

- [EDC Website](https://eclipse-edc.github.io)
- [GitHub Repository](https://github.com/eclipse-edc/Connector)
- [Sample Projects](https://github.com/eclipse-edc/Samples)
- [Decision Records](developer/decision-records/)
- [Contributing Guidelines](https://github.com/eclipse-edc/eclipse-edc.github.io/blob/main/CONTRIBUTING.md)

Happy data sharing! 🚀
