Here's a Linux-only Python function that finds and highlights regex matches with configurable colors, without using the `typing` module:

```python
import re

def highlight_regex(text, pattern, color='yellow', bold=False, underline=False, bg=False):
    """
    Highlights regex matches in text with specified color and style (Linux only)
    
    Args:
        text: Input text to search
        pattern: Regex pattern (string or compiled pattern)
        color: Color name (red, green, yellow, blue, magenta, cyan, white)
        bold: Make matches bold
        underline: Underline matches
        bg: Use background color instead of text color
    
    Returns:
        Text with matches highlighted
    
    Example:
        text = "Error: 404 Not Found"
        print(highlight_regex(text, r'\d+', color='red', bold=True))
    """
    # ANSI color codes
    colors = {
        'red': 31,
        'green': 32,
        'yellow': 33,
        'blue': 34,
        'magenta': 35,
        'cyan': 36,
        'white': 37
    }
    
    if color not in colors:
        raise ValueError(f"Invalid color. Choose from: {', '.join(colors.keys())}")
    
    # Build ANSI escape sequence
    styles = []
    if bold:
        styles.append('1')
    if underline:
        styles.append('4')
    
    color_code = colors[color]
    if bg:  # Background colors are +10 from foreground
        color_code += 10
    
    style_prefix = ';'.join(styles + [str(color_code)])
    
    # Compile pattern if it's a string
    if isinstance(pattern, str):
        pattern = re.compile(pattern)
    
    # Replacement function
    def replacer(match):
        matched_text = match.group()
        return f"\033[{style_prefix}m{matched_text}\033[0m"
    
    # Apply highlighting
    return pattern.sub(replacer, text)
```

### Usage Examples:

1. **Highlight all numbers in red:**
```python
log = "Processed 23 items with total cost $456.78"
print(highlight_regex(log, r'\d+', color='red'))
```

2. **Highlight email addresses with yellow background:**
```python
contact = "Email admin@example.com or support@company.org"
print(highlight_regex(contact, r'\S+@\S+', color='yellow', bg=True))
```

3. **Bold and underline error messages:**
```python
error = "ERROR: Disk full, WARNING: Memory low"
print(highlight_regex(error, r'ERROR|WARNING', color='red', bold=True, underline=True))
```

### Key Features:

1. Pure Linux solution using ANSI escape codes
2. No external dependencies
3. Supports:
   - 7 different colors
   - Bold/underline formatting
   - Foreground or background highlighting
4. Preserves original text structure
5. Proper ANSI code reset after each match

### Color Options:
- red
- green
- yellow  
- blue
- magenta
- cyan
- white

Just call the function with your text and regex pattern, and it will return the string with matches highlighted according to your specifications.
