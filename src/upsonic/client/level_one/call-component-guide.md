# Comprehensive Guide to Upsonic Call System

## Basic Usage

### 1. Simple Calls
```python
from upsonic import Task, UpsonicClient

# Initialize client
client = UpsonicClient("localhost")

# Basic text task
task = Task("What is machine learning?")
result = client.call(task)
```

### 2. Model Selection
```python
# Using specific models
task = Task("Explain quantum computing")
result = client.call(task, llm_model="openai/gpt-4")

# Different models for different needs
basic_result = client.call(task, llm_model="openai/gpt-3.5-turbo")
advanced_result = client.call(task, llm_model="anthropic/claude-2")
```

## Structured Responses

### 1. Basic Response Formats
```python
from upsonic import ObjectResponse

class WeatherResponse(ObjectResponse):
    temperature: float
    condition: str
    humidity: int

task = Task(
    "What's the weather in London?",
    response_format=WeatherResponse
)
weather = client.call(task)
print(f"Temperature: {weather.temperature}°C")
```

### 2. Complex Response Structures
```python
class Article(ObjectResponse):
    title: str
    sections: list[str]
    summary: str
    keywords: list[str]

task = Task(
    "Write an article about AI safety",
    response_format=Article
)
article = client.call(task)
```

## Batch Processing

### 1. Multiple Tasks
```python
# Create multiple tasks
tasks = [
    Task("Summarize chapter 1"),
    Task("Summarize chapter 2"),
    Task("Summarize chapter 3")
]

# Process all tasks
results = client.call(tasks)
```

### 2. Parallel Processing with Different Models
```python
tasks_with_models = [
    (Task("Basic analysis"), "openai/gpt-3.5-turbo"),
    (Task("Complex analysis"), "openai/gpt-4"),
    (Task("Creative writing"), "anthropic/claude-2")
]

results = []
for task, model in tasks_with_models:
    result = client.call(task, llm_model=model)
    results.append(result)
```

## Tool Integration

### 1. Using Built-in Tools
```python
from upsonic.tools import Search

# Task with search capability
task = Task(
    "Research recent AI developments",
    tools=[Search]
)
research = client.call(task)
```

### 2. Custom Tools
```python
class DatabaseTool:
    def query(self, sql: str):
        # Database query implementation
        pass

# Use custom tool
task = Task(
    "Get user statistics",
    tools=[DatabaseTool]
)
stats = client.call(task)
```

## Error Handling

### 1. Basic Error Handling
```python
from upsonic.exception import TimeoutException

try:
    result = client.call(task)
except TimeoutException:
    print("Request timed out")
except Exception as e:
    print(f"Error: {e}")
```

### 2. Advanced Error Management
```python
class CallManager:
    def __init__(self, client):
        self.client = client
        self.max_retries = 3
        
    def safe_call(self, task: Task):
        retries = 0
        while retries < self.max_retries:
            try:
                return self.client.call(task)
            except TimeoutException:
                retries += 1
                if retries == self.max_retries:
                    raise
                time.sleep(2 ** retries)  # Exponential backoff
```

## Performance Monitoring

### 1. Basic Timing
```python
import time

start_time = time.time()
result = client.call(task)
execution_time = time.time() - start_time

print(f"Task completed in {execution_time:.2f} seconds")
```

### 2. Detailed Performance Tracking
```python
class PerformanceTracker:
    def __init__(self):
        self.calls = []
        
    def track_call(self, task: Task):
        start = time.time()
        result = client.call(task)
        end = time.time()
        
        self.calls.append({
            'task': task.description,
            'model': task.model,
            'duration': end - start,
            'success': result is not None
        })
        
        return result
```

## Advanced Usage Patterns

### 1. Context Management
```python
class ContextualCall:
    def __init__(self, client):
        self.client = client
        self.context = []
        
    def add_context(self, info: str):
        self.context.append(info)
        
    def call_with_context(self, task: Task):
        task.context = self.context
        return self.client.call(task)
```

### 2. Response Processing Pipeline
```python
class ResponsePipeline:
    def __init__(self):
        self.processors = []
        
    def add_processor(self, func):
        self.processors.append(func)
        
    def process(self, result):
        for processor in self.processors:
            result = processor(result)
        return result
```

## Best Practices

### 1. Task Organization
```python
# Group related tasks
class TaskGroup:
    def __init__(self, name: str):
        self.name = name
        self.tasks = []
        
    def add_task(self, task: Task):
        self.tasks.append(task)
        
    def execute_all(self, client):
        return client.call(self.tasks)
```

### 2. Resource Management
```python
class ResourceManager:
    def __init__(self, client):
        self.client = client
        self.active_tasks = 0
        self.max_concurrent = 5
        
    async def managed_call(self, task: Task):
        while self.active_tasks >= self.max_concurrent:
            await asyncio.sleep(1)
            
        self.active_tasks += 1
        try:
            return self.client.call(task)
        finally:
            self.active_tasks -= 1
```

## Debugging Tips

1. **Enable Debug Mode**:
```python
client = UpsonicClient("localhost", debug=True)
```

2. **Response Validation**:
```python
def validate_response(response, expected_type):
    assert isinstance(response, expected_type), f"Expected {expected_type}, got {type(response)}"
```

3. **Request Logging**:
```python
class LoggedCall:
    def __init__(self, client):
        self.client = client
        self.log = []
        
    def call_with_logging(self, task: Task):
        self.log.append({
            'time': time.time(),
            'task': task.description
        })
        result = self.client.call(task)
        self.log[-1]['result'] = result
        return result
```

## Common Issues and Solutions

1. **Timeout Handling**:
```python
def call_with_timeout(client, task: Task, max_retries: int = 3):
    for attempt in range(max_retries):
        try:
            return client.call(task)
        except TimeoutException:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)
```

2. **Memory Management**:
```python
def clean_call(client, task: Task):
    """Make a call with proper cleanup"""
    try:
        return client.call(task)
    finally:
        # Cleanup code
        import gc
        gc.collect()
```

3. **Server Connection Issues**:
```python
def ensure_connection(client):
    """Ensure server connection is active"""
    if not client.status():
        raise ConnectionError("Server not available")
```

## Performance Optimization

1. **Response Caching**:
```python
from functools import lru_cache

class CachedCall:
    def __init__(self, client):
        self.client = client
        
    @lru_cache(maxsize=100)
    def cached_call(self, task_description: str):
        task = Task(task_description)
        return self.client.call(task)
```

2. **Batch Processing Optimization**:
```python
def optimized_batch_call(client, tasks: list[Task], batch_size: int = 5):
    """Process tasks in optimized batches"""
    results = []
    for i in range(0, len(tasks), batch_size):
        batch = tasks[i:i + batch_size]
        results.extend(client.call(batch))
    return results
```

## Monitoring and Analytics

1. **Usage Tracking**:
```python
class UsageTracker:
    def __init__(self):
        self.total_calls = 0
        self.total_tokens = 0
        self.model_usage = {}
        
    def track(self, task: Task, result):
        self.total_calls += 1
        # Add more tracking logic
```

2. **Performance Metrics**:
```python
class MetricsCollector:
    def __init__(self):
        self.response_times = []
        self.error_rates = {}
        
    def collect(self, duration: float, success: bool):
        self.response_times.append(duration)
        # Add more metrics
```
