# Comprehensive Guide to Direct LLM Calls in Upsonic

## Overview
Direct LLM calls provide a lightweight way to interact with language models without the full agent system. This is useful for simple tasks, quick prototyping, or when you don't need the advanced features of agents.

## Basic Usage

### 1. Simple Text Tasks
```python
from upsonic import Direct, Task

# Basic text task
task = Task("Summarize this paragraph: [your text here]")
result = Direct.print_do(task)
```

### 2. Model Selection
```python
# Using specific models
gpt4_task = Direct.do(
    Task("Complex analysis task here"),
    model="openai/gpt-4"
)

gpt35_task = Direct.do(
    Task("Simpler task here"),
    model="openai/gpt-3.5-turbo"
)

claude_task = Direct.do(
    Task("Another task"),
    model="anthropic/claude-2"
)
```

## Advanced Features

### 1. Response Format Control
```python
from upsonic import ObjectResponse

# Define custom response format
class Analysis(ObjectResponse):
    key_points: list[str]
    summary: str
    sentiment: str

# Create task with format
task = Task(
    "Analyze this customer feedback: [feedback text]",
    response_format=Analysis
)

# Get structured response
result = Direct.do(task)
print(f"Summary: {result.summary}")
print(f"Sentiment: {result.sentiment}")
```

### 2. Tool Integration
```python
from upsonic.tools import Search

# Task with search capability
search_task = Task(
    "Find recent news about AI developments",
    tools=[Search]
)

# Execute with tools
result = Direct.do(search_task)
```

### 3. Custom Client Usage
```python
from upsonic import UpsonicClient

# Create custom client
custom_client = UpsonicClient(
    url="http://your-server:8000",
    debug=True
)

# Use custom client for calls
result = Direct.do(
    task=Task("Your task here"),
    client=custom_client
)
```

## Debug and Development

### 1. Debug Mode
```python
# Enable debug mode for detailed logging
debug_result = Direct.do(
    Task("Test task"),
    debug=True
)
```

### 2. Error Handling
```python
from upsonic.exception import TimeoutException

try:
    result = Direct.do(
        Task("Complex task with potential timeout")
    )
except TimeoutException:
    print("Task timed out after 600 seconds")
except Exception as e:
    print(f"Error occurred: {e}")
```

## Common Use Cases

### 1. Text Processing
```python
# Translation
translation_task = Task("Translate to Spanish: Hello, how are you?")
Direct.print_do(translation_task)

# Summarization
summary_task = Task("Summarize this article: [article text]")
Direct.print_do(summary_task)
```

### 2. Data Analysis
```python
class DataAnalysis(ObjectResponse):
    trends: list[str]
    insights: list[str]
    recommendations: list[str]

analysis_task = Task(
    "Analyze this sales data: [data]",
    response_format=DataAnalysis
)

result = Direct.do(analysis_task)
```

### 3. Content Generation
```python
# Blog post generation
blog_task = Task(
    "Write a blog post about AI safety",
    response_format=str
)

# Code generation
code_task = Task(
    "Write a Python function to sort a list",
    response_format=str
)
```

## Best Practices

1. **Task Definition**:
   - Be clear and specific in task descriptions
   - Provide necessary context
   - Use appropriate response formats

2. **Model Selection**:
   - Choose models based on task complexity
   - Consider cost vs. performance
   - Test with different models for optimal results

3. **Performance**:
   - Use structured response formats for data processing
   - Handle timeouts appropriately
   - Enable debug mode when developing

4. **Error Handling**:
   - Implement proper try-catch blocks
   - Handle timeouts gracefully
   - Log errors for debugging

## Comparison: Direct vs. Agent

| Feature | Direct Call | Agent |
|---------|-------------|-------|
| Setup Complexity | Low | Higher |
| Task Management | Basic | Advanced |
| Memory/Context | No | Yes |
| Reliability Features | No | Yes |
| Use Case | Simple, one-off tasks | Complex, multi-step tasks |
| Response Time | Faster | May be slower |

## Examples in Different Contexts

### 1. API Integration
```python
# Using Direct calls in an API
from fastapi import FastAPI
app = FastAPI()

@app.post("/analyze")
async def analyze_text(text: str):
    task = Task(f"Analyze this text: {text}")
    result = Direct.do(task)
    return {"analysis": result}
```

### 2. Batch Processing
```python
# Process multiple items
texts = ["Text 1", "Text 2", "Text 3"]
results = []

for text in texts:
    task = Task(f"Process this: {text}")
    result = Direct.do(task)
    results.append(result)
```

### 3. Interactive Applications
```python
# Simple chat interface
def chat_interface():
    while True:
        user_input = input("You: ")
        if user_input.lower() == 'exit':
            break
            
        task = Task(user_input)
        response = Direct.print_do(task)
```

## Performance Tips

1. **Response Format Optimization**:
   - Use structured formats for consistent outputs
   - Keep formats simple for better performance
   - Use appropriate data types

2. **Client Management**:
   - Reuse clients when possible
   - Handle connection issues gracefully
   - Configure timeouts appropriately

3. **Resource Usage**:
   - Monitor API usage
   - Handle rate limiting
   - Implement appropriate error handling

## Troubleshooting Common Issues

1. **Timeout Issues**:
   - Default timeout is 600 seconds
   - Break large tasks into smaller ones
   - Use appropriate model for task size

2. **Connection Problems**:
   - Check server URL
   - Verify API keys
   - Monitor network connectivity

3. **Response Format Errors**:
   - Verify format definitions
   - Check data type compatibility
   - Handle null/missing values
