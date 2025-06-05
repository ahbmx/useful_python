Here's a version of the logger function that doesn't use the `typing` module:

```python
import logging
import sys

def setup_logger(
    name,
    log_file=None,
    console_output=True,
    log_level='INFO'
):
    """
    Configure and return a logger with specified settings.
    
    Args:
        name: Name of the logger (usually __name__)
        log_file: Path to log file (None for no file logging)
        console_output: Whether to log to console (True/False)
        log_level: Logging level as string ('DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL')
    
    Returns:
        Configured logger instance
    """
    # Create logger
    logger = logging.getLogger(name)
    
    # Convert string log level to logging constant
    level = getattr(logging, log_level.upper(), None)
    if not isinstance(level, int):
        raise ValueError('Invalid log level: %s' % log_level)
    
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

Key changes made:
1. Removed the `from typing import Optional` import
2. Removed all type hints from the function signature
3. Changed the error message formatting from f-string to %-formatting for broader Python version compatibility

The functionality remains exactly the same - you can still configure file/console output and set the log level using a string. The only difference is the lack of type annotations, which makes the code compatible with older Python versions that don't support the typing module.
