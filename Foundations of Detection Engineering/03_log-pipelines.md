# Log Pipelines: Processing Data at Scale

## What is a Log Pipeline?

A **log pipeline** is the infrastructure and processes that transform raw telemetry data into actionable security intelligence. It sits between log shippers and your SIEM/analytics platform, performing critical data operations.

```
[Sources] → [Shippers] → [Pipeline] → [Destinations]
                            ↓
                  Parse | Filter | Enrich
                  Transform | Route | Buffer
```

## Why Use a Log Pipeline?

### The Problem Without Pipelines
- **Inconsistent formats** - Each source has different field names
- **High costs** - Sending everything to expensive SIEM storage
- **Poor data quality** - Noisy, redundant, or useless data
- **Vendor lock-in** - Hard to switch destinations
- **No context** - Missing threat intelligence enrichment

### The Solution: Purpose-Built Pipelines
- ✅ **Normalize** data to common schemas
- ✅ **Filter** out noise and duplicates
- ✅ **Enrich** with context and threat intel
- ✅ **Route** to appropriate destinations based on value
- ✅ **Transform** to reduce costs
- ✅ **Buffer** during outages

## Core Pipeline Operations

### 1. Parsing
**Extract structure from unstructured data**

Raw log:
```
192.168.1.100 - admin [05/Nov/2025:14:23:45 -0500] "GET /api/users HTTP/1.1" 200 1234
```

Parsed:
```json
{
  "source_ip": "192.168.1.100",
  "user": "admin",
  "timestamp": "2025-11-05T14:23:45-05:00",
  "method": "GET",
  "uri": "/api/users",
  "status_code": 200,
  "bytes": 1234
}
```

**Common parsing techniques:**
- Regex patterns
- Grok patterns (Logstash)
- JSON parsing
- CSV parsing
- XML parsing

### 2. Filtering
**Remove unnecessary or low-value data**

**Examples:**
- Drop normal Windows logon activity (ANONYMOUS LOGON, Local System Account)
- Remove health check requests from web logs
- Exclude verbose debug messages
- Drop duplicate events
- Use log reduction tools such as [LogSlash](https://github.com/FoxIO-LLC/LogSlash)

### 3. Normalization
**Map diverse field names to a common schema**

**The Challenge:**
- Source A uses: `source_ip`
- Source B uses: `src_ip`
- Source C uses: `source_ipaddr`
- Source D uses: `srcIp`

**Without normalization:**
```sql
-- Query AWS CloudTrail
source_type="cloudtrail" sourceIPAddress=1.2.3.4

-- Query Azure Activity Logs  
source_type="azure" CallerIPAddress=1.2.3.4

-- Query O365 Audit Logs
source_type="o365" ClientIP=1.2.3.4

-- Different queries for the same question!
```

**With normalization (ECS):**
```sql
-- Query all sources
source.ip=1.2.3.4
```

**Solution: Pick a Standard**

#### Elastic Common Schema (ECS)
- Open source normalization framework
- Well-documented field mappings
- Used by Elastic Security
- Reference: https://www.elastic.co/guide/en/ecs/current/

**Example ECS fields:**
- `source.ip` - Source IP address
- `destination.ip` - Destination IP
- `user.name` - Username
- `event.action` - What happened
- `event.outcome` - Success or failure

#### Splunk Common Information Model (CIM)
- Splunk's normalization standard
- Focused on use cases (authentication, network traffic)
- Required for Splunk Enterprise Security
- Reference: https://docs.splunk.com/Documentation/CIM/

**Example CIM fields:**
- `src` - Source IP address
- `dest` - Destination IP
- `user` - Username
- `action` - What happened (allowed, blocked, success, failure)

#### Open Cybersecurity Schema Framework (OCSF)
- Vendor-neutral, open standard
- Backed by AWS, Splunk, and others
- Goal: Universal security log format
- Reference: https://schema.ocsf.io/

**Example OCSF fields:**
- `src_endpoint.ip` - Source IP address
- `dst_endpoint.ip` - Destination IP
- `actor.user.name` - Username
- `activity_name` - What happened

**Choosing a schema:**
- Use your SIEM vendor's standard (CIM for Splunk, ECS for Elastic)
- Consider OCSF for multi-platform or future flexibility
- Be consistent across all sources

### 4. Enrichment
**Add context to make data more valuable**

#### GeoIP Enrichment
Add geographic information based on IP address:
```json
{
  "source_ip": "8.8.8.8",
  "source_geo": {
    "country": "United States",
    "region": "California",
    "city": "Mountain View",
    "latitude": 37.386,
    "longitude": -122.0838,
    "asn": "AS15169",
    "organization": "Google LLC"
  }
}
```

#### Threat Intelligence Enrichment
Flag known malicious indicators:
```json
{
  "destination_ip": "185.220.101.1",
  "threat_intel": {
    "is_malicious": true,
    "reputation_score": 95,
    "categories": ["botnet", "tor_exit_node"],
    "first_seen": "2024-03-15",
    "sources": ["abuse.ch", "alienvault_otx"]
  }
}
```

#### Asset Enrichment
Add organizational context:
```json
{
  "hostname": "web-prod-01",
  "asset_info": {
    "owner": "Engineering Team",
    "criticality": "high",
    "environment": "production",
    "data_classification": "confidential",
    "location": "us-east-1"
  }
}
```

#### User Enrichment
Add identity information:
```json
{
  "username": "jdoe",
  "user_info": {
    "full_name": "John Doe",
    "department": "Finance",
    "title": "VP of Operations",
    "manager": "Jane Smith",
    "risk_score": 75,
    "is_privileged": true
  }
}
```

#### MITRE ATT&CK Mapping
Tag events with relevant techniques:
```json
{
  "event": "Process created: powershell.exe -encodedcommand ...",
  "mitre_attack": {
    "tactics": ["Execution", "Defense Evasion"],
    "techniques": ["T1059.001", "T1027"],
    "technique_names": ["PowerShell", "Obfuscated Files or Information"]
  }
}
```

**Enrichment sources:**
- GeoIP databases (MaxMind, IP2Location)
- Threat intel feeds (MISP, AlienVault OTX, abuse.ch)
- CMDB/asset inventory
- Active Directory / LDAP
- Custom lookup tables

### 5. Transformation
**Modify data to optimize for storage and cost**

#### Sampling
Keep a percentage of high-volume, low-value logs:
```
Keep 100% of failed authentication attempts
Keep 10% of successful authentication attempts
Keep 1% of successful web requests (200 OK)
```

#### Aggregation
Combine similar events:
```
Instead of:
  - 10:00:01 - User login success
  - 10:00:03 - User login success
  - 10:00:05 - User login success

Store:
  - 10:00:00-10:00:05 - User login success (count: 3)
```

#### Redaction
Remove sensitive data:
```
Before: {"credit_card": "4532-1234-5678-9010"}
After:  {"credit_card": "4532-****-****-9010"}
```

#### Field Removal
Drop unnecessary fields to reduce size:
```
Before (2KB):
{
  "timestamp": "...",
  "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)...",
  "accept_language": "en-US,en;q=0.9",
  "accept_encoding": "gzip, deflate, br",
  "referer": "https://...",
  ...50 more fields...
}

After (500 bytes):
{
  "timestamp": "...",
  "source_ip": "...",
  "method": "GET",
  "uri": "/api/users",
  "status_code": 200
}
```

### 6. Routing
**Send different data to different destinations**

**Example routing logic:**
```
IF event_type = "authentication_failure" 
  → Send to SIEM (hot storage)
  
IF event_type = "firewall_allow" AND source_ip = internal
  → Send to data lake (warm storage)
  
IF event_type = "compliance_audit"
  → Send to SIEM AND compliance archive
  
IF threat_intel.is_malicious = true
  → Send to SIEM AND SOAR platform AND Slack alert
```

**Benefits:**
- Cost optimization (expensive vs cheap storage)
- Compliance requirements (specific retention policies)
- Real-time vs batch processing
- Multi-tenancy (different teams, different SIEMs)

## Pipeline Tools & Platforms

### Open Source / Vendor-Agnostic

#### Logstash (Elastic)
- Data processing pipeline with input, filter, and output plugins
- Powerful transformation and enrichment capabilities
- Mature ecosystem with extensive plugin library
- Reference: https://www.elastic.co/guide/en/logstash/current/

#### Fluentd
- Full-featured log aggregator with 500+ plugins
- Unified logging layer for data collection and consumption
- Extensive plugin ecosystem for diverse integrations
- Reference: https://www.fluentd.org/

#### Vector
- Modern observability data pipeline
- Written in Rust for speed and reliability
- Built-in testing and observability features
- Reference: https://vector.dev/

#### Cribl Stream
- Visual pipeline builder with drag-and-drop interface
- Pre-built packs for common use cases
- Powerful for cost reduction and data optimization
- Reference: https://cribl.io/stream/

### Message Queues & Event Streaming

#### Apache Kafka
- Distributed message queue and event streaming platform
- Buffer between shippers and processors
- Enables real-time stream processing

#### Redis Streams
- In-memory data structure for event streaming
- Fast, lightweight alternative to Kafka

### Cloud-Native Pipeline Services
- **AWS Kinesis Data Firehose** - Managed log delivery
- **Azure Event Hubs** - Event streaming service
- **Google Cloud Dataflow** - Stream and batch processing

## Pipeline Monitoring & Observability

### Key Metrics to Track

#### Throughput
- Events per second (EPS)
- Bytes per second
- Peak vs average load

#### Latency
- End-to-end processing time
- Per-stage processing time
- Time to SIEM availability

#### Data Quality
- Parsing success rate
- Failed enrichments
- Schema validation errors

#### Resource Usage
- CPU utilization
- Memory consumption
- Network bandwidth
- Disk I/O

#### Error Rates
- Failed deliveries
- Dropped events
- Retry attempts

### Alerting on Pipeline Health
```
ALERT: Parsing failure rate > 5% for 10 minutes
ALERT: Kafka lag > 1 million messages
ALERT: Pipeline CPU usage > 80% for 15 minutes
ALERT: Elasticsearch delivery failures > 100/min
```

## Key Takeaways
- **Pipelines are essential** for cost control and data quality
- **Normalize early** to simplify detection engineering
- **Enrich with context** to make data actionable
- **Filter aggressively** to control costs
- **Monitor pipeline health** as critically as you monitor logs
- **Version control** all pipeline configurations
- **Test before deploying** to production