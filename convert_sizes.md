

Here's a Python function that converts between different storage size units (from bytes up to petabytes) with customizable precision:

```python
import os
script_dir = os.path.dirname(os.path.abspath(__file__))
```


```python
def convert_storage_size(value, from_unit='B', to_unit='B', precision=2):
    """
    Convert between digital storage units with customizable precision.
    
    Args:
        value (float): The value to convert
        from_unit (str): The unit to convert from (case-insensitive)
                        Options: 'B', 'KB', 'MB', 'GB', 'TB', 'PB'
        to_unit (str): The unit to convert to (case-insensitive)
                      Options: 'B', 'KB', 'MB', 'GB', 'TB', 'PB'
        precision (int): Number of decimal places to round to (default: 2)
    
    Returns:
        float: The converted value with specified precision
    
    Example:
        >>> convert_storage_size(1024, 'KB', 'MB')
        1.0
        >>> convert_storage_size(1.5, 'TB', 'GB', precision=0)
        1536
    """
    # Standardize unit case
    from_unit = from_unit.upper()
    to_unit = to_unit.upper()
    
    # Conversion factors (binary standard - powers of 1024)
    units = {
        'B': 1,
        'KB': 1024,
        'MB': 1024**2,
        'GB': 1024**3,
        'TB': 1024**4,
        'PB': 1024**5
    }
    
    # Validate units
    if from_unit not in units or to_unit not in units:
        raise ValueError(f"Invalid unit. Use: {', '.join(units.keys())}")
    
    # Convert to bytes first, then to target unit
    bytes_value = value * units[from_unit]
    converted_value = bytes_value / units[to_unit]
    
    # Apply precision
    if precision is not None:
        converted_value = round(converted_value, precision)
        if precision == 0:
            converted_value = int(converted_value)
    
    return converted_value
```

### Usage Examples:

```python
# Convert 1024 KB to MB
print(convert_storage_size(1024, 'KB', 'MB'))  # Output: 1.0

# Convert 1.5 TB to GB with no decimals
print(convert_storage_size(1.5, 'TB', 'GB', 0))  # Output: 1536

# Convert 500 MB to GB with 4 decimal places
print(convert_storage_size(500, 'MB', 'GB', 4))  # Output: 0.4883

# Convert 1 PB to TB
print(convert_storage_size(1, 'PB', 'TB'))  # Output: 1024.0
```

### Features:

1. Supports all common storage units (Bytes, KB, MB, GB, TB, PB)
2. Uses binary (1024-based) conversion factors
3. Customizable precision (set to None to get full precision)
4. Automatic conversion to integer when precision=0
5. Case-insensitive unit names
6. Input validation with helpful error messages

### Note:

This function uses binary (IEC) standard where:
- 1 KB = 1024 B
- 1 MB = 1024 KB
- etc.

If you need decimal (SI) standard conversion (where 1 KB = 1000 B), just change the conversion factors to powers of 1000.
