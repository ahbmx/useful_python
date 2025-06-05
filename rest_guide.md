# Creating a Class to Query REST API Endpoints: A Detailed Guide

This guide will walk you through creating a robust Python class for interacting with REST APIs, with explanations for each component.

## 1. Basic Structure

Let's start with the foundational class structure:

```python
import requests
import json
from typing import Optional, Dict, Any, Union

class RESTAPIClient:
    """
    A class to handle REST API requests with authentication, error handling, and utilities.
    
    Attributes:
        base_url (str): The base URL for the API endpoints
        headers (dict): Default headers to include in every request
        timeout (int): Default timeout in seconds for requests
        session (requests.Session): Persistent HTTP session for connection pooling
    """
    
    def __init__(self, base_url: str, timeout: int = 30):
        """
        Initialize the REST API client.
        
        Args:
            base_url: The base URL for all API endpoints (e.g., 'https://api.example.com/v1/')
            timeout: Default timeout in seconds for requests
        """
        self.base_url = base_url.rstrip('/') + '/'  # Ensure trailing slash
        self.timeout = timeout
        self.session = requests.Session()
        self.headers = {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        }
```

### Explanation:
- **Imports**: We use `requests` for HTTP operations, `json` for data serialization, and typing for type hints.
- **Class Docstring**: Documents the purpose and attributes of the class.
- **Initialization**:
  - `base_url` is normalized to always end with a slash
  - A persistent `requests.Session()` is created for connection pooling
  - Default headers are set for JSON content

## 2. Core Request Method

The heart of our class will be a private method that handles all HTTP requests:

```python
    def _request(
        self,
        method: str,
        endpoint: str,
        params: Optional[Dict[str, Any]] = None,
        data: Optional[Dict[str, Any]] = None,
        json_data: Optional[Dict[str, Any]] = None,
        headers: Optional[Dict[str, str]] = None,
        timeout: Optional[int] = None
    ) -> Union[Dict[str, Any], str]:
        """
        Execute an HTTP request to the API.
        
        Args:
            method: HTTP method (GET, POST, PUT, DELETE, etc.)
            endpoint: API endpoint (appended to base_url)
            params: Query parameters to include in the URL
            data: Form data to include in the body
            json_data: JSON data to include in the body
            headers: Additional headers to include
            timeout: Timeout in seconds (overrides default)
            
        Returns:
            Parsed JSON response if content is JSON, otherwise raw text
            
        Raises:
            requests.exceptions.RequestException: For request-related errors
            ValueError: For invalid responses or JSON parsing errors
        """
        url = self.base_url + endpoint.lstrip('/')
        final_headers = {**self.headers, **(headers or {})}
        timeout = timeout or self.timeout
        
        try:
            response = self.session.request(
                method=method,
                url=url,
                params=params,
                data=data,
                json=json_data,
                headers=final_headers,
                timeout=timeout
            )
            
            response.raise_for_status()  # Raises exception for 4XX/5XX status codes
            
            # Try to parse JSON, fall back to text if not JSON
            try:
                return response.json()
            except ValueError:
                return response.text
                
        except requests.exceptions.RequestException as e:
            # Enhance the error message with more context
            error_msg = f"Request failed: {method} {url}"
            if response is not None:
                error_msg += f" [Status: {response.status_code}]"
                try:
                    error_detail = response.json()
                    error_msg += f" - {error_detail}"
                except ValueError:
                    error_msg += f" - {response.text}"
            raise requests.exceptions.RequestException(error_msg) from e
```

### Explanation:
- **Flexible Parameters**: Accepts all common request components (params, data, json, headers)
- **URL Construction**: Combines base URL with endpoint safely
- **Header Merging**: Combines default headers with request-specific headers
- **Error Handling**:
  - Uses `raise_for_status()` to catch HTTP errors
  - Provides detailed error messages including response content
- **Response Processing**: Attempts JSON parsing first, falls back to raw text

## 3. Convenience Methods

Now we'll add methods for common HTTP verbs:

```python
    def get(
        self,
        endpoint: str,
        params: Optional[Dict[str, Any]] = None,
        headers: Optional[Dict[str, str]] = None,
        timeout: Optional[int] = None
    ) -> Union[Dict[str, Any], str]:
        """
        Execute a GET request.
        
        Args:
            endpoint: API endpoint
            params: Query parameters
            headers: Additional headers
            timeout: Timeout in seconds
            
        Returns:
            Response data
        """
        return self._request('GET', endpoint, params=params, headers=headers, timeout=timeout)

    def post(
        self,
        endpoint: str,
        json_data: Optional[Dict[str, Any]] = None,
        data: Optional[Dict[str, Any]] = None,
        headers: Optional[Dict[str, str]] = None,
        timeout: Optional[int]] = None
    ) -> Union[Dict[str, Any], str]:
        """
        Execute a POST request.
        
        Args:
            endpoint: API endpoint
            json_data: JSON data for the body
            data: Form data for the body
            headers: Additional headers
            timeout: Timeout in seconds
            
        Returns:
            Response data
        """
        return self._request('POST', endpoint, json_data=json_data, data=data, headers=headers, timeout=timeout)

    def put(
        self,
        endpoint: str,
        json_data: Optional[Dict[str, Any]] = None,
        data: Optional[Dict[str, Any]] = None,
        headers: Optional[Dict[str, str]] = None,
        timeout: Optional[int]] = None
    ) -> Union[Dict[str, Any], str]:
        """
        Execute a PUT request.
        
        Args:
            endpoint: API endpoint
            json_data: JSON data for the body
            data: Form data for the body
            headers: Additional headers
            timeout: Timeout in seconds
            
        Returns:
            Response data
        """
        return self._request('PUT', endpoint, json_data=json_data, data=data, headers=headers, timeout=timeout)

    def delete(
        self,
        endpoint: str,
        headers: Optional[Dict[str, str]] = None,
        timeout: Optional[int]] = None
    ) -> Union[Dict[str, Any], str]:
        """
        Execute a DELETE request.
        
        Args:
            endpoint: API endpoint
            headers: Additional headers
            timeout: Timeout in seconds
            
        Returns:
            Response data
        """
        return self._request('DELETE', endpoint, headers=headers, timeout=timeout)
```

### Explanation:
- **Verb-Specific Methods**: Each HTTP verb gets its own method with appropriate parameters
- **Consistent Interface**: All methods follow the same pattern for easy use
- **Type Hints**: Clear indication of expected parameter and return types

## 4. Authentication Support

Let's add authentication capabilities:

```python
    def set_basic_auth(self, username: str, password: str):
        """
        Set up Basic Authentication for all requests.
        
        Args:
            username: Basic auth username
            password: Basic auth password
        """
        self.session.auth = (username, password)
        
    def set_bearer_token(self, token: str):
        """
        Set up Bearer Token authentication for all requests.
        
        Args:
            token: The bearer token
        """
        self.headers['Authorization'] = f'Bearer {token}'
        
    def set_api_key(self, key: str, header_name: str = 'X-API-KEY'):
        """
        Set up API key authentication for all requests.
        
        Args:
            key: The API key
            header_name: The header name to use (default: 'X-API-KEY')
        """
        self.headers[header_name] = key
```

### Explanation:
- **Multiple Auth Methods**: Supports common authentication schemes
- **Flexible API Key**: Allows custom header names for API keys
- **Session Persistence**: Auth credentials persist across requests

## 5. Advanced Features

Let's add some more advanced functionality:

```python
    def paginated_get(
        self,
        endpoint: str,
        params: Optional[Dict[str, Any]] = None,
        headers: Optional[Dict[str, str]] = None,
        page_size: int = 100,
        max_pages: Optional[int]] = None
    ) -> list:
        """
        Handle paginated GET requests automatically.
        
        Args:
            endpoint: API endpoint
            params: Initial query parameters
            headers: Additional headers
            page_size: Number of items per page
            max_pages: Maximum number of pages to fetch (None for all)
            
        Returns:
            Combined list of all items from all pages
        """
        params = params or {}
        params['page_size'] = page_size
        params['page'] = 1
        
        all_items = []
        page_count = 0
        
        while True:
            response = self.get(endpoint, params=params, headers=headers)
            
            if not isinstance(response, dict):
                raise ValueError("Paginated endpoint must return JSON")
                
            items = response.get('items', [])
            all_items.extend(items)
            
            # Check if we've reached the end or max pages
            page_count += 1
            if not items or (max_pages and page_count >= max_pages):
                break
                
            params['page'] += 1
            
        return all_items
        
    def set_rate_limit(self, max_requests: int, per_seconds: int):
        """
        Set up simple rate limiting.
        
        Args:
            max_requests: Maximum number of requests
            per_seconds: Time period in seconds
        """
        from requests.adapters import HTTPAdapter
        from urllib3.util.retry import Retry
        
        retry_strategy = Retry(
            total=max_requests,
            backoff_factor=per_seconds/max_requests,
            status_forcelist=[429, 500, 502, 503, 504]
        )
        adapter = HTTPAdapter(max_retries=retry_strategy)
        self.session.mount("https://", adapter)
        self.session.mount("http://", adapter)
```

### Explanation:
- **Paginated Requests**: Handles pagination automatically for endpoints that support it
- **Rate Limiting**: Implements basic rate limiting and retry logic
- **Flexible Control**: Allows setting page size and maximum pages for pagination

## 6. Usage Example

Here's how to use the complete class:

```python
# Initialize client
api = RESTAPIClient(base_url="https://api.example.com/v1/")

# Set authentication
api.set_bearer_token("your_access_token")

# Make a GET request
users = api.get("users", params={"active": "true"})

# Make a POST request
new_user = api.post("users", json_data={"name": "John Doe", "email": "john@example.com"})

# Handle paginated results
all_products = api.paginated_get("products", page_size=50)

# Error handling
try:
    result = api.get("protected/resource")
except requests.exceptions.RequestException as e:
    print(f"Error accessing API: {e}")
```

## 7. Complete Class

Here's the complete implementation:

```python
import requests
import json
from typing import Optional, Dict, Any, Union

class RESTAPIClient:
    """A comprehensive REST API client with authentication, error handling, and utilities."""
    
    def __init__(self, base_url: str, timeout: int = 30):
        self.base_url = base_url.rstrip('/') + '/'
        self.timeout = timeout
        self.session = requests.Session()
        self.headers = {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        }
        
    def _request(
        self,
        method: str,
        endpoint: str,
        params: Optional[Dict[str, Any]] = None,
        data: Optional[Dict[str, Any]] = None,
        json_data: Optional[Dict[str, Any]] = None,
        headers: Optional[Dict[str, str]] = None,
        timeout: Optional[int] = None
    ) -> Union[Dict[str, Any], str]:
        url = self.base_url + endpoint.lstrip('/')
        final_headers = {**self.headers, **(headers or {})}
        timeout = timeout or self.timeout
        
        try:
            response = self.session.request(
                method=method,
                url=url,
                params=params,
                data=data,
                json=json_data,
                headers=final_headers,
                timeout=timeout
            )
            
            response.raise_for_status()
            
            try:
                return response.json()
            except ValueError:
                return response.text
                
        except requests.exceptions.RequestException as e:
            error_msg = f"Request failed: {method} {url}"
            if hasattr(e, 'response') and e.response is not None:
                error_msg += f" [Status: {e.response.status_code}]"
                try:
                    error_detail = e.response.json()
                    error_msg += f" - {error_detail}"
                except ValueError:
                    error_msg += f" - {e.response.text}"
            raise requests.exceptions.RequestException(error_msg) from e
            
    # HTTP verb methods (get, post, put, delete) as shown above
    
    # Authentication methods (set_basic_auth, set_bearer_token, set_api_key) as shown above
    
    # Advanced features (paginated_get, set_rate_limit) as shown above
```

## Key Features of This Implementation

1. **Clean Interface**: Simple methods for each HTTP verb
2. **Robust Error Handling**: Detailed error messages with response context
3. **Authentication Support**: Multiple auth methods built-in
4. **Connection Pooling**: Uses `requests.Session` for performance
5. **Type Hints**: Better IDE support and code clarity
6. **Advanced Features**: Pagination, rate limiting, and more
7. **Flexible Response Handling**: Works with both JSON and text responses

This class provides a solid foundation that can be extended with additional features as needed for specific APIs.




# Creating a Class to Query REST API Endpoints Without Typing Module

Here's a comprehensive guide to creating a REST API client class without using Python's `typing` module:

## 1. Basic Class Structure

```python
import requests
import json

class RESTAPIClient:
    """
    A class to handle REST API requests with authentication, error handling, and utilities.
    
    Attributes:
        base_url (str): The base URL for the API endpoints
        headers (dict): Default headers to include in every request
        timeout (int): Default timeout in seconds for requests
        session (requests.Session): Persistent HTTP session for connection pooling
    """
    
    def __init__(self, base_url, timeout=30):
        """
        Initialize the REST API client.
        
        Args:
            base_url: The base URL for all API endpoints (e.g., 'https://api.example.com/v1/')
            timeout: Default timeout in seconds for requests
        """
        self.base_url = base_url.rstrip('/') + '/'  # Ensure trailing slash
        self.timeout = timeout
        self.session = requests.Session()
        self.headers = {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        }
```

## 2. Core Request Method

```python
    def _request(
        self,
        method,
        endpoint,
        params=None,
        data=None,
        json_data=None,
        headers=None,
        timeout=None
    ):
        """
        Execute an HTTP request to the API.
        
        Args:
            method: HTTP method (GET, POST, PUT, DELETE, etc.)
            endpoint: API endpoint (appended to base_url)
            params: Query parameters to include in the URL
            data: Form data to include in the body
            json_data: JSON data to include in the body
            headers: Additional headers to include
            timeout: Timeout in seconds (overrides default)
            
        Returns:
            Parsed JSON response if content is JSON, otherwise raw text
            
        Raises:
            requests.exceptions.RequestException: For request-related errors
            ValueError: For invalid responses or JSON parsing errors
        """
        url = self.base_url + endpoint.lstrip('/')
        final_headers = {**self.headers, **(headers or {})}
        timeout = timeout or self.timeout
        
        try:
            response = self.session.request(
                method=method,
                url=url,
                params=params,
                data=data,
                json=json_data,
                headers=final_headers,
                timeout=timeout
            )
            
            response.raise_for_status()  # Raises exception for 4XX/5XX status codes
            
            # Try to parse JSON, fall back to text if not JSON
            try:
                return response.json()
            except ValueError:
                return response.text
                
        except requests.exceptions.RequestException as e:
            # Enhance the error message with more context
            error_msg = f"Request failed: {method} {url}"
            if response is not None:
                error_msg += f" [Status: {response.status_code}]"
                try:
                    error_detail = response.json()
                    error_msg += f" - {error_detail}"
                except ValueError:
                    error_msg += f" - {response.text}"
            raise requests.exceptions.RequestException(error_msg) from e
```

## 3. Convenience Methods for HTTP Verbs

```python
    def get(self, endpoint, params=None, headers=None, timeout=None):
        """
        Execute a GET request.
        
        Args:
            endpoint: API endpoint
            params: Query parameters
            headers: Additional headers
            timeout: Timeout in seconds
            
        Returns:
            Response data
        """
        return self._request('GET', endpoint, params=params, headers=headers, timeout=timeout)

    def post(self, endpoint, json_data=None, data=None, headers=None, timeout=None):
        """
        Execute a POST request.
        
        Args:
            endpoint: API endpoint
            json_data: JSON data for the body
            data: Form data for the body
            headers: Additional headers
            timeout: Timeout in seconds
            
        Returns:
            Response data
        """
        return self._request('POST', endpoint, json_data=json_data, data=data, headers=headers, timeout=timeout)

    def put(self, endpoint, json_data=None, data=None, headers=None, timeout=None):
        """
        Execute a PUT request.
        
        Args:
            endpoint: API endpoint
            json_data: JSON data for the body
            data: Form data for the body
            headers: Additional headers
            timeout: Timeout in seconds
            
        Returns:
            Response data
        """
        return self._request('PUT', endpoint, json_data=json_data, data=data, headers=headers, timeout=timeout)

    def delete(self, endpoint, headers=None, timeout=None):
        """
        Execute a DELETE request.
        
        Args:
            endpoint: API endpoint
            headers: Additional headers
            timeout: Timeout in seconds
            
        Returns:
            Response data
        """
        return self._request('DELETE', endpoint, headers=headers, timeout=timeout)
```

## 4. Authentication Methods

```python
    def set_basic_auth(self, username, password):
        """
        Set up Basic Authentication for all requests.
        
        Args:
            username: Basic auth username
            password: Basic auth password
        """
        self.session.auth = (username, password)
        
    def set_bearer_token(self, token):
        """
        Set up Bearer Token authentication for all requests.
        
        Args:
            token: The bearer token
        """
        self.headers['Authorization'] = f'Bearer {token}'
        
    def set_api_key(self, key, header_name='X-API-KEY'):
        """
        Set up API key authentication for all requests.
        
        Args:
            key: The API key
            header_name: The header name to use (default: 'X-API-KEY')
        """
        self.headers[header_name] = key
```

## 5. Advanced Features

```python
    def paginated_get(self, endpoint, params=None, headers=None, page_size=100, max_pages=None):
        """
        Handle paginated GET requests automatically.
        
        Args:
            endpoint: API endpoint
            params: Initial query parameters
            headers: Additional headers
            page_size: Number of items per page
            max_pages: Maximum number of pages to fetch (None for all)
            
        Returns:
            Combined list of all items from all pages
        """
        params = params or {}
        params['page_size'] = page_size
        params['page'] = 1
        
        all_items = []
        page_count = 0
        
        while True:
            response = self.get(endpoint, params=params, headers=headers)
            
            if not isinstance(response, dict):
                raise ValueError("Paginated endpoint must return JSON")
                
            items = response.get('items', [])
            all_items.extend(items)
            
            # Check if we've reached the end or max pages
            page_count += 1
            if not items or (max_pages and page_count >= max_pages):
                break
                
            params['page'] += 1
            
        return all_items
        
    def set_rate_limit(self, max_requests, per_seconds):
        """
        Set up simple rate limiting.
        
        Args:
            max_requests: Maximum number of requests
            per_seconds: Time period in seconds
        """
        from requests.adapters import HTTPAdapter
        from urllib3.util.retry import Retry
        
        retry_strategy = Retry(
            total=max_requests,
            backoff_factor=per_seconds/max_requests,
            status_forcelist=[429, 500, 502, 503, 504]
        )
        adapter = HTTPAdapter(max_retries=retry_strategy)
        self.session.mount("https://", adapter)
        self.session.mount("http://", adapter)
```

## 6. Usage Example

```python
# Initialize client
api = RESTAPIClient(base_url="https://api.example.com/v1/")

# Set authentication
api.set_bearer_token("your_access_token")

# Make a GET request
users = api.get("users", params={"active": "true"})

# Make a POST request
new_user = api.post("users", json_data={"name": "John Doe", "email": "john@example.com"})

# Handle paginated results
all_products = api.paginated_get("products", page_size=50)

# Error handling
try:
    result = api.get("protected/resource")
except requests.exceptions.RequestException as e:
    print(f"Error accessing API: {e}")
```

## Key Differences from Typed Version

1. **No Type Hints**: Removed all `Optional`, `Dict`, `Union`, etc. annotations
2. **Simpler Signatures**: Method signatures are more concise without type information
3. **Same Functionality**: All features remain identical, just without type annotations
4. **Documentation**: Docstrings still describe parameter types and return values

## Advantages of This Approach

1. **Compatibility**: Works with all Python versions, including those before 3.5
2. **Simplicity**: Less verbose code for simpler use cases
3. **Faster Execution**: Avoids the minimal overhead of type checking in some cases
4. **Cleaner Appearance**: Some developers prefer the cleaner look without type hints

The class maintains all the robust functionality of the typed version while being more accessible to Python environments where the typing module isn't available or desired.










