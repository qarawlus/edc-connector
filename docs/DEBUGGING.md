# EDC Connector Debugging Guide

## Overview

This guide provides practical debugging strategies and tips for troubleshooting issues when working with the EDC Connector framework - whether you're developing the framework itself, building extensions, or debugging a connector implementation.

## Table of Contents

1. [General Debugging Strategies](#general-debugging-strategies)
2. [Debugging in Different Environments](#debugging-in-different-environments)
3. [Common Issues and Solutions](#common-issues-and-solutions)
4. [Logging and Diagnostics](#logging-and-diagnostics)
5. [Performance Debugging](#performance-debugging)
6. [Network and Protocol Debugging](#network-and-protocol-debugging)

## General Debugging Strategies

### 1. Start with Logs

Always check the logs first. The connector uses SLF4J for logging.

**Enable Debug Logging:**

Set logging level via configuration:
```properties
edc.logging.level=DEBUG
```

Or set environment variable:
```bash
export EDC_LOGGING_LEVEL=DEBUG
```

**Increase Logging for Specific Packages:**

```properties
# For control plane transfer debugging
edc.logging.org.eclipse.edc.connector.controlplane.transfer=DEBUG

# For DSP protocol debugging
edc.logging.org.eclipse.edc.protocol.dsp=TRACE
```

### 2. Use Breakpoints Strategically

**Key Breakpoint Locations:**

- **Extension Loading**: `ServiceExtension.initialize()`
- **Transfer Process State Changes**: `TransferProcessManager` state transitions
- **Contract Negotiation**: `ContractNegotiationManager` state transitions
- **Data Plane Transfer**: `DataPlaneManager.transfer()`
- **Policy Evaluation**: `PolicyEngine.evaluate()`

### 3. Understand the Flow

Before debugging, understand the expected flow:

1. **Contract Negotiation Flow**:
   ```
   Request Catalog → Initiate Negotiation → Exchange Offers → 
   Agreement → Store Contract
   ```

2. **Transfer Process Flow**:
   ```
   Request Transfer → Validate Contract → Provision Resources →
   Initiate Data Plane Transfer → Monitor → Complete
   ```

### 4. Check State Machines

The framework uses state machines extensively. Common states:

**Transfer Process States:**
- `INITIAL` → `PROVISIONING` → `PROVISIONED` → `REQUESTED` → 
- `STARTED` → `COMPLETED` / `TERMINATED` / `DEPROVISIONING`

**Contract Negotiation States:**
- `REQUESTING` → `REQUESTED` → `OFFERED` → `ACCEPTED` → 
- `AGREED` / `TERMINATED`

## Debugging in Different Environments

### IntelliJ IDEA

#### Debug a Test

1. Right-click on test class/method
2. Select "Debug 'TestName'"
3. Set breakpoints as needed
4. Use "Step Over" (F8), "Step Into" (F7), "Step Out" (Shift+F8)

#### Debug a Running Connector Implementation

1. **Create Remote Debug Configuration**:
   - Run → Edit Configurations → Add New → Remote JVM Debug
   - Host: `localhost`
   - Port: `5005` (or your debug port)

2. **Start your connector with debug enabled**:
   ```bash
   java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 \
        -jar your-connector.jar
   ```

3. **Attach Debugger**:
   - Click Debug button in IntelliJ

#### Evaluate Expressions

During debugging:
- Select expression → Alt+F8 (Evaluate Expression)
- View variables in Variables panel
- Add watches in Watches panel

### VS Code

#### Debug Configuration

Create `.vscode/launch.json`:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Debug EDC Connector",
            "request": "attach",
            "hostName": "localhost",
            "port": 5005
        },
        {
            "type": "java",
            "name": "Debug Current Test",
            "request": "launch",
            "mainClass": "org.junit.platform.console.ConsoleLauncher",
            "args": "--scan-classpath"
        }
    ]
}
```

### Command Line Debugging

#### Using JDB (Java Debugger)

```bash
# Start your connector with debug enabled
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=*:5005 \
     -jar your-connector.jar

# In another terminal, attach JDB
jdb -attach 5005

# JDB commands
> stop at org.eclipse.edc.connector.core.CoreServicesExtension:45
> run
> locals
> print variableName
> step
> cont
```

## Common Issues and Solutions

### Issue: Extension Not Loading

**Symptoms:**
- Extension class not instantiated
- Services not registered
- NoClassDefFoundError

**Debug Steps:**

1. **Check Service Provider File**:
   ```bash
   cat src/main/resources/META-INF/services/org.eclipse.edc.spi.system.ServiceExtension
   ```
   Ensure your extension class is listed.

2. **Verify Extension is on Classpath**:
   ```bash
   jar -tf launcher.jar | grep MyExtension.class
   ```

3. **Check Extension Dependencies**:
   - Ensure all `@Inject` dependencies are available
   - Add breakpoint in extension's `initialize()` method
   - Check logs for dependency resolution errors

4. **Enable Extension Loading Logs**:
   ```properties
   edc.logging.org.eclipse.edc.boot=DEBUG
   ```

### Issue: Transfer Process Stuck

**Symptoms:**
- Transfer doesn't progress
- State remains unchanged
- No error messages

**Debug Steps:**

1. **Check Transfer Process State**:
   
   Using your connector implementation's Management API:
   ```bash
   curl http://localhost:8181/management/v3/transferprocesses/{id}
   ```
   
   **Note**: Actual endpoint URL depends on your connector implementation's configuration.

2. **Enable State Machine Logging**:
   ```properties
   edc.logging.org.eclipse.edc.connector.controlplane.transfer=DEBUG
   ```

3. **Set Breakpoint in State Transitions**:
   - `TransferProcessManagerImpl.processTransfers()`
   - State-specific methods (e.g., `processProvisioning()`)

4. **Check for Exceptions**:
   - Review logs for stack traces
   - Check callback URLs are reachable
   - Verify data plane is accessible

5. **Inspect State Store**:
   ```sql
   SELECT * FROM edc_transfer_process WHERE transfer_process_id = 'your-id';
   ```

### Issue: Contract Negotiation Failing

**Symptoms:**
- Negotiation ends in TERMINATED state
- Policy evaluation errors
- Protocol communication issues

**Debug Steps:**

1. **Enable Protocol Logging**:
   ```properties
   edc.logging.org.eclipse.edc.protocol.dsp=TRACE
   ```

2. **Check Policy Evaluation**:
   - Set breakpoint in `PolicyEngineImpl.evaluate()`
   - Check constraint function registration
   - Verify policy definitions

3. **Inspect Negotiation Messages**:
   - Enable HTTP logging to see DSP messages
   - Check request/response payloads

4. **Verify Participant Identity**:
   - Check participant ID extraction
   - Verify token validation
   - Check trust anchors configuration

### Issue: Data Plane Transfer Failing

**Symptoms:**
- Transfer process reports failure
- No data transferred
- Timeout errors

**Debug Steps:**

1. **Check Data Plane Selection**:
   ```properties
   edc.logging.org.eclipse.edc.connector.dataplane.selector=DEBUG
   ```

2. **Verify Data Plane Registration**:
   
   Using your connector implementation's Management API:
   ```bash
   curl http://localhost:8181/management/v3/dataplanes
   ```
   
   **Note**: Endpoint availability depends on your connector configuration.

3. **Check Data Source/Sink Configuration**:
   - Set breakpoint in source/sink implementations
   - Verify credentials and endpoints
   - Check network connectivity

4. **Monitor Data Plane Logs**:
   ```properties
   edc.logging.org.eclipse.edc.connector.dataplane.framework=DEBUG
   ```

5. **Test Data Access Directly**:
   - Try accessing source/destination outside connector
   - Verify authentication works independently

### Issue: Configuration Not Applied

**Symptoms:**
- Settings don't take effect
- Default values used instead

**Debug Steps:**

1. **Check Configuration Loading**:
   ```properties
   edc.logging.org.eclipse.edc.boot.system.DefaultServiceExtensionContext=DEBUG
   ```

2. **Verify Configuration Key**:
   - Check for typos in property names
   - Ensure correct prefix (e.g., `edc.`)

3. **Check Configuration Source**:
   - Environment variables override properties files
   - System properties override environment variables

4. **Print Configuration**:
   ```java
   var value = context.getConfig().getString("edc.my.property", null);
   context.getMonitor().info("Config value: " + value);
   ```

## Logging and Diagnostics

### Structured Logging

**Add Contextual Information:**

```java
monitor.debug("Processing transfer", () -> Map.of(
    "transferProcessId", transferProcess.getId(),
    "state", transferProcess.getState(),
    "assetId", transferProcess.getAssetId()
));
```

### Correlation IDs

Track requests across components:

```java
var correlationId = UUID.randomUUID().toString();
MDC.put("correlation-id", correlationId);
try {
    // Your code
} finally {
    MDC.remove("correlation-id");
}
```

### Health Checks

Check connector health (if health check extensions are included):

```bash
# Check connector health
curl http://localhost:8181/api/check/health

# Check liveness
curl http://localhost:8181/api/check/liveness

# Check readiness
curl http://localhost:8181/api/check/readiness
```

**Note**: Health check endpoints depend on your connector implementation and configuration.

### Metrics

Enable and monitor metrics:

```properties
edc.metrics.enabled=true
edc.metrics.system.enabled=true
```

Access metrics:
```bash
curl http://localhost:9090/metrics
```

## Performance Debugging

### Enable Performance Logging

```properties
edc.logging.org.eclipse.edc.transaction=DEBUG
```

### Profile with JFR (Java Flight Recorder)

```bash
# Start with JFR enabled
java -XX:StartFlightRecording=duration=60s,filename=recording.jfr \
     -jar connector.jar

# Analyze recording
jfr print recording.jfr
```

### Monitor Resource Usage

```bash
# Check thread usage
jstack <pid>

# Check memory usage
jmap -heap <pid>

# Monitor GC
java -Xlog:gc* -jar connector.jar
```

### Identify Slow Operations

Add timing logs:

```java
var start = System.currentTimeMillis();
try {
    // Operation
} finally {
    var duration = System.currentTimeMillis() - start;
    monitor.debug("Operation took " + duration + "ms");
}
```

## Network and Protocol Debugging

### HTTP Request/Response Logging

Enable HTTP client logging:

```properties
edc.logging.okhttp3=DEBUG
```

### Capture Network Traffic

Using tcpdump:
```bash
sudo tcpdump -i any -w connector-traffic.pcap port 8282
```

Analyze with Wireshark:
```bash
wireshark connector-traffic.pcap
```

### DSP Protocol Debugging

**Log DSP Messages:**

```properties
edc.logging.org.eclipse.edc.protocol.dsp.http=TRACE
```

**Inspect Message Transformers:**

Set breakpoints in:
- `JsonObjectFromXxxTransformer`
- `JsonObjectToXxxTransformer`

**Validate JSON-LD:**

Use online validators or tools:
```bash
# Pretty-print JSON-LD
jq . message.json

# Validate JSON-LD context
curl -X POST https://json-ld.org/playground/ -d @message.json
```

### OAuth2 Token Debugging

**Enable OAuth2 Logging:**

```properties
edc.logging.org.eclipse.edc.iam.oauth2=DEBUG
```

**Decode JWT Tokens:**

```bash
# Split token and decode
echo "eyJ..." | cut -d. -f2 | base64 -d | jq .
```

## Debugging Best Practices

1. **Reproduce Consistently**: Create a minimal reproducible example
2. **Isolate the Problem**: Use unit tests to isolate components
3. **Check Assumptions**: Verify preconditions and data
4. **Use Version Control**: Compare with working versions
5. **Document Findings**: Keep notes on what you've tried
6. **Ask for Help**: Use Discord, GitHub Discussions when stuck

## Troubleshooting Checklist

- [ ] Check logs for errors and warnings
- [ ] Verify configuration is correct
- [ ] Ensure all dependencies are satisfied
- [ ] Check network connectivity
- [ ] Verify authentication/authorization
- [ ] Review recent code changes
- [ ] Check system resources (memory, disk, network)
- [ ] Test with minimal configuration
- [ ] Compare with working environment
- [ ] Review relevant decision records

## Additional Resources

- [Developer Guide](DEVELOPER_GUIDE.md)
- [Architecture Documentation](ARCHITECTURE.md)
- [Decision Records](developer/decision-records/)
- [EDC Samples](https://github.com/eclipse-edc/Samples)
- [Community Discord](https://discord.gg/n4sD9qtjMQ)

## Getting Help

If you're still stuck:
1. Search GitHub issues for similar problems
2. Ask on Discord in the #help channel
3. Create a detailed GitHub issue with:
   - Steps to reproduce
   - Expected vs actual behavior
   - Relevant logs and configuration
   - Environment details
