# pytest-reportportal Function Reference

This document provides comprehensive documentation for all public functions in the pytest-reportportal plugin.

## Table of Contents

- [Utility Functions](#utility-functions)
- [Plugin Hook Functions](#plugin-hook-functions)
- [Service Functions](#service-functions)
- [Logging Functions](#logging-functions)
- [Helper Functions](#helper-functions)
- [Function Usage Patterns](#function-usage-patterns)

## Utility Functions

### `timestamp()`

**Module**: `pytest_reportportal.service`  
**Type**: Static utility function

#### Description

Generates a timestamp string in milliseconds since Unix epoch, formatted for ReportPortal API calls.

#### Signature

```python
def timestamp() -> str
```

#### Parameters

None

#### Returns

- **Type**: `str`
- **Description**: Timestamp in milliseconds since Unix epoch (e.g., "1640995200000")

#### Usage Examples

```python
from pytest_reportportal.service import timestamp

# Get current timestamp
current_time = timestamp()
print(current_time)  # Output: "1640995200000"

# Use in ReportPortal API calls
launch_data = {
    'name': 'Test Launch',
    'start_time': timestamp(),
    'description': 'Automated test launch'
}
```

#### Implementation Details

```python
from time import time

def timestamp():
    return str(int(time() * 1000))
```

---

### `trim_docstring(docstring)`

**Module**: `pytest_reportportal.service`  
**Type**: Static utility function

#### Description

Trims and formats docstrings according to PEP 257 conventions, removing common indentation and leading/trailing blank lines.

#### Signature

```python
def trim_docstring(docstring: str) -> str
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `docstring` | str | Yes | The docstring to trim and format |

#### Returns

- **Type**: `str`
- **Description**: Formatted docstring with proper indentation and spacing

#### Algorithm

1. Convert tabs to spaces using Python's standard rules
2. Split into lines
3. Determine minimum indentation (excluding first line)
4. Remove common indentation from all lines except the first
5. Strip leading and trailing blank lines
6. Join lines back into a single string

#### Usage Examples

```python
from pytest_reportportal.service import trim_docstring

def example_function():
    """
    This is an example function.
    
        It has some indented content.
        And multiple lines.
    
    Returns something useful.
    """
    pass

# Trim the docstring
trimmed = trim_docstring(example_function.__doc__)
print(trimmed)
# Output:
# This is an example function.
# 
#     It has some indented content.
#     And multiple lines.
# 
# Returns something useful.

# Use with pytest test descriptions
def test_user_login():
    """
    Test user login functionality.
    
    This test verifies:
    - Valid credentials allow login
    - Invalid credentials are rejected
    - Proper error messages are displayed
    """
    pass

description = trim_docstring(test_user_login.__doc__)
# Description will be properly formatted for ReportPortal
```

#### Edge Cases

```python
# Empty or None docstring
assert trim_docstring(None) == ''
assert trim_docstring('') == ''

# Single line docstring
single_line = "Simple description"
assert trim_docstring(single_line) == "Simple description"

# Docstring with only whitespace
whitespace_only = "   \n\t\n   "
assert trim_docstring(whitespace_only) == ""
```

---

## Plugin Hook Functions

### `is_master(config)`

**Module**: `pytest_reportportal.plugin`  
**Type**: Plugin utility function

#### Description

Determines whether the current pytest process is running as a master node (not under xdist distributed testing) or if xdist is not being used at all.

#### Signature

```python
def is_master(config) -> bool
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `config` | pytest.Config | Yes | Pytest configuration object |

#### Returns

- **Type**: `bool`
- **Description**: True if running as master node or not using xdist, False if running as worker node

#### Logic

The function checks for the presence of the `slaveinput` attribute on the config object, which is added by pytest-xdist for worker nodes.

#### Usage Examples

```python
from pytest_reportportal.plugin import is_master

def pytest_sessionstart(session):
    """Handle session start - only on master node."""
    if is_master(session.config):
        # This code runs only on the master node
        print("Starting ReportPortal launch on master node")
        session.config.py_test_service.start_launch("Master Launch")
    else:
        # This code runs on worker nodes
        print("Worker node - no launch creation needed")

def pytest_configure(config):
    """Configure plugin based on node type."""
    if is_master(config):
        # Master-specific configuration
        config.py_test_service = PyTestServiceClass()
    else:
        # Worker-specific configuration
        import pickle
        config.py_test_service = pickle.loads(config.slaveinput['py_test_service'])
```

#### Integration with xdist

```python
# Example of complete xdist integration
def pytest_sessionfinish(session):
    """Handle session finish."""
    if not session.config.option.rp_enabled:
        return
    
    if is_master(session.config):
        # Only master node finishes the launch
        session.config.py_test_service.finish_launch(status='RP_Launch')
    
    # All nodes (master and workers) terminate their service
    session.config.py_test_service.terminate_service()
```

---

### `wait_launch(rp_client)`

**Module**: `pytest_reportportal.plugin`  
**Type**: Plugin utility function

#### Description

Waits for a ReportPortal launch to be created and available, with a configurable timeout. This is primarily used in distributed testing scenarios where synchronization is needed.

#### Signature

```python
def wait_launch(rp_client) -> None
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `rp_client` | ReportPortalServiceAsync | Yes | ReportPortal client instance |

#### Returns

- **Type**: `None`
- **Raises**: `Exception` if launch is not found within timeout period

#### Configuration

- **Timeout**: Defined by `LAUNCH_WAIT_TIMEOUT` constant (default: 10 seconds)
- **Poll Interval**: 1 second

#### Usage Examples

```python
from pytest_reportportal.plugin import wait_launch
from pytest_reportportal import LAUNCH_WAIT_TIMEOUT

def pytest_sessionstart(session):
    """Start session with launch synchronization."""
    if not session.config.option.rp_enabled:
        return
    
    if is_master(session.config):
        # Master starts the launch
        session.config.py_test_service.start_launch(
            session.config.option.rp_launch,
            tags=session.config.getini('rp_launch_tags')
        )
        
        # If using xdist, wait for launch to be available
        if session.config.pluginmanager.hasplugin('xdist'):
            try:
                wait_launch(session.config.py_test_service.RP.rp_client)
                print(f"Launch ready after waiting (max {LAUNCH_WAIT_TIMEOUT}s)")
            except Exception as e:
                print(f"Launch wait failed: {e}")
                raise

# Custom timeout handling
def wait_launch_with_custom_timeout(rp_client, timeout=30):
    """Wait for launch with custom timeout."""
    import time
    
    start_time = time.time()
    while not rp_client.launch_id:
        if time.time() - start_time > timeout:
            raise Exception(f"Launch not found within {timeout} seconds")
        time.sleep(1)
    
    print(f"Launch {rp_client.launch_id} is ready")
```

#### Error Handling

```python
def robust_wait_launch(rp_client, max_retries=3):
    """Wait for launch with retry logic."""
    for attempt in range(max_retries):
        try:
            wait_launch(rp_client)
            return  # Success
        except Exception as e:
            if attempt == max_retries - 1:
                raise  # Last attempt failed
            print(f"Launch wait attempt {attempt + 1} failed: {e}")
            time.sleep(2)  # Wait before retry
```

---

### `pytest_addoption(parser)`

**Module**: `pytest_reportportal.plugin`  
**Type**: pytest hook implementation

#### Description

Adds command-line options and configuration file options for the ReportPortal plugin. This hook is called during pytest initialization to register all available options.

#### Signature

```python
def pytest_addoption(parser) -> None
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `parser` | pytest.Parser | Yes | pytest argument parser |

#### Added Options

#### Command Line Options

| Option | Type | Description |
|--------|------|-------------|
| `--reportportal` | flag | Enable ReportPortal plugin |
| `--rp-launch` | string | Override launch name |
| `--rp-launch-description` | string | Override launch description |
| `--rp-log-level` | string | Set logging level for automated logs |

#### Configuration File Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `rp_uuid` | string | Required | User authentication token |
| `rp_endpoint` | string | Required | ReportPortal server URL |
| `rp_project` | string | Required | Project name |
| `rp_launch` | string | 'Pytest Launch' | Launch name |
| `rp_launch_tags` | args | - | Launch tags |
| `rp_tests_tags` | args | - | Tags for all test items |
| `rp_launch_description` | string | '' | Launch description |
| `rp_log_batch_size` | string | '20' | Log batch size |
| `rp_ignore_errors` | bool | False | Ignore ReportPortal errors |
| `rp_ignore_tags` | args | - | Ignore specified markers |
| `rp_hierarchy_dirs` | bool | False | Enable directory hierarchy |
| `rp_hierarchy_module` | bool | True | Enable module hierarchy |
| `rp_hierarchy_class` | bool | True | Enable class hierarchy |
| `rp_hierarchy_parametrize` | bool | False | Enable parametrized test hierarchy |
| `rp_hierarchy_dirs_level` | int | 0 | Directory hierarchy starting level |
| `rp_issue_marks` | args | '' | Marks for issue information |
| `rp_issue_system_url` | string | '' | Issue system URL template |
| `rp_verify_ssl` | bool | True | Verify SSL certificates |
| `rp_display_suite_test_file` | bool | True | Include file path in suite names |

#### Usage Examples

```python
# The function is automatically called by pytest
# Users interact with it through configuration

# pytest.ini example
"""
[pytest]
rp_uuid = your-token-here
rp_endpoint = http://reportportal.local:8080
rp_project = my_project
rp_launch = CI Build Tests
rp_launch_tags = ci automated
rp_launch_description = Automated tests from CI pipeline
rp_log_batch_size = 50
rp_ignore_errors = True
rp_ignore_tags = skip slow
rp_hierarchy_dirs = True
rp_issue_marks = xfail bug
rp_issue_system_url = https://jira.example.com/browse/{issue_id}
"""

# Command line usage
"""
pytest --reportportal --rp-launch "Manual Test Run" --rp-launch-description "Manual testing session"
"""

# Programmatic access to options
def pytest_configure(config):
    """Access configured options."""
    # Command line options
    rp_enabled = config.option.rp_enabled
    launch_name = config.option.rp_launch
    
    # Configuration file options  
    endpoint = config.getini('rp_endpoint')
    project = config.getini('rp_project')
    uuid = config.getini('rp_uuid')
    
    print(f"ReportPortal enabled: {rp_enabled}")
    print(f"Endpoint: {endpoint}")
    print(f"Project: {project}")
```

---

## Service Functions

### `async_error_handler(exc_info)`

**Module**: `pytest_reportportal.service.PyTestServiceClass`  
**Type**: Instance method

#### Description

Handles asynchronous errors from the ReportPortal client, terminating the service and queuing error information for later processing.

#### Signature

```python
def async_error_handler(self, exc_info) -> None
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `exc_info` | tuple | Yes | Exception information tuple (type, value, traceback) |

#### Process

1. Terminates the ReportPortal service immediately
2. Sets the RP client to None
3. Queues the exception information for error reporting

#### Usage Examples

```python
from pytest_reportportal.service import PyTestServiceClass

# The error handler is automatically set during service initialization
service = PyTestServiceClass()
service.init_service(
    endpoint='http://reportportal.example.com',
    project='test_project',
    uuid='token',
    log_batch_size=20,
    ignore_errors=False,
    ignored_tags=[],
    verify_ssl=True
)

# Error handler is set internally:
# self.RP = ReportPortalServiceAsync(
#     error_handler=self.async_error_handler,
#     ...
# )

# Manual error simulation (for testing)
def simulate_error():
    try:
        raise ValueError("Simulated ReportPortal error")
    except Exception:
        import sys
        service.async_error_handler(sys.exc_info())

# Check for errors
def check_service_errors(service):
    """Check if service has queued errors."""
    try:
        exc, msg, tb = service._errors.get(False)
        print(f"Service error detected: {msg}")
        return True
    except queue.Empty:
        print("No service errors")
        return False
```

---

### `_stop_if_necessary()`

**Module**: `pytest_reportportal.service.PyTestServiceClass`  
**Type**: Instance method (private)

#### Description

Checks for queued asynchronous errors and handles them according to the `ignore_errors` configuration. This method is called before most service operations to ensure error handling.

#### Signature

```python
def _stop_if_necessary(self) -> None
```

#### Parameters

None

#### Process

1. Attempts to retrieve error from the error queue (non-blocking)
2. If error found, prints the traceback to stderr
3. If `ignore_errors` is False, exits pytest with the error message
4. If `ignore_errors` is True, continues execution

#### Usage Examples

```python
# This method is called internally before service operations
class PyTestServiceClass:
    def post_log(self, message, loglevel='INFO', attachment=None):
        self._stop_if_necessary()  # Check for errors first
        if self.RP is None:
            return
        # ... rest of method

    def start_pytest_item(self, test_item=None):
        self._stop_if_necessary()  # Check for errors first
        if self.RP is None:
            return
        # ... rest of method

# Custom error checking
def safe_operation_with_error_check(service, operation_func, *args, **kwargs):
    """Perform service operation with explicit error checking."""
    # Manual error check
    service._stop_if_necessary()
    
    if service.RP is None:
        print("Service not available")
        return None
    
    try:
        return operation_func(*args, **kwargs)
    except Exception as e:
        print(f"Operation failed: {e}")
        if not service.ignore_errors:
            raise
        return None
```

---

### `get_issue_types()`

**Module**: `pytest_reportportal.service.PyTestServiceClass`  
**Type**: Instance method

#### Description

Retrieves available issue types from the ReportPortal project settings and creates a mapping from short names to issue type locators.

#### Signature

```python
def get_issue_types(self) -> dict
```

#### Parameters

None

#### Returns

- **Type**: `dict`
- **Description**: Mapping of issue type short names to their locators
- **Example**: `{"PB": "product_bug_locator", "AB": "automation_bug_locator"}`

#### Issue Type Categories

- **AUTOMATION_BUG**: Issues related to test automation
- **PRODUCT_BUG**: Issues in the product being tested
- **SYSTEM_ISSUE**: Infrastructure or system-related issues
- **NO_DEFECT**: Test failures that are not defects
- **TO_INVESTIGATE**: Issues requiring investigation

#### Usage Examples

```python
from pytest_reportportal.service import PyTestServiceClass

service = PyTestServiceClass()
service.init_service(
    endpoint='http://reportportal.example.com',
    project='my_project',
    uuid='token',
    log_batch_size=20,
    ignore_errors=True,
    ignored_tags=[],
    verify_ssl=True
)

# Get available issue types
issue_types = service.get_issue_types()
print("Available issue types:")
for short_name, locator in issue_types.items():
    print(f"  {short_name}: {locator}")

# Example output:
# Available issue types:
#   PB: product_bug
#   AB: automation_bug
#   SI: system_issue
#   ND: no_defect
#   TI: to_investigate

# Use issue types in test reporting
def report_test_with_issue(service, test_item, status, issue_type_short=None, comment=None):
    """Report test result with issue information."""
    issue = None
    
    if status == 'FAILED' and issue_type_short:
        issue_types = service.get_issue_types()
        if issue_type_short in issue_types:
            issue = {
                'issue_type': issue_types[issue_type_short],
                'comment': comment or f'Test failed - {issue_type_short}'
            }
    
    service.finish_pytest_item(test_item, status, issue)

# Usage with pytest markers
import pytest

@pytest.mark.issue(issue_id="BUG-123", issue_type="PB", reason="Known product bug")
def test_with_known_issue():
    """Test that has a known product bug."""
    # Test implementation
    assert False, "This test fails due to known bug"

# Custom issue type validation
def validate_issue_type(service, issue_type_short):
    """Validate that an issue type is available in the project."""
    available_types = service.get_issue_types()
    if issue_type_short not in available_types:
        raise ValueError(f"Issue type '{issue_type_short}' not available. "
                        f"Available types: {list(available_types.keys())}")
    return available_types[issue_type_short]
```

---

## Logging Functions

### `patching_logger_class()`

**Module**: `pytest_reportportal.rp_logging`  
**Type**: Context manager function

#### Description

A context manager that temporarily patches the current logging class to support ReportPortal-specific functionality, particularly attachment handling. This allows standard Python loggers to work with ReportPortal attachments.

#### Signature

```python
@contextmanager
def patching_logger_class() -> Generator[None, None, None]
```

#### Parameters

None

#### Returns

- **Type**: Context manager
- **Yields**: None

#### Functionality

1. **Patches `_log` method**: Adds support for `attachment` parameter
2. **Patches `makeRecord` method**: Handles attachment data in log records
3. **Automatic restoration**: Restores original methods when context exits
4. **Thread-safe**: Uses proper context management

#### Usage Examples

```python
import logging
from pytest_reportportal.rp_logging import patching_logger_class

# Basic usage
def test_with_patched_logging():
    """Test using patched logging for attachments."""
    with patching_logger_class():
        logger = logging.getLogger(__name__)
        
        # Now logger supports attachment parameter
        logger.info(
            "Test step completed",
            attachment={
                "name": "step_result.txt",
                "data": "Step execution details...",
                "mime": "text/plain"
            }
        )

# Integration with pytest fixtures
@pytest.fixture
def patched_logger():
    """Provide a logger with ReportPortal attachment support."""
    with patching_logger_class():
        logger = logging.getLogger(__name__)
        yield logger

def test_with_fixture(patched_logger):
    """Test using patched logger fixture."""
    patched_logger.info("Starting test")
    
    # Simulate test data collection
    test_data = {"result": "success", "duration": 1.5}
    
    patched_logger.info(
        "Test data collected",
        attachment={
            "name": "test_data.json",
            "data": json.dumps(test_data, indent=2),
            "mime": "application/json"
        }
    )

# Advanced usage with custom handler
class CustomHandler(logging.Handler):
    """Custom handler that works with patched logging."""
    
    def emit(self, record):
        # Access attachment from patched record
        if hasattr(record, 'attachment') and record.attachment:
            print(f"Log with attachment: {record.attachment['name']}")
        else:
            print(f"Regular log: {record.getMessage()}")

def test_custom_handler():
    """Test with custom handler and patched logging."""
    with patching_logger_class():
        logger = logging.getLogger(__name__)
        handler = CustomHandler()
        logger.addHandler(handler)
        logger.setLevel(logging.INFO)
        
        # Regular log
        logger.info("Regular message")
        
        # Log with attachment
        logger.info(
            "Message with attachment",
            attachment={"name": "file.txt", "data": "content"}
        )

# Error handling
def test_patching_error_handling():
    """Test error handling in patched logging."""
    with patching_logger_class():
        logger = logging.getLogger(__name__)
        
        try:
            # This might fail if attachment format is wrong
            logger.info(
                "Test message",
                attachment="invalid_format"  # Should be dict
            )
        except Exception as e:
            logger.error(f"Logging failed: {e}")
```

#### Implementation Details

```python
@contextmanager
def patching_logger_class():
    logger_class = logging.getLoggerClass()
    original_log = logger_class._log
    original_makeRecord = logger_class.makeRecord
    
    try:
        def wrap_log(original_func):
            @wraps(original_func)
            def _log(self, *args, **kwargs):
                attachment = kwargs.pop('attachment', None)
                if attachment is not None:
                    kwargs.setdefault('extra', {}).update(
                        {'attachment': attachment})
                return original_func(self, *args, **kwargs)
            return _log
        
        def wrap_makeRecord(original_func):
            @wraps(original_func)
            def makeRecord(self, name, level, fn, lno, msg, args, exc_info,
                           func=None, extra=None, sinfo=None):
                if extra is not None:
                    attachment = extra.pop('attachment', None)
                else:
                    attachment = None
                
                # Handle different Python versions
                try:
                    record = original_func(self, name, level, fn, lno, msg,
                                         args, exc_info, func=func,
                                         extra=extra, sinfo=sinfo)
                except TypeError:
                    record = original_func(self, name, level, fn, lno, msg,
                                         args, exc_info, func=func,
                                         extra=extra)
                
                record.attachment = attachment
                return record
            return makeRecord
        
        # Apply patches only if not already patched
        if not logger_class == RPLogger and not hasattr(logger_class, "_patched"):
            logger_class._log = wrap_log(logger_class._log)
            logger_class.makeRecord = wrap_makeRecord(logger_class.makeRecord)
            logger_class._patched = True
        
        yield
        
    finally:
        # Restore original methods
        logger_class._log = original_log
        logger_class.makeRecord = original_makeRecord
```

---

## Helper Functions

### `_get_item_name(test_item)`

**Module**: `pytest_reportportal.service.PyTestServiceClass`  
**Type**: Static method

#### Description

Extracts and formats the display name for a test item, ensuring it meets ReportPortal's name length constraints (maximum 256 characters).

#### Signature

```python
@staticmethod
def _get_item_name(test_item) -> str
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `test_item` | pytest.Item | Yes | Pytest test item object |

#### Returns

- **Type**: `str`
- **Description**: Formatted test item name, truncated if necessary

#### Constraints

- Maximum length: 256 characters
- If truncated, a warning is issued to the test item

#### Usage Examples

```python
from pytest_reportportal.service import PyTestServiceClass

# Example test items
class MockTestItem:
    def __init__(self, name):
        self._rp_name = name
    
    def warn(self, code, message):
        print(f"Warning {code}: {message}")

# Normal length name
normal_item = MockTestItem("test_user_authentication")
name = PyTestServiceClass._get_item_name(normal_item)
print(name)  # "test_user_authentication"

# Long name that needs truncation
long_name = "test_very_long_function_name_that_exceeds_the_maximum_allowed_length_" * 10
long_item = MockTestItem(long_name)
truncated_name = PyTestServiceClass._get_item_name(long_item)
print(len(truncated_name))  # 256
print(truncated_name.endswith("..."))  # False (just truncated)

# Real pytest integration
def pytest_runtest_protocol(item):
    """Example of using _get_item_name in pytest hook."""
    service = PyTestServiceClass()
    
    # Get properly formatted name
    item_name = service._get_item_name(item)
    
    print(f"Starting test: {item_name}")
    
    # Use in ReportPortal API call
    service.RP.start_test_item(
        name=item_name,
        item_type='STEP',
        start_time=timestamp()
    )
```

---

### `_get_item_description(test_item)`

**Module**: `pytest_reportportal.service.PyTestServiceClass`  
**Type**: Static method

#### Description

Extracts and formats the description for a test item from its docstring or other documentation sources.

#### Signature

```python
@staticmethod
def _get_item_description(test_item) -> str
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `test_item` | pytest.Item | Yes | Pytest test item object |

#### Returns

- **Type**: `str`
- **Description**: Formatted test item description

#### Supported Item Types

- **Class**: Uses class docstring
- **Function**: Uses function docstring  
- **Module**: Uses module docstring
- **Item**: Uses object docstring
- **DoctestItem**: Uses doctest report info

#### Usage Examples

```python
from pytest_reportportal.service import PyTestServiceClass

# Test function with docstring
def test_user_login():
    """
    Test user authentication functionality.
    
    This test verifies that users can successfully log in
    with valid credentials and receive appropriate error
    messages for invalid credentials.
    """
    pass

# Mock test item
class MockTestItem:
    def __init__(self, obj):
        self.obj = obj

# Get description
test_item = MockTestItem(test_user_login)
description = PyTestServiceClass._get_item_description(test_item)
print(description)
# Output: "Test user authentication functionality.\n\nThis test verifies..."

# Class with docstring
class TestUserManagement:
    """Test suite for user management operations."""
    
    def test_create_user(self):
        """Test user creation functionality."""
        pass

class_item = MockTestItem(TestUserManagement)
class_description = PyTestServiceClass._get_item_description(class_item)
print(class_description)  # "Test suite for user management operations."

# No docstring case
def test_without_docstring():
    pass

no_doc_item = MockTestItem(test_without_docstring)
no_description = PyTestServiceClass._get_item_description(no_doc_item)
print(repr(no_description))  # None or empty string

# Integration with ReportPortal
def start_test_with_description(service, test_item):
    """Start test item with proper description."""
    description = service._get_item_description(test_item)
    
    service.RP.start_test_item(
        name=service._get_item_name(test_item),
        description=description,
        item_type='STEP',
        start_time=timestamp()
    )
```

---

### `_get_item_tags(test_item)`

**Module**: `pytest_reportportal.service.PyTestServiceClass`  
**Type**: Instance method

#### Description

Extracts pytest markers from a test item and converts them to ReportPortal tags, filtering out ignored markers and adding configured test tags.

#### Signature

```python
def _get_item_tags(self, test_item) -> list
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `test_item` | pytest.Item | Yes | Pytest test item object |

#### Returns

- **Type**: `list`
- **Description**: List of tag strings for ReportPortal

#### Tag Processing

1. Extracts pytest markers from the test item
2. Filters out markers listed in `ignored_tags`
3. Formats markers with arguments (e.g., "parametrize:value")
4. Adds configured test tags from `rp_tests_tags`

#### Usage Examples

```python
import pytest
from pytest_reportportal.service import PyTestServiceClass

# Test with multiple markers
@pytest.mark.smoke
@pytest.mark.api
@pytest.mark.priority("high")
@pytest.mark.parametrize("user_id", [1, 2, 3])
def test_api_functionality(user_id):
    """Test API with different user IDs."""
    pass

# Mock service configuration
service = PyTestServiceClass()
service.ignored_tags = ['parametrize', 'skip']

# Mock test item with markers
class MockTestItem:
    def __init__(self, markers, session_config):
        self.own_markers = markers
        self.session = MockSession(session_config)
    
    def get_closest_marker(self, name):
        for marker in self.own_markers:
            if marker.name == name:
                return marker
        return None

class MockMarker:
    def __init__(self, name, args=None):
        self.name = name
        self.args = args or []

class MockSession:
    def __init__(self, config):
        self.config = MockConfig(config)

class MockConfig:
    def __init__(self, test_tags):
        self._test_tags = test_tags
    
    def getini(self, option):
        if option == 'rp_tests_tags':
            return self._test_tags
        return []

# Create test item with markers
markers = [
    MockMarker('smoke'),
    MockMarker('api'), 
    MockMarker('priority', ['high']),
    MockMarker('parametrize', ['user_id'])  # Will be filtered out
]

test_item = MockTestItem(markers, ['automated', 'regression'])

# Get tags
tags = service._get_item_tags(test_item)
print(tags)
# Output: ['smoke', 'api', 'priority:high', 'automated', 'regression']

# Real pytest integration
def pytest_runtest_setup(item):
    """Example of using tags in pytest hook."""
    service = PyTestServiceClass()
    service.ignored_tags = ['skip', 'xfail']
    
    # Get tags for the test item
    tags = service._get_item_tags(item)
    
    print(f"Test tags: {tags}")
    
    # Use tags in ReportPortal
    service.RP.start_test_item(
        name=service._get_item_name(item),
        tags=tags,
        item_type='STEP'
    )

# Custom tag processing
def process_custom_tags(service, test_item, additional_tags=None):
    """Process tags with custom additions."""
    # Get standard tags
    tags = service._get_item_tags(test_item)
    
    # Add custom tags
    if additional_tags:
        tags.extend(additional_tags)
    
    # Remove duplicates while preserving order
    seen = set()
    unique_tags = []
    for tag in tags:
        if tag not in seen:
            seen.add(tag)
            unique_tags.append(tag)
    
    return unique_tags
```

---

### `_get_parameters(test_item)`

**Module**: `pytest_reportportal.service.PyTestServiceClass`  
**Type**: Instance method

#### Description

Extracts test parameters from parametrized pytest test items for display in ReportPortal.

#### Signature

```python
def _get_parameters(self, test_item) -> dict
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `test_item` | pytest.Item | Yes | Pytest test item object |

#### Returns

- **Type**: `dict`
- **Description**: Dictionary of parameter names to values, empty dict if no parameters

#### Requirements

- Only works when `RP_SUPPORTS_PARAMETERS` is True
- Requires test item to have `callspec` attribute (parametrized tests)

#### Usage Examples

```python
import pytest
from pytest_reportportal.service import PyTestServiceClass

# Parametrized test
@pytest.mark.parametrize("username,password,expected", [
    ("admin", "secret", True),
    ("user", "pass", True),
    ("invalid", "wrong", False)
])
def test_login(username, password, expected):
    """Test login with different credentials."""
    # Test implementation
    pass

# Mock parametrized test item
class MockCallSpec:
    def __init__(self, params):
        self.params = params

class MockTestItem:
    def __init__(self, callspec=None):
        if callspec:
            self.callspec = callspec

# Service with parameter support
service = PyTestServiceClass()
service.RP_SUPPORTS_PARAMETERS = True

# Test item with parameters
callspec = MockCallSpec({
    'username': 'admin',
    'password': 'secret', 
    'expected': True
})
test_item = MockTestItem(callspec)

# Get parameters
params = service._get_parameters(test_item)
print(params)
# Output: {'username': 'admin', 'password': 'secret', 'expected': True}

# Test item without parameters
non_param_item = MockTestItem()
no_params = service._get_parameters(non_param_item)
print(no_params)  # Output: {}

# Integration with test reporting
def report_parametrized_test(service, test_item):
    """Report parametrized test with parameters."""
    if service.RP_SUPPORTS_PARAMETERS:
        parameters = service._get_parameters(test_item)
        
        service.RP.start_test_item(
            name=service._get_item_name(test_item),
            description=service._get_item_description(test_item),
            tags=service._get_item_tags(test_item),
            parameters=parameters,  # Include parameters
            item_type='STEP',
            start_time=timestamp()
        )
    else:
        # Fallback for older ReportPortal versions
        service.RP.start_test_item(
            name=service._get_item_name(test_item),
            description=service._get_item_description(test_item),
            tags=service._get_item_tags(test_item),
            item_type='STEP',
            start_time=timestamp()
        )

# Custom parameter formatting
def format_parameters_for_display(service, test_item):
    """Format parameters for readable display."""
    params = service._get_parameters(test_item)
    
    if not params:
        return "No parameters"
    
    # Format as key=value pairs
    param_strings = [f"{key}={value}" for key, value in params.items()]
    return "Parameters: " + ", ".join(param_strings)

# Example usage in test
def test_with_parameter_logging(test_item, rp_logger):
    """Test that logs its parameters."""
    service = PyTestServiceClass()
    param_info = format_parameters_for_display(service, test_item)
    rp_logger.info(param_info)
    
    # Test logic here
    pass
```

---

## Function Usage Patterns

### Pattern 1: Service Initialization and Lifecycle

```python
from pytest_reportportal.service import PyTestServiceClass, timestamp

def complete_service_lifecycle():
    """Complete example of service lifecycle management."""
    # 1. Get singleton instance
    service = PyTestServiceClass()
    
    # 2. Initialize service
    service.init_service(
        endpoint='http://reportportal.example.com:8080',
        project='my_project',
        uuid='authentication-token',
        log_batch_size=20,
        ignore_errors=True,
        ignored_tags=['skip', 'slow'],
        verify_ssl=True
    )
    
    # 3. Start launch
    service.start_launch(
        launch_name='Automated Test Suite',
        tags=['automation', 'regression'],
        description='Daily regression test execution'
    )
    
    # 4. Post logs during execution
    service.post_log(
        message='Test suite initialized successfully',
        loglevel='INFO',
        attachment={
            'name': 'config.json',
            'data': '{"timeout": 30, "retries": 3}',
            'mime': 'application/json'
        }
    )
    
    # 5. Finish launch
    service.finish_launch(status='PASSED')
    
    # 6. Terminate service
    service.terminate_service()
```

### Pattern 2: Robust Error Handling

```python
import queue
import traceback
from pytest_reportportal.service import PyTestServiceClass

def robust_service_operations():
    """Service operations with comprehensive error handling."""
    service = PyTestServiceClass()
    
    try:
        # Initialize with error checking
        service.init_service(
            endpoint='http://reportportal.example.com:8080',
            project='test_project',
            uuid='token',
            log_batch_size=20,
            ignore_errors=False,  # Don't ignore errors for this example
            ignored_tags=[],
            verify_ssl=True
        )
        
        # Check for initialization errors
        service._stop_if_necessary()
        
        # Start launch with error handling
        service.start_launch('Error Handling Test')
        service._stop_if_necessary()
        
        # Perform operations with error checking
        for i in range(5):
            service.post_log(f'Operation {i+1} completed', loglevel='INFO')
            service._stop_if_necessary()
        
        service.finish_launch(status='PASSED')
        
    except Exception as e:
        print(f"Service operation failed: {e}")
        traceback.print_exc()
        
        # Check for queued async errors
        try:
            while True:
                exc, msg, tb = service._errors.get(False)
                print(f"Async error: {msg}")
                traceback.print_exception(exc, msg, tb)
        except queue.Empty:
            pass
        
        # Ensure service is terminated
        service.terminate_service(nowait=True)
        raise
    
    finally:
        # Final cleanup
        if service.RP is not None:
            service.terminate_service()
```

### Pattern 3: Advanced Logging with Attachments

```python
import json
import base64
from pytest_reportportal.rp_logging import patching_logger_class
from pytest_reportportal import RPLogger, RPLogHandler

def advanced_logging_pattern():
    """Advanced logging pattern with multiple attachment types."""
    
    # Setup patched logging
    with patching_logger_class():
        logging.setLoggerClass(RPLogger)
        logger = logging.getLogger(__name__)
        
        # Add ReportPortal handler
        service = PyTestServiceClass()
        handler = RPLogHandler(service, level=logging.INFO)
        logger.addHandler(handler)
        
        # Text attachment
        logger.info(
            "Configuration loaded",
            attachment={
                "name": "app_config.txt",
                "data": "debug=true\ntimeout=30\nretries=3",
                "mime": "text/plain"
            }
        )
        
        # JSON attachment
        test_data = {
            "test_suite": "user_management",
            "tests_run": 25,
            "passed": 23,
            "failed": 2,
            "duration": 45.7
        }
        
        logger.info(
            "Test execution summary",
            attachment={
                "name": "test_summary.json",
                "data": json.dumps(test_data, indent=2),
                "mime": "application/json"
            }
        )
        
        # Binary attachment (e.g., screenshot)
        # Simulated binary data
        fake_image_data = b'\x89PNG\r\n\x1a\n...'  # PNG header
        
        logger.info(
            "Screenshot captured",
            attachment={
                "name": "test_screenshot.png",
                "data": base64.b64encode(fake_image_data).decode(),
                "mime": "image/png"
            }
        )
        
        # HTML report attachment
        html_report = """
        <html>
        <head><title>Test Report</title></head>
        <body>
            <h1>Test Results</h1>
            <table border="1">
                <tr><th>Test</th><th>Status</th></tr>
                <tr><td>test_login</td><td>PASSED</td></tr>
                <tr><td>test_logout</td><td>PASSED</td></tr>
            </table>
        </body>
        </html>
        """
        
        logger.info(
            "HTML test report generated",
            attachment={
                "name": "test_report.html",
                "data": html_report,
                "mime": "text/html"
            }
        )
```

### Pattern 4: pytest Integration with Custom Hooks

```python
import pytest
from pytest_reportportal.plugin import is_master, wait_launch
from pytest_reportportal.service import PyTestServiceClass, timestamp

def pytest_configure(config):
    """Custom pytest configuration with ReportPortal."""
    # Check if ReportPortal is configured
    if not all([config.getini('rp_uuid'), 
               config.getini('rp_endpoint'), 
               config.getini('rp_project')]):
        config._reportportal_configured = False
        return
    
    config._reportportal_configured = True
    
    # Initialize service based on node type
    if is_master(config):
        config.py_test_service = PyTestServiceClass()
    else:
        # Worker node - deserialize service
        import pickle
        config.py_test_service = pickle.loads(config.slaveinput['py_test_service'])

def pytest_sessionstart(session):
    """Custom session start with enhanced ReportPortal integration."""
    if not session.config._reportportal_configured:
        return
    
    if not session.config.option.rp_enabled:
        return
    
    if is_master(session.config):
        service = session.config.py_test_service
        
        # Initialize service
        service.init_service(
            endpoint=session.config.getini('rp_endpoint'),
            project=session.config.getini('rp_project'),
            uuid=session.config.getini('rp_uuid'),
            log_batch_size=int(session.config.getini('rp_log_batch_size')),
            ignore_errors=bool(session.config.getini('rp_ignore_errors')),
            ignored_tags=session.config.getini('rp_ignore_tags'),
            verify_ssl=bool(session.config.getini('rp_verify_ssl'))
        )
        
        # Start launch with custom metadata
        launch_name = session.config.option.rp_launch or 'Custom Launch'
        
        service.start_launch(
            launch_name=launch_name,
            tags=list(session.config.getini('rp_launch_tags')) + ['custom'],
            description=f"Launch started at {timestamp()}"
        )
        
        # Wait for launch if using xdist
        if session.config.pluginmanager.hasplugin('xdist'):
            wait_launch(service.RP.rp_client)
        
        # Log session start
        service.post_log(
            f"Test session started with {len(session.items)} tests",
            loglevel='INFO'
        )

def pytest_sessionfinish(session):
    """Custom session finish with summary reporting."""
    if not session.config._reportportal_configured:
        return
    
    if not session.config.option.rp_enabled:
        return
    
    if is_master(session.config):
        service = session.config.py_test_service
        
        # Generate session summary
        summary = {
            "total_tests": len(session.items),
            "session_duration": "calculated_duration",
            "timestamp": timestamp()
        }
        
        service.post_log(
            "Test session completed",
            loglevel='INFO',
            attachment={
                "name": "session_summary.json",
                "data": json.dumps(summary, indent=2),
                "mime": "application/json"
            }
        )
        
        service.finish_launch(status='RP_Launch')
    
    # Terminate service for all nodes
    session.config.py_test_service.terminate_service()
```

This comprehensive function reference provides detailed documentation for all public functions in the pytest-reportportal plugin, including usage examples, parameters, return values, and integration patterns. Use this reference alongside the class and API documentation for complete understanding of the plugin's functionality.