# EDC Connector Documentation Index

## Welcome to EDC Connector Documentation

This index provides a comprehensive guide to all available documentation for the Eclipse Dataspace Components (EDC) Connector **framework**.

**Important**: EDC Connector is a framework for building dataspace connectors, not a standalone application. This documentation covers:
- The framework itself (for contributors and extension developers)
- How to build connector implementations using the framework
- Production implementations like [Tractus-X EDC Connector](https://github.com/eclipse-tractusx/tractusx-edc)

## Getting Started

### For New Users

If you're new to the EDC framework, start here:

1. **[Getting Started Guide](GETTING_STARTED.md)** - Learn about the framework and how to build connectors
   - Framework vs. implementation distinction
   - Using sample implementations
   - Building custom connectors
   - Basic operations and configuration

2. **[Architecture Overview](ARCHITECTURE.md)** - Understand what EDC framework provides
   - High-level architecture
   - Framework components (Control Plane, Data Plane, etc.)
   - Design principles
   - Deployment patterns

### For Developers

If you want to contribute to the framework or build extensions:

1. **[Developer Guide](DEVELOPER_GUIDE.md)** - Complete development setup and workflow
   - Prerequisites and setup
   - Framework structure walkthrough
   - Building and testing
   - Creating extensions
   - Code style and conventions

2. **[Debugging Guide](DEBUGGING.md)** - Troubleshooting and debugging strategies
   - Debugging the framework
   - Debugging connector implementations
   - Common issues and solutions
   - Logging and diagnostics

3. **[Component Relationships](COMPONENT_RELATIONSHIPS.md)** - Deep dive into framework architecture
   - Component hierarchy and dependencies
   - Data flow through the system
   - Communication patterns
   - Extension points

## Architecture and Design

### Core Documentation

- **[Architecture Overview](ARCHITECTURE.md)** - System architecture and design principles
  - Architectural layers (SPI, Core, Extensions)
  - Component descriptions
  - Data flow diagrams
  - Security architecture

- **[Component Relationships](COMPONENT_RELATIONSHIPS.md)** - How components interact
  - Control Plane flow
  - Data Plane flow
  - Protocol integration
  - State machines

- **[Management Domains](developer/management-domains/management-domains.md)** - Deployment topologies
  - Single instance deployment
  - Distributed deployments
  - Cluster configuration
  - High availability patterns

### Decision Records

All architectural decisions are documented in [Decision Records](developer/decision-records/):

Key decision records include:
- Transfer process state transitions
- Token handling and security
- Policy engine design
- Event framework
- Protocol implementations

Browse all: [Decision Records Index](developer/decision-records/README.md)

## Development Guides

### Setup and Configuration

- **[Developer Guide - Prerequisites](DEVELOPER_GUIDE.md#prerequisites)** - Required tools and software
- **[Developer Guide - Getting Started](DEVELOPER_GUIDE.md#getting-started)** - Clone, build, and run
- **[Getting Started - Configuration](GETTING_STARTED.md#configuration)** - Runtime configuration

### Development Workflow

- **[Developer Guide - Development Workflow](DEVELOPER_GUIDE.md#development-workflow)** - Git workflow and practices
- **[Developer Guide - Building](DEVELOPER_GUIDE.md#building-the-project)** - Build commands and options
- **[Developer Guide - Testing](DEVELOPER_GUIDE.md#running-tests)** - Test execution and coverage

### Creating Extensions

- **[Developer Guide - Creating Extensions](DEVELOPER_GUIDE.md#creating-extensions)** - Extension development guide
- **[Component Relationships - Extension Points](COMPONENT_RELATIONSHIPS.md#extension-points-and-customization)** - Available extension points
- **[Architecture - Extension Development](ARCHITECTURE.md#extension-development)** - Extension architecture

### Code Quality

- **[Developer Guide - Code Style](DEVELOPER_GUIDE.md#code-style-and-conventions)** - Coding standards
- **[Developer Guide - Common Tasks](DEVELOPER_GUIDE.md#common-development-tasks)** - Frequent operations

## Troubleshooting and Support

### Debugging

- **[Debugging Guide](DEBUGGING.md)** - Complete debugging reference
  - IDE debugging setup (IntelliJ, VS Code, Eclipse)
  - Common issues and solutions
  - Logging configuration
  - Performance profiling
  - Network debugging

### Common Issues

Quick links to solutions:
- [Extension Not Loading](DEBUGGING.md#issue-extension-not-loading)
- [Transfer Process Stuck](DEBUGGING.md#issue-transfer-process-stuck)
- [Contract Negotiation Failing](DEBUGGING.md#issue-contract-negotiation-failing)
- [Data Plane Transfer Failing](DEBUGGING.md#issue-data-plane-transfer-failing)
- [Configuration Not Applied](DEBUGGING.md#issue-configuration-not-applied)

### Getting Help

- **Discord**: [Join the EDC community](https://discord.gg/n4sD9qtjMQ)
- **GitHub Issues**: [Report bugs or request features](https://github.com/eclipse-edc/Connector/issues)
- **GitHub Discussions**: [Ask questions](https://github.com/eclipse-edc/Connector/discussions)
- **Documentation Website**: [eclipse-edc.github.io](https://eclipse-edc.github.io)

## API and Operations

### Using the Connector

- **[Getting Started - Basic Operations](GETTING_STARTED.md#basic-operations)** - REST API examples
  - Creating assets
  - Defining policies
  - Contract definitions
  - Catalog queries
  - Contract negotiations
  - Transfer processes

### API Reference

- **Management API** - Control plane management operations
- **DSP API** - Dataspace Protocol endpoints
- **Data Plane API** - Data transfer operations

Full API documentation: [EDC API Docs](https://eclipse-edc.github.io/docs)

## Project Structure

### Directory Overview

From the [README](../README.md#directory-structure):

```
edc-connector/
├── spi/              # Service Provider Interfaces (extension points)
├── core/             # Core implementations
├── extensions/       # Technology-specific extensions
├── data-protocols/   # Protocol implementations (DSP)
├── launchers/        # Runnable connector configurations
├── docs/            # Documentation (you are here)
├── system-tests/    # End-to-end tests
└── tests/           # Test utilities
```

### Key Directories

- **[SPI](ARCHITECTURE.md#spi-service-provider-interface-layer)** - Extension interfaces
- **[Core](ARCHITECTURE.md#core-implementations)** - Essential functionality
- **[Extensions](ARCHITECTURE.md#extensions-layer)** - Technology implementations
- **[Data Protocols](ARCHITECTURE.md#protocol-support)** - DSP implementation
- **[Launchers](../launchers/README.md)** - Runnable configurations

## Deployment

### Running the Connector

- **[Getting Started - Quick Start](GETTING_STARTED.md#quick-start)** - Run your first connector
- **[Getting Started - Configuration](GETTING_STARTED.md#configuration)** - Configure the connector
- **[Getting Started - Two-Connector Scenario](GETTING_STARTED.md#two-connector-scenario)** - Test setup

### Deployment Patterns

- **[Architecture - Deployment Patterns](ARCHITECTURE.md#deployment-patterns)** - Architecture patterns
- **[Management Domains](developer/management-domains/management-domains.md)** - Detailed deployment topologies

### Security

- **[Getting Started - Security](GETTING_STARTED.md#security-considerations)** - Security basics
- **[Architecture - Security](ARCHITECTURE.md#security-architecture)** - Security architecture

## Contributing

### How to Contribute

- **[Contributing Guidelines](https://github.com/eclipse-edc/eclipse-edc.github.io/blob/main/CONTRIBUTING.md)** - Contribution process
- **[Developer Guide](DEVELOPER_GUIDE.md)** - Development setup
- **[Code Style](DEVELOPER_GUIDE.md#code-style-and-conventions)** - Coding standards

### Community

- **Discord**: [EDC Chat](https://discord.gg/n4sD9qtjMQ)
- **GitHub**: [Connector Repository](https://github.com/eclipse-edc/Connector)
- **Samples**: [Example Projects](https://github.com/eclipse-edc/Samples)

## Advanced Topics

### Protocol Implementation

- **[Component Relationships - Protocol Layer](COMPONENT_RELATIONSHIPS.md#protocol-layer-integration)** - DSP integration
- **[Debugging - Protocol Debugging](DEBUGGING.md#network-and-protocol-debugging)** - Protocol troubleshooting

### Performance

- **[Debugging - Performance](DEBUGGING.md#performance-debugging)** - Performance analysis
- **[Component Relationships - Thread Model](COMPONENT_RELATIONSHIPS.md#thread-and-concurrency-model)** - Concurrency

### Observability

- **[Architecture - Observability](ARCHITECTURE.md#observability)** - Monitoring capabilities
- **[Debugging - Logging](DEBUGGING.md#logging-and-diagnostics)** - Logging configuration
- **[Component Relationships - Observability](COMPONENT_RELATIONSHIPS.md#observability-integration)** - Monitoring integration

## Reference Materials

### Specifications

- [Dataspace Protocol (DSP)](https://docs.internationaldataspaces.org/ids-knowledgebase/dataspace-protocol)
- [Decentralized Claims Protocol (DCP)](https://github.com/eclipse-tractusx/tutorial-resources/tree/main/mxd)
- [JSON-LD](https://json-ld.org/)
- [OAuth 2.0](https://oauth.net/2/)

### External Documentation

- [EDC Official Website](https://eclipse-edc.github.io)
- [EDC Samples Repository](https://github.com/eclipse-edc/Samples)
- [Eclipse Foundation](https://www.eclipse.org/)

## Documentation Maintenance

This documentation is maintained by the EDC community. To suggest improvements:

1. Open an issue: [GitHub Issues](https://github.com/eclipse-edc/Connector/issues)
2. Submit a pull request with documentation updates
3. Discuss on [Discord](https://discord.gg/n4sD9qtjMQ)

### Documentation Standards

- Keep documentation up-to-date with code changes
- Use clear, concise language
- Include code examples where appropriate
- Link related documents
- Update this index when adding new documentation

## Quick Reference

### Common Commands

```bash
# Build the project
./gradlew build

# Run tests
./gradlew test

# Build a launcher
./gradlew :launchers:generic:build

# Run the connector
java -jar launcher.jar

# Check code style
./gradlew checkstyleMain checkstyleTest
```

### Important Files

- `README.md` - Project overview and directory structure
- `SECURITY.md` - Security policy and vulnerability reporting
- `LICENSE` - Apache 2.0 license
- `NOTICE.md` - Legal notices and attributions

### Key Concepts

- **Asset** - Data to be shared
- **Policy** - Access control rules
- **Contract** - Agreement for data access
- **Transfer Process** - Data transfer workflow
- **Control Plane** - Manages negotiations and orchestration
- **Data Plane** - Executes data transfers
- **Extension** - Customization module
- **SPI** - Service Provider Interface (extension point)

---

**Last Updated**: 2025-11-11

For the most up-to-date documentation, visit the [EDC Documentation Website](https://eclipse-edc.github.io).
