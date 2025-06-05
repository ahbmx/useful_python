# Python Function to Search for Multiple Keys in Dictionary or JSON

Here's a Python function that searches for multiple keys in a dictionary or JSON object without using the typing module:

```python
import json

def search_multiple_keys(data, keys, nested=False):
    """
    Search for multiple keys in a dictionary or JSON object and return their values.
    
    Args:
        data: Dictionary or JSON string to search in
        keys: List of keys to search for
        nested: If True, searches nested dictionaries recursively (default: False)
    
    Returns:
        Dictionary with found keys and their values
    """
    # If input is JSON string, parse it to dict
    if isinstance(data, str):
        try:
            data = json.loads(data)
        except json.JSONDecodeError:
            return {"error": "Invalid JSON string"}
    
    if not isinstance(data, dict):
        return {"error": "Input must be a dictionary or JSON string"}
    
    result = {}
    
    def search_recursive(d, current_path=""):
        for key, value in d.items():
            full_key = f"{current_path}.{key}" if current_path else key
            
            # Check if current key is in our search list
            if key in keys:
                result[key] = value
            
            # If nested search is enabled and value is a dict, search recursively
            if nested and isinstance(value, dict):
                search_recursive(value, full_key)
            # Also check lists for nested dictionaries if nested search is enabled
            elif nested and isinstance(value, list):
                for item in value:
                    if isinstance(item, dict):
                        search_recursive(item, full_key)
    
    search_recursive(data)
    
    # Add not found keys with None value
    for key in keys:
        if key not in result:
            result[key] = None
    
    return result


# Example usage
if __name__ == "__main__":
    # Example dictionary
    sample_dict = {
        "name": "John",
        "age": 30,
        "address": {
            "street": "123 Main St",
            "city": "New York",
            "zip": "10001"
        },
        "contacts": [
            {"type": "email", "value": "john@example.com"},
            {"type": "phone", "value": "555-1234"}
        ]
    }
    
    # Example JSON string
    sample_json = json.dumps(sample_dict)
    
    # Search in dictionary
    print("Dictionary search:")
    print(search_multiple_keys(sample_dict, ["name", "age", "street", "type"]))
    
    # Search in dictionary with nested=True
    print("\nNested dictionary search:")
    print(search_multiple_keys(sample_dict, ["name", "street", "type"], nested=True))
    
    # Search in JSON
    print("\nJSON search:")
    print(search_multiple_keys(sample_json, ["name", "age", "nonexistent"]))
```

## Key Changes Made:
1. Removed all imports and references to the `typing` module
2. Removed type hints from function parameters and return value
3. Kept all the functionality exactly the same

The function works exactly as before, just without type annotations. It still:
- Handles both dictionaries and JSON strings
- Supports searching for multiple keys at once
- Offers optional recursive nested searching
- Returns a comprehensive result with None for missing keys
- Includes error handling for invalid inputs
