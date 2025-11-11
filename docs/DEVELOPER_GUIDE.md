# EDC Connector Developer Guide

## Welcome

This guide will help you get started with developing the EDC Connector **framework**. Whether you're contributing to the framework itself or building custom extensions and implementations, this document provides the essential information you need.

**Important**: The EDC Connector is a framework, not a standalone application. This guide covers:
- Contributing to the framework itself
- Building custom extensions
- Creating connector implementations using the framework

For building a complete, runnable connector, see the [Getting Started Guide](GETTING_STARTED.md).

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Getting Started](#getting-started)
3. [Project Structure](#project-structure)
4. [Development Workflow](#development-workflow)
5. [Building the Project](#building-the-project)
6. [Running Tests](#running-tests)
7. [Debugging](#debugging)
8. [Creating Extensions](#creating-extensions)
9. [Code Style and Conventions](#code-style-and-conventions)
10. [Common Development Tasks](#common-development-tasks)

## Prerequisites

Before you begin, ensure you have the following installed:

- **Java Development Kit (JDK) 17 or higher**
  - Recommended: Eclipse Temurin (formerly AdoptOpenJDK)
  - Verify: `java -version`

- **Gradle** (optional, wrapper included)
  - The project includes Gradle wrapper (`./gradlew`)
  - Verify: `./gradlew --version`

- **Git**
  - For version control and contributing
  - Verify: `git --version`

- **IDE** (recommended)
  - IntelliJ IDEA (Community or Ultimate Edition)
  - Eclipse IDE
  - VS Code with Java extensions

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/eclipse-edc/Connector.git
cd Connector
```

### 2. Build the Project

```bash
./gradlew build
```

This will:
- Compile all modules
- Run unit tests
- Generate documentation
- Create distribution artifacts

### 3. Run Tests

```bash
./gradlew test
```

### 4. Import into IDE

#### IntelliJ IDEA
1. Open IntelliJ IDEA
2. Select "File" → "Open"
3. Navigate to the cloned repository
4. Select the `build.gradle.kts` file
5. Click "Open as Project"
6. Wait for Gradle sync to complete

#### Eclipse
1. Open Eclipse
2. Select "File" → "Import"
3. Choose "Gradle" → "Existing Gradle Project"
4. Navigate to the cloned repository
5. Click "Finish"

#### VS Code
1. Open VS Code
2. Install "Extension Pack for Java"
3. Open the cloned repository folder
4. VS Code will automatically detect the Gradle project

## Project Structure

The EDC Connector framework is organized into several top-level directories:

```
edc-connector/
├── spi/                    # Service Provider Interfaces (extension points)
│   ├── common/            # Common SPIs used across components
│   ├── control-plane/     # Control plane SPIs
│   ├── data-plane/        # Data plane SPIs
│   └── data-plane-selector/
│
├── core/                   # Core implementation modules
│   ├── common/            # Common core services
│   ├── control-plane/     # Control plane implementations
│   ├── data-plane/        # Data plane implementations
│   └── data-plane-selector/
│
├── extensions/            # Technology-specific extensions
│   ├── common/           # Common extensions (APIs, SQL, etc.)
│   ├── control-plane/    # Control plane extensions
│   ├── data-plane/       # Data plane extensions
│   └── data-plane-selector/
│
├── data-protocols/        # Protocol implementations
│   └── dsp/              # Dataspace Protocol implementation
│
├── launchers/             # Runnable connector configurations
│   ├── generic/          # Generic launcher
│   └── dpf-selector/     # Data plane selector launcher
│
├── docs/                  # Documentation
│   └── developer/        # Developer documentation
│
├── system-tests/          # End-to-end system tests
│
└── tests/                 # Test utilities and fixtures
```

### Key Directories Explained

#### `spi/` - Service Provider Interfaces
This is where all extension points are defined. If you want to customize the connector's behavior, you'll implement interfaces defined here.

**Example**: To create a custom asset store, implement `AssetIndex` from `spi/control-plane/asset-spi/`.

#### `core/` - Core Implementations
Contains the essential building blocks of the connector. This code runs in every connector instance and provides fundamental functionality.

**Example**: `TransferProcessManager` in `core/control-plane/control-plane-transfer-manager/` orchestrates the transfer process.

#### `extensions/` - Technology-Specific Extensions
Provides concrete implementations for various technologies and platforms.

**Example**: `extensions/control-plane/store/sql/asset-index-sql/` provides a SQL-based asset store.

#### `data-protocols/` - Protocol Implementations
Contains implementations of dataspace communication protocols.

**Example**: `data-protocols/dsp/` implements the Dataspace Protocol.

#### `launchers/` - Reference Launcher Configurations
Provides minimal reference configurations showing how to build runnable connectors.

**Note**: These are reference examples, not production-ready connectors. For production use:
- Build your own connector implementation
- Use existing implementations like [Tractus-X EDC](https://github.com/eclipse-tractusx/tractusx-edc)
- Study [EDC Samples](https://github.com/eclipse-edc/Samples) for complete working examples

## Development Workflow

### Typical Development Cycle

1. **Create a feature branch**
   ```bash
   git checkout -b feature/my-new-feature
   ```

2. **Make your changes**
   - Edit code
   - Add tests
   - Update documentation

3. **Build and test locally**
   ```bash
   ./gradlew build
   ```

4. **Run specific module tests**
   ```bash
   ./gradlew :core:common:connector-core:test
   ```

5. **Check code style**
   ```bash
   ./gradlew checkstyleMain checkstyleTest
   ```

6. **Commit your changes**
   ```bash
   git add .
   git commit -m "feat: add new feature"
   ```

7. **Push and create pull request**
   ```bash
   git push origin feature/my-new-feature
   ```

## Building the Project

### Full Build

```bash
./gradlew clean build
```

### Build Specific Module

```bash
./gradlew :core:control-plane:control-plane-core:build
```

### Skip Tests

```bash
./gradlew build -x test
```

### Build with Parallel Execution

```bash
./gradlew build --parallel
```

### Generate Distribution

Build a reference launcher:

```bash
./gradlew :launchers:generic:build
```

The distribution will be in `launchers/generic/build/distributions/`.

**Note**: The generic launcher is a minimal reference. For production connectors:
- Build your own implementation with required extensions
- Follow patterns from [EDC Samples](https://github.com/eclipse-edc/Samples)
- Use production implementations like [Tractus-X EDC](https://github.com/eclipse-tractusx/tractusx-edc)

## Running Tests

### Run All Tests

```bash
./gradlew test
```

### Run Tests for Specific Module

```bash
./gradlew :spi:common:core-spi:test
```

### Run Integration Tests

```bash
./gradlew :system-tests:tests:test
```

### Run Single Test Class

```bash
./gradlew test --tests "org.eclipse.edc.connector.core.CoreServicesExtensionTest"
```

### Run Tests with Logging

```bash
./gradlew test --info
```

### Generate Test Coverage Report

```bash
./gradlew jacocoTestReport
```

Coverage reports will be in `build/reports/jacoco/test/html/index.html` for each module.

## Debugging

### Debugging in IntelliJ IDEA

1. **Set Breakpoints**
   - Click in the gutter next to the line number
   - Or press `Ctrl+F8` (Windows/Linux) or `Cmd+F8` (Mac)

2. **Debug a Test**
   - Right-click on the test class or method
   - Select "Debug 'TestName'"

3. **Debug a Launcher**
   - Create a Run Configuration
   - Set Main class (e.g., from a launcher module)
   - Click "Debug" button

4. **Remote Debugging**
   - Start connector with debug agent:
     ```bash
     java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -jar connector.jar
     ```
   - In IntelliJ: Run → Edit Configurations → Add New → Remote JVM Debug
   - Set port to 5005
   - Click "Debug"

### Debugging Tests

Add this to see detailed test output:

```bash
./gradlew test --debug-jvm
```

Then attach a remote debugger to port 5005.

### Common Debugging Scenarios

#### Issue: Service not found at runtime
- **Cause**: Missing dependency injection or extension not loaded
- **Solution**: Check `@Inject` annotations and ensure extension is on classpath

#### Issue: Build fails with compilation error
- **Cause**: Missing dependency or API change
- **Solution**: Run `./gradlew clean build` and check dependency versions

#### Issue: Tests fail intermittently
- **Cause**: Race conditions or shared state
- **Solution**: Check for proper test isolation and use of test utilities

For more debugging tips, see [DEBUGGING.md](DEBUGGING.md).

## Creating Extensions

### Extension Basics

Extensions are the primary way to customize the connector. They implement the SPI interfaces and are loaded automatically at runtime.

### Creating a Simple Extension

1. **Create a new module** in the appropriate location (e.g., `extensions/my-extension/`)

2. **Add build.gradle.kts**:
   ```kotlin
   plugins {
       `java-library`
   }
   
   dependencies {
       api(project(":spi:common:core-spi"))
       implementation(project(":core:common:util"))
   }
   ```

3. **Create Extension class**:
   ```java
   package com.example.myextension;
   
   import org.eclipse.edc.runtime.metamodel.annotation.Extension;
   import org.eclipse.edc.runtime.metamodel.annotation.Inject;
   import org.eclipse.edc.spi.system.ServiceExtension;
   import org.eclipse.edc.spi.system.ServiceExtensionContext;
   
   @Extension(value = "My Custom Extension")
   public class MyCustomExtension implements ServiceExtension {
       
       @Inject
       private MyDependency dependency;
       
       @Override
       public String name() {
           return "My Custom Extension";
       }
       
       @Override
       public void initialize(ServiceExtensionContext context) {
           var service = new MyServiceImpl(dependency);
           context.registerService(MyService.class, service);
       }
   }
   ```

4. **Create service provider configuration**:
   Create `src/main/resources/META-INF/services/org.eclipse.edc.spi.system.ServiceExtension`:
   ```
   com.example.myextension.MyCustomExtension
   ```

5. **Add to build**:
   Include your module in `settings.gradle.kts`

### Extension Best Practices

- **Single Responsibility**: Each extension should have one clear purpose
- **Minimal Dependencies**: Only depend on what you need
- **Configuration**: Use settings for customization
- **Logging**: Use SLF4J for consistent logging
- **Testing**: Write unit and integration tests

## Code Style and Conventions

### Java Code Style

The project uses Checkstyle to enforce code style. Configuration: `resources/edc-checkstyle-config.xml`

Key conventions:
- **Indentation**: 4 spaces (no tabs)
- **Line Length**: Maximum 140 characters
- **Braces**: Always use braces for control structures
- **Naming**:
  - Classes: `PascalCase`
  - Methods: `camelCase`
  - Constants: `UPPER_SNAKE_CASE`
  - Packages: lowercase

### Documentation

- **Javadoc**: Required for public APIs
- **Inline Comments**: Use sparingly, prefer self-documenting code
- **Package Info**: Add `package-info.java` for package-level documentation

Example:
```java
/**
 * Manages the lifecycle of transfer processes.
 * <p>
 * This service orchestrates the state machine for data transfers,
 * handling provisioning, data transfer, and deprovisioning phases.
 */
public interface TransferProcessManager {
    // ...
}
```

### Commit Messages

Follow conventional commits:
- `feat:` new features
- `fix:` bug fixes
- `docs:` documentation changes
- `refactor:` code refactoring
- `test:` test additions or changes
- `chore:` maintenance tasks

Example: `feat: add support for custom authentication provider`

## Common Development Tasks

### Adding a New Dependency

1. **Add to version catalog** (`gradle/libs.versions.toml`):
   ```toml
   [versions]
   mylib = "1.2.3"
   
   [libraries]
   mylib = { module = "com.example:mylib", version.ref = "mylib" }
   ```

2. **Use in module** (`build.gradle.kts`):
   ```kotlin
   dependencies {
       implementation(libs.mylib)
   }
   ```

### Adding a New Module

1. **Create directory structure**:
   ```
   extensions/my-module/
   ├── build.gradle.kts
   └── src/
       ├── main/java/
       └── test/java/
   ```

2. **Add to settings.gradle.kts**:
   ```kotlin
   include(":extensions:my-module")
   ```

3. **Create build.gradle.kts**:
   ```kotlin
   plugins {
       `java-library`
   }
   
   dependencies {
       // your dependencies
   }
   ```

### Running a Launcher Locally

For testing and development, you can run a reference launcher or sample:

1. **Build a reference launcher**:
   ```bash
   ./gradlew :launchers:generic:build
   ```

2. **Extract distribution**:
   ```bash
   cd launchers/generic/build/distributions
   unzip generic.zip
   cd generic
   ```

3. **Run**:
   ```bash
   java -jar lib/generic.jar
   ```

**Better Approach for Learning**: Use the [EDC Samples](https://github.com/eclipse-edc/Samples) repository which provides complete, documented examples:

```bash
git clone https://github.com/eclipse-edc/Samples.git
cd Samples/basic/basic-01-basic-connector
./gradlew build
java -jar build/libs/basic-connector.jar
```

### Viewing Available Tasks

```bash
./gradlew tasks
```

### Cleaning Build Artifacts

```bash
./gradlew clean
```

## Getting Help

- **Documentation**: Check [docs/](../docs/) and [decision records](developer/decision-records/)
- **Discord**: Join the [EDC Discord community](https://discord.gg/n4sD9qtjMQ)
- **GitHub Issues**: Search or create issues on GitHub
- **Discussions**: Use GitHub Discussions for questions

## Next Steps

- Review the [Architecture Documentation](ARCHITECTURE.md) to understand the framework design
- Explore [Decision Records](developer/decision-records/) for design rationale
- Study [EDC Samples](https://github.com/eclipse-edc/Samples) for complete working examples
- Examine [Tractus-X EDC](https://github.com/eclipse-tractusx/tractusx-edc) as a production implementation reference
- Read the [Contributing Guidelines](https://github.com/eclipse-edc/eclipse-edc.github.io/blob/main/CONTRIBUTING.md)

Happy coding! 🚀
