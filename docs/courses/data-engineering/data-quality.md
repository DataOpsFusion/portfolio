# Data Quality

Learn to build data systems that deliver trustworthy, accurate data. Because garbage in, garbage out is still true.

## Why Data Quality Matters

Bad data leads to:
- Wrong business decisions
- Failed ML models
- Angry stakeholders
- Wasted engineering time
- Lost revenue

Good data quality practices save time, money, and sanity.

## The Data Quality Framework

### 1. Accuracy
Is the data correct and error-free?

**Common Issues**:
- Typos and data entry errors
- Outdated information
- Wrong data types
- Invalid values

**Solutions**:
- Input validation
- Regular data audits
- Source system improvements
- Business rule validation

### 2. Completeness
Is all expected data present?

**Common Issues**:
- Missing records
- NULL values where they shouldn't be
- Incomplete data loads
- Partial system failures

**Solutions**:
- Completeness checks
- Required field validation
- Data load monitoring
- Source system SLAs

### 3. Consistency
Is data uniform across systems and time?

**Common Issues**:
- Different formats across systems
- Conflicting business rules
- Time zone inconsistencies
- Naming convention variations

**Solutions**:
- Data standardization
- Master data management
- Cross-system validation
- Canonical data models

### 4. Timeliness
Is data available when needed?

**Common Issues**:
- Delayed data loads
- Stale cache data
- Batch processing delays
- Network issues

**Solutions**:
- Real-time monitoring
- SLA definitions
- Alerting systems
- Data freshness checks

### 5. Validity
Does data conform to defined rules?

**Common Issues**:
- Invalid email formats
- Out-of-range values
- Foreign key violations
- Schema mismatches

**Solutions**:
- Schema validation
- Business rule engines
- Data type enforcement
- Constraint checking

## Implementing Data Quality

### Data Quality Checks

```python
# Example: Email validation
def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

# Example: Completeness check
def check_completeness(df, required_columns):
    missing_data = {}
    for col in required_columns:
        missing_count = df[col].isnull().sum()
        missing_data[col] = {
            'missing_count': missing_count,
            'missing_percentage': (missing_count / len(df)) * 100
        }
    return missing_data
```

### Data Profiling
Understand your data before you work with it:

- **Statistical analysis**: Min, max, mean, median, standard deviation
- **Data distribution**: Histograms, frequency counts
- **Pattern analysis**: Common formats, outliers
- **Relationship analysis**: Correlations, dependencies

### Data Quality Metrics

Track these KPIs:
- **Error rate**: Percentage of records with errors
- **Completeness score**: Percentage of complete records
- **Freshness**: Time since last update
- **Consistency score**: Cross-system data agreement
- **Business rule compliance**: Percentage passing validation

### Tools and Frameworks

#### Open Source
- **Great Expectations**: Data validation framework
- **Apache Griffin**: Data quality service
- **Deequ**: Data quality library for Apache Spark
- **Pandas Profiling**: Automated data profiling

#### Commercial
- **Talend Data Quality**: Enterprise data quality platform
- **Informatica Data Quality**: Comprehensive DQ suite
- **AWS Glue DataBrew**: Visual data preparation with quality checks

## Practical Implementations

### Project 1: Building a Data Quality Dashboard
Create a real-time dashboard showing:
- Data quality scores by dataset
- Trend analysis over time
- Alert notifications
- Root cause analysis

### Project 2: Automated Data Quality Pipeline
Build a pipeline that:
- Validates incoming data
- Quarantines bad records
- Alerts on quality issues
- Reports quality metrics

### Project 3: Cross-System Data Validation
Implement checks that:
- Compare data across systems
- Detect data drift
- Validate business rules
- Monitor data lineage

## Best Practices

### 1. Fail Fast
Catch data quality issues as early as possible in the pipeline.

### 2. Document Everything
Keep clear documentation of:
- Data quality rules
- Expected data formats
- Business logic
- Known issues and workarounds

### 3. Automate Quality Checks
Manual checks don't scale. Automate everything you can.

### 4. Make Quality Visible
Data quality should be visible to everyone who uses the data.

### 5. Continuous Improvement
Data quality is not a one-time effort - it's an ongoing process.

---

*Data quality isn't glamorous, but it's the foundation of everything else you'll build. Get this right and everything else becomes easier.*
