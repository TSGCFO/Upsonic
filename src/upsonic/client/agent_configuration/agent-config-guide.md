# Comprehensive Guide to Upsonic Agent Configuration

## Basic Agent Configuration

### 1. Creating a Simple Agent
```python
from upsonic import AgentConfiguration

# Basic agent
basic_agent = AgentConfiguration(
    job_title="General Assistant",
    model="openai/gpt-4o"
)
```

### 2. Identity and Context Configuration
```python
# Agent with full identity and context
professional_agent = AgentConfiguration(
    job_title="Business Analyst",
    company_url="https://mycompany.com",
    company_objective="Optimize business processes and improve efficiency",
    name="James Smith",  # Optional: Give agent a name
    contact="james@mycompany.com"  # Optional: Contact information
)
```

## Advanced Features Configuration

### 1. Reliability Features
```python
# Define reliability layer
class ReliabilityLayer:
    prevent_hallucination = 10  # Higher number = more strict checking
    verify_sources = True
    fact_checking = True

# Create agent with reliability features
reliable_agent = AgentConfiguration(
    job_title="Research Analyst",
    reliability_layer=ReliabilityLayer,
    reflection=True,  # Enables more thorough self-checking
    memory=True      # Enables context retention
)
```

### 2. Caching Configuration
```python
# Agent with custom caching settings
caching_agent = AgentConfiguration(
    job_title="Data Processor",
    caching=True,
    cache_expiry=3600,  # Cache expiry in seconds (default: 1 hour)
    context_compress=True  # Enable context compression for large datasets
)
```

### 3. Tools Integration
```python
from upsonic.tools import Search, ComputerUse

# Define custom tool
class CustomDatabaseTool:
    command = "query_db"
    args = ["query"]

# Agent with multiple tools
tooled_agent = AgentConfiguration(
    job_title="Technical Assistant",
    tools=[
        Search,           # Built-in search capability
        ComputerUse,      # Computer control capability
        CustomDatabaseTool # Custom tool
    ]
)
```

### 4. Knowledge Base Integration
```python
from upsonic import KnowledgeBase

# Create knowledge base
kb = KnowledgeBase()
kb.add_document("path/to/document.pdf")
kb.add_webpage("https://example.com/docs")

# Agent with knowledge base
knowledgeable_agent = AgentConfiguration(
    job_title="Domain Expert",
    knowledge_base=kb
)
```

## Specialized Agent Configurations

### 1. Development Agent
```python
# Agent specialized for code development
dev_agent = AgentConfiguration(
    job_title="Senior Developer",
    model="openai/gpt-4o",
    tools=[
        "git",
        "docker",
        "code_analysis"
    ],
    reflection=True,  # Enable thorough code review
    sub_task=True    # Break down complex coding tasks
)
```

### 2. Research Agent
```python
# Agent for research tasks
research_agent = AgentConfiguration(
    job_title="Research Assistant",
    tools=[Search],
    reliability_layer=ReliabilityLayer,
    reflection=True,
    memory=True,
    caching=True,
    cache_expiry=24*3600  # 24 hours cache
)
```

### 3. Customer Service Agent
```python
# Agent for customer interactions
service_agent = AgentConfiguration(
    job_title="Customer Service Representative",
    company_objective="Provide excellent customer support",
    name="Support Team",
    tools=[
        "knowledge_base",
        "ticket_system",
        "email_tools"
    ],
    memory=True,  # Remember conversation context
    sub_task=False  # Direct responses preferred
)
```

## Usage Examples

### 1. Basic Task Execution
```python
from upsonic import Task

# Create agent and task
agent = AgentConfiguration(job_title="Assistant")
task = Task("Summarize this article: [article text]")

# Execute task
result = agent.do(task)  # Execute silently
agent.print_do(task)     # Execute and print result
```

### 2. Multi-Task Processing
```python
# Create multiple tasks
tasks = [
    Task("Research latest AI trends"),
    Task("Summarize findings"),
    Task("Create presentation outline")
]

# Process tasks
for task in tasks:
    agent.do(task)
```

### 3. Agent with Custom Client
```python
from upsonic import UpsonicClient

# Create custom client
custom_client = UpsonicClient(
    url="http://custom-server:8000",
    debug=True
)

# Agent with custom client
custom_agent = AgentConfiguration(
    job_title="Custom Service Agent",
    client=custom_client
)
```

## Best Practices

1. **Reliability Settings**:
   - Enable `reflection` for critical tasks
   - Use `reliability_layer` for fact-checking
   - Enable `memory` for conversational tasks

2. **Caching Optimization**:
   - Enable caching for repetitive tasks
   - Set appropriate cache_expiry
   - Use context_compress for large datasets

3. **Tool Integration**:
   - Only include necessary tools
   - Combine built-in and custom tools
   - Test tool compatibility

4. **Performance Optimization**:
   - Use sub_task=False for simple tasks
   - Enable caching for frequent operations
   - Set appropriate retry limits

## Debugging and Monitoring

```python
# Debug mode agent
debug_agent = AgentConfiguration(
    job_title="Debug Assistant",
    debug=True,  # Enable detailed logging
    reflection=True,  # See thought process
    client=UpsonicClient("localhost", debug=True)
)
```

## Configuration Properties Reference

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| job_title | str | Required | Agent's role/purpose |
| model | str | "openai/gpt-4o" | AI model to use |
| debug | bool | False | Enable debug mode |
| sub_task | bool | True | Break tasks into subtasks |
| reflection | bool | False | Enable self-checking |
| memory | bool | False | Enable context retention |
| caching | bool | True | Enable response caching |
| cache_expiry | int | 3600 | Cache duration (seconds) |
| context_compress | bool | False | Compress context data |
| tools | List[Any] | [] | Available tools |
| reliability_layer | Any | None | Reliability settings |
