# pytest-reportportal Documentation

Welcome to the comprehensive documentation for the pytest-reportportal plugin. This plugin enables seamless integration between pytest test framework and ReportPortal test management platform.

## Overview

The pytest-reportportal plugin allows you to:
- Report pytest test results to ReportPortal in real-time
- Organize tests hierarchically by directories, modules, and classes
- Attach files, logs, and screenshots to test results
- Track test execution history and trends
- Integrate with CI/CD pipelines for automated reporting

## Quick Start

### Installation

```bash
pip install pytest-reportportal
```

### Basic Configuration

Create a `pytest.ini` file in your project root:

```ini
[pytest]
rp_uuid = your-reportportal-uuid
rp_endpoint = http://your-reportportal-server:8080
rp_project = your_project_name
```

### Run Tests

```bash
pytest --reportportal
```

## Documentation Structure

This documentation is organized into several comprehensive guides:

### 📚 Core Documentation

#### [API Reference](API_REFERENCE.md)
Complete API documentation covering all public classes, methods, and functions with detailed parameters, return values, and examples.

**Contents:**
- Public API Classes (RPLogger, RPLogHandler, PyTestServiceClass, RPReportListener)
- Public Functions (timestamp, trim_docstring, patching_logger_class, etc.)
- Configuration Options
- Pytest Hooks
- Usage Examples

#### [Class Reference](CLASS_REFERENCE.md)
In-depth documentation for all public classes including inheritance relationships, usage patterns, and advanced examples.

**Contents:**
- RPLogger - Custom logger with ReportPortal attachment support
- RPLogHandler - Logging handler for ReportPortal integration
- PyTestServiceClass - Core service managing ReportPortal communication
- RPReportListener - Event listener coordinating with pytest hooks
- Singleton - Metaclass ensuring single service instance
- Class relationships and interaction patterns

#### [Function Reference](FUNCTION_REFERENCE.md)
Detailed documentation for all public functions with comprehensive examples and usage patterns.

**Contents:**
- Utility Functions (timestamp, trim_docstring)
- Plugin Hook Functions (is_master, wait_launch, pytest_addoption)
- Service Functions (async_error_handler, get_issue_types)
- Logging Functions (patching_logger_class)
- Helper Functions (_get_item_name, _get_item_tags, _get_parameters)

#### [Usage Examples](USAGE_EXAMPLES.md)
Comprehensive tutorials and examples covering everything from basic setup to advanced integration scenarios.

**Contents:**
- Quick Start Guide
- Configuration Examples (minimal, complete, environment-specific)
- Basic Usage Examples (simple tests, logging, markers)
- Advanced Usage Examples (attachments, parametrized tests, error handling)
- Integration Examples (Selenium, Database, REST API testing)
- Troubleshooting Examples
- Best Practices

#### [Configuration Reference](CONFIGURATION_REFERENCE.md)
Complete configuration guide covering all available options, environment variables, and advanced configuration scenarios.

**Contents:**
- Required and optional configuration options
- Command line arguments and environment variables
- Configuration examples for different environments
- Advanced configuration patterns
- Troubleshooting configuration issues

## Key Features

### 🔄 Real-time Test Reporting

Tests are reported to ReportPortal in real-time as they execute, providing immediate visibility into test progress and results.

```python
def test_example(rp_logger):
    rp_logger.info("Starting test execution")
    # Test logic here
    assert True
    rp_logger.info("Test completed successfully")
```

### 📁 Hierarchical Test Organization

Organize your tests in a clear hierarchy based on:
- **Directories**: Group tests by feature areas
- **Modules**: Organize by test files
- **Classes**: Group related test methods
- **Parametrized Tests**: Handle test variations

```ini
[pytest]
rp_hierarchy_dirs = True
rp_hierarchy_module = True  
rp_hierarchy_class = True
rp_hierarchy_parametrize = False
```

### 📎 Rich Attachments Support

Attach various file types to your test results:

```python
# Text attachment
rp_logger.info(
    "Configuration loaded",
    attachment={
        "name": "config.txt",
        "data": "debug=true\ntimeout=30",
        "mime": "text/plain"
    }
)

# JSON attachment
rp_logger.info(
    "Test results",
    attachment={
        "name": "results.json", 
        "data": json.dumps({"passed": 15, "failed": 2}),
        "mime": "application/json"
    }
)

# Screenshot attachment (base64 encoded)
rp_logger.info(
    "Screenshot captured",
    attachment={
        "name": "screenshot.png",
        "data": base64_encoded_image,
        "mime": "image/png"
    }
)
```

### 🏷️ Flexible Tagging System

Use pytest markers as tags for better test organization:

```python
@pytest.mark.smoke
@pytest.mark.api
@pytest.mark.critical
def test_api_endpoint():
    """Test critical API functionality."""
    pass
```

### 🔍 Issue Tracking Integration

Link test failures to issue tracking systems:

```python
@pytest.mark.issue(issue_id="BUG-123", reason="Known API bug", issue_type="PB")
def test_with_known_issue():
    """Test that has a known issue."""
    assert False, "This test fails due to known bug"
```

### ⚙️ Extensive Configuration Options

Configure every aspect of the integration:

```ini
[pytest]
# Required settings
rp_uuid = your-uuid-token
rp_endpoint = http://reportportal.server:8080
rp_project = project_name

# Launch settings
rp_launch = Launch Name
rp_launch_tags = tag1 tag2 tag3
rp_launch_description = Launch description

# Hierarchy settings
rp_hierarchy_dirs = True
rp_hierarchy_module = True
rp_hierarchy_class = True

# Logging settings
rp_log_batch_size = 20
rp_ignore_errors = True
rp_ignore_tags = skip slow

# Issue tracking
rp_issue_marks = xfail bug
rp_issue_system_url = https://jira.company.com/browse/{issue_id}
```

## Architecture Overview

The plugin consists of several key components working together:

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   pytest       │    │ pytest-          │    │ ReportPortal    │
│   Framework     │◄──►│ reportportal     │◄──►│ Server          │
│                 │    │ Plugin           │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │ Plugin Components│
                    └──────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ RPLogger    │    │ PyTestService   │    │ RPReportListener│
│ RPLogHandler│    │ Class           │    │                 │
└─────────────┘    └─────────────────┘    └─────────────────┘
```

### Component Responsibilities

- **RPLogger/RPLogHandler**: Handle logging with ReportPortal attachments
- **PyTestServiceClass**: Core service managing ReportPortal API communication
- **RPReportListener**: Coordinates with pytest hooks for test lifecycle events
- **Plugin Hooks**: Integrate with pytest's execution flow

## Getting Started

### 1. Prerequisites

- Python 3.6+
- pytest 3.0.7+
- Access to a ReportPortal server instance

### 2. Installation

```bash
# Install from PyPI
pip install pytest-reportportal

# Or install development version
pip install git+https://github.com/reportportal/agent-python-pytest.git
```

### 3. Configuration

Create `pytest.ini` in your project root:

```ini
[pytest]
# Required configuration
rp_uuid = your-reportportal-uuid-token
rp_endpoint = http://your-reportportal-server:8080
rp_project = your_project_name

# Optional configuration
rp_launch = My Test Launch
rp_launch_description = Automated test execution
rp_launch_tags = automated regression
```

### 4. Basic Usage

```python
# conftest.py
import pytest
import logging
from pytest_reportportal import RPLogger, RPLogHandler

@pytest.fixture(scope="session")
def rp_logger(request):
    logging.setLoggerClass(RPLogger)
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.DEBUG)
    
    rp_handler = RPLogHandler(request.node.config.py_test_service)
    rp_handler.setLevel(logging.INFO)
    logger.addHandler(rp_handler)
    
    return logger
```

```python
# test_example.py
def test_basic_functionality(rp_logger):
    """Test basic application functionality."""
    rp_logger.info("Starting basic functionality test")
    
    # Test logic
    result = 2 + 2
    rp_logger.info(f"Calculation result: {result}")
    
    assert result == 4
    rp_logger.info("Test passed successfully")
```

### 5. Run Tests

```bash
# Run tests with ReportPortal reporting
pytest --reportportal

# Run with custom launch name
pytest --reportportal --rp-launch "Custom Launch Name"

# Run with custom description
pytest --reportportal --rp-launch-description "Custom test run"
```

## Advanced Features

### Distributed Testing Support

The plugin supports pytest-xdist for parallel test execution:

```bash
# Run tests in parallel with ReportPortal
pytest --reportportal -n 4
```

### CI/CD Integration

#### GitHub Actions Example

```yaml
name: Tests with ReportPortal
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.8'
    
    - name: Install dependencies
      run: |
        pip install pytest pytest-reportportal
        pip install -r requirements.txt
    
    - name: Run tests
      env:
        RP_UUID: ${{ secrets.RP_UUID }}
        RP_ENDPOINT: ${{ secrets.RP_ENDPOINT }}
        RP_PROJECT: ${{ secrets.RP_PROJECT }}
      run: |
        pytest --reportportal \
               --rp-launch "GitHub Actions Build ${{ github.run_number }}" \
               --rp-launch-description "Automated tests for commit ${{ github.sha }}"
```

#### Jenkins Pipeline Example

```groovy
pipeline {
    agent any
    
    environment {
        RP_UUID = credentials('reportportal-uuid')
        RP_ENDPOINT = 'http://reportportal.company.com:8080'
        RP_PROJECT = 'my_project'
    }
    
    stages {
        stage('Test') {
            steps {
                sh '''
                    pytest --reportportal \
                           --rp-launch "Jenkins Build ${BUILD_NUMBER}" \
                           --rp-launch-description "Automated tests for build ${BUILD_NUMBER}"
                '''
            }
        }
    }
}
```

### Custom Extensions

You can extend the plugin with custom functionality:

```python
# Custom ReportPortal integration
from pytest_reportportal.service import PyTestServiceClass
from pytest_reportportal.listener import RPReportListener

class CustomRPReportListener(RPReportListener):
    """Extended ReportPortal listener with custom features."""
    
    def pytest_runtest_makereport(self, item):
        # Custom report processing
        report = super().pytest_runtest_makereport(item)
        
        # Add custom metadata
        if hasattr(item, 'custom_metadata'):
            self.PyTestService.post_log(
                f"Custom metadata: {item.custom_metadata}",
                loglevel='INFO'
            )
        
        return report
```

## Troubleshooting

### Common Issues

#### 1. Connection Issues

```ini
[pytest]
# Disable SSL verification if needed
rp_verify_ssl = False

# Ignore connection errors during development
rp_ignore_errors = True
```

#### 2. Large Log Volumes

```ini
[pytest]
# Increase batch size for better performance
rp_log_batch_size = 50

# Filter out verbose markers
rp_ignore_tags = debug verbose parametrize
```

#### 3. Timeout Issues

```python
# Custom timeout handling
from pytest_reportportal.plugin import wait_launch

def pytest_sessionstart(session):
    if is_master(session.config):
        service.start_launch("My Launch")
        
        try:
            wait_launch(service.RP.rp_client)
        except Exception as e:
            print(f"Launch wait timeout: {e}")
            # Continue anyway
```

### Debug Mode

Enable debug logging to troubleshoot issues:

```python
import logging
logging.basicConfig(level=logging.DEBUG)

# Or in pytest.ini
[pytest]
log_cli = true
log_cli_level = DEBUG
```

## Contributing

We welcome contributions! Please see the [Contributing Guide](../CONTRIBUTING.rst) for details on:
- Setting up development environment
- Running tests
- Submitting pull requests
- Code style guidelines

## Support

- **Documentation**: This comprehensive guide and API reference
- **Issues**: Report bugs and feature requests on [GitHub Issues](https://github.com/reportportal/agent-python-pytest/issues)
- **Community**: Join discussions in the ReportPortal community
- **Email**: Contact the maintainers at SupportEPMC-TSTReportPortal@epam.com

## License

This project is licensed under the Apache 2.0 License - see the [LICENSE](../LICENSE) file for details.

## Version History

- **1.0.5**: Current stable version
- **1.0.4**: Previous stable release
- **1.0.3**: Legacy version

## Related Projects

- [ReportPortal](https://reportportal.io/) - Main ReportPortal platform
- [reportportal-client](https://github.com/reportportal/client-Python) - Python client library
- [pytest](https://pytest.org/) - Python testing framework

---

## Quick Navigation

| Document | Description |
|----------|-------------|
| [API Reference](API_REFERENCE.md) | Complete API documentation with all classes, methods, and functions |
| [Class Reference](CLASS_REFERENCE.md) | Detailed class documentation with inheritance and relationships |
| [Function Reference](FUNCTION_REFERENCE.md) | Comprehensive function documentation with examples |
| [Usage Examples](USAGE_EXAMPLES.md) | Tutorials and examples from basic to advanced usage |
| [Configuration Reference](CONFIGURATION_REFERENCE.md) | Complete configuration guide with all options and examples |

**Happy Testing with pytest-reportportal! 🚀**