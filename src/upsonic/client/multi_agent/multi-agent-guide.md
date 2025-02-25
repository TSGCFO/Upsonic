# Comprehensive Guide to Upsonic Multi-Agent System

## Introduction and Core Concepts

The Multi-Agent system is one of Upsonic's most powerful features. It allows you to create a team of specialized AI agents that work together to solve complex problems. Unlike a single agent approach, the multi-agent system can distribute tasks based on specialization, leading to more accurate and comprehensive results.

At its core, the system works by:
1. Taking a collection of specialized agents, each configured for specific tasks
2. Taking a set of tasks that need to be completed
3. Intelligently matching tasks to the most appropriate agents
4. Coordinating the workflow between agents when tasks depend on each other

This creates an AI workforce that can tackle complex, multi-step processes that would be difficult for a single agent to handle effectively.

## Basic Usage

### 1. Creating a Multi-Agent Setup

This example shows how to create three specialized agents with different roles and have them work together on related tasks:

```python
from upsonic import MultiAgent, AgentConfiguration, Task

# Create specialized agents - each with a different area of expertise
researcher = AgentConfiguration(
    job_title="Research Specialist",  # Focused on information gathering
    model="openai/gpt-4"              # Using a model good at research
)

analyst = AgentConfiguration(
    job_title="Data Analyst",         # Specialized in data interpretation
    model="openai/gpt-4"              # Using analytical capabilities
)

writer = AgentConfiguration(
    job_title="Content Writer",       # Expert in clear communication
    model="anthropic/claude-2"        # Using a model with strong writing skills
)

# Create a sequence of tasks forming a workflow
tasks = [
    Task("Research recent AI developments in healthcare for the past 6 months"),
    Task("Analyze findings to identify the top 3 emerging trends"),
    Task("Write a 2-page executive summary report for hospital administrators")
]

# Execute tasks with multiple agents
# The system will automatically match tasks to the appropriate agents
result = MultiAgent.do(
    agents=[researcher, analyst, writer],
    tasks=tasks
)

# Each task will be completed by the most suitable agent
# The researcher will gather information
# The analyst will identify patterns and trends
# The writer will create the final report
```

The key benefit here is task specialization - each agent focuses on what it does best, rather than trying to have one agent handle all aspects of a complex workflow.

### 2. Simple Task Distribution

This example demonstrates how agents are automatically matched to appropriate tasks:

```python
# Define simple tasks requiring different expertise
tasks = [
    Task("What are the latest developments in AI safety research?"),
    Task("What implications do these developments have for healthcare businesses?"),
    Task("Create a 5-step action plan for hospitals based on these findings")
]

# Create agents with different roles - each with their own perspective
agents = [
    AgentConfiguration(job_title="Tech Researcher"),     # Focused on technical information
    AgentConfiguration(job_title="Business Analyst"),    # Specializes in business implications
    AgentConfiguration(job_title="Strategy Consultant")  # Expert in actionable planning
]

# Execute with output printing - Upsonic will match agents to appropriate tasks
# The Tech Researcher will handle the research task
# The Business Analyst will analyze implications
# The Strategy Consultant will create the action plan
MultiAgent.print_do(agents, tasks)
```

Behind the scenes, Upsonic uses the job titles and agent configurations to determine the best agent for each task. The system isn't just randomly assigning tasks - it's intelligently matching based on the relationship between agent expertise and task requirements.

## Advanced Configuration

### 1. Agents with Different Tools

This example shows how to equip agents with different tools for specialized capabilities:

```python
from upsonic.tools import Search, ComputerUse

# Research agent with search capability
# This agent can access external information sources
researcher = AgentConfiguration(
    job_title="Researcher",
    tools=[Search]  # Can search the internet for information
)

# Automation agent with computer control
# This agent can interact with the operating system
automator = AgentConfiguration(
    job_title="Automator",
    tools=[ComputerUse]  # Can control mouse/keyboard and perform actions
)

# Each task requires different capabilities
tasks = [
    # This task needs external information
    Task("Research market trends for electric vehicles in Europe", tools=[Search]),
    
    # This task needs computer interaction abilities
    Task("Automate data collection from the sales dashboard", tools=[ComputerUse])
]

# The system matches tasks to agents partly based on tool capabilities
# The researcher will handle the market research using Search
# The automator will handle the automation task using ComputerUse
MultiAgent.do(
    agents=[researcher, automator],
    tasks=tasks
)
```

This is particularly powerful because it creates a team of agents with complementary capabilities. Each agent brings different tools to the table, allowing the team to handle a wider range of tasks than any single agent could.

### 2. Custom Client Configuration

This example demonstrates how to use different servers or configurations for different agents:

```python
from upsonic import UpsonicClient

# Create custom clients - perhaps connecting to different servers
# or with different configurations
client1 = UpsonicClient(
    "http://server1:8000",  # Perhaps a high-security environment
    debug=True              # With debugging enabled
)

client2 = UpsonicClient(
    "http://server2:8000",  # Maybe optimized for throughput
    debug=False             # Without debug overhead
)

# Configure agents with custom clients
agent1 = AgentConfiguration(
    job_title="Secure Data Processor",
    client=client1  # Uses the secure server
)

agent2 = AgentConfiguration(
    job_title="High-Volume Processor",
    client=client2  # Uses the high-throughput server
)

# Tasks can be distributed across different servers/environments
tasks = [
    Task("Process sensitive customer data"),
    Task("Generate high-volume analytics report")
]

# The system preserves client associations when distributing tasks
MultiAgent.do(
    agents=[agent1, agent2],
    tasks=tasks
)
```

This configuration allows you to connect to different infrastructure based on task requirements - perhaps using more secure environments for sensitive data or higher-powered computing for complex calculations.

## Task Coordination

### 1. Sequential Task Processing

This example shows how to create a workflow where each task depends on previous results:

```python
class ResearchProject:
    def __init__(self):
        # Create a team with different specializations
        self.agents = [
            AgentConfiguration(job_title="Researcher"),   # Gathers information
            AgentConfiguration(job_title="Analyst"),      # Interprets findings
            AgentConfiguration(job_title="Writer")        # Communicates results
        ]
        
    def execute(self, topic: str):
        # Create a sequence of dependent tasks
        tasks = [
            Task(f"Research {topic} and gather key information"),
            Task("Analyze research findings and identify patterns"),
            Task("Write a comprehensive report based on the analysis")
        ]
        
        # Create a dependency chain - each task uses previous results
        # This is critical: it ensures information flows through the workflow
        for i in range(1, len(tasks)):
            tasks[i].context = tasks[i-1]  # Each task gets the previous task as context
        
        # Execute the connected workflow
        # The system handles the sequential processing automatically
        return MultiAgent.do(self.agents, tasks)
```

This creates an assembly line of agents, where each specialist builds upon the work of the previous agent. The context passing is key - it ensures that insights gathered by the researcher influence the analyst's work, and both inform the writer's final report.

### 2. Parallel Task Processing

This example demonstrates handling multiple independent tasks simultaneously:

```python
class ParallelProcessor:
    def __init__(self, agent_count: int):
        # Create a pool of similar agents for parallel processing
        self.agents = [
            AgentConfiguration(
                job_title=f"Processor-{i}",
                model="openai/gpt-3.5-turbo"  # Using a faster, more cost-effective model
            )
            for i in range(agent_count)
        ]
        
    def process_batch(self, items: list[str]):
        # Create one task per item
        tasks = [
            Task(f"Process item: {item}")
            for item in items
        ]
        
        # Process all tasks in parallel across the agent pool
        # This is much faster than sequential processing for independent tasks
        return MultiAgent.do(self.agents, tasks)
```

This pattern is ideal for situations where you have many similar, independent tasks that don't rely on each other's results. It allows you to process them all simultaneously, dramatically improving throughput compared to sequential processing.

## Advanced Usage Patterns

### 1. Dynamic Agent Selection

This example shows how to create a system that selects appropriate agents based on task types:

```python
class DynamicAgentSystem:
    def __init__(self):
        # Create a pool of specialized agents
        self.agent_pool = {
            'research': AgentConfiguration(
                job_title="Researcher",
                tools=[Search]  # Has search capability
            ),
            'analysis': AgentConfiguration(
                job_title="Analyst",
                # Configured for data processing
                reflection=True  # More careful analysis
            ),
            'writing': AgentConfiguration(
                job_title="Writer",
                model="anthropic/claude-2"  # Good writing model
            ),
            'coding': AgentConfiguration(
                job_title="Programmer",
                tools=["code_analysis"]  # Coding tools
            )
        }
        
    def select_agents(self, task_types: list[str]):
        # Find the right agents for each task type
        return [
            self.agent_pool[task_type]
            for task_type in task_types
            if task_type in self.agent_pool
        ]
        
    def execute_tasks(self, task_list: list[tuple[str, str]]):
        # Parse the task list to get types and descriptions
        # task_list contains tuples of (task_type, task_description)
        task_types = [task_type for task_type, _ in task_list]
        tasks = [Task(desc) for _, desc in task_list]
        
        # Select appropriate agents for these specific tasks
        agents = self.select_agents(task_types)
        
        # Execute with the tailored agent selection
        return MultiAgent.do(agents, tasks)
```

This system is highly flexible because it builds custom agent teams based on the needs of each specific set of tasks. Rather than having a fixed team, it assembles the right specialists for the job at hand.

### 2. Specialized Task Teams

This example demonstrates creating specialized teams of agents for different project types:

```python
class AgentTeam:
    def __init__(self, team_type: str):
        # Configure team based on project type
        if team_type == "research":
            # Research team with complementary roles
            self.agents = [
                AgentConfiguration(
                    job_title="Lead Researcher",
                    # Coordinates research effort
                    model="openai/gpt-4"
                ),
                AgentConfiguration(
                    job_title="Data Collector",
                    # Gathers raw information
                    tools=[Search]
                ),
                AgentConfiguration(
                    job_title="Research Validator",
                    # Verifies accuracy
                    reliability_layer={"verify_sources": True}
                )
            ]
        elif team_type == "development":
            # Development team structure
            self.agents = [
                AgentConfiguration(
                    job_title="Architect",
                    # Designs solutions
                    reflection=True
                ),
                AgentConfiguration(
                    job_title="Developer",
                    # Implements solutions
                    tools=["code_analysis"]
                ),
                AgentConfiguration(
                    job_title="Tester",
                    # Verifies quality
                    reliability_layer={"prevent_hallucination": 10}
                )
            ]
            
    def execute_project(self, project_tasks: list[Task]):
        # Run the specialized team on project tasks
        return MultiAgent.do(self.agents, project_tasks)
```

This approach creates coordinated teams where each member plays a specific role, much like human teams in a workplace. The team composition is tailored to the type of project, ensuring all necessary specializations are covered.

## Error Handling and Monitoring

### 1. Task Monitoring

This example shows how to add monitoring to track the status of tasks:

```python
class TaskMonitor:
    def __init__(self):
        # Storage for task status information
        self.task_status = {}
        
    def monitor_execution(self, agents: list, tasks: list[Task]):
        try:
            # Attempt to execute the tasks
            result = MultiAgent.do(agents, tasks)
            
            # Record successful completion
            self.record_success(tasks)
            
            return result
        except Exception as e:
            # Capture and record any failures
            self.record_failure(tasks, str(e))
            
            # Re-raise the exception for further handling
            raise
            
    def record_success(self, tasks):
        # Update status for all completed tasks
        for task in tasks:
            self.task_status[task.price_id] = {
                "status": "completed",
                "completion_time": time.time()
            }
            
    def record_failure(self, tasks, error):
        # Record failure details for debugging
        for task in tasks:
            self.task_status[task.price_id] = {
                "status": "failed",
                "error": error,
                "failure_time": time.time()
            }
            
    def get_task_report(self):
        # Generate a summary of all task statuses
        completed = sum(1 for status in self.task_status.values() 
                       if status.get("status") == "completed")
        failed = sum(1 for status in self.task_status.values() 
                    if status.get("status") == "failed")
        
        return {
            "total_tasks": len(self.task_status),
            "completed": completed,
            "failed": failed,
            "success_rate": completed / len(self.task_status) if self.task_status else 0
        }
```

The monitoring system provides visibility into what's happening with your agents, helping you track progress, identify issues, and measure performance over time.

### 2. Performance Tracking

This example demonstrates tracking agent performance metrics:

```python
import time

class PerformanceTracker:
    def __init__(self):
        # Storage for performance statistics
        self.agent_stats = {}
        self.task_timings = {}
        
    def track_execution(self, agents: list, tasks: list[Task]):
        # Record starting time
        start_time = time.time()
        
        # Execute the tasks
        result = MultiAgent.do(agents, tasks)
        
        # Calculate total duration
        duration = time.time() - start_time
        
        # Record performance for each agent
        for agent in agents:
            if agent.job_title not in self.agent_stats:
                self.agent_stats[agent.job_title] = {
                    "count": 0,
                    "total_duration": 0,
                    "tasks_handled": []
                }
            
            self.agent_stats[agent.job_title]["count"] += 1
            self.agent_stats[agent.job_title]["total_duration"] += duration
            
            # Track which tasks were handled by this agent
            for task in tasks:
                self.agent_stats[agent.job_title]["tasks_handled"].append(task.price_id)
                
                # Record individual task timing
                self.task_timings[task.price_id] = {
                    "duration": duration / len(tasks),  # Approximate
                    "agent": agent.job_title
                }
        
        return result
        
    def get_average_duration(self, job_title: str):
        # Calculate average processing time for an agent type
        if job_title in self.agent_stats:
            stats = self.agent_stats[job_title]
            if stats["count"] > 0:
                return stats["total_duration"] / stats["count"]
        return None
        
    def get_performance_report(self):
        # Generate comprehensive performance summary
        return {
            "agents": {
                job_title: {
                    "tasks_completed": stats["count"],
                    "avg_duration": stats["total_duration"] / stats["count"] if stats["count"] > 0 else 0,
                    "total_time": stats["total_duration"]
                }
                for job_title, stats in self.agent_stats.items()
            },
            "task_count": len(self.task_timings),
            "avg_task_time": sum(t["duration"] for t in self.task_timings.values()) / len(self.task_timings) if self.task_timings else 0
        }
```

This detailed tracking helps you understand which agents are most effective for different types of tasks, allowing you to optimize your agent configurations and task assignments over time.

## Best Practices

### 1. Agent Organization

This pattern demonstrates how to manage a complex organization of agents and teams:

```python
class AgentManager:
    def __init__(self):
        # Storage for agents and teams
        self.agents = {}
        self.teams = {}
        
    def register_agent(self, role: str, agent_config: dict):
        """Create and register an agent with specific capabilities"""
        # Create the agent with specified configuration
        agent = AgentConfiguration(
            job_title=role,
            # Apply all other configuration options
            **agent_config
        )
        
        # Store for later use
        self.agents[role] = agent
        return agent
        
    def create_team(self, team_name: str, roles: list[str]):
        """Build a team from existing agent roles"""
        # Verify all required roles exist
        if all(role in self.agents for role in roles):
            # Create the team with the specified agents
            self.teams[team_name] = [
                self.agents[role] for role in roles
            ]
            return True
        else:
            # Identify missing roles
            missing = [role for role in roles if role not in self.agents]
            print(f"Cannot create team, missing roles: {missing}")
            return False
            
    def execute_team_task(self, team_name: str, tasks: list[Task]):
        """Execute tasks using a specific team"""
        if team_name in self.teams:
            # Run the tasks with the specified team
            return MultiAgent.do(self.teams[team_name], tasks)
        else:
            raise ValueError(f"Team '{team_name}' not found")
            
    def list_available_teams(self):
        """Get all available teams and their composition"""
        return {
            team_name: [agent.job_title for agent in team]
            for team_name, team in self.teams.items()
        }
```

This organizational approach lets you build a library of specialized agents and combine them into purpose-built teams, making it easier to manage complex multi-agent systems at scale.

### 2. Task Management

This pattern shows how to manage task queues for efficient processing:

```python
class TaskManager:
    def __init__(self, agent_manager):
        # Task queues for processing
        self.task_queue = []
        self.priority_queue = []
        self.completed_tasks = []
        self.failed_tasks = []
        
        # Reference to agent management
        self.agent_manager = agent_manager
        
    def add_task(self, task: Task, priority: bool = False):
        """Add a task to the appropriate queue"""
        if priority:
            self.priority_queue.append(task)
        else:
            self.task_queue.append(task)
        
    def process_queue(self, team_name: str, batch_size: int = 5):
        """Process tasks in batches using the specified team"""
        # Get the team to use
        team = self.agent_manager.teams.get(team_name)
        if not team:
            raise ValueError(f"Team '{team_name}' not found")
            
        # Process all tasks in batches
        results = []
        
        # First process priority queue
        while self.priority_queue:
            # Take up to batch_size tasks
            current_batch = self.priority_queue[:batch_size]
            self.priority_queue = self.priority_queue[batch_size:]
            
            try:
                # Process the batch
                batch_results = MultiAgent.do(team, current_batch)
                results.extend(batch_results)
                self.completed_tasks.extend(current_batch)
            except Exception as e:
                # Handle failures
                self.failed_tasks.extend(current_batch)
                print(f"Error processing batch: {e}")
        
        # Then process regular queue
        while self.task_queue:
            current_batch = self.task_queue[:batch_size]
            self.task_queue = self.task_queue[batch_size:]
            
            try:
                batch_results = MultiAgent.do(team, current_batch)
                results.extend(batch_results)
                self.completed_tasks.extend(current_batch)
            except Exception as e:
                self.failed_tasks.extend(current_batch)
                print(f"Error processing batch: {e}")
                
        return results
        
    def get_queue_status(self):
        """Get current queue status"""
        return {
            "pending_regular": len(self.task_queue),
            "pending_priority": len(self.priority_queue),
            "completed": len(self.completed_tasks),
            "failed": len(self.failed_tasks)
        }
```

This queue-based system helps you efficiently manage large volumes of tasks, with support for prioritization, batching, and failure handling.

## Integration Examples

### 1. API Integration

This example shows how to expose multi-agent capabilities through an API:

```python
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
app = FastAPI()

# Store for long-running tasks
task_results = {}

# Setup agent team
research_agents = [
    AgentConfiguration(job_title="Internet Researcher", tools=[Search]),
    AgentConfiguration(job_title="Content Analyst"),
    AgentConfiguration(job_title="Report Writer")
]

class TaskRequest(BaseModel):
    task_descriptions: list[str]
    priority: bool = False
    callback_url: str = None

@app.post("/process")
async def process_tasks(request: TaskRequest, background_tasks: BackgroundTasks):
    """API endpoint for submitting tasks to agents"""
    # Generate a unique job ID
    job_id = str(uuid.uuid4())
    
    # Create tasks from descriptions
    tasks = [
        Task(description)
        for description in request.task_descriptions
    ]
    
    # Store initial status
    task_results[job_id] = {
        "status": "processing",
        "tasks": len(tasks),
        "completed": 0,
        "results": None
    }
    
    # Process in background to avoid blocking
    background_tasks.add_task(
        process_in_background, 
        job_id, 
        tasks, 
        request.callback_url
    )
    
    return {"job_id": job_id, "status": "processing"}

async def process_in_background(job_id: str, tasks: list[Task], callback_url: str = None):
    """Process tasks in background and store results"""
    try:
        # Execute with multi-agent system
        results = MultiAgent.do(research_agents, tasks)
        
        # Update status
        task_results[job_id] = {
            "status": "completed",
            "tasks": len(tasks),
            "completed": len(tasks),
            "results": [str(task.response) for task in tasks]
        }
        
        # Send callback if requested
        if callback_url:
            requests.post(callback_url, json={
                "job_id": job_id,
                "status": "completed",
                "results": task_results[job_id]
            })
            
    except Exception as e:
        # Handle failures
        task_results[job_id] = {
            "status": "failed",
            "error": str(e),
            "tasks": len(tasks),
            "completed": 0
        }
        
        if callback_url:
            requests.post(callback_url, json={
                "job_id": job_id,
                "status": "failed",
                "error": str(e)
            })

@app.get("/status/{job_id}")
async def get_status(job_id: str):
    """Check status of submitted job"""
    if job_id in task_results:
        return task_results[job_id]
    return {"status": "not_found"}
```

This API integration provides a production-ready service that can process tasks asynchronously with a multi-agent system, including status tracking and callback notifications.

### 2. Workflow Integration

This example demonstrates integrating multi-agent teams into complex workflows:

```python
class WorkflowSystem:
    def __init__(self):
        # Initialize workflow registry
        self.workflows = {}
        self.agent_manager = AgentManager()
        
        # Setup default agents
        self._setup_default_agents()
        
    def _setup_default_agents(self):
        """Create standard agent roles"""
        default_configs = {
            "researcher": {"tools": [Search]},
            "analyzer": {"reflection": True},
            "writer": {"model": "anthropic/claude-2"},
            "coder": {"tools": ["code_analysis"]},
            "data_processor": {"memory": True}
        }
        
        for role, config in default_configs.items():
            self.agent_manager.register_agent(role, config)
            
        # Create standard teams
        self.agent_manager.create_team("research", ["researcher", "analyzer", "writer"])
        self.agent_manager.create_team("development", ["researcher", "coder", "writer"])
        self.agent_manager.create_team("data", ["researcher", "data_processor", "analyzer"])
        
    def register_workflow(self, name: str, steps: list[dict]):
        """Register a new workflow with steps"""
        # Steps should contain team and task description
        self.workflows[name] = steps
        
    def execute_workflow(self, name: str, input_data: str):
        """Execute a complete workflow"""
        if name not in self.workflows:
            raise ValueError(f"Workflow '{name}' not found")
            
        # Get workflow definition
        steps = self.workflows[name]
        
        # Process each step sequentially
        results = []
        context = input_data
        
        for i, step in enumerate(steps):
            # Create the task with accumulated context
            task = Task(
                f"{step['description']}: {context}",
                tools=step.get('tools', []),
                response_format=step.get('format')
            )
            
            # Get the team to use
            team_name = step['team']
            team = self.agent_manager.teams.get(team_name, [])
            
            # Execute this workflow step
            print(f"Executing step {i+1}/{len(steps)}: {step['description']}")
            result = MultiAgent.do(team, [task])
            
            # Use this result as context for the next step
            context = task.response
            results.append({
                "step": i+1,
                "description": step['description'],
                "result": context
            })
            
        return {
            "workflow": name,
            "steps_completed": len(steps),
            "results": results,
            "final_result": context
        }
        
    def list_available_workflows(self):
        """Get all registered workflows"""
        return {
            name: [step['description'] for step in workflow]
            for name, workflow in self.workflows.items()
        }
```

This workflow system creates a powerful orchestration layer that can coordinate complex, multi-step processes using specialized agent teams for each step.

In production environments, the multi-agent system allows you to build sophisticated AI solutions that mirror human team structures, with specialists collaborating on complex workflows that would be impossible for a single agent to handle effectively.
