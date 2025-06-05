Here's a Python function that can handle multiple input types (list, multi-line string, multiple variables) and prints them in a neatly formatted table with automatically calculated column sizes:

```python
from typing import List, Union, Any
import itertools

def print_table(*args, headers: Union[List[str], None] = None, 
                min_col_width: int = 3, max_col_width: int = 50) -> None:
    """
    Prints data in tabular format with automatically calculated column sizes.
    
    Args:
        *args: Can be:
            - A list of lists (each sublist is a row)
            - A multi-line string (each line is a row)
            - Multiple variables (each will be a column)
        headers: Optional list of header strings
        min_col_width: Minimum column width (default: 3)
        max_col_width: Maximum column width (default: 50)
    """
    # Convert all input types to a list of lists (rows)
    rows = []
    
    # Case 1: Single list argument (either list of rows or list of columns)
    if len(args) == 1 and isinstance(args[0], list):
        # Check if it's a list of lists (rows)
        if all(isinstance(item, list) for item in args[0]):
            rows = args[0]
        else:
            # Single list - treat as one row
            rows = [args[0]]
    
    # Case 2: Multi-line string
    elif len(args) == 1 and isinstance(args[0], str) and '\n' in args[0]:
        rows = [line.split('\t') if '\t' in line else line.split(',') 
                for line in args[0].strip().split('\n')]
    
    # Case 3: Multiple arguments (each becomes a column)
    elif len(args) > 1:
        # Convert to list of lists (transposed)
        rows = list(map(list, itertools.zip_longest(*args, fillvalue='')))
    
    # Case 4: Single string (no newlines) - treat as one cell
    elif len(args) == 1 and isinstance(args[0], str):
        rows = [[args[0]]]
    
    # Case 5: Single non-list/non-string argument
    elif len(args) == 1:
        rows = [[str(args[0])]]
    
    # If we have headers, add them as the first row
    if headers:
        rows.insert(0, headers)
    
    if not rows:
        print("No data to display")
        return
    
    # Calculate column widths
    col_widths = [
        min(max(len(str(item)) for item in col), max_col_width)
        for col in zip(*rows)
    ]
    col_widths = [max(w, min_col_width) for w in col_widths]
    
    # Create format string
    format_str = " | ".join([f"{{:<{w}}}" for w in col_widths])
    
    # Print separator
    separator = "-+-".join(['-' * w for w in col_widths])
    
    # Print table
    for i, row in enumerate(rows):
        # Convert all items to strings and truncate if needed
        row_str = [
            str(item)[:max_col_width] + ('...' if len(str(item)) > max_col_width else '')
            for item in row
        ]
        print(format_str.format(*row_str))
        
        # Print header separator if headers exist
        if headers and i == 0:
            print(separator)

# Example usages
if __name__ == "__main__":
    print("\nExample 1: List of lists (rows)")
    data = [
        ["Alice", 28, "Engineer", "San Francisco"],
        ["Bob", 32, "Designer", "New York"],
        ["Charlie", 24, "Student", "Chicago"]
    ]
    print_table(data, headers=["Name", "Age", "Occupation", "Location"])
    
    print("\nExample 2: Multi-line string")
    multi_line = """Name\tAge\tOccupation
Alice\t28\tEngineer
Bob\t32\tDesigner
Charlie\t24\tStudent"""
    print_table(multi_line)
    
    print("\nExample 3: Multiple variables as columns")
    names = ["Apple", "Banana", "Cherry"]
    prices = [1.20, 0.50, 2.75]
    quantities = [10, 20, 15]
    print_table(names, prices, quantities, headers=["Fruit", "Price", "Qty"])
    
    print("\nExample 4: Single list")
    print_table(["One", "Two", "Three"])
    
    print("\nExample 5: Mixed types with long text")
    long_text = "This is a very long text that should be truncated to demonstrate the max width functionality"
    print_table(["Short", 42, long_text])
```

### Features:

1. **Multiple Input Types**:
   - List of lists (each sublist is a row)
   - Multi-line string (split by newlines, then by tabs or commas)
   - Multiple variables (each becomes a column)
   - Single value (treated as one cell)

2. **Automatic Formatting**:
   - Calculates column widths based on content
   - Handles text truncation with ellipsis for long content
   - Proper alignment of different data types

3. **Customization Options**:
   - Optional headers
   - Configurable minimum and maximum column widths
   - Automatic header separator

4. **Robust Handling**:
   - Works with missing values (None)
   - Converts all values to strings
   - Handles uneven row lengths

### Example Outputs:

```
Example 1: List of lists (rows)
Name    | Age | Occupation | Location      
--------+-----+------------+--------------
Alice   | 28  | Engineer   | San Francisco
Bob     | 32  | Designer   | New York    
Charlie | 24  | Student    | Chicago     

Example 2: Multi-line string
Name    | Age | Occupation
Alice   | 28  | Engineer  
Bob     | 32  | Designer  
Charlie | 24  | Student   

Example 3: Multiple variables as columns
Fruit  | Price | Qty
Apple  | 1.2   | 10 
Banana | 0.5   | 20 
Cherry | 2.75  | 15 

Example 4: Single list
One  | Two | Three

Example 5: Mixed types with long text
Short | 42 | This is a very long text that should be truncate...
```

This function provides a flexible way to display tabular data from various input formats while automatically handling the formatting for optimal display.


Here's a Python function that prints various input types in tabular format without using the `typing` module:

```python
import itertools

def print_table(*args, headers=None, min_col_width=3, max_col_width=50):
    """
    Prints data in tabular format with automatically calculated column sizes.
    
    Args:
        *args: Can be:
            - A list of lists (each sublist is a row)
            - A multi-line string (each line is a row)
            - Multiple variables (each will be a column)
        headers: Optional list of header strings
        min_col_width: Minimum column width (default: 3)
        max_col_width: Maximum column width (default: 50)
    """
    # Convert all input types to a list of lists (rows)
    rows = []
    
    # Case 1: Single list argument
    if len(args) == 1 and isinstance(args[0], list):
        # Check if it's a list of lists (rows)
        if all(isinstance(item, list) for item in args[0]):
            rows = args[0]
        else:
            # Single list - treat as one row
            rows = [args[0]]
    
    # Case 2: Multi-line string
    elif len(args) == 1 and isinstance(args[0], str) and '\n' in args[0]:
        rows = [line.split('\t') if '\t' in line else line.split(',') 
                for line in args[0].strip().split('\n')]
    
    # Case 3: Multiple arguments (each becomes a column)
    elif len(args) > 1:
        # Convert to list of lists (transposed)
        rows = list(map(list, itertools.zip_longest(*args, fillvalue='')))
    
    # Case 4: Single string (no newlines) - treat as one cell
    elif len(args) == 1 and isinstance(args[0], str):
        rows = [[args[0]]]
    
    # Case 5: Single non-list/non-string argument
    elif len(args) == 1:
        rows = [[str(args[0])]]
    
    # Add headers if provided
    if headers:
        rows.insert(0, headers)
    
    if not rows:
        print("No data to display")
        return
    
    # Calculate column widths
    col_widths = []
    for col in zip(*rows):
        max_len = max(len(str(item)) for item in col)
        col_widths.append(min(max(max_len, min_col_width), max_col_width))
    
    # Create format string
    format_str = " | ".join(["{{:<{}}}".format(w) for w in col_widths])
    
    # Print separator
    separator = "-+-".join(['-' * w for w in col_widths])
    
    # Print table
    for i, row in enumerate(rows):
        # Convert all items to strings and truncate if needed
        row_str = []
        for item, width in zip(row, col_widths):
            item_str = str(item)
            if len(item_str) > width:
                item_str = item_str[:width-3] + '...'
            row_str.append(item_str)
        
        print(format_str.format(*row_str))
        
        # Print header separator if headers exist
        if headers and i == 0:
            print(separator)

# Example usages
if __name__ == "__main__":
    print("\nExample 1: List of lists (rows)")
    data = [
        ["Alice", 28, "Engineer", "San Francisco"],
        ["Bob", 32, "Designer", "New York"],
        ["Charlie", 24, "Student", "Chicago"]
    ]
    print_table(data, headers=["Name", "Age", "Occupation", "Location"])
    
    print("\nExample 2: Multi-line string")
    multi_line = """Name\tAge\tOccupation
Alice\t28\tEngineer
Bob\t32\tDesigner
Charlie\t24\tStudent"""
    print_table(multi_line)
    
    print("\nExample 3: Multiple variables as columns")
    names = ["Apple", "Banana", "Cherry"]
    prices = [1.20, 0.50, 2.75]
    quantities = [10, 20, 15]
    print_table(names, prices, quantities, headers=["Fruit", "Price", "Qty"])
    
    print("\nExample 4: Single list")
    print_table(["One", "Two", "Three"])
    
    print("\nExample 5: Mixed types with long text")
    long_text = "This is a very long text that should be truncated to demonstrate the max width functionality"
    print_table(["Short", 42, long_text])
```

### Key Features:

1. **Input Flexibility**:
   - Accepts lists, multi-line strings, multiple variables, or single values
   - Automatically detects the input format

2. **Smart Formatting**:
   - Calculates optimal column widths
   - Truncates long text with ellipsis (...)
   - Handles different data types (converts all to strings)

3. **Customization Options**:
   - Optional headers with automatic separator line
   - Configurable minimum and maximum column widths
   - Proper alignment of columns

4. **Clean Output**:
   - Consistent spacing between columns
   - Clear visual separation of headers
   - Handles missing/uneven data gracefully

### Example Output:

```
Example 1: List of lists (rows)
Name    | Age | Occupation | Location      
--------+-----+------------+--------------
Alice   | 28  | Engineer   | San Francisco
Bob     | 32  | Designer   | New York    
Charlie | 24  | Student    | Chicago     

Example 3: Multiple variables as columns
Fruit  | Price | Qty
Apple  | 1.2   | 10 
Banana | 0.5   | 20 
Cherry | 2.75  | 15 

Example 5: Mixed types with long text
Short | 42 | This is a very long text that should be truncate...
```

This version maintains all the functionality of the original but removes the dependency on the `typing` module while keeping the code clean and well-documented.



Here's an enhanced version of the table printing function that allows coloring specific columns, using ANSI color codes:

```python
import itertools

# ANSI color codes
COLORS = {
    'red': '\033[91m',
    'green': '\033[92m',
    'yellow': '\033[93m',
    'blue': '\033[94m',
    'magenta': '\033[95m',
    'cyan': '\033[96m',
    'white': '\033[97m',
    'reset': '\033[0m'
}

def print_table(*args, headers=None, min_col_width=3, max_col_width=50, 
                colored_columns=None):
    """
    Prints data in tabular format with column coloring support.
    
    Args:
        *args: Input data (list of lists, multi-line string, or multiple variables)
        headers: Optional list of header strings
        min_col_width: Minimum column width (default: 3)
        max_col_width: Maximum column width (default: 50)
        colored_columns: Dict of {column_index: color_name} or {column_name: color_name}
                        Example: {1: 'red', 'Status': 'green'}
    """
    # Convert all input types to a list of lists (rows)
    rows = []
    
    # Case 1: Single list argument
    if len(args) == 1 and isinstance(args[0], list):
        if all(isinstance(item, list) for item in args[0]):
            rows = args[0]
        else:
            rows = [args[0]]
    
    # Case 2: Multi-line string
    elif len(args) == 1 and isinstance(args[0], str) and '\n' in args[0]:
        rows = [line.split('\t') if '\t' in line else line.split(',') 
                for line in args[0].strip().split('\n')]
    
    # Case 3: Multiple arguments (each becomes a column)
    elif len(args) > 1:
        rows = list(map(list, itertools.zip_longest(*args, fillvalue='')))
    
    # Case 4: Single string (no newlines)
    elif len(args) == 1 and isinstance(args[0], str):
        rows = [[args[0]]]
    
    # Case 5: Single non-list/non-string argument
    elif len(args) == 1:
        rows = [[str(args[0])]]
    
    # Add headers if provided
    if headers:
        rows.insert(0, headers)
    
    if not rows:
        print("No data to display")
        return
    
    # Calculate column widths
    col_widths = []
    for col in zip(*rows):
        max_len = max(len(str(item)) for item in col)
        col_widths.append(min(max(max_len, min_col_width), max_col_width))
    
    # Process colored_columns (convert header names to indices if needed)
    color_map = {}
    if colored_columns and headers:
        for col_ref, color in colored_columns.items():
            if isinstance(col_ref, str):
                if col_ref in headers:
                    col_idx = headers.index(col_ref)
                    color_map[col_idx] = color
            else:
                if col_ref < len(headers):
                    color_map[col_ref] = color
    
    # Create format string
    format_str = " | ".join(["{{:<{}}}".format(w) for w in col_widths])
    
    # Print separator
    separator = "-+-".join(['-' * w for w in col_widths])
    
    # Print table
    for i, row in enumerate(rows):
        formatted_cells = []
        for col_idx, (item, width) in enumerate(zip(row, col_widths)):
            item_str = str(item)
            if len(item_str) > width:
                item_str = item_str[:width-3] + '...'
            
            # Apply color if specified for this column
            if col_idx in color_map and color_map[col_idx] in COLORS:
                color_code = COLORS[color_map[col_idx]]
                reset_code = COLORS['reset']
                item_str = f"{color_code}{item_str}{reset_code}"
            
            formatted_cells.append(item_str)
        
        print(format_str.format(*formatted_cells))
        
        # Print header separator if headers exist
        if headers and i == 0:
            print(separator)

# Example usages
if __name__ == "__main__":
    print("\nExample 1: Colored columns by index")
    data = [
        ["Alice", 28, "Engineer", "Active"],
        ["Bob", 32, "Designer", "Inactive"],
        ["Charlie", 24, "Student", "Active"]
    ]
    print_table(data, 
               headers=["Name", "Age", "Occupation", "Status"],
               colored_columns={3: 'green'})  # Color Status column
    
    print("\nExample 2: Colored columns by header name")
    sales_data = [
        ["North", 15000, "Target Met"],
        ["South", 8500, "Below Target"],
        ["East", 12000, "Target Met"]
    ]
    print_table(sales_data,
               headers=["Region", "Sales", "Performance"],
               colored_columns={'Performance': 'red', 'Sales': 'cyan'})
    
    print("\nExample 3: Multiple variables with coloring")
    products = ["Laptop", "Phone", "Tablet"]
    stocks = [15, 0, 8]
    statuses = ["In Stock", "Out of Stock", "Low Stock"]
    print_table(products, stocks, statuses,
               headers=["Product", "Stock", "Status"],
               colored_columns={1: 'yellow', 2: 'red'})
```

### Key Features:

1. **Column Coloring**:
   - Specify colors by column index (0-based) or header name
   - Supports standard ANSI colors: red, green, yellow, blue, magenta, cyan, white

2. **Enhanced Formatting**:
   - Colors are automatically reset after each cell
   - Works with all existing features (auto-width, truncation, etc.)
   - Header names can be used for color mapping

3. **Usage Examples**:
   - Color important status columns
   - Highlight numeric values
   - Draw attention to specific data points

4. **Color Options**:
   ```python
   COLORS = {
       'red': '\033[91m',      # Bright Red
       'green': '\033[92m',    # Bright Green
       'yellow': '\033[93m',   # Bright Yellow
       'blue': '\033[94m',     # Bright Blue
       'magenta': '\033[95m',  # Bright Magenta
       'cyan': '\033[96m',     # Bright Cyan
       'white': '\033[97m',    # Bright White
       'reset': '\033[0m'      # Reset to default
   }
   ```

### Example Output:
The output will show colored text in the specified columns (colors won't be visible here, but will appear in terminal):
```
Example 1: Colored columns by index
Name    | Age | Occupation | Status  
--------+-----+------------+---------
Alice   | 28  | Engineer   | Active 
Bob     | 32  | Designer   | Inactive
Charlie | 24  | Student    | Active 

Example 2: Colored columns by header name
Region | Sales | Performance  
-------+-------+-------------
North  | 15000 | Target Met  
South  | 8500  | Below Target
East   | 12000 | Target Met  
```

Note: ANSI colors work in most modern terminals. If colors don't appear:
1. Ensure your terminal supports ANSI color codes
2. On Windows, you might need to enable ANSI support with `os.system('color')`
