# Comprehensive Guide to Upsonic Tasks

## Task Creation and Basic Usage

### 1. Simple Task Creation
Shows how to create basic tasks with just a description:
```python
from upsonic import Task

# Simple task
simple_task = Task("Summarize this text")

# Task with more details
detailed_task = Task(
    description="Analyze this data",
    tools=[],  # No special tools
    response_format=None  # Default string response
)
```

### 2. Tasks with Response Formats
Demonstrates how to create tasks that expect specific response structures:
```python
from upsonic import ObjectResponse

class AnalysisResult(ObjectResponse):
    summary: str
    key_points: list[str]
    sentiment: float

# Task with structured response
analysis_task = Task(
    description="Analyze customer feedback",
    response_format=AnalysisResult
)
```

### 3. Tasks with Tools
Shows how to create tasks that use specific tools:
```python
from upsonic.tools import Search, ComputerUse

# Task with search capability
research_task = Task(
    description="Research recent AI developments",
    tools=[Search]
)

# Task with computer automation
automation_task = Task(
    description="Open the calculator app",
    tools=[ComputerUse]
)
```

## Working with Images

### 1. Image Processing Tasks
Demonstrates how to create tasks that work with images:
```python
# Task with image input
image_task = Task(
    description="Describe this image",
    images=["path/to/image.jpg"]
)

# Multiple images
multi_image_task = Task(
    description="Compare these images",
    images=[
        "path/to/image1.jpg",
        "path/to/image2.jpg"
    ]
)
```

### 2. Accessing Image Data
Shows how to work with base64-encoded image data:
```python
def process_task_images(task: Task):
    if task.images_base_64:
        for img_data in task.images_base_64:
            # Process base64 image data
            process_image(img_data)
```

## Context Management

### 1. Tasks with Context
Shows how to provide additional context to tasks:
```python
# Task with simple context
context_task = Task(
    description="Summarize this in context",
    context="Previous conversation about AI"
)

# Task with structured context
class Context(ObjectResponse):
    background: str
    requirements: list[str]

task_context = Context(
    background="Project background info",
    requirements=["Requirement 1", "Requirement 2"]
)

detailed_task = Task(
    description="Create project plan",
    context=task_context
)
```

### 2. Chaining Tasks
Demonstrates how to use previous task results as context:
```python
# First task
first_task = Task("Research topic X")
result = agent.do(first_task)

# Second task using first task's context
second_task = Task(
    description="Summarize findings",
    context=first_task
)
```

## Cost Tracking

### 1. Basic Cost Tracking
Shows how to track and monitor task costs:
```python
task = Task("Complex analysis")
agent.do(task)

# Get task cost
total_cost = task.get_total_cost()
print(f"Task cost: ${total_cost}")
```

### 2. Cost Management
Demonstrates cost tracking for multiple tasks:
```python
class CostTracker:
    def __init__(self):
        self.tasks = []
        
    def add_task(self, task: Task):
        self.tasks.append(task)
        
    def get_total_costs(self):
        return sum(task.get_total_cost() or 0 for task in self.tasks)
```

## Advanced Task Features

### 1. Subtasks
Shows how to break down complex tasks:
```python
# Main task with subtasks
main_task = Task(
    description="Create comprehensive report",
    not_main_task=False  # This is a main task
)

# Subtask
subtask = Task(
    description="Research section 1",
    not_main_task=True   # This is a subtask
)
```

### 2. Custom Task Types
Demonstrates how to create specialized task types:
```python
class ResearchTask(Task):
    def __init__(self, topic: str, **kwargs):
        super().__init__(
            description=f"Research about {topic}",
            tools=[Search],
            **kwargs
        )

class AnalysisTask(Task):
    def __init__(self, data: str, **kwargs):
        super().__init__(
            description=f"Analyze this data: {data}",
            response_format=AnalysisResult,
            **kwargs
        )
```

## Integration Examples

### 1. API Integration
Shows how to use tasks in an API context:
```python
from fastapi import FastAPI
app = FastAPI()

@app.post("/analyze")
async def analyze_text(text: str):
    task = Task(
        description=f"Analyze: {text}",
        response_format=AnalysisResult
    )
    result = agent.do(task)
    return {
        "result": result,
        "cost": task.get_total_cost()
    }
```

### 2. Batch Processing
Demonstrates processing multiple tasks efficiently:
```python
class BatchProcessor:
    def __init__(self, agent):
        self.agent = agent
        
    def process_batch(self, descriptions: list[str]):
        tasks = [
            Task(description=desc)
            for desc in descriptions
        ]
        
        results = []
        total_cost = 0
        
        for task in tasks:
            result = self.agent.do(task)
            results.append(result)
            total_cost += task.get_total_cost() or 0
            
        return results, total_cost
```

## Best Practices

### 1. Task Organization
```python
class TaskManager:
    def __init__(self):
        self.tasks = {}
        
    def create_task(self, category: str, description: str, **kwargs):
        task = Task(description=description, **kwargs)
        self.tasks[task.price_id] = {
            'task': task,
            'category': category,
            'status': 'created'
        }
        return task
        
    def update_status(self, task_id: str, status: str):
        if task_id in self.tasks:
            self.tasks[task_id]['status'] = status
            
    def get_category_costs(self):
        costs = {}
        for info in self.tasks.values():
            category = info['category']
            cost = info['task'].get_total_cost() or 0
            costs[category] = costs.get(category, 0) + cost
        return costs
```

### 2. Error Handling
```python
def safe_task_execution(task: Task, agent):
    try:
        result = agent.do(task)
        return {
            'success': True,
            'result': result,
            'cost': task.get_total_cost()
        }
    except Exception as e:
        return {
            'success': False,
            'error': str(e),
            'cost': task.get_total_cost()
        }
```

## Debugging and Testing

### 1. Task Debugging
```python
class DebugTask(Task):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        print(f"Created task with ID: {self.price_id}")
        
    @property
    def response(self):
        print(f"Accessing response for task: {self.price_id}")
        return super().response
```

### 2. Task Testing
```python
def test_task_creation():
    # Test basic task
    task = Task("Test description")
    assert task.description == "Test description"
    assert task.response is None
    
    # Test task with tools
    task_with_tools = Task(
        description="Test with tools",
        tools=[Search]
    )
    assert len(task_with_tools.tools) == 1
    
    # Test task with images
    image_task = Task(
        description="Test with image",
        images=["test.jpg"]
    )
    assert image_task.images is not None
```
