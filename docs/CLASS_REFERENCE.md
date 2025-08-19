# pytest-reportportal Class Reference

This document provides detailed documentation for all public classes in the pytest-reportportal plugin.

## Table of Contents

- [RPLogger](#rplogger)
- [RPLogHandler](#rploghandler)
- [PyTestServiceClass](#pytestserviceclass)
- [RPReportListener](#rpreportlistener)
- [Singleton](#singleton)
- [Class Relationships](#class-relationships)
- [Usage Patterns](#usage-patterns)

## RPLogger

**Module**: `pytest_reportportal.rp_logging`  
**Inherits**: `logging.getLoggerClass()`

### Overview

`RPLogger` is a custom logger class that extends Python's standard logging functionality to support ReportPortal-specific features, particularly file attachments. It maintains compatibility with the standard logging interface while adding ReportPortal-specific capabilities.

### Class Definition

```python
class RPLogger(logging.getLoggerClass()):
    def __init__(self, name, level=0):
        super(RPLogger, self).__init__(name, level=level)
```

### Constructor Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | str | Required | The logger name, typically `__name__` |
| `level` | int | 0 | The minimum logging level |

### Methods

#### `_log(level, msg, args, exc_info=None, extra=None, stack_info=False, attachment=None)`

The core logging method that handles ReportPortal-specific functionality.

**Parameters**:
- `level` (int): Logging level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
- `msg` (str): The log message template
- `args` (tuple): Arguments for message formatting
- `exc_info` (tuple, optional): Exception information tuple
- `extra` (dict, optional): Additional context information
- `stack_info` (bool): Whether to include stack trace information
- `attachment` (dict, optional): ReportPortal attachment specification

**Attachment Format**:
```python
attachment = {
    "name": "filename.ext",      # File name for the attachment
    "data": file_content,        # File content (string or bytes)
    "mime": "mime/type"          # MIME type of the file
}
```

### Usage Examples

#### Basic Usage

```python
import logging
from pytest_reportportal import RPLogger

# Set RPLogger as the default logger class
logging.setLoggerClass(RPLogger)

# Create logger instance
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# Standard logging works as expected
logger.info("This is a standard log message")
logger.error("This is an error message")
```

#### Logging with Attachments

```python
import json

# Log with JSON attachment
test_data = {"test_id": "TC001", "status": "passed", "duration": 1.5}
logger.info(
    "Test completed successfully",
    attachment={
        "name": "test_result.json",
        "data": json.dumps(test_data, indent=2),
        "mime": "application/json"
    }
)

# Log with text attachment
log_content = "Detailed execution log..."
logger.info(
    "Execution details attached",
    attachment={
        "name": "execution_log.txt",
        "data": log_content,
        "mime": "text/plain"
    }
)
```

#### Advanced Usage with Context

```python
import traceback

def test_function_with_logging():
    logger = logging.getLogger(__name__)
    
    try:
        logger.info("Starting test execution")
        
        # Test logic here
        result = perform_complex_operation()
        
        logger.info(
            "Operation completed",
            extra={"operation_id": "OP123", "duration": 2.3}
        )
        
        return result
        
    except Exception as e:
        logger.error(
            "Test failed with exception",
            exc_info=True,
            attachment={
                "name": "error_traceback.txt",
                "data": traceback.format_exc(),
                "mime": "text/plain"
            }
        )
        raise
```

---

## RPLogHandler

**Module**: `pytest_reportportal.rp_logging`  
**Inherits**: `logging.Handler`

### Overview

`RPLogHandler` is a custom logging handler that sends log records to ReportPortal. It maps Python logging levels to ReportPortal levels and handles the transmission of log messages and attachments.

### Class Definition

```python
class RPLogHandler(logging.Handler):
    _loglevel_map = {
        logging.NOTSET: 'TRACE',
        logging.DEBUG: 'DEBUG', 
        logging.INFO: 'INFO',
        logging.WARNING: 'WARN',
        logging.ERROR: 'ERROR',
        logging.CRITICAL: 'ERROR',
    }
```

### Constructor Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `py_test_service` | PyTestServiceClass | Required | The pytest service instance |
| `level` | int | `logging.NOTSET` | Minimum level to handle |
| `filter_reportportal_client_logs` | bool | False | Filter RP client logs |
| `endpoint` | str | None | ReportPortal endpoint for filtering |

### Attributes

#### Class Attributes

- `_loglevel_map` (dict): Maps Python logging levels to ReportPortal level names
- `_sorted_levelnos` (list): Sorted logging levels for efficient lookup

#### Instance Attributes

- `py_test_service`: Reference to the ReportPortal service
- `filter_reportportal_client_logs`: Whether to filter ReportPortal client logs
- `ignored_record_names`: Tuple of logger names to ignore
- `endpoint`: ReportPortal endpoint URL

### Methods

#### `filter(record)`

Determines whether a log record should be processed based on filtering rules.

**Parameters**:
- `record` (LogRecord): The log record to evaluate

**Returns**: `bool` - True if the record should be processed

**Filtering Logic**:
1. If filtering is disabled, all records pass
2. Records from ReportPortal client loggers are filtered out
3. urllib3 connection pool logs to the ReportPortal endpoint are filtered

#### `emit(record)`

Processes a log record and sends it to ReportPortal.

**Parameters**:
- `record` (LogRecord): The log record to emit

**Process**:
1. Formats the log message
2. Maps the logging level to ReportPortal level
3. Extracts attachment information if present
4. Sends the log to ReportPortal via the service

### Usage Examples

#### Basic Handler Setup

```python
import logging
from pytest_reportportal import RPLogHandler
from pytest_reportportal.service import PyTestServiceClass

# Get service instance
service = PyTestServiceClass()

# Create handler
handler = RPLogHandler(
    py_test_service=service,
    level=logging.INFO
)

# Configure logger
logger = logging.getLogger()
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Use logger normally
logger.info("This message will be sent to ReportPortal")
```

#### Advanced Handler Configuration

```python
import logging
from pytest_reportportal import RPLogHandler

class CustomRPLogHandler(RPLogHandler):
    """Custom ReportPortal handler with additional filtering."""
    
    def __init__(self, py_test_service, **kwargs):
        super().__init__(py_test_service, **kwargs)
        self.custom_filters = ['selenium', 'urllib3']
    
    def filter(self, record):
        # Apply parent filtering first
        if not super().filter(record):
            return False
        
        # Apply custom filtering
        for filter_name in self.custom_filters:
            if record.name.startswith(filter_name):
                return False
        
        return True
    
    def emit(self, record):
        # Add custom metadata
        if hasattr(record, 'extra'):
            record.extra['handler_version'] = '1.0'
        
        super().emit(record)

# Use custom handler
handler = CustomRPLogHandler(
    py_test_service=service,
    level=logging.DEBUG,
    filter_reportportal_client_logs=True,
    endpoint='http://reportportal.example.com'
)
```

#### Integration with pytest Fixtures

```python
import pytest
import logging
from pytest_reportportal import RPLogger, RPLogHandler

@pytest.fixture(scope="session")
def rp_logger(request):
    """Create ReportPortal logger for the session."""
    # Set logger class
    logging.setLoggerClass(RPLogger)
    
    # Create logger
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.DEBUG)
    
    # Create ReportPortal handler
    rp_handler = RPLogHandler(
        py_test_service=request.node.config.py_test_service,
        level=logging.INFO,
        filter_reportportal_client_logs=True
    )
    
    # Add custom formatter
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    rp_handler.setFormatter(formatter)
    
    logger.addHandler(rp_handler)
    
    return logger

def test_with_custom_handler(rp_logger):
    """Test using custom ReportPortal handler."""
    rp_logger.info("Test started with custom handler")
    
    # Test logic here
    assert True
    
    rp_logger.info("Test completed successfully")
```

---

## PyTestServiceClass

**Module**: `pytest_reportportal.service`  
**Metaclass**: `Singleton`

### Overview

`PyTestServiceClass` is the core service class that manages communication with ReportPortal. It implements the Singleton pattern to ensure only one instance exists per pytest session and handles all ReportPortal API interactions.

### Class Definition

```python
class PyTestServiceClass(with_metaclass(Singleton, object)):
    def __init__(self):
        self.RP = None
        self.RP_SUPPORTS_PARAMETERS = True  # Based on client version
        self.ignore_errors = True
        self.ignored_tags = []
        # ... other initialization
```

### Attributes

#### Core Attributes

- `RP`: ReportPortal client instance (`ReportPortalServiceAsync`)
- `RP_SUPPORTS_PARAMETERS` (bool): Whether the client supports test parameters
- `ignore_errors` (bool): Whether to ignore ReportPortal connection errors
- `ignored_tags` (list): List of pytest markers to ignore
- `issue_types` (dict): Available issue types from ReportPortal project
- `project_settiings` (dict): ReportPortal project settings

#### Internal Attributes

- `_errors` (Queue): Queue for handling async errors
- `_loglevels` (tuple): Supported log levels
- `_hier_parts` (dict): Hierarchy parts tracking
- `_item_parts` (dict): Test item parts mapping

### Methods

#### `init_service(endpoint, project, uuid, log_batch_size, ignore_errors, ignored_tags, verify_ssl=True)`

Initializes the ReportPortal service connection and configuration.

**Parameters**:
- `endpoint` (str): ReportPortal server URL (e.g., 'http://rp.example.com:8080')
- `project` (str): Project name in ReportPortal
- `uuid` (str): User authentication token/UUID
- `log_batch_size` (int): Size of log batch requests for performance
- `ignore_errors` (bool): Whether to ignore connection/API errors
- `ignored_tags` (list): Pytest markers to exclude from reporting
- `verify_ssl` (bool): Whether to verify SSL certificates

**Returns**: ReportPortal service instance

**Example**:
```python
service = PyTestServiceClass()
service.init_service(
    endpoint='https://reportportal.company.com',
    project='my_project',
    uuid='your-uuid-token-here',
    log_batch_size=20,
    ignore_errors=True,
    ignored_tags=['skip', 'slow', 'parametrize'],
    verify_ssl=True
)
```

#### `start_launch(launch_name, mode=None, tags=None, description=None)`

Starts a new test launch in ReportPortal.

**Parameters**:
- `launch_name` (str): Display name for the launch
- `mode` (str, optional): Launch mode ('DEFAULT', 'DEBUG')
- `tags` (list, optional): Tags to associate with the launch
- `description` (str, optional): Launch description

**Example**:
```python
service.start_launch(
    launch_name='Nightly Regression Tests',
    mode='DEFAULT',
    tags=['regression', 'automated', 'nightly'],
    description='Automated regression test suite run nightly'
)
```

#### `collect_tests(session)`

Collects and organizes pytest test items for hierarchical reporting.

**Parameters**:
- `session`: Pytest session object containing test items

**Functionality**:
- Builds hierarchical structure based on configuration
- Handles directory, module, class, and parametrized test hierarchies
- Prepares test items for ReportPortal reporting

#### `start_pytest_item(test_item)`

Starts reporting for a specific pytest test item.

**Parameters**:
- `test_item`: Pytest test item object

**Process**:
1. Starts parent suite items if not already started
2. Creates test item in ReportPortal with metadata
3. Handles test parameters if supported

#### `finish_pytest_item(test_item, status, issue=None)`

Finishes reporting for a pytest test item.

**Parameters**:
- `test_item`: Pytest test item object
- `status` (str): Test result ('PASSED', 'FAILED', 'SKIPPED')
- `issue` (dict, optional): Issue information for failed tests

**Issue Format**:
```python
issue = {
    'issue_type': 'PRODUCT_BUG',  # Issue type ID
    'comment': 'Test failed due to known bug #123'
}
```

#### `finish_launch(launch=None, status='rp_launch')`

Finishes the current test launch.

**Parameters**:
- `launch` (optional): Launch identifier (usually not needed)
- `status` (str): Launch completion status

#### `post_log(message, loglevel='INFO', attachment=None)`

Posts a log message to the current test item.

**Parameters**:
- `message` (str): Log message content
- `loglevel` (str): Log level ('TRACE', 'DEBUG', 'INFO', 'WARN', 'ERROR')
- `attachment` (dict, optional): File attachment

**Example**:
```python
# Simple log message
service.post_log("Test step completed", loglevel='INFO')

# Log with attachment
service.post_log(
    "Screenshot captured",
    loglevel='INFO',
    attachment={
        "name": "screenshot.png",
        "data": base64_encoded_image,
        "mime": "image/png"
    }
)
```

#### `get_issue_types()`

Retrieves available issue types from the ReportPortal project.

**Returns**: `dict` - Mapping of short names to issue type locators

#### `terminate_service(nowait=False)`

Terminates the ReportPortal service connection.

**Parameters**:
- `nowait` (bool): Whether to wait for pending operations to complete

### Usage Examples

#### Complete Service Usage

```python
from pytest_reportportal.service import PyTestServiceClass
import json

# Get singleton instance
service = PyTestServiceClass()

# Initialize service
service.init_service(
    endpoint='http://reportportal.example.com:8080',
    project='demo_project',
    uuid='your-uuid-here',
    log_batch_size=20,
    ignore_errors=False,
    ignored_tags=['skip']
)

# Start launch
service.start_launch(
    launch_name='API Test Suite',
    tags=['api', 'regression'],
    description='Comprehensive API testing'
)

# Simulate test execution
service.post_log("Starting API tests", loglevel='INFO')

# Post test results
service.post_log(
    "Test results summary",
    loglevel='INFO',
    attachment={
        "name": "results.json",
        "data": json.dumps({"passed": 15, "failed": 2}),
        "mime": "application/json"
    }
)

# Finish launch
service.finish_launch(status='PASSED')

# Clean up
service.terminate_service()
```

#### Error Handling

```python
import logging
from queue import Empty

def safe_service_operation(service, operation_func, *args, **kwargs):
    """Safely execute service operation with error handling."""
    try:
        return operation_func(*args, **kwargs)
    except Exception as e:
        logging.error(f"Service operation failed: {e}")
        
        # Check for queued errors
        try:
            exc, msg, tb = service._errors.get(False)
            logging.error(f"Queued error: {msg}")
        except Empty:
            pass
        
        if not service.ignore_errors:
            raise
        
        return None

# Usage
service = PyTestServiceClass()
safe_service_operation(
    service, 
    service.start_launch,
    launch_name='Safe Launch'
)
```

---

## RPReportListener

**Module**: `pytest_reportportal.listener`

### Overview

`RPReportListener` is the event listener class that integrates with pytest's hook system to coordinate test execution reporting with ReportPortal. It handles test lifecycle events and manages test results.

### Class Definition

```python
class RPReportListener(object):
    def __init__(self, py_test_service, log_level=logging.NOTSET, endpoint=None):
        self.PyTestService = py_test_service
        self.result = None
        self.issue = {}
        self._log_level = log_level
        # ... handler initialization
```

### Constructor Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `py_test_service` | PyTestServiceClass | Required | Service instance |
| `log_level` | int | `logging.NOTSET` | Logging level for automated logs |
| `endpoint` | str | None | ReportPortal endpoint URL |

### Attributes

- `PyTestService`: Reference to the ReportPortal service
- `result` (str): Current test result status
- `issue` (dict): Issue information for failed tests
- `_log_level` (int): Configured logging level
- `_log_handler` (RPLogHandler): Log handler instance

### Hook Methods

#### `pytest_runtest_protocol(item)` (Hook Wrapper)

Wraps the entire test execution protocol with ReportPortal reporting.

**Process**:
1. Starts the test item in ReportPortal
2. Sets up logging capture if available
3. Yields control to pytest for test execution
4. Finishes the test item with final status

#### `pytest_runtest_makereport(item)` (Hook Wrapper)

Processes test reports from different phases (setup, call, teardown).

**Report Processing**:
- **Setup phase**: Handles fixture failures and skipped tests
- **Call phase**: Processes main test execution results
- **Teardown phase**: Handles cleanup failures

**Status Mapping**:
- `report.passed` → 'PASSED'
- `report.failed` → 'FAILED' 
- `report.skipped` → 'SKIPPED'

### Private Methods

#### `_add_issue_info(item, report)`

Extracts issue information from pytest markers and report data.

**Supported Markers**:
- `@pytest.mark.issue(issue_id="BUG-123", reason="Known bug", issue_type="PB")`
- `@pytest.mark.xfail(reason="Expected to fail")`

### Usage Examples

#### Integration in Plugin

```python
# In plugin.py
def pytest_configure(config):
    """Configure ReportPortal listener."""
    if config._reportportal_configured:
        log_level = logging.INFO  # or from config
        
        config._reporter = RPReportListener(
            config.py_test_service,
            log_level=log_level,
            endpoint=config.getini('rp_endpoint')
        )
        
        config.pluginmanager.register(config._reporter)

def pytest_unconfigure(config):
    """Unconfigure ReportPortal listener."""
    if hasattr(config, '_reporter'):
        reporter = config._reporter
        del config._reporter
        config.pluginmanager.unregister(reporter)
```

#### Custom Listener Extension

```python
from pytest_reportportal.listener import RPReportListener
import pytest

class CustomRPReportListener(RPReportListener):
    """Extended ReportPortal listener with custom functionality."""
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.custom_data = {}
    
    @pytest.hookimpl(hookwrapper=True)
    def pytest_runtest_protocol(self, item):
        """Enhanced test protocol with custom tracking."""
        # Pre-test custom logic
        self.custom_data[item.nodeid] = {
            'start_time': time.time(),
            'custom_markers': [m.name for m in item.own_markers]
        }
        
        # Call parent implementation
        yield from super().pytest_runtest_protocol(item)
        
        # Post-test custom logic
        if item.nodeid in self.custom_data:
            duration = time.time() - self.custom_data[item.nodeid]['start_time']
            self.PyTestService.post_log(
                f"Test duration: {duration:.2f} seconds",
                loglevel='INFO'
            )
    
    @pytest.hookimpl(hookwrapper=True) 
    def pytest_runtest_makereport(self, item):
        """Enhanced report processing with custom issue handling."""
        report = (yield).get_result()
        
        # Custom issue processing
        if report.failed and hasattr(item, 'custom_issue_handler'):
            custom_issue = item.custom_issue_handler(report)
            if custom_issue:
                self.issue.update(custom_issue)
        
        # Call parent processing
        super().pytest_runtest_makereport.__wrapped__(self, item)
```

---

## Singleton

**Module**: `pytest_reportportal.service`

### Overview

`Singleton` is a metaclass that implements the Singleton design pattern, ensuring only one instance of a class exists throughout the application lifecycle.

### Class Definition

```python
class Singleton(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]
```

### Usage Examples

#### Basic Singleton Usage

```python
from six import with_metaclass
from pytest_reportportal.service import Singleton

class MyService(with_metaclass(Singleton, object)):
    def __init__(self):
        self.data = {}
    
    def set_data(self, key, value):
        self.data[key] = value
    
    def get_data(self, key):
        return self.data.get(key)

# Usage
service1 = MyService()
service2 = MyService()

print(service1 is service2)  # True - same instance

service1.set_data('test', 'value')
print(service2.get_data('test'))  # 'value' - shared state
```

---

## Class Relationships

### Inheritance Hierarchy

```
logging.getLoggerClass()
└── RPLogger

logging.Handler
└── RPLogHandler

object (with Singleton metaclass)
└── PyTestServiceClass

object
└── RPReportListener

type (metaclass)
└── Singleton
```

### Dependency Graph

```
PyTestServiceClass (Singleton)
├── Used by RPLogHandler
├── Used by RPReportListener
└── Manages ReportPortalServiceAsync

RPLogger
└── Works with RPLogHandler

RPReportListener
├── Uses PyTestServiceClass
├── Integrates with pytest hooks
└── Manages RPLogHandler (optional)
```

### Interaction Flow

1. **Initialization**: `PyTestServiceClass` singleton is created
2. **Configuration**: Service is initialized with ReportPortal connection
3. **Logging Setup**: `RPLogger` and `RPLogHandler` are configured
4. **Test Execution**: `RPReportListener` coordinates with pytest hooks
5. **Reporting**: All components work together to send data to ReportPortal

## Usage Patterns

### Pattern 1: Basic Setup

```python
# Standard setup pattern used in most tests
import logging
from pytest_reportportal import RPLogger, RPLogHandler
from pytest_reportportal.service import PyTestServiceClass

# 1. Set logger class
logging.setLoggerClass(RPLogger)

# 2. Get service instance (singleton)
service = PyTestServiceClass()

# 3. Create and configure handler
handler = RPLogHandler(service)
logger = logging.getLogger(__name__)
logger.addHandler(handler)

# 4. Use normally
logger.info("Test message")
```

### Pattern 2: pytest Fixture Integration

```python
# Fixture-based pattern for pytest integration
@pytest.fixture(scope="session")
def rp_logger(request):
    logging.setLoggerClass(RPLogger)
    logger = logging.getLogger(__name__)
    
    handler = RPLogHandler(request.node.config.py_test_service)
    logger.addHandler(handler)
    
    return logger

def test_example(rp_logger):
    rp_logger.info("Test started")
    # Test logic
    rp_logger.info("Test completed")
```

### Pattern 3: Advanced Custom Integration

```python
# Custom integration with extended functionality
class CustomReportPortalIntegration:
    def __init__(self, config):
        self.service = PyTestServiceClass()
        self.service.init_service(**config)
        
        self.logger = self._setup_logger()
        self.listener = self._setup_listener()
    
    def _setup_logger(self):
        logging.setLoggerClass(RPLogger)
        logger = logging.getLogger(self.__class__.__name__)
        
        handler = RPLogHandler(
            self.service,
            level=logging.INFO,
            filter_reportportal_client_logs=True
        )
        logger.addHandler(handler)
        
        return logger
    
    def _setup_listener(self):
        return RPReportListener(
            self.service,
            log_level=logging.INFO
        )
    
    def start_test_session(self, name, tags=None):
        self.service.start_launch(name, tags=tags)
        self.logger.info(f"Started test session: {name}")
    
    def end_test_session(self):
        self.service.finish_launch()
        self.logger.info("Ended test session")
        self.service.terminate_service()

# Usage
integration = CustomReportPortalIntegration({
    'endpoint': 'http://rp.example.com',
    'project': 'my_project',
    'uuid': 'token'
})

integration.start_test_session("Custom Test Run", tags=['custom'])
# Run tests...
integration.end_test_session()
```

This comprehensive class reference provides detailed information about each public class, their relationships, and usage patterns. Use this documentation alongside the API reference and usage examples for complete understanding of the pytest-reportportal plugin.