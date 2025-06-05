# Python Logger Function with Configurable Output and Log Level

Here's a flexible logger function that can be configured to output to file, console, or both, and allows setting the log level using a string:

```python
import logging
import sys
from typing import Optional

def setup_logger(
    name: str,
    log_file: Optional[str] = None,
    console_output: bool = True,
    log_level: str = 'INFO'
) -> logging.Logger:
    """
    Configure and return a logger with specified settings.
    
    Args:
        name: Name of the logger (usually __name__)
        log_file: Path to log file (None for no file logging)
        console_output: Whether to log to console
        log_level: Logging level as string ('DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL')
    
    Returns:
        Configured logger instance
    """
    # Create logger
    logger = logging.getLogger(name)
    
    # Convert string log level to logging constant
    level = getattr(logging, log_level.upper(), None)
    if not isinstance(level, int):
        raise ValueError(f'Invalid log level: {log_level}')
    
    logger.setLevel(level)
    
    # Clear any existing handlers to avoid duplicate logs
    logger.handlers.clear()
    
    # Create formatter
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    
    # Add file handler if log_file is specified
    if log_file:
        file_handler = logging.FileHandler(log_file)
        file_handler.setLevel(level)
        file_handler.setFormatter(formatter)
        logger.addHandler(file_handler)
    
    # Add console handler if console_output is True
    if console_output:
        console_handler = logging.StreamHandler(sys.stdout)
        console_handler.setLevel(level)
        console_handler.setFormatter(formatter)
        logger.addHandler(console_handler)
    
    return logger


# Example usage
if __name__ == "__main__":
    # Logger with console output only, DEBUG level
    console_logger = setup_logger('console_logger', console_output=True, log_level='DEBUG')
    console_logger.debug('Debug message')
    console_logger.info('Info message')
    console_logger.warning('Warning message')
    
    # Logger with file output only, WARNING level
    file_logger = setup_logger('file_logger', log_file='app.log', console_output=False, log_level='WARNING')
    file_logger.info('This info message will not appear in the file')  # Won't be logged
    file_logger.warning('This warning will be in the file')
    
    # Logger with both file and console output, ERROR level
    combined_logger = setup_logger('combined_logger', log_file='combined.log', console_output=True, log_level='ERROR')
    combined_logger.warning('This warning will not appear')  # Won't be logged
    combined_logger.error('This error will appear in both places')
```

## Features:

1. **Flexible Output**: Can log to file, console, or both
2. **Configurable Log Level**: Accepts string values ('DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL')
3. **Clean Setup**: Removes any existing handlers to prevent duplicate logs
4. **Consistent Formatting**: Uses a standard format with timestamp, logger name, level, and message
5. **Type Hints**: Includes type annotations for better code clarity

## Usage Examples:

```python
# Basic console logger with INFO level (default)
logger = setup_logger(__name__)

# File logger with DEBUG level
logger = setup_logger(__name__, log_file='debug.log', console_output=False, log_level='DEBUG')

# Both file and console logger with WARNING level
logger = setup_logger(__name__, log_file='warnings.log', console_output=True, log_level='WARNING')
```

The logger will only output messages at or above the specified log level.
