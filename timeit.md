# Python Function Timer Decorator

Here's a simple yet powerful decorator that times function execution and stores the results in a list:

```python
import time

# List to store all timing results
timing_results = []

def timeit(func):
    """Decorator that times function execution and stores results"""
    def wrapper(*args, **kwargs):
        start_time = time.perf_counter()
        result = func(*args, **kwargs)
        end_time = time.perf_counter()
        
        timing_data = {
            'function_name': func.__name__,
            'args': args,
            'kwargs': kwargs,
            'result': result,
            'execution_time': end_time - start_time,
            'timestamp': time.time()
        }
        
        timing_results.append(timing_data)
        return result
    return wrapper

# Alternative version as a class decorator
class TimeIt:
    """Class-based decorator that stores timing results"""
    _results = []
    
    def __init__(self, func):
        self.func = func
    
    def __call__(self, *args, **kwargs):
        start_time = time.perf_counter()
        result = self.func(*args, **kwargs)
        end_time = time.perf_counter()
        
        timing_data = {
            'function_name': self.func.__name__,
            'args': args,
            'kwargs': kwargs,
            'result': result,
            'execution_time': end_time - start_time,
            'timestamp': time.time()
        }
        
        TimeIt._results.append(timing_data)
        return result
    
    @classmethod
    def get_results(cls):
        return cls._results
    
    @classmethod
    def clear_results(cls):
        cls._results = []
```

## How to Use the Decorators

### Using the simple function decorator:
```python
@timeit
def calculate_sum(n):
    return sum(range(n))

# When you call the function, it gets timed automatically
calculate_sum(1000000)

# Access all timing results
print("Timing results:", timing_results)
```

### Using the class-based decorator:
```python
@TimeIt
def process_data(data):
    return [x * 2 for x in data]

process_data([1, 2, 3, 4, 5])

# Access results through the class
print("All results:", TimeIt.get_results())

# Clear results if needed
TimeIt.clear_results()
```

## Features

1. **Simple Decorator Syntax**: Just add `@timeit` above your function
2. **Comprehensive Tracking**: Stores:
   - Function name
   - Arguments passed
   - Return value
   - Execution time
   - Timestamp
3. **Two Implementations**:
   - Simple function decorator (global list)
   - Class-based decorator (encapsulated results)
4. **No Dependencies**: Uses only standard library modules
5. **Flexible Access**: Results can be retrieved or cleared

## Advanced Usage Example

```python
# Track multiple functions
@timeit
def heavy_computation(n):
    return sum(i**2 for i in range(n))

@TimeIt
def data_processing(data):
    return sorted(data) * 100

# Run functions
heavy_computation(1000000)
data_processing([5, 2, 8, 1, 9])

# Analyze results
print("\nFunction performance:")
for result in timing_results + TimeIt.get_results():
    name = result['function_name']
    time_taken = result['execution_time']
    print(f"{name} took {time_taken:.4f} seconds")
```

You can extend this by adding features like:
- Saving results to a file
- Calculating statistics (average/min/max times)
- Adding context managers for timing code blocks
- Thread-safe implementations for concurrent use
