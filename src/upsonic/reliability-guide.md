# Comprehensive Guide to Upsonic Reliability Processor

The Reliability Processor is a critical component in Upsonic that verifies and validates AI-generated content to ensure accuracy and prevent hallucinations. It conducts multi-layered verification across different content types and can automatically correct problematic content.

## Core Concepts

### 1. Reliability Layers
Reliability layers define the level of verification to apply to AI outputs:

```python
from upsonic import AgentConfiguration

# Define a reliability layer with maximum hallucination prevention
class ReliabilityLayer:
    prevent_hallucination = 10  # Maximum verification

# Create an agent with reliability features
reliable_agent = AgentConfiguration(
    job_title="Verified Researcher",
    reliability_layer=ReliabilityLayer
)
```

The `prevent_hallucination` setting controls verification intensity:
- `0`: No verification (default)
- `10`: Maximum verification with multiple checks

### 2. Validation Process

The validation process involves four specialized checks:

```python
# Each validation focuses on a specific content type
validations = [
    ("url_validation", "Verifies URLs mentioned in content"),
    ("number_validation", "Verifies numerical claims"),
    ("information_validation", "Verifies factual statements"),
    ("code_validation", "Verifies code snippets and syntax")
]
```

For each validation type, a specialized agent:
1. Analyzes the content
2. Checks against provided context
3. Flags suspicious elements
4. Provides confidence scores

### 3. Result Editing

When suspicious content is detected, an editor agent automatically fixes it:

```python
# Simplified editor process
if validation_result.any_suspicion:
    editor_agent = AgentConfiguration("Editor")
    editor_task = Task(
        "Fix suspicious content by setting flagged items to None",
        context=[original_response, validation_result]
    )
    editor_agent.do(editor_task)
    return editor_task.response  # Return the cleaned version
```

The editor removes or replaces suspicious content while preserving the original structure and format.

## Validation Types

### 1. URL Validation
Checks the authenticity and source of URLs in content:

```python
url_validation_prompt = """
Focus on basic URL source validation:

Source Verification:
- Check if the source is come from the content. But dont make assumption just check the context and try to find exact things. If not flag it.
- If you can see the things in the context everything okay (Trusted Source).

IMPORTANT: If the URL source cannot be verified, flag it as suspicious.
"""
```

This validation:
- Detects URLs in content using regex
- Verifies each URL against provided context
- Flags URLs that cannot be verified in context
- Skips validation if no URLs are present

### 2. Number Validation
Verifies numerical claims in the content:

```python
number_validation_prompt = """
Focus on basic numerical validation:

Number Verification:
- Check if the source is come from the content. But dont make assumption just check the context and try to find exact things. If not flag it.
- If you can see the things in the context everything okay (Trusted Source).

IMPORTANT: If the numbers cannot be verified, flag them as suspicious.
"""
```

This validation:
- Identifies numerical statements and statistics
- Verifies them against provided context
- Flags numbers that don't appear in context
- Prevents fabricated statistics

### 3. Information Validation
Validates factual statements and claims:

```python
information_validation_prompt = """
Focus on basic information validation:

Information Verification:
- Check if the source is come from the content. But dont make assumption just check the context and try to find exact things. If not flag it.
- If you can see the things in the context everything okay (Trusted Source).

IMPORTANT: If the information cannot be verified, flag it as suspicious.
"""
```

This validation:
- Reviews general factual claims
- Compares each claim against context data
- Flags information not supported by context
- Maintains factual integrity

### 4. Code Validation
Verifies code snippets and programming examples:

```python
code_validation_prompt = """
Focus on basic code validation:

Code Verification:
- Check if the source is come from the content. But dont make assumption just check the context and try to find exact things. If not flag it.
- If you can see the things in the context everything okay (Trusted Source).

IMPORTANT: If the code cannot be verified or appears suspicious, flag it as suspicious.
"""
```

This validation:
- Examines code content for accuracy
- Verifies code against context
- Flags potentially problematic code
- Ensures code integrity

## Using the Reliability Processor

### 1. Basic Implementation
The simplest way to enable reliability processing:

```python
from upsonic import AgentConfiguration, Task

# Define reliability settings
class ReliabilitySettings:
    prevent_hallucination = 10  # Full verification

# Create agent with reliability
agent = AgentConfiguration(
    job_title="Reliable Assistant",
    reliability_layer=ReliabilitySettings
)

# Create and execute task
task = Task("Provide information about AI safety")
result = agent.do(task)

# Result will be automatically verified and cleaned
```

### 2. Custom Reliability Layer
Creating a more customized reliability layer:

```python
class EnhancedReliability:
    prevent_hallucination = 10
    
    # Additional settings could be added in future versions
    verify_sources = True
    confidence_threshold = 0.8
    fact_checking = True
    
    @property
    def max_verification(self):
        return self.prevent_hallucination == 10

agent = AgentConfiguration(
    job_title="Enhanced Reliable Agent",
    reliability_layer=EnhancedReliability()  # Instance with custom settings
)
```

### 3. Selective Reliability
Applying reliability only to specific tasks:

```python
# Standard agent with reliability
reliable_agent = AgentConfiguration(
    job_title="Careful Researcher",
    reliability_layer=ReliabilityLayer
)

# Regular agent without reliability
regular_agent = AgentConfiguration(
    job_title="Fast Assistant"
    # No reliability layer
)

# Choose based on task requirements
def process_task(task, require_verification=False):
    if require_verification:
        return reliable_agent.do(task)
    else:
        return regular_agent.do(task)
```

## Advanced Usage

### 1. Validation Result Analysis
Accessing and analyzing validation details:

```python
from upsonic.reliability_processor import ValidationResult, ValidationPoint

# Create task with reliability and execute
result = reliable_agent.do(task)

# Access validation results from agent's process
def inspect_validation(task):
    # Note: This is conceptual as validation details aren't 
    # directly exposed in the current implementation
    validation = task._validation_result
    
    if validation:
        print(f"Overall suspicious: {validation.any_suspicion}")
        print(f"Confidence score: {validation.overall_confidence}")
        
        # Inspect individual validations
        for val_type in ["url", "number", "information", "code"]:
            point = getattr(validation, f"{val_type}_validation")
            print(f"{val_type.title()} validation:")
            print(f"  Suspicious: {point.is_suspicious}")
            print(f"  Points: {point.suspicious_points}")
            print(f"  Feedback: {point.feedback}")
```

### 2. Custom Editor Implementation
Creating a customized editor behavior:

```python
# Custom editor prompt with specialized behavior
custom_editor_prompt = """
Custom editor with special requirements:

1. For suspicious URLs:
   - Replace with [URL REMOVED] placeholder
   - Maintain surrounding context

2. For suspicious numbers:
   - Replace with approximate ranges when possible
   - Use [UNVERIFIED] prefix otherwise

3. For suspicious information:
   - Replace with [REQUIRES VERIFICATION] tag
   - Suggest verification steps

4. For suspicious code:
   - Replace with commented code showing issues
   - Provide safe alternatives when possible
"""

# Configure specialized editor
editor_task = Task(
    custom_editor_prompt,
    context=[original_task, validation_result]
)
```

### 3. Reliability Pipeline
Creating a multi-stage reliability pipeline:

```python
class ReliabilityPipeline:
    def __init__(self):
        # Initial verification agent
        self.verifier = AgentConfiguration(
            job_title="Verification Agent",
            reliability_layer={"prevent_hallucination": 10}
        )
        
        # Specialized editor agent
        self.editor = AgentConfiguration(
            job_title="Editor Agent"
        )
        
        # Final review agent
        self.reviewer = AgentConfiguration(
            job_title="Review Agent"
        )
    
    def process(self, task):
        # Initial response generation
        initial_result = self.verifier.do(task)
        
        # If issues found, pass to editor
        if hasattr(initial_result, '_has_suspicious_content') and initial_result._has_suspicious_content:
            edit_task = Task(
                "Edit and correct suspicious content",
                context=[task, initial_result]
            )
            edited_result = self.editor.do(edit_task)
            
            # Final review for quality assurance
            review_task = Task(
                "Review edited content for quality and completeness",
                context=[task, initial_result, edited_result]
            )
            return self.reviewer.do(review_task)
        
        # No issues found, return initial result
        return initial_result
```

## Internal Components

### 1. ValidationPoint Class
Used to store validation results for a specific type:

```python
class ValidationPoint(ObjectResponse):
    is_suspicious: bool                           # Whether suspicious content was found
    feedback: str                                 # Detailed feedback
    suspicious_points: list[str]                  # Specific suspicious elements
    source_reliability: SourceReliability         # Source reliability assessment
    verification_method: str                      # Method used for verification
    confidence_score: float                       # Confidence in verification
```

### 2. ValidationResult Class
Aggregates results from multiple validation points:

```python
class ValidationResult(ObjectResponse):
    url_validation: ValidationPoint               # URL validation results
    number_validation: ValidationPoint            # Number validation results
    information_validation: ValidationPoint       # Information validation results
    code_validation: ValidationPoint              # Code validation results
    any_suspicion: bool                           # Whether any validation raised suspicion
    suspicious_points: list[str]                  # Combined list of suspicious points
    overall_feedback: str                         # Aggregated feedback
    overall_confidence: float                     # Overall confidence score
```

This class also includes a `calculate_suspicion()` method that:
- Determines if any validations found suspicious content
- Aggregates suspicious points from all validations
- Calculates overall confidence score
- Generates detailed feedback

### 3. URL Detection Functions
Utility functions for finding URLs in content:

```python
def find_urls_in_text(text: str) -> List[str]:
    """Find all URLs in the given text using regex pattern matching."""
    url_pattern = r'http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\\(\\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+'
    return re.findall(url_pattern, text)

def contains_urls(texts: List[str]) -> bool:
    """Check if any of the provided texts contain URLs."""
    for text in texts:
        if not isinstance(text, str):
            continue
        if find_urls_in_text(text):
            return True
    return False
```

## Best Practices

### 1. Appropriate Reliability Levels
Choose appropriate reliability settings based on use case:

```python
# High-stakes scenarios (medical, financial, legal)
class HighStakesReliability:
    prevent_hallucination = 10  # Maximum verification
    
# Medium-stakes scenarios (business reports, research)
class StandardReliability:
    prevent_hallucination = 10  # Full verification but less strict
    
# Low-stakes scenarios (creative content, brainstorming)
class MinimalReliability:
    prevent_hallucination = 0  # No verification needed
```

### 2. Context Preparation
Ensure proper context for reliable verification:

```python
def prepare_reliable_task(query, documents):
    # Create comprehensive context with source attribution
    context = []
    
    for doc in documents:
        # Add explicit source markers
        context.append(f"Source '{doc.title}' (Trusted): {doc.content}")
    
    # Create task with rich context
    return Task(
        description=query,
        context=context
    )
```

### 3. Performance Considerations
Balance reliability with performance needs:

```python
def choose_reliability_level(task_type, urgency):
    if task_type == "critical" and urgency != "immediate":
        # Full verification for critical non-urgent tasks
        return {"prevent_hallucination": 10}
    elif task_type == "important":
        # Standard verification for important tasks
        return {"prevent_hallucination": 10}
    else:
        # No verification for routine or urgent tasks
        return None
```

## Implementation Examples

### 1. Fact-Checking System
Using reliability for automated fact-checking:

```python
class FactCheckingSystem:
    def __init__(self):
        self.research_agent = AgentConfiguration(
            job_title="Research Agent",
            tools=[Search]  # Can search for information
        )
        
        self.verification_agent = AgentConfiguration(
            job_title="Verification Agent",
            reliability_layer={"prevent_hallucination": 10}
        )
    
    def fact_check(self, claim, sources):
        # Research relevant information
        research_task = Task(
            f"Research information related to: {claim}",
            tools=[Search]
        )
        self.research_agent.do(research_task)
        
        # Verify claim against research
        verification_task = Task(
            f"Verify this claim: {claim}",
            context=[research_task, sources]
        )
        
        # Result will be automatically verified
        return self.verification_agent.do(verification_task)
```

### 2. Medical Information System
Using reliability for medical information:

```python
class MedicalInfoSystem:
    def __init__(self):
        # Create agent with maximum reliability
        self.agent = AgentConfiguration(
            job_title="Medical Information Specialist",
            reliability_layer={"prevent_hallucination": 10}
        )
        
        # Load medical knowledge base
        self.knowledge_base = KnowledgeBase()
        self.knowledge_base.add_file("medical_guidelines.pdf")
        self.knowledge_base.add_file("drug_interactions.pdf")
    
    def get_medical_info(self, query):
        # Create task with knowledge base context
        task = Task(
            description=query,
            context=self.knowledge_base
        )
        
        # Process with reliability checks
        result = self.agent.do(task)
        
        # Add disclaimer
        return {
            "response": result,
            "disclaimer": "This information has been automatically verified against medical sources, but should not replace professional medical advice."
        }
```
