# Comprehensive Guide to Upsonic Task Responses

## Basic Response Types

### 1. Object Response
This example shows how to create a basic response structure for storing person information with specific data types for each field:
```python
from upsonic import ObjectResponse

# Basic object response
class PersonInfo(ObjectResponse):
    name: str
    age: int
    email: str

# Usage
task = Task(
    "Get person info for John",
    response_format=PersonInfo
)
```

### 2. Simple Type Responses
Shows how to create responses for basic data types. Each response type is specialized for a specific kind of data (text, numbers, true/false values):
```python
from upsonic import StrResponse, IntResponse, FloatResponse, BoolResponse

# String response
NameResponse = StrResponse("name")
task = Task("What's your name?", response_format=NameResponse)

# Integer response
AgeResponse = IntResponse("age")
task = Task("How old are you?", response_format=AgeResponse)

# Float response
TemperatureResponse = FloatResponse("temperature")
task = Task("What's the temperature?", response_format=TemperatureResponse)

# Boolean response
AvailabilityResponse = BoolResponse("is_available")
task = Task("Is the service available?", response_format=AvailabilityResponse)
```

### 3. List Response
Demonstrates how to handle responses that contain lists of strings, useful for tags, categories, or any collection of text items:
```python
from upsonic import StrInListResponse

# String list response
TagsResponse = StrInListResponse("tags")
task = Task("List relevant tags", response_format=TagsResponse)
```

## Complex Response Structures

### 1. Nested Objects
Shows how to create complex response structures with nested objects, useful for related data like customer information with address details:
```python
class Address(ObjectResponse):
    street: str
    city: str
    country: str

class Customer(ObjectResponse):
    name: str
    age: int
    address: Address
    active: bool

# Usage
task = Task(
    "Get customer details",
    response_format=Customer
)
```

### 2. List of Objects
Demonstrates how to handle collections of complex objects, perfect for inventory systems or any list of structured data:
```python
class Product(ObjectResponse):
    name: str
    price: float
    quantity: int

class Inventory(ObjectResponse):
    products: List[Product]
    total_value: float
    last_updated: str

# Usage
task = Task(
    "Get inventory status",
    response_format=Inventory
)
```

## Custom Response Types

### 1. Creating Custom Response Types
Shows how to create specialized response types with custom output formatting, useful for tailored data presentation:
```python
class CustomTaskResponse(ObjectResponse):
    def output(self):
        """Custom output formatting"""
        return self.dict()

class AnalysisResponse(CustomTaskResponse):
    text: str
    sentiment: float
    keywords: List[str]
    
    def output(self):
        return {
            'sentiment_analysis': self.sentiment,
            'key_findings': self.keywords
        }
```

### 2. Response Type with Validation
Demonstrates how to add data validation to responses, ensuring that values fall within acceptable ranges:
```python
from pydantic import validator

class ScoreResponse(ObjectResponse):
    score: float
    confidence: float

    @validator('score')
    def validate_score(cls, v):
        if not 0 <= v <= 100:
            raise ValueError('Score must be between 0 and 100')
        return v

    @validator('confidence')
    def validate_confidence(cls, v):
        if not 0 <= v <= 1:
            raise ValueError('Confidence must be between 0 and 1')
        return v
```

## Response Processing

### 1. Basic Response Processing
Shows the fundamental way to process response data, converting it into a dictionary format:
```python
def process_response(response: ObjectResponse):
    """Process a basic response"""
    data = response.dict()
    # Add processing logic
    return data
```

### 2. Response Transformation
Illustrates how to create a flexible system for transforming responses into different formats:
```python
class ResponseTransformer:
    def transform(self, response: ObjectResponse):
        if hasattr(response, 'output'):
            return response.output()
        return response.dict()

    def format_output(self, response: ObjectResponse):
        data = self.transform(response)
        # Add formatting logic
        return data
```

## Best Practices

### 1. Response Type Selection
Shows how to automatically select the appropriate response type based on data characteristics:
```python
def select_response_type(data_type: str):
    """Select appropriate response type based on data"""
    type_mapping = {
        'text': StrResponse('text'),
        'number': IntResponse('number'),
        'decimal': FloatResponse('decimal'),
        'flag': BoolResponse('flag'),
        'list': StrInListResponse('items')
    }
    return type_mapping.get(data_type, ObjectResponse)
```

### 2. Response Validation
Demonstrates how to add self-validation capabilities to response objects:
```python
class ValidatedResponse(ObjectResponse):
    def validate_response(self):
        """Validate response data"""
        try:
            self.dict()
            return True
        except Exception as e:
            print(f"Validation error: {e}")
            return False
```

## Advanced Usage

### 1. Dynamic Response Types
Shows how to create response types on the fly based on runtime requirements:
```python
def create_dynamic_response(fields: dict):
    """Create dynamic response type based on fields"""
    class DynamicResponse(ObjectResponse):
        pass
    
    for field_name, field_type in fields.items():
        setattr(DynamicResponse, field_name, field_type)
    
    return DynamicResponse

# Usage
fields = {
    'name': str,
    'count': int,
    'active': bool
}
DynamicResponse = create_dynamic_response(fields)
```

### 2. Response Inheritance
Demonstrates how to create a hierarchy of response types with shared basic fields:
```python
class BaseResponse(ObjectResponse):
    timestamp: str
    version: str

class DetailedResponse(BaseResponse):
    data: dict
    status: str
    
class ErrorResponse(BaseResponse):
    error_code: int
    message: str
```

## Error Handling

### 1. Basic Error Handling
Shows simple error handling when creating response objects:
```python
def safe_response_handling(response_type, data):
    try:
        return response_type(**data)
    except Exception as e:
        print(f"Error creating response: {e}")
        return None
```

### 2. Advanced Error Handling
Demonstrates comprehensive error handling with logging and validation:
```python
class ResponseHandler:
    def __init__(self):
        self.error_responses = []
        
    def handle_response(self, response_type, data):
        try:
            response = response_type(**data)
            if hasattr(response, 'validate_response'):
                if not response.validate_response():
                    raise ValueError("Response validation failed")
            return response
        except Exception as e:
            self.error_responses.append({
                'type': response_type.__name__,
                'error': str(e),
                'data': data
            })
            return None
```

## Performance Optimization

### 1. Response Caching
Shows how to implement caching for response objects to improve performance:
```python
from functools import lru_cache

class CachedResponseHandler:
    @lru_cache(maxsize=100)
    def get_response(self, response_type, data_key):
        # Response generation logic
        pass
```

### 2. Batch Response Processing
Demonstrates how to efficiently process multiple responses at once:
```python
class BatchResponseProcessor:
    def process_batch(self, responses: List[ObjectResponse]):
        results = []
        for response in responses:
            processed = self.process_single(response)
            results.append(processed)
        return results
```

## Integration Examples

### 1. API Integration
Shows how to integrate response handling with a FastAPI endpoint:
```python
from fastapi import FastAPI
app = FastAPI()

@app.post("/process")
async def process_response(data: dict):
    response_type = select_response_type(data['type'])
    response = response_type(**data['content'])
    return response.dict()
```

### 2. Database Integration
Demonstrates how to save and load responses from a database:
```python
class DatabaseResponseHandler:
    def save_response(self, response: ObjectResponse):
        data = response.dict()
        # Database save logic
        
    def load_response(self, response_type, id: int):
        # Database load logic
        pass
```

## Debugging and Testing

### 1. Response Debugging
Shows how to add debugging information to response creation:
```python
class DebugResponse(ObjectResponse):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        print(f"Creating response of type {self.__class__.__name__}")
        print(f"Data: {self.dict()}")
```

### 2. Response Testing
Demonstrates how to create test functions for response types:
```python
def test_response(response_type, test_data):
    """Test response type with sample data"""
    try:
        response = response_type(**test_data)
        assert response.dict()
        print("Response test passed")
        return True
    except Exception as e:
        print(f"Response test failed: {e}")
        return False
```
