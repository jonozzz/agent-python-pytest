# pytest-reportportal Configuration Reference

This document provides comprehensive documentation for all configuration options available in the pytest-reportportal plugin.

## Table of Contents

- [Configuration Methods](#configuration-methods)
- [Required Configuration](#required-configuration)
- [Optional Configuration](#optional-configuration)
- [Command Line Options](#command-line-options)
- [Environment Variables](#environment-variables)
- [Configuration Examples](#configuration-examples)
- [Advanced Configuration](#advanced-configuration)
- [Troubleshooting Configuration](#troubleshooting-configuration)

## Configuration Methods

The pytest-reportportal plugin supports multiple configuration methods:

1. **pytest.ini file** (recommended)
2. **Command line arguments**
3. **Environment variables**
4. **Programmatic configuration**

### Priority Order

Configuration values are resolved in the following priority order (highest to lowest):

1. Command line arguments (`--rp-launch`)
2. Environment variables (`RP_LAUNCH`)
3. pytest.ini file (`rp_launch`)
4. Default values

## Required Configuration

These settings are mandatory for the plugin to function:

### `rp_uuid`

**Description**: User authentication token/UUID for ReportPortal access  
**Type**: String  
**Required**: Yes  
**Example**: `fb586627-32be-47dd-93c1-678873458a5f`

```ini
[pytest]
rp_uuid = fb586627-32be-47dd-93c1-678873458a5f
```

**How to obtain**:
1. Log into your ReportPortal instance
2. Go to User Profile → API Keys
3. Copy the UUID token

### `rp_endpoint`

**Description**: ReportPortal server URL  
**Type**: String  
**Required**: Yes  
**Format**: `http(s)://hostname:port`  
**Example**: `http://reportportal.company.com:8080`

```ini
[pytest]
rp_endpoint = http://reportportal.company.com:8080
```

### `rp_project`

**Description**: Project name in ReportPortal  
**Type**: String  
**Required**: Yes  
**Example**: `my_project`

```ini
[pytest]
rp_project = my_project
```

**Note**: The project must exist in ReportPortal and you must have access to it.

## Optional Configuration

### Launch Configuration

#### `rp_launch`

**Description**: Name for the test launch  
**Type**: String  
**Default**: `'Pytest Launch'`  
**Example**: `'Nightly Regression Tests'`

```ini
[pytest]
rp_launch = Nightly Regression Tests
```

**Command line override**:
```bash
pytest --reportportal --rp-launch "Custom Launch Name"
```

#### `rp_launch_description`

**Description**: Description for the test launch  
**Type**: String  
**Default**: `''` (empty)  
**Example**: `'Automated regression test suite'`

```ini
[pytest]
rp_launch_description = Automated regression test suite run nightly
```

**Command line override**:
```bash
pytest --reportportal --rp-launch-description "Custom description"
```

#### `rp_launch_tags`

**Description**: Tags to apply to the launch  
**Type**: Space-separated list  
**Default**: None  
**Example**: `'regression automated nightly'`

```ini
[pytest]
rp_launch_tags = regression automated nightly
```

#### `rp_tests_tags`

**Description**: Tags to apply to all test items  
**Type**: Space-separated list  
**Default**: None  
**Example**: `'pytest automated'`

```ini
[pytest]
rp_tests_tags = pytest automated
```

### Hierarchy Configuration

#### `rp_hierarchy_dirs`

**Description**: Enable hierarchical reporting by directories  
**Type**: Boolean  
**Default**: `False`  
**Note**: Does not support xdist plugin

```ini
[pytest]
rp_hierarchy_dirs = True
```

#### `rp_hierarchy_dirs_level`

**Description**: Directory level to start hierarchy from (relative to pytest.ini)  
**Type**: Integer  
**Default**: `0`  
**Example**: `1` (skip root directory)

```ini
[pytest]
rp_hierarchy_dirs_level = 1
```

#### `rp_hierarchy_module`

**Description**: Enable hierarchical reporting by modules  
**Type**: Boolean  
**Default**: `True`  
**Note**: Does not support xdist plugin

```ini
[pytest]
rp_hierarchy_module = True
```

#### `rp_hierarchy_class`

**Description**: Enable hierarchical reporting by classes  
**Type**: Boolean  
**Default**: `True`  
**Note**: Does not support xdist plugin

```ini
[pytest]
rp_hierarchy_class = True
```

#### `rp_hierarchy_parametrize`

**Description**: Enable hierarchical reporting for parametrized tests  
**Type**: Boolean  
**Default**: `False`  
**Note**: Does not support xdist plugin

```ini
[pytest]
rp_hierarchy_parametrize = True
```

#### `rp_display_suite_test_file`

**Description**: Include suite's relative file path in launch name  
**Type**: Boolean  
**Default**: `True`  
**Format**: `<RELATIVE_FILE_PATH>::<SUITE_NAME>`  
**Note**: Only relevant when `rp_hierarchy_module` is `False`

```ini
[pytest]
rp_display_suite_test_file = False
```

### Logging Configuration

#### `rp_log_batch_size`

**Description**: Size of batch log requests for async mode  
**Type**: Integer  
**Default**: `20`  
**Range**: `1-100`  
**Performance**: Higher values improve performance but use more memory

```ini
[pytest]
rp_log_batch_size = 50
```

#### `rp_log_level`

**Description**: Logging level for automated log records reporting  
**Type**: String  
**Default**: None  
**Values**: `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`  
**Note**: Only available with pytest >= 3.3

```ini
[pytest]
rp_log_level = INFO
```

**Command line override**:
```bash
pytest --reportportal --rp-log-level DEBUG
```

### Error Handling Configuration

#### `rp_ignore_errors`

**Description**: Ignore ReportPortal connection/API errors  
**Type**: Boolean  
**Default**: `False`  
**Recommendation**: Set to `True` for development, `False` for CI/CD

```ini
[pytest]
rp_ignore_errors = True
```

#### `rp_verify_ssl`

**Description**: Verify SSL certificates when connecting to ReportPortal  
**Type**: Boolean  
**Default**: `True`  
**Note**: Set to `False` only for development with self-signed certificates

```ini
[pytest]
rp_verify_ssl = False
```

### Tag Filtering Configuration

#### `rp_ignore_tags`

**Description**: Pytest markers to ignore (exclude from ReportPortal)  
**Type**: Space-separated list  
**Default**: None  
**Common values**: `'skip slow parametrize'`

```ini
[pytest]
rp_ignore_tags = skip slow parametrize xfail
```

### Issue Tracking Configuration

#### `rp_issue_marks`

**Description**: Pytest markers that contain issue information  
**Type**: Space-separated list  
**Default**: `''` (empty)  
**Example**: `'xfail issue bug'`

```ini
[pytest]
rp_issue_marks = xfail issue bug
```

#### `rp_issue_system_url`

**Description**: URL template for issue tracking system  
**Type**: String with `{issue_id}` placeholder  
**Default**: `''` (empty)  
**Example**: `'https://jira.company.com/browse/{issue_id}'`

```ini
[pytest]
rp_issue_system_url = https://jira.company.com/browse/{issue_id}
```

## Command Line Options

### `--reportportal`

**Description**: Enable ReportPortal plugin  
**Type**: Flag  
**Default**: Disabled

```bash
pytest --reportportal
```

### `--rp-launch`

**Description**: Override launch name  
**Type**: String  
**Overrides**: `rp_launch` config option

```bash
pytest --reportportal --rp-launch "Custom Launch Name"
```

### `--rp-launch-description`

**Description**: Override launch description  
**Type**: String  
**Overrides**: `rp_launch_description` config option

```bash
pytest --reportportal --rp-launch-description "Custom description"
```

### `--rp-log-level`

**Description**: Set logging level for automated reporting  
**Type**: String  
**Values**: `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`  
**Requires**: pytest >= 3.3

```bash
pytest --reportportal --rp-log-level DEBUG
```

## Environment Variables

You can use environment variables for configuration, especially useful in CI/CD:

### Standard Environment Variables

| Environment Variable | Configuration Option | Example |
|---------------------|---------------------|---------|
| `RP_UUID` | `rp_uuid` | `fb586627-32be-47dd-93c1-678873458a5f` |
| `RP_ENDPOINT` | `rp_endpoint` | `http://rp.company.com:8080` |
| `RP_PROJECT` | `rp_project` | `my_project` |
| `RP_LAUNCH` | `rp_launch` | `CI Build Tests` |
| `RP_LAUNCH_DESCRIPTION` | `rp_launch_description` | `Automated CI tests` |

### Usage Example

```bash
export RP_UUID="your-uuid-token"
export RP_ENDPOINT="http://reportportal.company.com:8080"
export RP_PROJECT="my_project"
export RP_LAUNCH="GitHub Actions Build"

pytest --reportportal
```

### CI/CD Integration

#### GitHub Actions

```yaml
env:
  RP_UUID: ${{ secrets.RP_UUID }}
  RP_ENDPOINT: ${{ secrets.RP_ENDPOINT }}
  RP_PROJECT: ${{ secrets.RP_PROJECT }}
  RP_LAUNCH: "GitHub Actions Build ${{ github.run_number }}"
```

#### Jenkins

```groovy
environment {
    RP_UUID = credentials('reportportal-uuid')
    RP_ENDPOINT = 'http://reportportal.company.com:8080'
    RP_PROJECT = 'my_project'
    RP_LAUNCH = "Jenkins Build ${BUILD_NUMBER}"
}
```

## Configuration Examples

### Minimal Configuration

```ini
# pytest.ini - Minimal setup
[pytest]
rp_uuid = fb586627-32be-47dd-93c1-678873458a5f
rp_endpoint = http://reportportal.local:8080
rp_project = demo_project
```

### Development Configuration

```ini
# pytest-dev.ini - Development environment
[pytest]
rp_uuid = dev-uuid-token
rp_endpoint = http://dev-reportportal:8080
rp_project = myproject_dev
rp_launch = Development Tests
rp_ignore_errors = True
rp_verify_ssl = False
rp_log_batch_size = 10
rp_ignore_tags = skip slow debug
```

### Production Configuration

```ini
# pytest-prod.ini - Production environment
[pytest]
rp_uuid = prod-uuid-token
rp_endpoint = https://reportportal.company.com
rp_project = myproject_prod
rp_launch = Production Smoke Tests
rp_launch_description = Critical path validation
rp_launch_tags = production smoke critical
rp_ignore_errors = False
rp_verify_ssl = True
rp_log_batch_size = 50
rp_hierarchy_dirs = True
rp_hierarchy_module = True
rp_hierarchy_class = True
rp_issue_marks = xfail bug
rp_issue_system_url = https://jira.company.com/browse/{issue_id}
```

### Complete Configuration

```ini
# pytest.ini - All available options
[pytest]
# Required settings
rp_uuid = fb586627-32be-47dd-93c1-678873458a5f
rp_endpoint = http://reportportal.company.com:8080
rp_project = comprehensive_project

# Launch settings
rp_launch = Comprehensive Test Suite
rp_launch_description = Complete test suite with all features
rp_launch_tags = comprehensive regression automated
rp_tests_tags = pytest automated

# Hierarchy settings
rp_hierarchy_dirs = True
rp_hierarchy_dirs_level = 0
rp_hierarchy_module = True
rp_hierarchy_class = True
rp_hierarchy_parametrize = False
rp_display_suite_test_file = True

# Logging settings
rp_log_batch_size = 30
rp_log_level = INFO

# Error handling
rp_ignore_errors = True
rp_verify_ssl = True

# Tag filtering
rp_ignore_tags = skip slow debug parametrize

# Issue tracking
rp_issue_marks = xfail issue bug defect
rp_issue_system_url = https://jira.company.com/browse/{issue_id}
```

## Advanced Configuration

### Multiple Environment Configuration

Create separate configuration files for different environments:

#### Structure

```
project/
├── pytest.ini              # Default configuration
├── pytest-dev.ini          # Development
├── pytest-staging.ini      # Staging
├── pytest-prod.ini         # Production
└── tests/
```

#### Usage

```bash
# Development
pytest -c pytest-dev.ini --reportportal

# Staging
pytest -c pytest-staging.ini --reportportal

# Production
pytest -c pytest-prod.ini --reportportal
```

### Dynamic Configuration

```python
# conftest.py - Dynamic configuration
import os
import pytest

def pytest_configure(config):
    """Configure ReportPortal based on environment."""
    
    # Get environment
    env = os.getenv('TEST_ENV', 'dev')
    
    # Set environment-specific values
    if env == 'prod':
        config.option.rp_launch = f"Production Tests - {os.getenv('BUILD_NUMBER', 'manual')}"
        config.option.rp_ignore_errors = False
    elif env == 'staging':
        config.option.rp_launch = f"Staging Tests - {os.getenv('BUILD_NUMBER', 'manual')}"
        config.option.rp_ignore_errors = True
    else:  # dev
        config.option.rp_launch = f"Development Tests - {os.getenv('USER', 'unknown')}"
        config.option.rp_ignore_errors = True
```

### Conditional Configuration

```python
# conftest.py - Conditional ReportPortal activation
import pytest

def pytest_configure(config):
    """Only enable ReportPortal in CI environment."""
    
    # Check if running in CI
    is_ci = any([
        os.getenv('CI'),
        os.getenv('GITHUB_ACTIONS'),
        os.getenv('JENKINS_URL'),
        os.getenv('GITLAB_CI')
    ])
    
    if not is_ci and not config.option.rp_enabled:
        # Disable ReportPortal for local development
        config.option.rp_enabled = False
        print("ReportPortal disabled - not running in CI")
```

### Configuration Validation

```python
# conftest.py - Configuration validation
import pytest
import sys

def pytest_configure(config):
    """Validate ReportPortal configuration."""
    
    if not config.option.rp_enabled:
        return
    
    # Required settings
    required = ['rp_uuid', 'rp_endpoint', 'rp_project']
    missing = []
    
    for setting in required:
        if not config.getini(setting):
            missing.append(setting)
    
    if missing:
        print(f"Error: Missing required ReportPortal settings: {missing}")
        print("Please configure these in pytest.ini or as environment variables")
        sys.exit(1)
    
    # Validate endpoint format
    endpoint = config.getini('rp_endpoint')
    if not (endpoint.startswith('http://') or endpoint.startswith('https://')):
        print(f"Error: Invalid endpoint format: {endpoint}")
        print("Endpoint must start with http:// or https://")
        sys.exit(1)
    
    # Validate batch size
    try:
        batch_size = int(config.getini('rp_log_batch_size'))
        if not 1 <= batch_size <= 100:
            print(f"Warning: Batch size {batch_size} outside recommended range (1-100)")
    except ValueError:
        print(f"Error: Invalid batch size: {config.getini('rp_log_batch_size')}")
        sys.exit(1)
    
    print("ReportPortal configuration validated successfully")
```

## Troubleshooting Configuration

### Common Configuration Issues

#### 1. Missing Required Settings

**Error**: Plugin not starting or tests not reported

**Solution**: Verify all required settings are present:

```bash
# Check configuration
pytest --reportportal --collect-only -v
```

**Debug script**:
```python
# debug_config.py
import pytest
from _pytest.config import get_config

config = get_config()
config.parse([])

required = ['rp_uuid', 'rp_endpoint', 'rp_project']
for setting in required:
    value = config.getini(setting)
    print(f"{setting}: {'✓' if value else '✗'} {value}")
```

#### 2. Connection Issues

**Symptoms**: Connection timeouts, SSL errors

**Solutions**:

```ini
[pytest]
# Disable SSL verification for development
rp_verify_ssl = False

# Ignore connection errors
rp_ignore_errors = True

# Reduce batch size for unstable connections
rp_log_batch_size = 5
```

#### 3. Performance Issues

**Symptoms**: Slow test execution, memory usage

**Solutions**:

```ini
[pytest]
# Increase batch size for better performance
rp_log_batch_size = 50

# Reduce hierarchy levels
rp_hierarchy_dirs = False
rp_hierarchy_parametrize = False

# Filter verbose markers
rp_ignore_tags = debug verbose parametrize
```

#### 4. Tag Filtering Issues

**Problem**: Too many or unwanted tags in ReportPortal

**Solution**:

```ini
[pytest]
# Filter out common pytest markers
rp_ignore_tags = skip skipif xfail parametrize usefixtures

# Or be more specific
rp_ignore_tags = slow integration manual
```

### Configuration Testing

#### Test Configuration Script

```python
# test_rp_config.py
"""Test ReportPortal configuration."""

import pytest
from pytest_reportportal.service import PyTestServiceClass

def test_reportportal_configuration():
    """Test that ReportPortal is properly configured."""
    
    # This will only run if ReportPortal is enabled
    if not pytest.config.option.rp_enabled:
        pytest.skip("ReportPortal not enabled")
    
    # Get service instance
    service = PyTestServiceClass()
    
    # Test service initialization
    assert service is not None
    print(f"Service instance: {service}")
    
    # Test configuration values
    config = pytest.config
    
    uuid = config.getini('rp_uuid')
    endpoint = config.getini('rp_endpoint')
    project = config.getini('rp_project')
    
    assert uuid, "rp_uuid not configured"
    assert endpoint, "rp_endpoint not configured"
    assert project, "rp_project not configured"
    
    print(f"UUID: {uuid[:8]}...")  # Partial for security
    print(f"Endpoint: {endpoint}")
    print(f"Project: {project}")
    
    # Test optional settings
    launch = config.getini('rp_launch')
    batch_size = config.getini('rp_log_batch_size')
    
    print(f"Launch: {launch}")
    print(f"Batch size: {batch_size}")
    
    assert True, "Configuration test passed"

if __name__ == "__main__":
    pytest.main([__file__, "-v", "--reportportal"])
```

#### Run Configuration Test

```bash
# Test configuration
python test_rp_config.py

# Or with pytest
pytest test_rp_config.py -v --reportportal
```

### Configuration Best Practices

1. **Use separate configurations for different environments**
2. **Store sensitive data (UUID) in environment variables**
3. **Use meaningful launch names with build/version info**
4. **Enable error ignoring for development, disable for CI/CD**
5. **Adjust batch size based on network conditions**
6. **Filter unnecessary tags to reduce noise**
7. **Use hierarchy features judiciously (impacts performance)**
8. **Validate configuration in CI/CD pipelines**

This comprehensive configuration reference should help you properly set up and customize the pytest-reportportal plugin for your specific needs and environments.