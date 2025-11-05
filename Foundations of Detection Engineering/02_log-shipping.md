# Log Shipping: Getting Data from Source to Destination

## What is Log Shipping?

**Log shipping** is the process of collecting, forwarding, and delivering log data from sources (endpoints, servers, applications) to destinations (SIEM, data lake, analytics platform). The tools that perform this function are called **log shippers** or **forwarders**.

## Key Functions of Log Shippers

1. **Collection** - Read logs from files, APIs, or event streams
2. **Buffering** - Store logs temporarily if the destination is unavailable
3. **Filtering** - Include or exclude specific events
4. **Parsing** - Structure unstructured log data
5. **Transformation** - Modify field names or values
6. **Routing** - Send different logs to different destinations
7. **Compression** - Reduce bandwidth usage
8. **Encryption** - Secure data in transit

## Types of Log Shippers

### Vendor-Agnostic (Open Source / Multi-Platform)

These tools can send data to multiple different destinations and aren't tied to a specific vendor's platform.

#### Beats (Elastic)
- Filebeat - Log files and text-based data
- Winlogbeat - Windows Event Logs
- Auditbeat - Audit data (file integrity, user activity)
- Metricbeat - System and service metrics
- Packetbeat - Network packet analysis
- Heartbeat - Uptime monitoring
- **Advantages:** Small footprint (specific agents by purpose), well-documented, easy to configure

#### Fluentd / Fluent Bit
- **Fluentd**: Full-featured log aggregator with 500+ plugins
- **Fluent Bit**: Lightweight version with only the most common plugins
- **Advantages**: Extensive plugin ecosystem, unified logging layer

#### NXLog
- **Community Edition** (free) and **Enterprise Edition** (paid)
- **Advantages**: Strong Windows Event Log support, format conversion

#### Vector (Datadog / Open Source)
- Modern, high-performance observability data pipeline
- Written in Rust for speed and reliability
- **Advantages**: Low resource usage, built-in testing, observability

#### Logstash (Elastic)
- Data processing pipeline with input, filter, and output plugins
- More heavyweight than Beats
- **Advantages**: Powerful transformation capabilities, mature ecosystem
- **Use case**: Complex log parsing and enrichment scenarios


### Vendor-Specific (Platform-Native)

These tools are designed for specific vendor platforms but may support limited multi-platform scenarios.

#### Azure Monitor Agent (AMA)
- **Platform**: Microsoft Azure, Azure Sentinel
- Replacement for Log Analytics agent (MMA/OMS)
- **Use case**: Azure-centric environments

#### AWS CloudWatch Agent
- **Platform**: Amazon Web Services
- Collects logs and metrics from EC2, on-premises servers
- **Use case**: AWS workloads and hybrid environments

#### Google Cloud Logging Agent
- **Platform**: Google Cloud Platform
- Based on Fluentd
- **Use case**: GCP environments

#### Splunk Universal Forwarder
- **Platform**: Splunk Enterprise, Splunk Cloud
- Lightweight agent for forwarding data to Splunk
- **Use case**: Splunk deployments

#### Elastic Agent
- **Platform**: Elastic Stack (Elasticsearch, Security)
- Unified agent replacing individual Beats
- **Use case**: Modern Elastic deployments

#### Datadog Agent
- **Platform**: Datadog
- Collects logs, metrics, and traces
- **Use case**: Datadog customers

## Syslog: The Classic Protocol

**Syslog** is a standard protocol (RFC 5424) for message logging, particularly common in Unix/Linux environments.

### Syslog Implementations

#### Rsyslog
- **Most common** Linux syslog implementation
- High performance, modular design
- Supports TCP, TLS, relaying
- **Use case**: Linux server logging

#### Syslog-ng
- Advanced syslog daemon
- Better filtering and routing capabilities than rsyslog
- Correlation and pattern matching
- **Use case**: Complex syslog routing scenarios

### Syslog Considerations
- **Severity Levels**: Emergency (0) through Debug (7)
- **Facilities**: Categorize message source (kern, user, mail, etc.)
- **Transport**: UDP (unreliable, fast) vs TCP (reliable, slower)
- **Security**: Use TLS for encryption
- **Limitations**: No guaranteed delivery with UDP, limited structure

## Windows Event Forwarding (WEF)

**Windows Event Forwarding** is a native Windows capability to forward events between Windows systems without third-party agents.

### How WEF Works
1. **Event Collector** - Central Windows server receives events
2. **Event Source** - Windows endpoints send events
3. **Subscriptions** - Define which events to collect
4. **WinRM** - Uses Windows Remote Management protocol

### WEF Advantages
- ✅ **No additional agents** - Built into Windows
- ✅ **Free** - No licensing costs
- ✅ **Centralized collection** - Aggregate before SIEM ingestion
- ✅ **Compression** - Reduces bandwidth

### WEF Limitations
- ❌ **Windows only** - Doesn't work for Linux/macOS
- ❌ **Limited transformation** - Can't parse or enrich
- ❌ **Scalability concerns** - Large environments may struggle

### Resources
- **Palantir WEF Guidance**: https://github.com/palantir/windows-event-forwarding
- **Microsoft WEF Documentation**: https://docs.microsoft.com/en-us/windows/security/threat-protection/use-windows-event-forwarding-to-assist-in-intrusion-detection

## Choosing the Right Log Shipper

Consider these factors when selecting a log shipper:

| Factor | Questions to Ask |
|--------|------------------|
| **Platform Support** | What operating systems do you need to support? Vendor Lock-in? |
| **Destination** | Where is the data going? (SIEM, data lake, multiple?) |
| **Volume** | How much data will you collect? |
| **Performance** | What are the resource constraints? |
| **Features** | Do you need parsing, filtering, or enrichment? |
| **Management** | How will you deploy and configure at scale? |
| **Cost** | Licensing, bandwidth, storage implications? |

## Deployment Best Practices

### 1. Start Small, Scale Gradually
- Begin with a pilot group
- Test performance and reliability
- Roll out incrementally

### 2. Implement Monitoring
- Track agent health
- Monitor collection rates
- Alert on failures

### 3. Use Configuration Management
- Ansible, Puppet, Chef for agent deployment
- Version control configurations
- Automate updates

### 4. Secure the Pipeline
- Encrypt data in transit (TLS/SSL)
- Authenticate collectors
- Segment networks appropriately

### 5. Plan for Failure
- Buffer locally when destination unavailable
- Implement retry logic
- Have fallback destinations

### 6. Test Performance Impact
- Measure CPU, memory, disk I/O
- Test during peak loads
- Optimize collection intervals

## Key Takeaways

- **Consider transformation needs** - Some shippers are better at parsing and enrichment
- **Plan for scale** - Easy configuration management is critical for large deployments
- **Monitor your shippers** - They're a critical part of your detection infrastructure

## Additional Resources

- **Elastic Beats**: https://www.elastic.co/beats/
- **Fluentd**: https://www.fluentd.org/
- **Vector**: https://vector.dev/
- **NXLog**: https://nxlog.co/
