Here's an updated version that:
1. If the output file exists, returns its content instead of executing the command
2. If no output file is specified, creates a sanitized filename from the command
3. Only executes the command if the output file doesn't exist
4. Still captures both stdout and stderr

```python
import subprocess
import re
from pathlib import Path

def sanitize_filename(command):
    """Convert a command to a safe filename."""
    if isinstance(command, list):
        cmd_str = ' '.join(command)
    else:
        cmd_str = str(command)
    
    # Remove special characters and replace spaces with underscores
    sanitized = re.sub(r'[^\w\-_\. ]', '', cmd_str)
    sanitized = re.sub(r'\s+', '_', sanitized.strip())
    
    # Trim to reasonable length and add .log extension
    return f"{sanitized[:200]}.log" if sanitized else "command_output.log"

def execute_command_to_file(command, output_file=None, force_execute=False):
    """
    Execute a command and redirect both stdout and stderr to a file.
    If file exists and force_execute is False, return its content instead.
    
    Args:
        command (str or list): The command to execute
        output_file (str or Path, optional): Output file path
        force_execute (bool): If True, always execute the command
    
    Returns:
        tuple: (return_code, output_content)
               return_code is None if reading from existing file
    """
    # Determine output filename
    if output_file is None:
        output_file = sanitize_filename(command)
    
    output_path = Path(output_file)
    
    # If file exists and we're not forcing execution, return its content
    if output_path.exists() and not force_execute:
        with open(output_path, 'r') as f:
            return (None, f.read())
    
    # Execute the command
    try:
        result = subprocess.run(
            command,
            stdout=subprocess.PIPE,
            stderr=subprocess.STDOUT,
            shell=isinstance(command, str),
            text=True,
            check=False
        )
        
        output_content = result.stdout
        
        # Write output to file
        output_path.parent.mkdir(parents=True, exist_ok=True)
        with open(output_path, 'w') as f:
            f.write(output_content)
            
        return (result.returncode, output_content)
    
    except Exception as e:
        return (-1, f"Command execution failed: {str(e)}")

# Example usage:
if __name__ == "__main__":
    # Example 1: With specified filename
    return_code, output = execute_command_to_file(["ls", "-l"], "directory_listing.txt")
    print(f"Return code: {return_code}")
    print(f"Output:\n{output[:200]}...")  # Print first 200 chars
    
    # Example 2: Auto-generated filename
    return_code, output = execute_command_to_file("echo 'Hello World' && date")
    print(f"\nAuto-generated filename example:")
    print(f"Return code: {return_code}")
    print(f"Output:\n{output}")
    
    # Example 3: Read existing file without execution
    _, cached_output = execute_command_to_file(["ls", "-l"], "directory_listing.txt")
    print(f"\nCached content:\n{cached_output[:200]}...")
    
    # Example 4: Force execution even if file exists
    return_code, output = execute_command_to_file(
        ["ls", "-l"], 
        "directory_listing.txt",
        force_execute=True
    )
    print(f"\nForced execution return code: {return_code}")
```

Key improvements:
1. New `sanitize_filename()` function creates safe filenames from commands
2. If output file exists and `force_execute` is False, returns cached content
3. Returns a tuple with (return_code, output_content)
4. Better error handling
5. Maintains all previous functionality

The filename sanitization:
- Removes special characters
- Replaces spaces with underscores
- Limits filename length
- Adds .log extension

Example auto-generated filenames:
- `ls -l` → `ls_l.log`
- `echo "Hello" && date` → `echo_Hello__date.log`
