<div align="center">
  <img src="assets/datatidy-logo-pypi.png" alt="DataTidy Logo" width="300">
  
  <h3>Configuration-Driven Data Processing Made Simple</h3>
  
  [![PyPI version](https://badge.fury.io/py/datatidy.svg)](https://pypi.org/project/datatidy/)
  [![Python versions](https://img.shields.io/pypi/pyversions/datatidy.svg)](https://pypi.org/project/datatidy/)
  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
  [![Downloads](https://pepy.tech/badge/datatidy)](https://pepy.tech/project/datatidy)
</div>

# DataTidy

A powerful, configuration-driven data processing and cleaning package for Python with robust fallback capabilities. DataTidy allows you to define complex data transformations, validations, and cleanings through simple YAML configuration files, ensuring 100% reliability in production environments.

## 🚀 Key Features

- **🔧 Configuration-Driven**: Define all transformations in YAML - no code required
- **📊 Multiple Data Sources**: CSV, Excel, databases (PostgreSQL, MySQL, Snowflake, etc.)
- **🔗 Multi-Input Joins**: Combine data from multiple sources with flexible join operations
- **⚡ Advanced Operations**: Map/reduce/filter with lambda functions and chained operations
- **🧠 Dependency Resolution**: Automatic execution order planning for complex transformations
- **📈 Time Series Support**: Lag operations and rolling window calculations
- **🛡️ Safe Expressions**: Secure evaluation with whitelist-based security
- **🎯 Data Validation**: Comprehensive validation rules with detailed error reporting
- **⚙️ CLI Interface**: Easy-to-use command-line tools for batch processing

### 🔄 Enhanced Fallback System (v0.1.0)

- **🛡️ 100% Reliability**: Dashboard never fails to load data with automatic fallback mechanisms
- **⚖️ Graceful Degradation**: Gets sophisticated transformations when possible, basic data when needed
- **🔍 Enhanced Error Logging**: Detailed error categorization with actionable debugging suggestions
- **📊 Data Quality Metrics**: Compare DataTidy results with fallback data for quality assessment
- **🎛️ Multiple Processing Modes**: Strict, partial, and fallback modes for different reliability requirements
- **🔧 Partial Processing**: Skip problematic columns while processing successful ones
- **📋 Processing Recommendations**: Get specific suggestions for improving configurations


## Installation

```bash
pip install datatidy
```

For development installation:
```bash
git clone https://github.com/your-repo/datatidy.git
cd datatidy
pip install -e ".[dev]"
```

## Quick Start

### 1. Create a sample configuration

```bash
datatidy sample config.yaml
```

### 2. Process your data

```bash
datatidy process config.yaml -i input.csv -o output.csv
```

### 3. Or use programmatically

```python
from datatidy import DataTidy

# Initialize with configuration
dt = DataTidy('config.yaml')

# Standard processing
result = dt.process_data('input.csv')

# Enhanced processing with fallback
result = dt.process_data_with_fallback('input.csv')

# Save result
dt.process_and_save('output.csv', 'input.csv')
```

## Configuration Structure

DataTidy uses YAML configuration files to define data processing pipelines:

```yaml
input:
  type: csv                    # csv, excel, database
  source: "data/input.csv"     # file path or SQL query
  options:
    encoding: utf-8
    delimiter: ","

output:
  columns:
    user_id:
      source: "id"             # Source column name
      type: int                # Data type conversion
      validation:
        required: true
        min_value: 1
    
    full_name:
      source: "name"
      type: string
      transformation: "str.title()"  # Python expression
      validation:
        required: true
        min_length: 2
        max_length: 100
    
    age_group:
      transformation: "'adult' if age >= 18 else 'minor'"
      type: string
      validation:
        allowed_values: ["adult", "minor"]

  filters:
    - condition: "age >= 0"
      action: keep

  sort:
    - column: user_id
      ascending: true

global_settings:
  ignore_errors: false
  max_errors: 100
  
  # Enhanced fallback settings
  processing_mode: partial           # strict, partial, or fallback
  enable_partial_processing: true
  enable_fallback: true
  max_column_failures: 5
  failure_threshold: 0.3             # 30% failure rate triggers fallback
  
  # Fallback transformations for problematic columns
  fallback_transformations:
    age_group:
      type: default_value
      value: "unknown"
```

## Examples

### Basic CSV Processing

```python
from datatidy import DataTidy

config = {
    "input": {
        "type": "csv",
        "source": "users.csv"
    },
    "output": {
        "columns": {
            "clean_name": {
                "source": "name",
                "transformation": "str.strip().title()",
                "type": "string"
            },
            "age_category": {
                "transformation": "'senior' if age > 65 else ('adult' if age >= 18 else 'minor')",
                "type": "string"
            }
        }
    }
}

dt = DataTidy()
dt.load_config(config)
result = dt.process_data()
print(result)
```

### Database Processing

```yaml
input:
  type: database
  source: 
    query: "SELECT * FROM users WHERE active = true"
    connection_string: "postgresql://user:pass@localhost/db"

output:
  columns:
    user_email:
      source: "email"
      type: string
      validation:
        pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    
    signup_date:
      source: "created_at"
      type: datetime
      format: "%Y-%m-%d"
```

### Excel Processing with Complex Transformations

```yaml
input:
  type: excel
  source:
    path: "sales_data.xlsx"
    sheet_name: "Q1_Sales"
    options:
      header: 0
      skiprows: 2

output:
  columns:
    revenue_category:
      transformation: |
        'high' if revenue > 100000 else (
          'medium' if revenue > 50000 else 'low'
        )
      validation:
        allowed_values: ["high", "medium", "low"]
    
    formatted_date:
      source: "sale_date"
      type: datetime
      format: "%Y-%m-%d"
    
    clean_product_name:
      source: "product"
      transformation: "str.strip().upper().replace('_', ' ')"
      validation:
        min_length: 1
        max_length: 50

  filters:
    - condition: "revenue > 0"
      action: keep
    - condition: "product != 'DELETED'"
      action: keep
```

## Enhanced Fallback Processing

### Production-Ready Data Processing
```python
from datatidy import DataTidy

# Initialize with fallback-enabled configuration
dt = DataTidy('config.yaml')

# Define fallback database query
def fallback_database_query():
    return pd.read_sql("SELECT * FROM facilities", db_connection)

# Process with guaranteed results
result = dt.process_data_with_fallback(
    data=input_df,
    fallback_query_func=fallback_database_query
)

# Your application always gets data!
if result.fallback_used:
    logger.warning("DataTidy processing failed, using database fallback")

# Check processing results
summary = dt.get_processing_summary()
print(f"Success: {summary['success']}")
print(f"Successful columns: {summary['successful_columns']}")
print(f"Failed columns: {summary['failed_columns']}")

# Get improvement recommendations
recommendations = dt.get_processing_recommendations()
for rec in recommendations:
    print(f"💡 {rec}")

# Compare data quality when both available
if not result.fallback_used:
    fallback_data = fallback_database_query()
    quality = dt.compare_with_fallback(fallback_data)
    print(f"Overall quality score: {quality.overall_quality_score:.2f}")
```

### Data Quality Monitoring
```python
from datatidy.fallback.metrics import DataQualityMetrics

# Compare processing results
comparison = DataQualityMetrics.compare_results(
    datatidy_df=processed_data,
    fallback_df=fallback_data,
    datatidy_time=2.3,
    fallback_time=0.8
)

# Print detailed comparison
DataQualityMetrics.print_comparison_summary(comparison)

# Export for analysis
DataQualityMetrics.export_comparison_report(
    comparison, 
    'quality_report.json'
)
```

## Command Line Usage

### Enhanced Processing Modes
```bash
# Strict mode (default) - fails on any error
datatidy process config.yaml --mode strict

# Partial mode - skip problematic columns
datatidy process config.yaml --mode partial --show-summary

# Fallback mode - use fallback transformations
datatidy process config.yaml --mode fallback

# Development mode with detailed feedback
datatidy process config.yaml --mode partial \\
  --show-summary \\
  --show-recommendations \\
  --error-log debug.json
```

### Process Data
```bash
# Basic processing
datatidy process config.yaml

# With input/output files
datatidy process config.yaml -i input.csv -o output.csv

# Ignore validation errors
datatidy process config.yaml --ignore-errors
```

### Validate Configuration
```bash
datatidy validate config.yaml
```

### Create Sample Configuration
```bash
datatidy sample my_config.yaml
```

## Expression System

DataTidy includes a safe expression parser that supports:

### Basic Operations
- Arithmetic: `+`, `-`, `*`, `/`, `//`, `%`, `**`
- Comparison: `==`, `!=`, `<`, `<=`, `>`, `>=`
- Logical: `and`, `or`, `not`
- Membership: `in`, `not in`

### Functions
- Type conversion: `str()`, `int()`, `float()`, `bool()`
- Math: `abs()`, `max()`, `min()`, `round()`
- String methods: `upper()`, `lower()`, `strip()`, `replace()`, etc.

### Examples
```yaml
transformations:
  # Conditional expressions
  status: "'active' if last_login_days < 30 else 'inactive'"
  
  # String operations
  clean_name: "name.strip().title()"
  
  # Mathematical calculations
  bmi: "weight / (height / 100) ** 2"
  
  # Complex conditions
  risk_level: |
    'high' if (age > 65 and income < 30000) else (
      'medium' if age > 40 else 'low'
    )
```

## Validation Rules

DataTidy supports comprehensive validation:

```yaml
validation:
  required: true              # Field must not be null
  nullable: false             # Field cannot be null
  min_value: 0               # Minimum numeric value
  max_value: 100             # Maximum numeric value
  min_length: 2              # Minimum string length
  max_length: 50             # Maximum string length
  pattern: "^[A-Za-z]+$"     # Regex pattern
  allowed_values: ["A", "B"] # Whitelist of values
```

## Error Handling

```python
dt = DataTidy('config.yaml')
result = dt.process_data('input.csv')

# Check for errors
if dt.has_errors():
    for error in dt.get_errors():
        print(f"Error: {error['message']}")
```

## API Reference

### DataTidy Class

#### Core Methods
- `load_config(config)`: Load configuration from file or dict
- `process_data(data=None)`: Process data according to configuration
- `process_and_save(output_path, data=None)`: Process and save data
- `get_errors()`: Get list of processing errors
- `has_errors()`: Check if errors occurred

#### Enhanced Fallback Methods
- `process_data_with_fallback(data=None, fallback_query_func=None)`: Process with fallback capabilities
- `get_processing_summary()`: Get detailed processing summary with metrics
- `get_error_report()`: Get categorized error report with debugging info
- `get_processing_recommendations()`: Get actionable recommendations for improvements
- `compare_with_fallback(fallback_df)`: Compare DataTidy results with fallback data
- `export_error_log(file_path)`: Export detailed error log to JSON
- `set_processing_mode(mode)`: Set processing mode (strict, partial, fallback)

### Processing Result Class

#### Properties
- `success`: Boolean indicating overall processing success
- `data`: Processed DataFrame result
- `processing_mode`: Mode used for processing
- `successful_columns`: List of successfully processed columns
- `failed_columns`: List of failed columns
- `fallback_used`: Boolean indicating if fallback was activated
- `processing_time`: Time taken for processing
- `error_log`: Detailed list of processing errors

### Data Quality Metrics

#### Static Methods
- `DataQualityMetrics.compare_results(datatidy_df, fallback_df)`: Compare two DataFrames
- `DataQualityMetrics.print_comparison_summary(comparison)`: Print formatted comparison
- `DataQualityMetrics.export_comparison_report(comparison, file_path)`: Export report to JSON

### Configuration Schema

See [Configuration Reference](docs/configuration.md) for complete schema documentation.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## License

MIT License - see LICENSE file for details.

## Changelog

### Version 0.1.0
- Initial release
- Basic CSV, Excel, and database support
- Safe expression engine
- Comprehensive validation system
- CLI interface