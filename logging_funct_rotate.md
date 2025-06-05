Here's an enhanced version of the logger function that adds log rotation by days, without using the `typing` module:

```python
import logging
import sys
from logging.handlers import TimedRotatingFileHandler

def setup_logger(
    name,
    log_file=None,
    console_output=True,
    log_level='INFO',
    rotate_days=0
):
    """
    Configure and return a logger with specified settings.
    
    Args:
        name: Name of the logger (usually __name__)
        log_file: Path to log file (None for no file logging)
        console_output: Whether to log to console (True/False)
        log_level: Logging level as string ('DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL')
        rotate_days: Number of days to keep rotated log files (0 for no rotation)
    
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
        if rotate_days > 0:
            # Use rotating file handler
            file_handler = TimedRotatingFileHandler(
                log_file,
                when='midnight',
                interval=1,
                backupCount=rotate_days
            )
            file_handler.suffix = "%Y-%m-%d.log"  # Add date to rotated files
        else:
            # Use regular file handler
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
    
    # Logger with file output only, WARNING level, rotating every day (keeps 7 days)
    file_logger = setup_logger(
        'file_logger', 
        log_file='app.log', 
        console_output=False, 
        log_level='WARNING',
        rotate_days=7
    )
    file_logger.info('This info message will not appear in the file')  # Won't be logged
    file_logger.warning('This warning will be in the file')
    
    # Logger with both file and console output, ERROR level, no rotation
    combined_logger = setup_logger(
        'combined_logger', 
        log_file='combined.log', 
        console_output=True, 
        log_level='ERROR',
        rotate_days=0
    )
    combined_logger.warning('This warning will not appear')  # Won't be logged
    combined_logger.error('This error will appear in both places')
```

Key enhancements:

1. Added `rotate_days` parameter:
   - When set to 0 (default), logs won't rotate
   - When set to a positive number, logs will rotate daily and keep the specified number of days

2. Used `TimedRotatingFileHandler` from `logging.handlers` for daily rotation:
   - Rotates logs at midnight
   - Adds date suffix to rotated files (e.g., "app.log.2023-11-15")
   - Automatically deletes old logs when exceeding the `rotate_days` count

3. Maintained all previous functionality:
   - Still configurable for file/console output
   - Still accepts log level as string
   - Still avoids duplicate handlers

The rotation feature helps manage log file sizes and automatically archives older logs while keeping a specified number of recent log files.
