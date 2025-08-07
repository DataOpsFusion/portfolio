# Database Fundamentals

Master the databases that power modern data systems - from traditional SQL to cutting-edge time-series databases.

## The Database Landscape

Choosing the right database is crucial. Here's a practical guide to when to use what:

### 🗃️ Relational Databases (SQL)
**Best for**: Transactions, complex queries, data integrity
- **PostgreSQL**: The Swiss Army knife of databases
- **MySQL**: Fast and reliable for web applications
- **SQLite**: Perfect for development and small applications

### 📊 Analytical Databases
**Best for**: Analytics, reporting, data warehousing
- **ClickHouse**: Blazing fast for time-series and analytics
- **BigQuery**: Serverless analytics at scale
- **Snowflake**: Cloud data warehouse with separation of compute/storage

### 🔄 NoSQL Databases
**Best for**: Flexible schemas, horizontal scaling
- **MongoDB**: Document database for JSON-like data
- **Cassandra**: Distributed database for high availability
- **DynamoDB**: Serverless NoSQL on AWS

### ⚡ In-Memory Databases
**Best for**: Caching, real-time applications
- **Redis**: The go-to cache and pub/sub system
- **Memcached**: Simple distributed caching

### 📈 Time-Series Databases
**Best for**: IoT data, monitoring, financial data
- **InfluxDB**: Purpose-built for time-series data
- **TimescaleDB**: PostgreSQL extension for time-series

## Database Design Principles

### Normalization vs Denormalization
Understanding when to normalize and when to denormalize:

**Normalize when**:
- Data integrity is critical
- Storage cost is high
- Write-heavy workloads

**Denormalize when**:
- Query performance is critical
- Read-heavy workloads
- Analytics use cases

### Indexing Strategies
Indexes are your friend (but choose wisely):

```sql
-- Primary indexes for unique identification
CREATE UNIQUE INDEX idx_user_email ON users(email);

-- Composite indexes for multi-column queries
CREATE INDEX idx_order_date_status ON orders(order_date, status);

-- Partial indexes for filtered queries
CREATE INDEX idx_active_users ON users(created_at) 
WHERE status = 'active';
```

### Partitioning
Split large tables for better performance:

- **Horizontal partitioning**: Split by rows (e.g., by date)
- **Vertical partitioning**: Split by columns
- **Functional partitioning**: Split by feature/service

## Hands-On Tutorials

### Tutorial 1: PostgreSQL Deep Dive
- Advanced SQL techniques
- Query optimization
- Index strategies
- JSON support in PostgreSQL

### Tutorial 2: Building a Data Warehouse
- Star vs snowflake schemas
- Slowly changing dimensions
- ETL vs ELT patterns
- Performance optimization

### Tutorial 3: NoSQL with MongoDB
- Document modeling
- Aggregation pipelines
- Sharding strategies
- Replica sets

### Tutorial 4: Time-Series with InfluxDB
- Schema design for time-series
- Retention policies
- Continuous queries
- Grafana integration

## Performance Optimization

### Query Optimization
- Explain plans and execution strategies
- Common anti-patterns to avoid
- Index usage optimization
- Query rewriting techniques

### Hardware Considerations
- CPU vs I/O bound workloads
- Memory sizing for databases
- Storage types and their impact
- Network considerations

---

*Database fundamentals never go out of style. Master these concepts and you'll be valuable in any data role.*
