# Monitoring & Logging

Build observable systems that tell you what's happening, what's broken, and what's about to break.

## The Three Pillars of Observability

### 1. Metrics
Numerical data about system behavior over time.
- **System metrics**: CPU, memory, disk, network
- **Application metrics**: Request rate, response time, error rate
- **Business metrics**: User signups, revenue, conversion rates

### 2. Logs
Detailed records of events happening in your system.
- **Structured logs**: JSON format for easy parsing
- **Centralized logging**: All logs in one place
- **Log levels**: DEBUG, INFO, WARN, ERROR, FATAL

### 3. Traces
End-to-end view of requests as they flow through distributed systems.
- **Distributed tracing**: Follow requests across services
- **Span analysis**: Understand bottlenecks
- **Dependency mapping**: Visualize service interactions

## The Prometheus Ecosystem

### Prometheus Fundamentals
Time-series database designed for monitoring:

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
  
  - job_name: 'my-app'
    static_configs:
      - targets: ['localhost:8080']
```

### PromQL Query Language
```promql
# CPU usage percentage
100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# HTTP request rate
rate(http_requests_total[5m])

# 95th percentile response time
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Error rate percentage
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100
```

### Application Metrics with Client Libraries
```python
# Python example with prometheus_client
from prometheus_client import Counter, Histogram, start_http_server

REQUEST_COUNT = Counter('requests_total', 'Total requests', ['method', 'endpoint'])
REQUEST_DURATION = Histogram('request_duration_seconds', 'Request duration')

@REQUEST_DURATION.time()
def process_request():
    REQUEST_COUNT.labels(method='GET', endpoint='/api/users').inc()
    # Your application logic here
```

## Grafana for Visualization

### Dashboard Design Principles
- **User-focused**: Start with what users care about
- **Layered approach**: Overview → drill-down → details
- **Context matters**: Add annotations and descriptions
- **Alert on what matters**: Not everything needs an alert

### Dashboard Examples

#### System Overview Dashboard
- CPU, Memory, Disk, Network usage
- System load and uptime
- Top processes and resource consumers

#### Application Performance Dashboard
- Request rate and response time
- Error rates and types
- Database connection pools
- Cache hit ratios

#### Business Metrics Dashboard
- Active users
- Revenue metrics
- Feature usage
- Conversion funnels

### Grafana Alerting
```yaml
# Alert rule example
- alert: HighErrorRate
  expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High error rate detected"
    description: "Error rate is {{ $value }} requests/second"
```

## ELK Stack for Logging

### Elasticsearch
Distributed search and analytics engine:
```bash
# Index mapping for application logs
PUT /app-logs
{
  "mappings": {
    "properties": {
      "timestamp": {"type": "date"},
      "level": {"type": "keyword"},
      "message": {"type": "text"},
      "service": {"type": "keyword"},
      "trace_id": {"type": "keyword"}
    }
  }
}
```

### Logstash
Data processing pipeline:
```ruby
# logstash.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][service] == "web-app" {
    grok {
      match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:msg}" }
    }
    date {
      match => [ "timestamp", "ISO8601" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
  }
}
```

### Kibana
Visualization and exploration:
- Log search and filtering
- Dashboard creation
- Index pattern management
- Alerting and monitoring

## Distributed Tracing with Jaeger

### Instrumenting Applications
```python
# Python with OpenTelemetry
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Setup tracing
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

jaeger_exporter = JaegerExporter(
    agent_host_name="jaeger",
    agent_port=6831,
)

span_processor = BatchSpanProcessor(jaeger_exporter)
trace.get_tracer_provider().add_span_processor(span_processor)

# Use in application
@tracer.start_as_current_span("process_order")
def process_order(order_id):
    with tracer.start_as_current_span("validate_order") as span:
        span.set_attribute("order.id", order_id)
        # Validation logic
        
    with tracer.start_as_current_span("charge_payment"):
        # Payment processing
        pass
```

## SLI, SLO, and SLA

### Service Level Indicators (SLIs)
Specific metrics that matter to users:
- **Availability**: Percentage of successful requests
- **Latency**: Time to process requests
- **Throughput**: Requests handled per second

### Service Level Objectives (SLOs)
Target values for SLIs:
- 99.9% availability
- 95% of requests under 200ms
- Handle 1000 RPS

### Service Level Agreements (SLAs)
Contractual commitments with consequences:
- Customer-facing commitments
- Penalties for not meeting SLAs
- Usually more lenient than SLOs

## Error Budgets
Mathematical way to balance reliability and feature velocity:

```
Error Budget = 1 - SLO
If SLO is 99.9%, error budget is 0.1%
Monthly budget = 43.8 minutes of downtime
```

## Alerting Best Practices

### Alert on Symptoms, Not Causes
```yaml
# Good: Alert on user impact
- alert: HighLatency
  expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5

# Not ideal: Alert on resource usage
- alert: HighCPU
  expr: cpu_usage > 80
```

### Runbook Integration
Every alert should have:
- Clear description of the problem
- Impact on users
- Investigation steps
- Escalation procedures

### Alert Fatigue Prevention
- Only alert on actionable items
- Use different severity levels
- Implement alert suppression
- Regular alert review and cleanup

## Hands-On Projects

### Project 1: Complete Monitoring Stack
Deploy and configure:
- Prometheus for metrics collection
- Grafana for visualization
- Alertmanager for notifications
- Custom exporters for application metrics

### Project 2: Centralized Logging
Build a logging pipeline:
- Filebeat for log shipping
- Logstash for processing
- Elasticsearch for storage
- Kibana for visualization

### Project 3: Distributed Tracing
Implement tracing across microservices:
- Jaeger for trace collection
- OpenTelemetry instrumentation
- Service dependency mapping
- Performance bottleneck identification

---

*Good monitoring is invisible when everything works and invaluable when things break. Invest time in observability and your future self will thank you.*
