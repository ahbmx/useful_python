Here's a Python function to highlight strings with different colors using ANSI escape codes:

```python
def highlight(text, color='yellow', bold=False, underline=False):
    """
    Highlight text with different colors and styles using ANSI escape codes.
    
    Args:
        text (str): Text to highlight
        color (str): Color name (red, green, yellow, blue, magenta, cyan, white)
        bold (bool): Whether to make text bold
        underline (bool): Whether to underline text
    
    Returns:
        str: Highlighted text with ANSI escape codes
    
    Example:
        print(highlight("Warning!", color="red", bold=True))
        print(highlight("Success", color="green"))
    """
    color_codes = {
        'red': 91,
        'green': 92,
        'yellow': 93,
        'blue': 94,
        'magenta': 95,
        'cyan': 96,
        'white': 97
    }
    
    if color.lower() not in color_codes:
        raise ValueError(f"Invalid color. Choose from: {', '.join(color_codes.keys())}")
    
    code = color_codes[color.lower()]
    styles = []
    
    if bold:
        styles.append('1')
    if underline:
        styles.append('4')
    
    style_prefix = ';'.join(styles) + ';' if styles else ''
    
    return f"\033[{style_prefix}{code}m{text}\033[0m"
```

### Usage Examples:

```python
# Basic highlighting
print(highlight("Important message", color="yellow"))
print(highlight("Error occurred", color="red"))
print(highlight("Success!", color="green"))

# With styles
print(highlight("Critical warning!", color="red", bold=True))
print(highlight("Click here", color="blue", underline=True))
print(highlight("Special notice", color="magenta", bold=True, underline=True))
```

### Features:

1. Supports multiple colors (red, green, yellow, blue, magenta, cyan, white)
2. Optional bold and underline formatting
3. Input validation for colors
4. Resets formatting after the text
5. Works in most terminals that support ANSI color codes

### For Windows Users:

If you're using Windows (especially older versions), you may need to enable ANSI support first:

```python
import os
import platform

if platform.system() == "Windows":
    os.system("color")  # Enable ANSI colors on Windows
```

### Alternative (using colorama):

For better cross-platform support, you can use the `colorama` package:

```python
from colorama import Fore, Style, init

def highlight(text, color='yellow', bold=False, underline=False):
    """Colorama version with cross-platform support"""
    init()  # Initialize colorama
    
    colors = {
        'red': Fore.RED,
        'green': Fore.GREEN,
        'yellow': Fore.YELLOW,
        'blue': Fore.BLUE,
        'magenta': Fore.MAGENTA,
        'cyan': Fore.CYAN,
        'white': Fore.WHITE
    }
    
    if color.lower() not in colors:
        raise ValueError(f"Invalid color. Choose from: {', '.join(colors.keys())}")
    
    result = colors[color.lower()] + text + Style.RESET_ALL
    
    if bold:
        result = Style.BRIGHT + result
    if underline:
        result += Style.RESET_ALL + '\033[4m' + text + Style.RESET_ALL
    
    return result
```
