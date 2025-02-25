# Comprehensive Guide to Upsonic Printing Module

The printing module handles all output formatting and display in Upsonic, providing rich, colorful console output for various types of information.

## Basic Output Functions

### 1. Server Connection Status
Shows how to display server connection information:
```python
from upsonic.client.printing import connected_to_server

# Show successful connection
connected_to_server("Local(Docker)", "Established")

# Show failed connection
connected_to_server("Cloud(Upsonic)", "Failed")
```

Output example:
```
┌─────────────────── Upsonic - Server Connection ───────────────────┐
│ Server Type:        Local(Docker)                                 │
│ Connection Status:  ✓ Established                                 │
└──────────────────────────────────────────────────────────────────┘
```

### 2. Call Results
Displays the results of LLM calls:
```python
from upsonic.client.printing import call_end

call_end(
    result="Analysis complete",
    llm_model="openai/gpt-4",
    response_format="str",
    start_time=start_time,
    end_time=end_time,
    usage={'input_tokens': 100, 'output_tokens': 50}
)
```

Output example:
```
┌───────────────────── Upsonic - Call Result ─────────────────────┐
│ LLM Model:         openai/gpt-4                                 │
│                                                                 │
│ Result:           Analysis complete                             │
│                                                                 │
│ Response Format:   str                                          │
│ Estimated Cost:    $0.0015                                      │
│ Time Taken:        2.34 seconds                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Agent Output Functions

### 1. Agent Results
Shows agent execution results with additional details:
```python
from upsonic.client.printing import agent_end

agent_end(
    result="Task completed successfully",
    llm_model="openai/gpt-4",
    response_format="str",
    start_time=start_time,
    end_time=end_time,
    usage={'input_tokens': 150, 'output_tokens': 75},
    tool_count=2,
    context_count=1,
    debug=False,
    price_id="task-123"
)
```

### 2. Cost Tracking
Shows how to display cost information:
```python
from upsonic.client.printing import agent_total_cost

agent_total_cost(
    total_input_tokens=1000,
    total_output_tokens=500,
    total_time=10.5,
    llm_model="openai/gpt-4"
)
```

## Price ID Summary

### 1. Displaying Task Costs
Shows how to track and display costs for specific tasks:
```python
from upsonic.client.printing import print_price_id_summary

# Print summary for a specific task
summary = print_price_id_summary("task-123", task)
```

Output example:
```
┌───────────────── Upsonic - Price ID Summary ─────────────────┐
│ Price ID:           task-123                                 │
│                                                             │
│ Input Tokens:       1,000                                   │
│ Output Tokens:      500                                     │
│ Total Estimated Cost: $0.0150                               │
└─────────────────────────────────────────────────────────────┘
```

### 2. Getting Cost Information
Shows how to retrieve cost information programmatically:
```python
from upsonic.client.printing import get_price_id_total_cost

# Get cost details for a task
cost_info = get_price_id_total_cost("task-123")
if cost_info:
    print(f"Input tokens: {cost_info['input_tokens']}")
    print(f"Output tokens: {cost_info['output_tokens']}")
    print(f"Estimated cost: ${cost_info['estimated_cost']:.4f}")
```

## Retry Information

### 1. Showing Retry Status
Displays retry attempts for tasks:
```python
from upsonic.client.printing import agent_retry

# Show retry status
agent_retry(retry_count=1, max_retries=3)
```

Output example:
```
┌───────────────────── Upsonic - Agent Retry ────────────────────┐
│ Retry Status:      Attempt 2 of 4                              │
└────────────────────────────────────────────────────────────────┘
```

## Custom Output Formatting

### 1. Rich Table Creation
Shows how to create custom formatted tables:
```python
from rich.table import Table
from rich.console import Console

def custom_output(data: dict):
    console = Console()
    table = Table(show_header=False, expand=True, box=None)
    table.width = 60
    
    for key, value in data.items():
        table.add_row(f"[bold]{key}:[/bold]", f"[green]{value}[/green]")
    
    console.print(table)
```

### 2. Panel Creation
Shows how to create custom panels:
```python
from rich.panel import Panel

def custom_panel(title: str, content: str):
    console = Console()
    panel = Panel(
        content,
        title=f"[bold cyan]{title}[/bold cyan]",
        border_style="cyan",
        expand=True,
        width=70
    )
    console.print(panel)
```

## Best Practices

### 1. Output Organization
```python
class OutputManager:
    def __init__(self):
        self.console = Console()
        
    def section(self, title: str):
        """Create a section header"""
        self.console.print(f"\n[bold cyan]═══ {title} ═══[/bold cyan]\n")
        
    def success(self, message: str):
        """Print success message"""
        self.console.print(f"[green]✓[/green] {message}")
        
    def error(self, message: str):
        """Print error message"""
        self.console.print(f"[red]✗[/red] {message}")
        
    def info(self, message: str):
        """Print info message"""
        self.console.print(f"[blue]ℹ[/blue] {message}")
```

### 2. Cost Tracking
```python
class CostTracker:
    def __init__(self):
        self.tasks = {}
        
    def add_task_cost(self, task_id: str, input_tokens: int, output_tokens: int):
        if task_id not in self.tasks:
            self.tasks[task_id] = {
                'input_tokens': 0,
                'output_tokens': 0
            }
        self.tasks[task_id]['input_tokens'] += input_tokens
        self.tasks[task_id]['output_tokens'] += output_tokens
        
    def print_summary(self):
        for task_id, data in self.tasks.items():
            print_price_id_summary(task_id, None)
```

## Debugging Tips

### 1. Debug Output
```python
def debug_print(data: Any, debug: bool = False):
    """Print detailed information when in debug mode"""
    if debug:
        console = Console()
        console.print("[bold red]DEBUG OUTPUT:[/bold red]")
        console.print(data)
```

### 2. Performance Logging
```python
import time

class PerformanceLogger:
    def __init__(self):
        self.start_times = {}
        
    def start(self, operation: str):
        self.start_times[operation] = time.time()
        
    def end(self, operation: str):
        if operation in self.start_times:
            duration = time.time() - self.start_times[operation]
            console = Console()
            console.print(f"[bold yellow]{operation}[/bold yellow]: {duration:.2f}s")
```

## Common Patterns

### 1. Progress Tracking
```python
def show_progress(current: int, total: int, description: str):
    percentage = (current / total) * 100
    console = Console()
    console.print(f"[bold blue]{description}[/bold blue]: {percentage:.1f}%")
```

### 2. Result Formatting
```python
def format_result(result: Any, max_length: int = 370):
    """Format result for display with truncation"""
    result_str = str(result)
    if len(result_str) > max_length:
        return result_str[:max_length] + "..."
    return result_str
```
