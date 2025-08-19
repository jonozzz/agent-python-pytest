# pytest-reportportal Usage Examples and Tutorials

This document provides comprehensive examples and tutorials for using the pytest-reportportal plugin effectively.

## Table of Contents

- [Quick Start Guide](#quick-start-guide)
- [Configuration Examples](#configuration-examples)
- [Basic Usage Examples](#basic-usage-examples)
- [Advanced Usage Examples](#advanced-usage-examples)
- [Integration Examples](#integration-examples)
- [Troubleshooting Examples](#troubleshooting-examples)
- [Best Practices](#best-practices)

## Quick Start Guide

### 1. Installation

```bash
pip install pytest-reportportal
```

### 2. Basic Configuration

Create a `pytest.ini` file in your project root:

```ini
[pytest]
rp_uuid = your-uuid-token-here
rp_endpoint = http://your-reportportal.com:8080
rp_project = your_project_name
rp_launch = My First Launch
```

### 3. Run Tests

```bash
pytest --reportportal
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

### Complete Configuration

```ini
# pytest.ini - Full configuration with all options
[pytest]
# Required settings
rp_uuid = fb586627-32be-47dd-93c1-678873458a5f
rp_endpoint = http://reportportal.local:8080
rp_project = demo_project

# Launch settings
rp_launch = Comprehensive Test Suite
rp_launch_tags = regression smoke api
rp_launch_description = Complete regression test suite with API and UI tests

# Test organization
rp_tests_tags = automated pytest
rp_hierarchy_dirs = True
rp_hierarchy_module = True
rp_hierarchy_class = True
rp_hierarchy_parametrize = False
rp_hierarchy_dirs_level = 0
rp_display_suite_test_file = True

# Logging and reporting
rp_log_batch_size = 50
rp_log_level = INFO
rp_ignore_errors = True
rp_ignore_tags = skip slow parametrize

# Issue tracking
rp_issue_marks = xfail issue bug
rp_issue_system_url = https://jira.company.com/browse/{issue_id}

# Security
rp_verify_ssl = True
```

### Environment-Specific Configuration

```ini
# pytest-dev.ini - Development environment
[pytest]
rp_uuid = dev-uuid-token
rp_endpoint = http://dev-reportportal:8080
rp_project = myproject_dev
rp_launch = Development Tests
rp_ignore_errors = True
```

```ini
# pytest-prod.ini - Production environment
[pytest]
rp_uuid = prod-uuid-token
rp_endpoint = https://reportportal.company.com
rp_project = myproject_prod
rp_launch = Production Smoke Tests
rp_ignore_errors = False
rp_verify_ssl = True
```

Usage:
```bash
# Development
pytest -c pytest-dev.ini --reportportal

# Production
pytest -c pytest-prod.ini --reportportal
```

## Basic Usage Examples

### Simple Test with Logging

```python
# conftest.py
import pytest
import logging
from pytest_reportportal import RPLogger, RPLogHandler

@pytest.fixture(scope="session")
def rp_logger(request):
    """Setup ReportPortal logger for the session."""
    logging.setLoggerClass(RPLogger)
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.DEBUG)
    
    # Create and configure ReportPortal handler
    rp_handler = RPLogHandler(request.node.config.py_test_service)
    rp_handler.setLevel(logging.INFO)
    
    return logger
```

```python
# test_basic.py
def test_simple_calculation(rp_logger):
    """Test basic arithmetic operations."""
    rp_logger.info("Starting calculation test")
    
    # Test addition
    result = 2 + 3
    rp_logger.info(f"2 + 3 = {result}")
    assert result == 5
    
    # Test multiplication
    result = 4 * 5
    rp_logger.info(f"4 * 5 = {result}")
    assert result == 20
    
    rp_logger.info("Calculation test completed successfully")
```

### Test with Docstring Description

```python
def test_user_authentication():
    """
    Test user authentication functionality.
    
    This test verifies that users can successfully authenticate
    with valid credentials and receive appropriate error messages
    for invalid credentials.
    
    Test Steps:
    1. Attempt login with valid credentials
    2. Verify successful authentication
    3. Attempt login with invalid credentials
    4. Verify error message
    """
    # Test implementation
    assert True
```

### Test with Markers and Tags

```python
import pytest

@pytest.mark.smoke
@pytest.mark.api
@pytest.mark.priority_high
def test_api_health_check(rp_logger):
    """Test API health endpoint availability."""
    rp_logger.info("Checking API health endpoint")
    
    # Simulate API call
    response_status = 200
    rp_logger.info(f"API responded with status: {response_status}")
    
    assert response_status == 200
    rp_logger.info("API health check passed")

@pytest.mark.ui
@pytest.mark.slow
def test_ui_login_flow(rp_logger):
    """Test complete UI login workflow."""
    rp_logger.info("Starting UI login test")
    
    # Simulate UI interactions
    steps = [
        "Navigate to login page",
        "Enter username",
        "Enter password", 
        "Click login button",
        "Verify dashboard loads"
    ]
    
    for step in steps:
        rp_logger.info(f"Step: {step}")
        # Simulate step execution
        assert True
    
    rp_logger.info("UI login test completed")
```

## Advanced Usage Examples

### Test with File Attachments

```python
import subprocess
import json
import tempfile
import os

def test_system_info_collection(rp_logger):
    """Collect and attach system information."""
    rp_logger.info("Collecting system information")
    
    # Collect system info
    try:
        system_info = subprocess.check_output("uname -a", shell=True, text=True)
        rp_logger.info(
            "System information collected",
            attachment={
                "name": "system_info.txt",
                "data": system_info,
                "mime": "text/plain"
            }
        )
    except Exception as e:
        rp_logger.error(f"Failed to collect system info: {e}")
    
    # Collect memory info
    try:
        memory_info = subprocess.check_output("free -h", shell=True, text=True)
        rp_logger.info(
            "Memory information collected",
            attachment={
                "name": "memory_info.txt", 
                "data": memory_info,
                "mime": "text/plain"
            }
        )
    except Exception as e:
        rp_logger.error(f"Failed to collect memory info: {e}")
    
    assert True

def test_json_report_attachment(rp_logger):
    """Test with JSON report attachment."""
    test_results = {
        "test_name": "json_attachment_test",
        "status": "passed",
        "duration": 1.25,
        "steps": [
            {"step": 1, "action": "Initialize", "result": "success"},
            {"step": 2, "action": "Execute", "result": "success"},
            {"step": 3, "action": "Verify", "result": "success"}
        ]
    }
    
    rp_logger.info(
        "Test results summary",
        attachment={
            "name": "test_results.json",
            "data": json.dumps(test_results, indent=2),
            "mime": "application/json"
        }
    )
    
    assert test_results["status"] == "passed"
```

### Parametrized Tests with Hierarchy

```python
import pytest

# Test data
test_data = [
    ("user1", "pass1", True),
    ("user2", "pass2", True), 
    ("invalid", "wrong", False),
    ("", "", False)
]

@pytest.mark.parametrize("username,password,expected", test_data)
def test_login_scenarios(username, password, expected, rp_logger):
    """Test various login scenarios with different credentials."""
    rp_logger.info(f"Testing login with username: '{username}'")
    
    # Simulate login attempt
    if username and password and username.startswith("user"):
        result = True
    else:
        result = False
    
    rp_logger.info(f"Login result: {result}, Expected: {expected}")
    assert result == expected

# API endpoint test data
api_endpoints = [
    ("/api/users", 200),
    ("/api/posts", 200),
    ("/api/invalid", 404),
    ("/api/forbidden", 403)
]

@pytest.mark.parametrize("endpoint,expected_status", api_endpoints)
def test_api_endpoints(endpoint, expected_status, rp_logger):
    """Test API endpoint responses."""
    rp_logger.info(f"Testing endpoint: {endpoint}")
    
    # Simulate API call
    if "invalid" in endpoint:
        status = 404
    elif "forbidden" in endpoint:
        status = 403
    else:
        status = 200
    
    rp_logger.info(f"Endpoint {endpoint} returned status: {status}")
    assert status == expected_status
```

### Error Handling and Issue Reporting

```python
import pytest

@pytest.mark.xfail(reason="Known issue with database connection")
def test_database_connection(rp_logger):
    """Test database connectivity - known to fail."""
    rp_logger.info("Attempting database connection")
    
    # Simulate database connection failure
    connection_successful = False
    
    if not connection_successful:
        rp_logger.error("Database connection failed")
        
    assert connection_successful

@pytest.mark.issue(issue_id="BUG-123", reason="API timeout issue", issue_type="PB")
def test_api_with_known_issue(rp_logger):
    """Test API call with known timeout issue."""
    rp_logger.info("Testing API with timeout issue")
    
    # Simulate timeout
    timeout_occurred = True
    
    if timeout_occurred:
        rp_logger.error("API call timed out after 30 seconds")
        pytest.fail("API timeout occurred")

@pytest.mark.issue(issue_id=["BUG-456", "BUG-789"], issue_type="AB")
def test_multiple_issues(rp_logger):
    """Test with multiple related issues."""
    rp_logger.info("Running test with multiple known issues")
    
    # Test logic that might fail due to multiple issues
    assert False, "Multiple issues causing test failure"
```

### Custom Test Classes

```python
import pytest

class TestUserManagement:
    """Test suite for user management functionality."""
    
    @pytest.fixture(autouse=True)
    def setup_user_test(self, rp_logger):
        """Setup for user management tests."""
        rp_logger.info("Setting up user management test environment")
        self.test_user = {"id": 1, "name": "testuser", "email": "test@example.com"}
        rp_logger.info(f"Test user created: {self.test_user}")
    
    def test_create_user(self, rp_logger):
        """Test user creation functionality."""
        rp_logger.info("Testing user creation")
        
        # Simulate user creation
        new_user = {"id": 2, "name": "newuser", "email": "new@example.com"}
        rp_logger.info(f"Creating user: {new_user}")
        
        # Verify user creation
        assert new_user["id"] is not None
        assert new_user["email"] is not None
        rp_logger.info("User creation successful")
    
    def test_update_user(self, rp_logger):
        """Test user update functionality."""
        rp_logger.info("Testing user update")
        
        # Update test user
        self.test_user["name"] = "updated_user"
        rp_logger.info(f"Updated user: {self.test_user}")
        
        assert self.test_user["name"] == "updated_user"
        rp_logger.info("User update successful")
    
    def test_delete_user(self, rp_logger):
        """Test user deletion functionality."""
        rp_logger.info("Testing user deletion")
        
        user_id = self.test_user["id"]
        rp_logger.info(f"Deleting user with ID: {user_id}")
        
        # Simulate deletion
        deleted = True
        
        assert deleted
        rp_logger.info("User deletion successful")

class TestAPIIntegration:
    """Test suite for API integration tests."""
    
    @pytest.fixture(autouse=True)
    def setup_api_test(self, rp_logger):
        """Setup API test environment."""
        rp_logger.info("Setting up API test environment")
        self.base_url = "https://api.example.com"
        self.headers = {"Content-Type": "application/json"}
        rp_logger.info(f"API base URL: {self.base_url}")
    
    def test_get_request(self, rp_logger):
        """Test GET API request."""
        endpoint = f"{self.base_url}/users"
        rp_logger.info(f"Making GET request to: {endpoint}")
        
        # Simulate API response
        response = {"status": 200, "data": [{"id": 1, "name": "user1"}]}
        rp_logger.info(f"API response: {response}")
        
        assert response["status"] == 200
        assert len(response["data"]) > 0
    
    def test_post_request(self, rp_logger):
        """Test POST API request."""
        endpoint = f"{self.base_url}/users"
        payload = {"name": "newuser", "email": "new@example.com"}
        
        rp_logger.info(f"Making POST request to: {endpoint}")
        rp_logger.info(f"Payload: {payload}")
        
        # Simulate API response
        response = {"status": 201, "data": {"id": 123, **payload}}
        rp_logger.info(f"API response: {response}")
        
        assert response["status"] == 201
        assert response["data"]["id"] is not None
```

## Integration Examples

### Selenium WebDriver Integration

```python
import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import base64

class TestWebApplication:
    """Web application test suite with Selenium."""
    
    @pytest.fixture(autouse=True)
    def setup_driver(self, rp_logger):
        """Setup Selenium WebDriver."""
        rp_logger.info("Initializing WebDriver")
        
        options = webdriver.ChromeOptions()
        options.add_argument("--headless")
        options.add_argument("--no-sandbox")
        
        self.driver = webdriver.Chrome(options=options)
        self.driver.implicitly_wait(10)
        
        rp_logger.info("WebDriver initialized successfully")
        
        yield
        
        # Cleanup
        rp_logger.info("Closing WebDriver")
        self.driver.quit()
    
    def take_screenshot(self, rp_logger, name="screenshot"):
        """Take and attach screenshot."""
        try:
            screenshot = self.driver.get_screenshot_as_png()
            rp_logger.info(
                f"Screenshot taken: {name}",
                attachment={
                    "name": f"{name}.png",
                    "data": base64.b64encode(screenshot).decode(),
                    "mime": "image/png"
                }
            )
        except Exception as e:
            rp_logger.error(f"Failed to take screenshot: {e}")
    
    def test_login_page(self, rp_logger):
        """Test login page functionality."""
        rp_logger.info("Testing login page")
        
        # Navigate to login page
        self.driver.get("https://example.com/login")
        rp_logger.info("Navigated to login page")
        self.take_screenshot(rp_logger, "login_page")
        
        # Find and fill login form
        username_field = self.driver.find_element(By.ID, "username")
        password_field = self.driver.find_element(By.ID, "password")
        login_button = self.driver.find_element(By.ID, "login-btn")
        
        username_field.send_keys("testuser")
        password_field.send_keys("testpass")
        rp_logger.info("Filled login credentials")
        
        # Take screenshot before login
        self.take_screenshot(rp_logger, "before_login")
        
        # Click login
        login_button.click()
        rp_logger.info("Clicked login button")
        
        # Wait for dashboard
        WebDriverWait(self.driver, 10).until(
            EC.presence_of_element_located((By.ID, "dashboard"))
        )
        
        # Take screenshot after login
        self.take_screenshot(rp_logger, "after_login")
        
        # Verify successful login
        assert "dashboard" in self.driver.current_url
        rp_logger.info("Login successful - dashboard loaded")
```

### Database Testing Integration

```python
import pytest
import sqlite3
import json

class TestDatabase:
    """Database integration test suite."""
    
    @pytest.fixture(autouse=True)
    def setup_database(self, rp_logger):
        """Setup test database."""
        rp_logger.info("Setting up test database")
        
        # Create in-memory database
        self.conn = sqlite3.connect(":memory:")
        self.cursor = self.conn.cursor()
        
        # Create test table
        self.cursor.execute("""
            CREATE TABLE users (
                id INTEGER PRIMARY KEY,
                name TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        
        # Insert test data
        test_users = [
            ("John Doe", "john@example.com"),
            ("Jane Smith", "jane@example.com"),
            ("Bob Johnson", "bob@example.com")
        ]
        
        self.cursor.executemany(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            test_users
        )
        self.conn.commit()
        
        rp_logger.info(f"Test database created with {len(test_users)} users")
        
        yield
        
        # Cleanup
        rp_logger.info("Closing database connection")
        self.conn.close()
    
    def test_select_users(self, rp_logger):
        """Test selecting users from database."""
        rp_logger.info("Testing user selection")
        
        self.cursor.execute("SELECT * FROM users")
        users = self.cursor.fetchall()
        
        rp_logger.info(f"Found {len(users)} users in database")
        
        # Log users as JSON attachment
        users_data = []
        for user in users:
            users_data.append({
                "id": user[0],
                "name": user[1], 
                "email": user[2],
                "created_at": user[3]
            })
        
        rp_logger.info(
            "Database query results",
            attachment={
                "name": "users_query_result.json",
                "data": json.dumps(users_data, indent=2),
                "mime": "application/json"
            }
        )
        
        assert len(users) == 3
        assert users[0][1] == "John Doe"
    
    def test_insert_user(self, rp_logger):
        """Test inserting new user."""
        rp_logger.info("Testing user insertion")
        
        new_user = ("Alice Brown", "alice@example.com")
        
        self.cursor.execute(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            new_user
        )
        self.conn.commit()
        
        rp_logger.info(f"Inserted new user: {new_user}")
        
        # Verify insertion
        self.cursor.execute("SELECT COUNT(*) FROM users")
        count = self.cursor.fetchone()[0]
        
        rp_logger.info(f"Total users after insertion: {count}")
        assert count == 4
    
    def test_update_user(self, rp_logger):
        """Test updating user information."""
        rp_logger.info("Testing user update")
        
        # Update user with ID 1
        self.cursor.execute(
            "UPDATE users SET name = ? WHERE id = ?",
            ("John Updated", 1)
        )
        self.conn.commit()
        
        rp_logger.info("Updated user with ID 1")
        
        # Verify update
        self.cursor.execute("SELECT name FROM users WHERE id = 1")
        updated_name = self.cursor.fetchone()[0]
        
        rp_logger.info(f"Updated name: {updated_name}")
        assert updated_name == "John Updated"
```

### REST API Testing with Requests

```python
import pytest
import requests
import json

class TestRESTAPI:
    """REST API test suite."""
    
    @pytest.fixture(autouse=True)
    def setup_api(self, rp_logger):
        """Setup API test environment."""
        rp_logger.info("Setting up API test environment")
        
        self.base_url = "https://jsonplaceholder.typicode.com"
        self.session = requests.Session()
        self.session.headers.update({
            "Content-Type": "application/json",
            "User-Agent": "pytest-reportportal-test"
        })
        
        rp_logger.info(f"API base URL: {self.base_url}")
    
    def log_api_call(self, rp_logger, method, url, response, request_data=None):
        """Log API call details."""
        call_details = {
            "method": method,
            "url": url,
            "status_code": response.status_code,
            "response_time": response.elapsed.total_seconds(),
            "request_data": request_data,
            "response_data": response.json() if response.content else None
        }
        
        rp_logger.info(
            f"{method} {url} - Status: {response.status_code}",
            attachment={
                "name": f"api_call_{method.lower()}.json",
                "data": json.dumps(call_details, indent=2),
                "mime": "application/json"
            }
        )
    
    def test_get_posts(self, rp_logger):
        """Test GET /posts endpoint."""
        rp_logger.info("Testing GET /posts endpoint")
        
        url = f"{self.base_url}/posts"
        response = self.session.get(url)
        
        self.log_api_call(rp_logger, "GET", url, response)
        
        assert response.status_code == 200
        posts = response.json()
        assert len(posts) > 0
        assert "title" in posts[0]
        
        rp_logger.info(f"Retrieved {len(posts)} posts successfully")
    
    def test_create_post(self, rp_logger):
        """Test POST /posts endpoint."""
        rp_logger.info("Testing POST /posts endpoint")
        
        url = f"{self.base_url}/posts"
        post_data = {
            "title": "Test Post",
            "body": "This is a test post created by pytest-reportportal",
            "userId": 1
        }
        
        response = self.session.post(url, json=post_data)
        
        self.log_api_call(rp_logger, "POST", url, response, post_data)
        
        assert response.status_code == 201
        created_post = response.json()
        assert created_post["title"] == post_data["title"]
        assert created_post["userId"] == post_data["userId"]
        
        rp_logger.info(f"Created post with ID: {created_post['id']}")
    
    def test_update_post(self, rp_logger):
        """Test PUT /posts/1 endpoint."""
        rp_logger.info("Testing PUT /posts/1 endpoint")
        
        url = f"{self.base_url}/posts/1"
        update_data = {
            "id": 1,
            "title": "Updated Test Post",
            "body": "This post has been updated",
            "userId": 1
        }
        
        response = self.session.put(url, json=update_data)
        
        self.log_api_call(rp_logger, "PUT", url, response, update_data)
        
        assert response.status_code == 200
        updated_post = response.json()
        assert updated_post["title"] == update_data["title"]
        
        rp_logger.info("Post updated successfully")
    
    def test_delete_post(self, rp_logger):
        """Test DELETE /posts/1 endpoint."""
        rp_logger.info("Testing DELETE /posts/1 endpoint")
        
        url = f"{self.base_url}/posts/1"
        response = self.session.delete(url)
        
        self.log_api_call(rp_logger, "DELETE", url, response)
        
        assert response.status_code == 200
        rp_logger.info("Post deleted successfully")
```

## Troubleshooting Examples

### Handling Connection Issues

```python
import pytest
from pytest_reportportal.service import PyTestServiceClass

def test_with_connection_handling(rp_logger):
    """Test with ReportPortal connection error handling."""
    rp_logger.info("Testing with connection handling")
    
    try:
        # Your test logic here
        result = perform_test_operation()
        rp_logger.info(f"Test operation result: {result}")
        assert result is not None
        
    except Exception as e:
        rp_logger.error(f"Test failed with error: {str(e)}")
        
        # Check if ReportPortal service is available
        service = PyTestServiceClass()
        if service.RP is None:
            rp_logger.warning("ReportPortal service not available, continuing test")
        
        # Re-raise the exception to fail the test
        raise

@pytest.fixture(scope="session")
def robust_rp_logger(request):
    """Robust ReportPortal logger with error handling."""
    import logging
    from pytest_reportportal import RPLogger, RPLogHandler
    
    logging.setLoggerClass(RPLogger)
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.DEBUG)
    
    try:
        # Try to create ReportPortal handler
        rp_handler = RPLogHandler(request.node.config.py_test_service)
        rp_handler.setLevel(logging.INFO)
        logger.addHandler(rp_handler)
        logger.info("ReportPortal logging enabled")
    except Exception as e:
        # Fall back to console logging if ReportPortal is unavailable
        console_handler = logging.StreamHandler()
        console_handler.setLevel(logging.INFO)
        logger.addHandler(console_handler)
        logger.warning(f"ReportPortal unavailable, using console logging: {e}")
    
    return logger
```

### Debugging Configuration Issues

```python
# conftest.py
import pytest
import logging

def pytest_configure(config):
    """Debug configuration issues."""
    logger = logging.getLogger(__name__)
    
    # Check required configuration
    required_settings = ['rp_uuid', 'rp_endpoint', 'rp_project']
    missing_settings = []
    
    for setting in required_settings:
        value = config.getini(setting)
        if not value:
            missing_settings.append(setting)
        else:
            logger.info(f"Configuration {setting}: {'*' * len(str(value))}")  # Mask sensitive data
    
    if missing_settings:
        logger.error(f"Missing required ReportPortal settings: {missing_settings}")
        logger.info("Please check your pytest.ini configuration file")
    
    # Check optional settings
    optional_settings = [
        'rp_launch', 'rp_launch_description', 'rp_launch_tags',
        'rp_log_batch_size', 'rp_ignore_errors', 'rp_verify_ssl'
    ]
    
    for setting in optional_settings:
        value = config.getini(setting)
        if value:
            logger.info(f"Optional setting {setting}: {value}")

@pytest.fixture(scope="session")
def debug_rp_service(request):
    """Debug ReportPortal service initialization."""
    logger = logging.getLogger(__name__)
    
    try:
        service = request.node.config.py_test_service
        logger.info(f"ReportPortal service initialized: {service is not None}")
        
        if hasattr(service, 'RP') and service.RP:
            logger.info("ReportPortal client is available")
            if hasattr(service.RP, 'rp_client'):
                logger.info("ReportPortal client connection established")
        else:
            logger.warning("ReportPortal client not available")
            
    except AttributeError as e:
        logger.error(f"ReportPortal service not properly configured: {e}")
    
    return service
```

## Best Practices

### 1. Structured Logging

```python
# Use structured logging with consistent format
def test_structured_logging(rp_logger):
    """Example of structured logging best practices."""
    
    # Use consistent log levels
    rp_logger.info("=== TEST START: Structured Logging Example ===")
    
    # Log test steps clearly
    rp_logger.info("STEP 1: Initialize test data")
    test_data = {"user_id": 123, "action": "login"}
    
    rp_logger.info("STEP 2: Perform test action")
    result = perform_action(test_data)
    
    rp_logger.info(f"STEP 3: Verify result - Expected: success, Actual: {result}")
    
    if result == "success":
        rp_logger.info("✅ TEST PASSED: All steps completed successfully")
    else:
        rp_logger.error("❌ TEST FAILED: Action did not complete successfully")
    
    rp_logger.info("=== TEST END ===")
    assert result == "success"
```

### 2. Effective Use of Attachments

```python
def test_attachment_best_practices(rp_logger):
    """Best practices for using attachments."""
    
    # Use meaningful attachment names
    config_data = {"timeout": 30, "retries": 3}
    rp_logger.info(
        "Test configuration loaded",
        attachment={
            "name": "test_config.json",
            "data": json.dumps(config_data, indent=2),
            "mime": "application/json"
        }
    )
    
    # Attach error logs when tests fail
    try:
        risky_operation()
    except Exception as e:
        error_details = {
            "error": str(e),
            "timestamp": datetime.now().isoformat(),
            "test_context": "attachment_best_practices"
        }
        
        rp_logger.error(
            "Test failed with exception",
            attachment={
                "name": "error_details.json",
                "data": json.dumps(error_details, indent=2),
                "mime": "application/json"
            }
        )
        raise
```

### 3. Test Organization

```python
# Organize tests in logical groups
class TestUserAuthentication:
    """Group related authentication tests."""
    
    @pytest.fixture(autouse=True)
    def setup_auth_test(self, rp_logger):
        rp_logger.info("Setting up authentication test environment")
        # Setup code
    
    def test_valid_login(self, rp_logger):
        """Test successful login with valid credentials."""
        pass
    
    def test_invalid_login(self, rp_logger):
        """Test failed login with invalid credentials."""
        pass
    
    def test_password_reset(self, rp_logger):
        """Test password reset functionality."""
        pass

class TestUserProfile:
    """Group related profile management tests."""
    
    def test_update_profile(self, rp_logger):
        """Test profile update functionality."""
        pass
    
    def test_delete_profile(self, rp_logger):
        """Test profile deletion functionality."""
        pass
```

### 4. Configuration Management

```bash
# Use different configurations for different environments
# Development
pytest -c pytest-dev.ini --reportportal tests/

# Staging  
pytest -c pytest-staging.ini --reportportal tests/

# Production smoke tests
pytest -c pytest-prod.ini --reportportal tests/smoke/
```

### 5. CI/CD Integration

```yaml
# GitHub Actions example
name: Test with ReportPortal
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
    
    - name: Run tests with ReportPortal
      env:
        RP_UUID: ${{ secrets.RP_UUID }}
        RP_ENDPOINT: ${{ secrets.RP_ENDPOINT }}
        RP_PROJECT: ${{ secrets.RP_PROJECT }}
      run: |
        pytest --reportportal \
               --rp-launch "GitHub Actions Build ${{ github.run_number }}" \
               --rp-launch-description "Automated tests for commit ${{ github.sha }}"
```

This comprehensive guide should help you effectively use pytest-reportportal in various scenarios and environments. Remember to always check the ReportPortal server status and configuration when troubleshooting issues.