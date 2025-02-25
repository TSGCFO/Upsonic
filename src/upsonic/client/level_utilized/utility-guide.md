# Comprehensive Guide to Upsonic Utility Module

The utility module in Upsonic contains essential helper functions that handle serialization, deserialization, error handling, and data formatting. These functions are used throughout the framework to ensure proper data handling between client and server components.

## Context Handling

### 1. Context Serialization
The `context_serializer` function prepares context data for transmission to the server:

```python
from upsonic.client.level_utilized.utility import context_serializer

# Prepare context for transmission
serialized_context = context_serializer(
    context=my_context_object,
    client=upsonic_client
)
```

This function:
- Converts single context items into a list
- Adds the current date and time to the context
- Removes any tools or response formats from context objects
- Serializes KnowledgeBase objects
- Pickles and base64 encodes the context for safe transmission

This is crucial for sending complex Python objects to the server while preserving their structure and relationships.

### 2. Context Preparation
The `serialize_context` function handles special context objects:

```python
from upsonic.client.level_utilized.utility import serialize_context

# Convert knowledge base to markdown for context
prepared_context = serialize_context(
    context=knowledge_base,
    client=upsonic_client
)
```

This function specializes in handling KnowledgeBase objects, converting them to markdown format that can be used as context by language models.

## Response Format Handling

### 1. Response Format Serialization
The `response_format_serializer` function prepares response format specifications:

```python
from upsonic.client.level_utilized.utility import response_format_serializer

# Convert response format to serialized string
serialized_format = response_format_serializer(
    response_format=MyResponseClass
)
```

This function:
- Defaults to "str" if no format is specified
- Pickles and base64 encodes Pydantic models or other types
- Ensures complex response format specifications can be safely transmitted

### 2. Response Format Deserialization
The `response_format_deserializer` function converts raw results to the specified format:

```python
from upsonic.client.level_utilized.utility import response_format_deserializer

# Convert raw result to specified format
processed_result = response_format_deserializer(
    response_format_str=serialized_format,
    result=raw_result
)
```

This function:
- Determines if decoding is needed based on the format string
- Decodes and unpickles complex response formats
- Returns the properly formatted result

## Tool Handling

### 1. Tool Serialization
The `tools_serializer` function prepares tool specifications for transmission:

```python
from upsonic.client.level_utilized.utility import tools_serializer

# Convert tool specifications to serialized format
serialized_tools = tools_serializer(
    tools_=[SearchTool, DatabaseTool]
)
```

This function:
- Handles class types by using their name
- Handles string identifiers
- Handles object instances by using their class name
- Adds a wildcard suffix for class-based tools

## Error Handling

### 1. Error Processing
The `error_handler` function processes server response errors:

```python
from upsonic.client.level_utilized.utility import error_handler

# Process response for potential errors
try:
    error_handler(response_result)
except Exception as e:
    print(f"Error: {e}")
```

This function checks the status code and raises the appropriate exception:
- 401: NoAPIKeyException - API key issues
- 402: ContextWindowTooSmallException - Context size problems
- 403: InvalidRequestException - Malformed requests
- 400: UnsupportedLLMModelException - Unsupported model
- 500: CallErrorException - Server-side errors

## Common Usage Patterns

### 1. Complete Request Preparation
A typical pattern for preparing a complete request:

```python
def prepare_request(task, client, model):
    # Serialize tools
    tools = tools_serializer(task.tools)
    
    # Serialize response format
    response_format = response_format_serializer(task.response_format)
    
    # Serialize context
    context = context_serializer(task.context, client)
    
    # Compile request data
    data = {
        "prompt": task.description,
        "images": task.images_base_64,
        "response_format": response_format,
        "tools": tools,
        "context": context,
        "llm_model": model
    }
    
    return data
```

### 2. Complete Response Processing
A typical pattern for processing a complete response:

```python
def process_response(response, response_format_str):
    # Handle any errors
    try:
        error_handler(response)
    except Exception as e:
        print(f"Error processing response: {e}")
        raise
    
    # Deserialize the result
    deserialized_result = response_format_deserializer(
        response_format_str, 
        response
    )
    
    return deserialized_result
```

## Edge Cases and Error Management

### 1. Handling Large Context
When dealing with large context objects:

```python
def safe_context_serialization(context, client):
    try:
        serialized = context_serializer(context, client)
        return serialized
    except MemoryError:
        # Handle large context
        print("Context too large, attempting to reduce")
        reduced_context = reduce_context_size(context)
        return context_serializer(reduced_context, client)
```

### 2. Fallback Formats
When dealing with format deserialization errors:

```python
def safe_format_deserialization(response_format_str, result):
    try:
        return response_format_deserializer(response_format_str, result)
    except Exception as e:
        print(f"Error deserializing: {e}")
        # Fall back to string format
        result["result"] = str(result["result"])
        return result
```

## Debugging and Troubleshooting

### 1. Debug Serialization
For debugging serialization issues:

```python
def debug_serialization(obj, client):
    print(f"Object type: {type(obj)}")
    
    try:
        serialized = context_serializer(obj, client)
        print(f"Serialized size: {len(serialized)} bytes")
        return serialized
    except Exception as e:
        print(f"Serialization error: {e}")
        # Try to identify problematic attributes
        if hasattr(obj, '__dict__'):
            for key, value in obj.__dict__.items():
                try:
                    test = cloudpickle.dumps(value)
                    print(f"Attribute {key} OK")
                except Exception as e:
                    print(f"Problematic attribute: {key}, Error: {e}")
        return None
```

### 2. Error Investigation
For investigating error responses:

```python
def investigate_error(response):
    print(f"Response status: {response.get('status_code')}")
    print(f"Response detail: {response.get('detail')}")
    
    try:
        error_handler(response)
        print("No errors detected")
    except Exception as e:
        print(f"Error type: {type(e).__name__}")
        print(f"Error message: {str(e)}")
        
        # Suggest remediation
        if isinstance(e, NoAPIKeyException):
            print("Suggestion: Check your API key configuration")
        elif isinstance(e, ContextWindowTooSmallException):
            print("Suggestion: Reduce context size or use a model with larger context window")
```

## Extending the Utility Functions

### 1. Custom Serializer
Creating a custom serializer for special objects:

```python
def custom_object_serializer(obj):
    # Register with cloudpickle
    the_module = dill.detect.getmodule(obj.__class__)
    if the_module is not None:
        cloudpickle.register_pickle_by_value(the_module)
    
    # Convert to pickle and base64
    pickled = cloudpickle.dumps(obj)
    encoded = base64.b64encode(pickled).decode("utf-8")
    
    return encoded
```

### 2. Custom Error Handler
Creating a custom error handler with retry logic:

```python
def custom_error_handler(result, max_retries=3):
    retry_count = 0
    
    while retry_count < max_retries:
        try:
            error_handler(result)
            return  # No error
        except CallErrorException:
            # Server error might be temporary, retry
            retry_count += 1
            print(f"Server error, retrying ({retry_count}/{max_retries})")
            time.sleep(2 ** retry_count)  # Exponential backoff
        except Exception as e:
            # Other errors, don't retry
            print(f"Non-retryable error: {e}")
            raise
    
    # Max retries exceeded
    raise CallErrorException("Max retries exceeded")
```
