# Comprehensive Guide to Upsonic Knowledge Base

## Basic Setup and Usage

### 1. Creating a Knowledge Base
```python
from upsonic import KnowledgeBase

# Create a new knowledge base
kb = KnowledgeBase()

# Create with initial sources
kb = KnowledgeBase(sources=["path/to/file1.pdf", "path/to/file2.md"])
```

### 2. Managing Sources
```python
# Add individual files
kb.add_file("documents/guide.pdf")
kb.add_file("documents/api_reference.md")

# Remove files
kb.remove_file("documents/old_guide.pdf")

# Check current sources
print(kb.sources)  # Lists all sources
```

## Integration with Agents

### 1. Basic Agent Integration
```python
from upsonic import AgentConfiguration, Task

# Create knowledge base
kb = KnowledgeBase()
kb.add_file("company_policies.pdf")
kb.add_file("product_manual.md")

# Create agent with knowledge base
agent = AgentConfiguration(
    job_title="Support Specialist",
    knowledge_base=kb
)

# Create task that will use knowledge base
task = Task("Explain our return policy")
result = agent.do(task)
```

### 2. Multiple Knowledge Bases
```python
# Technical documentation knowledge base
tech_kb = KnowledgeBase()
tech_kb.add_file("api_docs.md")
tech_kb.add_file("tech_specs.pdf")

# Business documentation knowledge base
business_kb = KnowledgeBase()
business_kb.add_file("policies.pdf")
business_kb.add_file("procedures.md")

# Create specialized agents
tech_agent = AgentConfiguration(
    job_title="Technical Support",
    knowledge_base=tech_kb
)

business_agent = AgentConfiguration(
    job_title="Business Analyst",
    knowledge_base=business_kb
)
```

## Working with Different File Types

### 1. PDF Documents
```python
# Adding PDF files
kb = KnowledgeBase()
kb.add_file("documents/report.pdf")
kb.add_file("documents/whitepaper.pdf")
```

### 2. Markdown Files
```python
# Adding markdown documentation
kb.add_file("docs/README.md")
kb.add_file("docs/CONTRIBUTING.md")
```

### 3. Converting to Markdown
```python
# Convert all sources to markdown format
markdown_content = kb.markdown(client)

# Access converted content
for source, content in markdown_content.knowledges.items():
    print(f"Source: {source}")
    print(f"Content: {content[:100]}...")  # Preview first 100 chars
```

## Advanced Usage Patterns

### 1. Dynamic Knowledge Base Updates
```python
class DynamicKnowledgeBase:
    def __init__(self):
        self.kb = KnowledgeBase()
        
    def update_knowledge(self, new_file: str):
        self.kb.add_file(new_file)
        
    def remove_outdated(self, old_file: str):
        self.kb.remove_file(old_file)
        
    def refresh_knowledge(self, files: list[str]):
        self.kb = KnowledgeBase(sources=files)
```

### 2. Context-Aware Knowledge Base
```python
class ContextualKnowledge:
    def __init__(self):
        self.technical_kb = KnowledgeBase()
        self.business_kb = KnowledgeBase()
        self.general_kb = KnowledgeBase()
    
    def get_knowledge_base(self, context: str) -> KnowledgeBase:
        if context == "technical":
            return self.technical_kb
        elif context == "business":
            return self.business_kb
        return self.general_kb
```

## Best Practices

### 1. File Management
- Keep files organized in logical directories
- Use consistent naming conventions
- Regularly update and maintain source files
- Remove outdated or irrelevant sources

### 2. Performance Optimization
- Limit the number of sources to necessary documents
- Use appropriate file formats
- Consider file size and processing time
- Regular cleanup of unused sources

### 3. Content Organization
```python
# Organize by categories
product_kb = KnowledgeBase()
product_kb.add_file("products/specifications.pdf")
product_kb.add_file("products/user_guides.pdf")

support_kb = KnowledgeBase()
support_kb.add_file("support/troubleshooting.md")
support_kb.add_file("support/faq.md")
```

## Integration Examples

### 1. Web API Integration
```python
from fastapi import FastAPI
app = FastAPI()

kb = KnowledgeBase()

@app.post("/add_knowledge")
async def add_knowledge(file_path: str):
    kb.add_file(file_path)
    return {"status": "added", "file": file_path}

@app.get("/get_knowledge")
async def get_knowledge():
    return {"sources": kb.sources}
```

### 2. Automated Knowledge Updates
```python
import schedule
import time

class KnowledgeManager:
    def __init__(self):
        self.kb = KnowledgeBase()
        
    def update_knowledge(self):
        # Add new files from a directory
        new_files = self.scan_directory("new_docs/")
        for file in new_files:
            self.kb.add_file(file)
            
    def run_schedule(self):
        schedule.every().day.at("00:00").do(self.update_knowledge)
        while True:
            schedule.run_pending()
            time.sleep(3600)
```

## Error Handling

### 1. Basic Error Handling
```python
def safe_add_file(kb: KnowledgeBase, file_path: str) -> bool:
    try:
        kb.add_file(file_path)
        return True
    except Exception as e:
        print(f"Error adding file: {e}")
        return False

def safe_remove_file(kb: KnowledgeBase, file_path: str) -> bool:
    try:
        kb.remove_file(file_path)
        return True
    except ValueError:
        print(f"File not found: {file_path}")
        return False
    except Exception as e:
        print(f"Error removing file: {e}")
        return False
```

### 2. Validation Functions
```python
def validate_file_path(file_path: str) -> bool:
    import os
    return os.path.exists(file_path)

def validate_file_type(file_path: str) -> bool:
    allowed_extensions = ['.pdf', '.md', '.txt']
    return any(file_path.endswith(ext) for ext in allowed_extensions)

# Usage
if validate_file_path(file_path) and validate_file_type(file_path):
    kb.add_file(file_path)
```

## Troubleshooting Common Issues

1. **File Not Found Issues**:
   - Verify file paths are correct and absolute
   - Check file permissions
   - Ensure files exist before adding

2. **Content Processing Issues**:
   - Verify file formats are supported
   - Check file encoding
   - Monitor file sizes

3. **Performance Issues**:
   - Limit number of sources
   - Monitor memory usage
   - Implement caching if needed

## Knowledge Base Tips

1. **Maintenance**:
   - Regularly review and update sources
   - Remove outdated information
   - Verify file integrity

2. **Organization**:
   - Group related files together
   - Use consistent naming conventions
   - Maintain documentation about sources

3. **Optimization**:
   - Use appropriate file formats
   - Monitor performance
   - Implement caching strategies
