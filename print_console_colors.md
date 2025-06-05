# Python Function to Print Available Console Colors and Combinations

Here's a Python function that displays all available console colors and their combinations using ANSI escape codes:

```python
def print_console_colors():
    """
    Prints all available console colors and combinations with their ANSI codes.
    """
    # Text colors
    colors = {
        'Black': '30',
        'Red': '31',
        'Green': '32',
        'Yellow': '33',
        'Blue': '34',
        'Magenta': '35',
        'Cyan': '36',
        'White': '37',
        'Bright Black': '90',
        'Bright Red': '91',
        'Bright Green': '92',
        'Bright Yellow': '93',
        'Bright Blue': '94',
        'Bright Magenta': '95',
        'Bright Cyan': '96',
        'Bright White': '97'
    }
    
    # Background colors
    bg_colors = {
        'Black': '40',
        'Red': '41',
        'Green': '42',
        'Yellow': '43',
        'Blue': '44',
        'Magenta': '45',
        'Cyan': '46',
        'White': '47',
        'Bright Black': '100',
        'Bright Red': '101',
        'Bright Green': '102',
        'Bright Yellow': '103',
        'Bright Blue': '104',
        'Bright Magenta': '105',
        'Bright Cyan': '106',
        'Bright White': '107'
    }
    
    # Text styles
    styles = {
        'Normal': '0',
        'Bold': '1',
        'Dim': '2',
        'Italic': '3',
        'Underline': '4',
        'Blink': '5',
        'Reverse': '7',
        'Hidden': '8',
        'Strikethrough': '9'
    }
    
    # Print header
    print("\n=== AVAILABLE CONSOLE COLORS AND COMBINATIONS ===\n")
    
    # Print basic colors
    print("BASIC COLORS:")
    for name, code in colors.items():
        print(f"\033[{code}m{name.ljust(12)}\033[0m", end=' ')
    print("\n")
    
    # Print background colors
    print("BACKGROUND COLORS:")
    for name, code in bg_colors.items():
        print(f"\033[{code}m{' ' * 12}\033[0m", end=' ')
        print(f"\033[0m{name.ljust(12)}\033[0m", end=' ')
    print("\n")
    
    # Print text styles
    print("TEXT STYLES:")
    for name, code in styles.items():
        print(f"\033[{code}m{name.ljust(12)}\033[0m", end=' ')
    print("\n")
    
    # Print some combinations
    print("SOME COMBINATIONS:")
    for fg_name, fg_code in colors.items():
        for bg_name, bg_code in bg_colors.items():
            if fg_name == bg_name:  # Skip same color combinations
                continue
            print(f"\033[{fg_code};{bg_code}m{fg_name[:3]}/{bg_name[:3]}\033[0m", end=' ')
        print()
    
    # Print styled combinations
    print("\nSTYLED COMBINATIONS:")
    style_examples = [
        ("Bold", "1"),
        ("Underline", "4"),
        ("Bold+Underline", "1;4"),
        ("Italic", "3"),
        ("Reverse", "7")
    ]
    for style_name, style_code in style_examples:
        for color_name, color_code in colors.items():
            print(f"\033[{style_code};{color_code}m{style_name[:3]}/{color_name[:3]}\033[0m", end=' ')
        print()
    
    print("\n=== END OF COLOR DEMONSTRATION ===")

# Call the function
print_console_colors()
```

## How to Use This Function

1. Simply call `print_console_colors()` in your Python script
2. The function will display:
   - All basic text colors
   - All background colors
   - Different text styles
   - Various combinations of foreground/background colors
   - Some styled text examples

## Notes

- This uses ANSI escape codes which work in most modern terminals
- Some styles (like Blink) may not be supported in all terminals
- The appearance may vary depending on your terminal emulator
- For Windows, you might need to enable ANSI support with `os.system('color')` or use the `colorama` library

For Windows compatibility, you can initialize colorama first:

```python
import colorama
colorama.init()
print_console_colors()
```
