# pytest-reportportal API Reference

This document provides comprehensive API documentation for the pytest-reportportal plugin, which enables reporting of pytest test results to ReportPortal.

## Table of Contents

- [Public API Classes](#public-api-classes)
- [Public Functions](#public-functions)  
- [Configuration Options](#configuration-options)
- [Pytest Hooks](#pytest-hooks)
- [Usage Examples](#usage-examples)

## Public API Classes

### RPLogger

**Module**: `pytest_reportportal.rp_logging`  
**Inherits**: `logging.getLoggerClass()`

A custom logger class that extends Python's standard logging functionality to support ReportPortal-specific features like attachments.

#### Constructor

```python
RPLogger(name, level=0)
```

**Parameters**:
- `name` (str): The logger name
- `level` (int): The logging level (default: 0)

#### Methods

##### `_log(level, msg, args, exc_info=None, extra=None, stack_info=False, attachment=None)`

Low-level logging routine that creates a LogRecord and handles it through all registered handlers.

**Parameters**:
- `level` (int): Logging level
- `msg` (str): Log message
- `args` (tuple): Arguments for message formatting
- `exc_info` (tuple, optional): Exception information
- `extra` (dict, optional): Extra context information
- `stack_info` (bool): Include stack information (default: False)
- `attachment` (dict, optional): File attachment for ReportPortal

**Example**:
```python
import logging
from pytest_reportportal import RPLogger

# Set up RPLogger
logging.setLoggerClass(RPLogger)
logger = logging.getLogger(__name__)

# Log with attachment
logger.info(
    "Test completed with screenshot",
    attachment={
        "name": "screenshot.png",
        "data": screenshot_data,
        "mime": "image/png"
    }
)
```

---

### RPLogHandler

**Module**: `pytest_reportportal.rp_logging`  
**Inherits**: `logging.Handler`

A logging handler that sends log records to ReportPortal.

#### Constructor

```python
RPLogHandler(py_test_service, level=logging.NOTSET, filter_reportportal_client_logs=False, endpoint=None)
```

**Parameters**:
- `py_test_service` (PyTestServiceClass): The pytest service instance
- `level` (int): Minimum logging level to handle (default: NOTSET)
- `filter_reportportal_client_logs` (bool): Filter out ReportPortal client logs (default: False)
- `endpoint` (str, optional): ReportPortal endpoint URL for filtering

#### Class Attributes

- `_loglevel_map` (dict): Maps Python logging levels to ReportPortal level names
- `_sorted_levelnos` (list): Sorted list of logging level numbers

#### Methods

##### `filter(record)`

Determines whether a log record should be processed.

**Parameters**:
- `record` (LogRecord): The log record to filter

**Returns**: `bool` - True if record should be processed

##### `emit(record)`

Processes and sends a log record to ReportPortal.

**Parameters**:
- `record` (LogRecord): The log record to emit

**Example**:
```python
import logging
from pytest_reportportal import RPLogHandler

# Create handler
handler = RPLogHandler(
    py_test_service=service_instance,
    level=logging.INFO,
    filter_reportportal_client_logs=True
)

# Add to logger
logger = logging.getLogger()
logger.addHandler(handler)
```

---

### PyTestServiceClass

**Module**: `pytest_reportportal.service`  
**Metaclass**: `Singleton`

The main service class that handles communication with ReportPortal. This class is implemented as a singleton to ensure only one instance exists per pytest session.

#### Constructor

```python
PyTestServiceClass()
```

Initializes the service with default values. No parameters required due to singleton pattern.

#### Attributes

- `RP`: ReportPortal client instance
- `RP_SUPPORTS_PARAMETERS` (bool): Whether the client supports test parameters
- `ignore_errors` (bool): Whether to ignore ReportPortal errors
- `ignored_tags` (list): List of pytest markers to ignore
- `issue_types` (dict): Available issue types from ReportPortal project
- `project_settiings` (dict): ReportPortal project settings

#### Methods

##### `init_service(endpoint, project, uuid, log_batch_size, ignore_errors, ignored_tags, verify_ssl=True)`

Initializes the ReportPortal service connection.

**Parameters**:
- `endpoint` (str): ReportPortal server URL
- `project` (str): Project name in ReportPortal
- `uuid` (str): User authentication token
- `log_batch_size` (int): Size of log batch requests
- `ignore_errors` (bool): Whether to ignore connection errors
- `ignored_tags` (list): Pytest markers to ignore
- `verify_ssl` (bool): Verify SSL certificates (default: True)

**Returns**: ReportPortal service instance

##### `start_launch(launch_name, mode=None, tags=None, description=None)`

Starts a new test launch in ReportPortal.

**Parameters**:
- `launch_name` (str): Name of the test launch
- `mode` (str, optional): Launch mode
- `tags` (list, optional): Launch tags
- `description` (str, optional): Launch description

##### `start_pytest_item(test_item)`

Starts reporting for a pytest test item.

**Parameters**:
- `test_item`: Pytest test item object

##### `finish_pytest_item(test_item, status, issue=None)`

Finishes reporting for a pytest test item.

**Parameters**:
- `test_item`: Pytest test item object
- `status` (str): Test result status ('PASSED', 'FAILED', 'SKIPPED')
- `issue` (dict, optional): Issue information for failed tests

##### `finish_launch(launch=None, status='rp_launch')`

Finishes the test launch.

**Parameters**:
- `launch` (optional): Launch identifier
- `status` (str): Launch status (default: 'rp_launch')

##### `post_log(message, loglevel='INFO', attachment=None)`

Posts a log message to ReportPortal.

**Parameters**:
- `message` (str): Log message
- `loglevel` (str): Log level ('TRACE', 'DEBUG', 'INFO', 'WARN', 'ERROR')
- `attachment` (dict, optional): File attachment

**Example**:
```python
from pytest_reportportal.service import PyTestServiceClass

# Get service instance (singleton)
service = PyTestServiceClass()

# Initialize service
service.init_service(
    endpoint='http://reportportal.example.com:8080',
    project='my_project',
    uuid='your-uuid-token',
    log_batch_size=20,
    ignore_errors=True,
    ignored_tags=['skip', 'parametrize']
)

# Start launch
service.start_launch(
    launch_name='My Test Launch',
    tags=['regression', 'api'],
    description='Automated regression tests'
)

# Post log message
service.post_log(
    message='Test step completed successfully',
    loglevel='INFO'
)
```

---

### RPReportListener

**Module**: `pytest_reportportal.listener`

Event listener class that handles pytest hooks and coordinates with ReportPortal service.

#### Constructor

```python
RPReportListener(py_test_service, log_level=logging.NOTSET, endpoint=None)
```

**Parameters**:
- `py_test_service` (PyTestServiceClass): Service instance
- `log_level` (int): Logging level (default: NOTSET)
- `endpoint` (str, optional): ReportPortal endpoint URL

#### Attributes

- `PyTestService`: Reference to the pytest service
- `result` (str): Current test result
- `issue` (dict): Issue information for failed tests

#### Methods

##### `pytest_runtest_protocol(item)` (Hook)

Pytest hook that wraps test execution with ReportPortal reporting.

**Parameters**:
- `item`: Pytest test item

##### `pytest_runtest_makereport(item)` (Hook)

Pytest hook that processes test reports and extracts results.

**Parameters**:
- `item`: Pytest test item

---

## Public Functions

### `timestamp()`

**Module**: `pytest_reportportal.service`

Generates a timestamp string for ReportPortal API calls.

**Returns**: `str` - Timestamp in milliseconds since epoch

**Example**:
```python
from pytest_reportportal.service import timestamp

current_time = timestamp()
print(current_time)  # Output: "1640995200000"
```

---

### `trim_docstring(docstring)`

**Module**: `pytest_reportportal.service`

Trims and formats docstrings according to PEP 257 conventions.

**Parameters**:
- `docstring` (str): The docstring to trim

**Returns**: `str` - Formatted docstring

**Example**:
```python
from pytest_reportportal.service import trim_docstring

def example_function():
    """
    This is an example function
    with a multi-line docstring.
    
    It has multiple paragraphs.
    """
    pass

trimmed = trim_docstring(example_function.__doc__)
print(trimmed)
# Output: "This is an example function\nwith a multi-line docstring.\n\nIt has multiple paragraphs."
```

---

### `patching_logger_class()`

**Module**: `pytest_reportportal.rp_logging`

Context manager that temporarily patches the logging class to support ReportPortal attachments.

**Returns**: Context manager

**Example**:
```python
from pytest_reportportal.rp_logging import patching_logger_class
import logging

with patching_logger_class():
    logger = logging.getLogger()
    logger.info("Message with attachment", attachment={"name": "file.txt", "data": "content"})
```

---

### `is_master(config)`

**Module**: `pytest_reportportal.plugin`

Determines if the current pytest process is a master node (not running under xdist).

**Parameters**:
- `config`: Pytest configuration object

**Returns**: `bool` - True if master node or not using xdist

---

### `wait_launch(rp_client)`

**Module**: `pytest_reportportal.plugin`

Waits for a ReportPortal launch to be created, with timeout.

**Parameters**:
- `rp_client`: ReportPortal client instance

**Raises**: `Exception` if launch is not found within timeout

---

## Configuration Options

The plugin supports extensive configuration through pytest.ini file or command line options:

### Required Configuration

- `rp_uuid`: User authentication token
- `rp_project`: Project name in ReportPortal
- `rp_endpoint`: ReportPortal server URL

### Optional Configuration

- `rp_launch`: Launch name (default: 'Pytest Launch')
- `rp_launch_tags`: Launch tags
- `rp_launch_description`: Launch description
- `rp_log_batch_size`: Log batch size (default: 20)
- `rp_ignore_errors`: Ignore ReportPortal errors (default: False)
- `rp_ignore_tags`: Pytest markers to ignore
- `rp_hierarchy_dirs`: Enable directory hierarchy (default: False)
- `rp_hierarchy_module`: Enable module hierarchy (default: True)
- `rp_hierarchy_class`: Enable class hierarchy (default: True)
- `rp_hierarchy_parametrize`: Enable parametrized test hierarchy (default: False)
- `rp_verify_ssl`: Verify SSL certificates (default: True)

## Pytest Hooks

The plugin implements several pytest hooks:

- `pytest_addoption`: Adds command line options
- `pytest_configure`: Configures the plugin
- `pytest_configure_node`: Configures xdist nodes
- `pytest_sessionstart`: Starts ReportPortal session
- `pytest_collection_modifyitems`: Modifies test collection
- `pytest_collection_finish`: Finishes test collection
- `pytest_sessionfinish`: Finishes ReportPortal session
- `pytest_unconfigure`: Unconfigures the plugin

## Usage Examples

### Basic Setup

```python
# conftest.py
import pytest
import logging
from pytest_reportportal import RPLogger, RPLogHandler

@pytest.fixture(scope="session")
def rp_logger(request):
    # Set up ReportPortal logger
    logging.setLoggerClass(RPLogger)
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.DEBUG)
    
    # Create ReportPortal handler
    rp_handler = RPLogHandler(request.node.config.py_test_service)
    rp_handler.setLevel(logging.INFO)
    
    return logger
```

### Test with Logging

```python
# test_example.py
def test_with_logging(rp_logger):
    """Test that demonstrates ReportPortal logging."""
    rp_logger.info("Starting test execution")
    
    # Test logic here
    result = 2 + 2
    rp_logger.info(f"Calculation result: {result}")
    
    assert result == 4
    rp_logger.info("Test completed successfully")
```

### Test with Attachments

```python
def test_with_attachment(rp_logger):
    """Test that includes file attachments."""
    import subprocess
    
    # Get system information
    system_info = subprocess.check_output("uname -a", shell=True)
    
    # Log with attachment
    rp_logger.info(
        "System information collected",
        attachment={
            "name": "system_info.txt",
            "data": system_info,
            "mime": "text/plain"
        }
    )
    
    assert True
```

### Configuration File Example

```ini
# pytest.ini
[pytest]
rp_uuid = your-uuid-token
rp_endpoint = http://reportportal.example.com:8080
rp_project = my_project
rp_launch = My Test Launch
rp_launch_tags = regression automated
rp_launch_description = Automated regression test suite
rp_ignore_errors = True
rp_ignore_tags = skip parametrize
rp_hierarchy_dirs = True
rp_hierarchy_module = True
rp_hierarchy_class = True
```

For more detailed examples and advanced usage, see the [Usage Examples](USAGE_EXAMPLES.md) documentation.