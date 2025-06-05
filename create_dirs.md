Here's a simple Python function that creates directories from either a string path or a list of paths:

```python
import os

def create_directories(paths):
    """
    Creates directories from either a single path string or a list of paths.
    
    Args:
        paths (str or list): Either a single directory path as a string,
                            or a list of directory paths to create.
    
    Example usage:
        create_directories("path/to/dir")  # Single directory
        create_directories(["path/to/dir1", "path/to/dir2"])  # Multiple directories
    """
    if isinstance(paths, str):
        paths = [paths]  # Convert single path to list for uniform handling
    
    for path in paths:
        try:
            os.makedirs(path, exist_ok=True)
            print(f"Created directory: {path}")
        except OSError as e:
            print(f"Error creating directory {path}: {e}")
```

### Features:
1. Accepts either a single path string or a list of paths
2. Creates parent directories as needed (like `mkdir -p` in Unix)
3. Won't raise an error if directories already exist (`exist_ok=True`)
4. Provides feedback about what was created
5. Includes basic error handling

### Example Usage:
```python
# Single directory
create_directories("project/src/utils")

# Multiple directories
create_directories([
    "project/src",
    "project/tests",
    "project/docs",
    "project/data/raw",
    "project/data/processed"
])
```

This is a more focused version of the previous function that just handles directory creation (not files) and accepts either strings or lists as input.
