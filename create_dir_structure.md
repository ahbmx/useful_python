# Python Function to Create a Directory Structure

Here's a Python function that creates a directory structure. It uses the `os` module to handle directory operations and includes error handling:

```python
import os

def create_directory_structure(base_path, structure):
    """
    Creates a directory structure from a nested dictionary.
    
    Args:
        base_path (str): The root directory where the structure should be created
        structure (dict): A dictionary representing the directory structure.
                          Keys are directory names, values can be either:
                          - None (empty directory)
                          - dict (subdirectory structure)
                          - str (file to create with given content)
    
    Returns:
        None
    
    Example:
        structure = {
            'project': {
                'src': {
                    'main.py': 'print("Hello World")',
                    'utils.py': '# Utility functions'
                },
                'tests': {
                    'test_main.py': '# Test cases'
                },
                'docs': None,
                'README.md': 'Project description'
            }
        }
        create_directory_structure('/path/to/base', structure)
    """
    try:
        # Create base path if it doesn't exist
        os.makedirs(base_path, exist_ok=True)
        
        for name, content in structure.items():
            path = os.path.join(base_path, name)
            
            if isinstance(content, dict):
                # It's a directory with sub-structure
                os.makedirs(path, exist_ok=True)
                create_directory_structure(path, content)
            elif content is None:
                # It's an empty directory
                os.makedirs(path, exist_ok=True)
            else:
                # It's a file with content
                with open(path, 'w') as f:
                    f.write(content)
                    
    except OSError as e:
        print(f"Error creating directory structure: {e}")
```

## How to Use This Function

1. Define your directory structure as a nested dictionary
2. Call the function with your base path and structure

### Example Usage:

```python
# Define your directory structure
project_structure = {
    'my_project': {
        'src': {
            '__init__.py': '',
            'main.py': 'def main():\n    print("Hello World")\n\nif __name__ == "__main__":\n    main()',
            'utils': {
                '__init__.py': '',
                'file_utils.py': '# File utility functions',
                'math_utils.py': '# Math utility functions'
            }
        },
        'tests': {
            '__init__.py': '',
            'test_main.py': '# Test cases for main.py'
        },
        'docs': None,  # Empty directory
        'data': {
            'raw': None,
            'processed': None
        },
        'README.md': '# My Awesome Project\n\nProject description goes here.',
        '.gitignore': '*.pyc\n__pycache__/\n*.log\n'
    }
}

# Create the structure
create_directory_structure('/path/to/your/projects', project_structure)
```

## Features:

1. Creates both directories and files
2. Handles nested directory structures
3. Can create empty directories (with `None` value)
4. Can create files with content
5. Uses `exist_ok=True` to avoid errors if directories already exist
6. Includes error handling for filesystem operations

You can modify this function to suit your specific needs, such as adding different file permissions or more complex directory creation logic.
