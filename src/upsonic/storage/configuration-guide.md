# Comprehensive Guide to Upsonic Configuration Storage

## Introduction and Purpose

The Configuration Storage module is a foundational component of Upsonic that provides robust, persistent storage for settings, credentials, and runtime data. Unlike simple configuration files or environment variables, this system offers:

- **Persistent Storage**: Settings remain available across application restarts
- **Secure Credential Management**: API keys and sensitive data are stored safely
- **Runtime Configurability**: Settings can be modified during execution
- **Structured Data Support**: Store complex nested data structures, not just simple values
- **Transactional Operations**: Ensures configuration integrity even during unexpected shutdowns
- **Multi-Instance Capability**: Separate configurations for system and client settings

This module serves as the backbone for many critical Upsonic features including:

1. **API Authentication**: Securely storing credentials for OpenAI, Anthropic, AWS, and other services
2. **Caching System**: Underlying storage for the caching framework
3. **User Preferences**: Persistence of user-selected settings and configurations
4. **System Identification**: Storage for system IDs and identifiers
5. **Runtime State Management**: Maintaining state between operations and sessions

## Core Components

### ConfigManager Class

The `ConfigManager` class is the central component that handles all configuration storage operations:

```python
from upsonic.storage.configuration import ConfigManager

# Create a custom configuration store
custom_config = ConfigManager(db_name="my_project_config.sqlite")

# Store a configuration value
custom_config.set("api_endpoint", "https://api.example.com/v2")

# Retrieve a configuration value
endpoint = custom_config.get("api_endpoint")
print(f"Using API endpoint: {endpoint}")
```

**Key Implementation Details:**

1. **SQLite Backend**: Uses SQLite for reliable, cross-platform storage without external dependencies
2. **JSON Serialization**: Automatically handles conversion between Python objects and stored values
3. **Signal Handling**: Ensures data is properly committed even during abnormal termination
4. **Graceful Error Management**: Fails safely with sensible defaults when errors occur
5. **Non-Blocking Design**: Minimizes impact on application performance

### Global Configuration Instances

The module provides two pre-configured instances for different purposes:

```python
from upsonic.storage.configuration import Configuration, ClientConfiguration

# System-level configuration (global settings)
system_api_key = Configuration.get("OPENAI_API_KEY")

# Client-specific configuration (per-client settings)
client_setting = ClientConfiguration.get("default_model")
```

**Configuration Contexts:**

1. **`Configuration`**: System-wide settings, API keys, and global preferences
2. **`ClientConfiguration`**: Client-specific settings, cached data, and runtime state

## Core Functionality

### Setting Configuration Values

Store configuration values of various types:

```python
# Store simple values
Configuration.set("max_response_tokens", 1000)
Configuration.set("enable_logging", True)
Configuration.set("service_name", "Upsonic Agent")

# Store complex data structures
agent_config = {
    "model": "openai/gpt-4",
    "temperature": 0.7,
    "max_tokens": 1500,
    "allowed_tools": ["search", "code", "math"]
}
Configuration.set("default_agent_config", agent_config)

# Store nested structures with mixed types
system_status = {
    "services": {
        "main_server": {"running": True, "port": 7541, "uptime": 3600},
        "tools_server": {"running": False, "port": 7542, "error": "Failed to bind port"}
    },
    "resources": {
        "disk_free_mb": 15240,
        "memory_usage_percent": 42.5
    },
    "startup_time": "2023-07-15T14:30:22Z"
}
Configuration.set("system_status", system_status)
```

**How It Works:**

1. The value is converted to a JSON string representation
2. SQLite's REPLACE command ensures atomic updates
3. Any existing value with the same key is overwritten
4. The operation is committed immediately to ensure persistence
5. The function returns a boolean indicating success or failure

### Retrieving Configuration Values

Retrieve stored values with type preservation:

```python
# Get values with defaults for missing keys
max_tokens = Configuration.get("max_response_tokens", 512)  # Default: 512
logging_enabled = Configuration.get("enable_logging", False)  # Default: False

# Retrieve complex structures
agent_config = Configuration.get("default_agent_config")
if agent_config:
    print(f"Using model: {agent_config['model']} at temperature {agent_config['temperature']}")
    
# Access nested values
system_status = Configuration.get("system_status")
if system_status and system_status.get("services", {}).get("main_server", {}).get("running"):
    print("Main server is running")
```

**How It Works:**

1. The key is looked up in the SQLite database
2. The JSON string value is deserialized back to Python objects
3. If the key doesn't exist, the provided default value is returned
4. The original data types and structure are preserved
5. The operation is read-only and doesn't affect the database

### Deleting Configuration Values

Remove configuration entries:

```python
# Remove a configuration entry
success = Configuration.delete("temporary_token")

# Conditionally delete
if Configuration.get("feature_enabled") == False:
    Configuration.delete("feature_settings")
    
# Check if deletion was successful
if Configuration.delete("old_setting"):
    print("Setting removed successfully")
else:
    print("Setting didn't exist or couldn't be removed")
```

**How It Works:**

1. The key is deleted from the SQLite database
2. The change is committed immediately to ensure persistence
3. The function returns a boolean indicating if the key existed and was deleted
4. If the key doesn't exist, the operation succeeds but returns False
5. If an error occurs, the function returns False

### Environment Variable Integration

Automatically load configuration from environment variables:

```python
# Initialize from environment variables
Configuration.initialize("DATABASE_URL")
Configuration.initialize("LOG_LEVEL")

# Check if the initialization was successful
if Configuration.get("DATABASE_URL"):
    print("Database URL loaded from environment")
else:
    print("Database URL not found in environment")
```

**How It Works:**

1. The `.env` file is loaded if present (via python-dotenv)
2. The specified environment variable is checked
3. If the variable exists, its value is stored in the configuration
4. Existing configuration values are not overwritten
5. The operation is idempotent and can be called multiple times safely

## Advanced Usage

### Transaction Management

Ensure configuration integrity during complex operations:

```python
def update_system_configuration(new_settings):
    """Update multiple related configuration settings atomically"""
    try:
        # Make multiple changes
        for key, value in new_settings.items():
            Configuration.set(key, value)
            
        # Explicitly commit all changes
        Configuration.dump()
        return True
    except Exception as e:
        print(f"Configuration update failed: {e}")
        return False
```

**Benefits:**

1. **Atomicity**: Multiple related changes succeed or fail together
2. **Consistency**: Configuration remains in a valid state even after errors
3. **Durability**: Explicit commit ensures all changes are persisted
4. **Isolation**: Changes don't interfere with other operations

### Managing Multiple Configuration Stores

Work with multiple independent configuration stores:

```python
def setup_multi_tenant_configuration():
    """Set up separate configuration stores for different tenants"""
    tenant_configs = {}
    
    # Create configuration stores for each tenant
    tenant_ids = ["tenant1", "tenant2", "tenant3"]
    for tenant_id in tenant_ids:
        tenant_configs[tenant_id] = ConfigManager(db_name=f"{tenant_id}_config.sqlite")
        
        # Initialize with default settings
        tenant_configs[tenant_id].set("created_at", time.time())
        tenant_configs[tenant_id].set("storage_quota_mb", 1000)
        
    return tenant_configs
```

**Use Cases:**

1. **Multi-Tenant Applications**: Separate configuration for each client or tenant
2. **Component Isolation**: Independent configuration for different application components
3. **Testing Environments**: Separate configurations for testing without affecting production
4. **Hierarchical Settings**: Different scopes of configuration (user, project, system)

### Working with Sensitive Data

Handling credentials and sensitive information:

```python
def setup_secure_credentials(keyring_available=False):
    """Configure API credentials with appropriate security"""
    if keyring_available:
        # For desktop applications, use keyring for superior security
        import keyring
        openai_key = keyring.get_password("upsonic", "openai_api_key")
        if openai_key:
            Configuration.set("OPENAI_API_KEY", openai_key)
    else:
        # For server applications, load from environment or prompt
        from getpass import getpass
        
        # Try environment first
        Configuration.initialize("OPENAI_API_KEY")
        
        # If not in environment, prompt securely
        if not Configuration.get("OPENAI_API_KEY"):
            key = getpass("Enter OpenAI API Key: ")
            Configuration.set("OPENAI_API_KEY", key)
```

**Security Considerations:**

1. **Transport Security**: SQLite database is local-only, not exposed over networks
2. **File Security**: Access controlled by operating system file permissions
3. **No Plain-Text Configuration Files**: Avoids sensitive data in easily-readable files
4. **Environment Integration**: Works with secure environment variable management
5. **Memory Protection**: Minimizes exposure of sensitive data in memory

## Best Practices

### Namespace Keys

Use consistent naming conventions for configuration keys:

```python
# Good key naming practices
Configuration.set("server.port", 7541)
Configuration.set("server.host", "localhost")
Configuration.set("logging.level", "info")
Configuration.set("logging.file_path", "/var/log/upsonic.log")

# Use functions to ensure consistent namespacing
def get_config(section, key, default=None):
    """Get a configuration value with consistent namespacing"""
    return Configuration.get(f"{section}.{key}", default)
    
def set_config(section, key, value):
    """Set a configuration value with consistent namespacing"""
    return Configuration.set(f"{section}.{key}", value)
```

**Benefits:**

1. **Organization**: Logical grouping of related settings
2. **Avoid Conflicts**: Prevent key collisions between components
3. **Clarity**: Easier to understand the purpose of each setting
4. **Consistency**: Standardized approach across the application

### Configuration Validation

Validate configuration values for correctness:

```python
def validate_api_configuration():
    """Validate API configuration settings"""
    required_keys = ["OPENAI_API_KEY", "server.port", "server.host"]
    validation_results = {}
    
    # Check for required keys
    for key in required_keys:
        value = Configuration.get(key)
        validation_results[key] = {
            "present": value is not None,
            "value": value if key != "OPENAI_API_KEY" else "[REDACTED]" if value else None
        }
        
    # Validate port range
    port = Configuration.get("server.port")
    if port is not None:
        if isinstance(port, int) and 1024 <= port <= 65535:
            validation_results["server.port"]["valid_range"] = True
        else:
            validation_results["server.port"]["valid_range"] = False
            
    return validation_results
```

**Validation Strategies:**

1. **Required Keys**: Ensure all necessary configuration exists
2. **Type Checking**: Verify values have correct data types
3. **Range Validation**: Confirm numeric values are within acceptable ranges
4. **Format Verification**: Check that strings match expected patterns
5. **Dependency Checking**: Ensure related settings are compatible

### Configuration Documentation

Document configuration options for users:

```python
def generate_configuration_documentation():
    """Generate documentation for available configuration options"""
    config_docs = {
        "server": {
            "port": {
                "description": "Port number for the Upsonic server",
                "type": "integer",
                "default": 7541,
                "range": [1024, 65535]
            },
            "host": {
                "description": "Host interface to bind the server to",
                "type": "string",
                "default": "localhost",
                "options": ["localhost", "0.0.0.0", "127.0.0.1"]
            }
        },
        "logging": {
            "level": {
                "description": "Logging verbosity level",
                "type": "string",
                "default": "info",
                "options": ["debug", "info", "warning", "error"]
            }
        }
    }
    
    return config_docs
```

**Documentation Components:**

1. **Description**: Clear explanation of what the setting controls
2. **Type Information**: Expected data type for the value
3. **Default Values**: What's used when not explicitly configured
4. **Valid Options**: Allowed values or ranges
5. **Examples**: Sample configurations for common scenarios

## Troubleshooting

### Configuration Diagnostics

Tools for diagnosing configuration issues:

```python
def diagnose_configuration():
    """Run diagnostics on the configuration system"""
    results = {
        "database": {
            "exists": False,
            "writable": False,
            "readable": False,
            "path": None
        },
        "key_count": 0,
        "sample_keys": [],
        "environment_vars": {}
    }
    
    # Check configuration database
    import os
    from upsonic.storage.folder import BASE_PATH
    
    db_path = os.path.join(BASE_PATH, "config.sqlite")
    results["database"]["path"] = db_path
    results["database"]["exists"] = os.path.exists(db_path)
    
    if results["database"]["exists"]:
        results["database"]["readable"] = os.access(db_path, os.R_OK)
        results["database"]["writable"] = os.access(db_path, os.W_OK)
        
    # Test basic operations
    test_key = "_diagnostic_test_key"
    test_value = {"timestamp": time.time()}
    
    write_succeeded = Configuration.set(test_key, test_value)
    read_result = Configuration.get(test_key)
    delete_succeeded = Configuration.delete(test_key)
    
    results["operations"] = {
        "write": write_succeeded,
        "read": read_result is not None,
        "read_matches": read_result == test_value if read_result else False,
        "delete": delete_succeeded
    }
    
    # Get information about existing keys
    try:
        import sqlite3
        conn = sqlite3.connect(db_path)
        cursor = conn.cursor()
        cursor.execute("SELECT COUNT(*) FROM config_store")
        results["key_count"] = cursor.fetchone()[0]
        
        cursor.execute("SELECT key FROM config_store LIMIT 10")
        results["sample_keys"] = [row[0] for row in cursor.fetchall()]
        conn.close()
    except Exception as e:
        results["error"] = str(e)
        
    # Check environment variables
    for key in ["OPENAI_API_KEY", "ANTHROPIC_API_KEY", "AWS_ACCESS_KEY_ID"]:
        results["environment_vars"][key] = key in os.environ
        
    return results
```

**Diagnostic Capabilities:**

1. **Database Checks**: Verify the SQLite database exists and is accessible
2. **Operation Tests**: Confirm basic CRUD operations work correctly
3. **Content Analysis**: Examine existing configuration entries
4. **Environment Integration**: Check environment variable availability
5. **Error Detection**: Identify common configuration issues

### Configuration Recovery

Recover from configuration corruption:

```python
def repair_configuration():
    """Attempt to repair corrupted configuration"""
    import os
    import sqlite3
    import shutil
    from upsonic.storage.folder import BASE_PATH
    
    db_path = os.path.join(BASE_PATH, "config.sqlite")
    backup_path = os.path.join(BASE_PATH, "config_backup.sqlite")
    
    # Create backup if it doesn't exist
    if not os.path.exists(backup_path) and os.path.exists(db_path):
        shutil.copy2(db_path, backup_path)
        
    results = {
        "actions": [],
        "recovered_keys": 0,
        "success": False
    }
    
    try:
        # Try to open and query the database
        conn = sqlite3.connect(db_path)
        cursor = conn.cursor()
        cursor.execute("SELECT COUNT(*) FROM config_store")
        count = cursor.fetchone()[0]
        conn.close()
        
        # Database seems fine
        results["actions"].append("Database integrity verified")
        results["success"] = True
        return results
    except sqlite3.DatabaseError:
        # Database is corrupted, try to recreate it
        results["actions"].append("Detected database corruption")
        
        try:
            # Rename corrupted database
            corrupt_path = db_path + ".corrupted"
            if os.path.exists(db_path):
                os.rename(db_path, corrupt_path)
                results["actions"].append(f"Moved corrupted database to {corrupt_path}")
                
            # Create new configuration instance
            new_config = ConfigManager()
            results["actions"].append("Created new configuration database")
            
            # Try to recover from backup if available
            if os.path.exists(backup_path):
                try:
                    backup_conn = sqlite3.connect(backup_path)
                    backup_cursor = backup_conn.cursor()
                    backup_cursor.execute("SELECT key, value FROM config_store")
                    rows = backup_cursor.fetchall()
                    backup_conn.close()
                    
                    # Restore data from backup
                    for key, value in rows:
                        try:
                            new_config.set(key, json.loads(value))
                            results["recovered_keys"] += 1
                        except:
                            continue
                            
                    results["actions"].append(f"Recovered {results['recovered_keys']} keys from backup")
                except:
                    results["actions"].append("Failed to recover from backup")
            
            results["success"] = True
            return results
        except Exception as e:
            results["actions"].append(f"Repair failed: {str(e)}")
            results["success"] = False
            return results
```

**Recovery Approaches:**

1. **Backup Creation**: Automatic backup of working configuration
2. **Integrity Verification**: Checking database structure integrity
3. **Corruption Handling**: Moving corrupted databases aside
4. **Data Recovery**: Restoring settings from backups
5. **Clean Restart**: Creating fresh configuration when needed

## Integration Examples

### Web Application Configuration

Integrating with a FastAPI web application:

```python
from fastapi import FastAPI, Depends, HTTPException
from upsonic.storage.configuration import Configuration
from pydantic import BaseModel

app = FastAPI()

class ServerSettings(BaseModel):
    port: int
    host: str
    worker_threads: int = 4
    debug_mode: bool = False

def get_settings():
    """Dependency for retrieving server settings"""
    settings = Configuration.get("server_settings")
    if not settings:
        # Use defaults if not configured
        return ServerSettings(port=7541, host="localhost")
    return ServerSettings(**settings)

@app.get("/settings")
def read_settings(settings: ServerSettings = Depends(get_settings)):
    """API endpoint to read current settings"""
    return settings

@app.post("/settings")
def update_settings(new_settings: ServerSettings):
    """API endpoint to update settings"""
    # Store the new settings
    Configuration.set("server_settings", new_settings.dict())
    return {"status": "updated", "settings": new_settings}

@app.on_event("startup")
def load_initial_configuration():
    """Initialize configuration on application startup"""
    Configuration.initialize("SERVER_PORT")
    Configuration.initialize("SERVER_HOST")
    
    # Convert environment variables to proper settings format
    port = Configuration.get("SERVER_PORT")
    host = Configuration.get("SERVER_HOST")
    
    if port or host:
        current = Configuration.get("server_settings", {})
        if port:
            current["port"] = int(port)
        if host:
            current["host"] = host
        Configuration.set("server_settings", current)
```

**Integration Features:**

1. **Settings Models**: Using Pydantic for type validation
2. **Dependency Injection**: Clean retrieval of configuration in handlers
3. **API Access**: Endpoints for reading and updating settings
4. **Startup Initialization**: Loading configuration when app starts
5. **Environment Override**: Allowing environment variables to control settings

### CLI Application Configuration

Integrating with a command-line application:

```python
import argparse
import os
from upsonic.storage.configuration import Configuration

def main():
    """Main entry point for CLI application"""
    # Set up argument parser
    parser = argparse.ArgumentParser(description="Upsonic CLI")
    parser.add_argument("--config", help="Path to configuration file")
    parser.add_argument("--api-key", help="OpenAI API Key")
    parser.add_argument("--port", type=int, help="Server port")
    parser.add_argument("--verbose", action="store_true", help="Enable verbose output")
    
    subparsers = parser.add_subparsers(dest="command")
    
    # Add subcommands
    start_parser = subparsers.add_parser("start", help="Start the server")
    stop_parser = subparsers.add_parser("stop", help="Stop the server")
    status_parser = subparsers.add_parser("status", help="Check server status")
    
    # Parse arguments
    args = parser.parse_args()
    
    # Load configuration file if specified
    if args.config:
        load_config_file(args.config)
    
    # Override with command line arguments
    if args.api_key:
        Configuration.set("OPENAI_API_KEY", args.api_key)
    if args.port:
        Configuration.set("server.port", args.port)
    if args.verbose:
        Configuration.set("logging.verbose", True)
    
    # Execute command
    if args.command == "start":
        start_server()
    elif args.command == "stop":
        stop_server()
    elif args.command == "status":
        check_status()
    else:
        parser.print_help()

def load_config_file(config_path):
    """Load configuration from a file"""
    import json
    
    try:
        with open(config_path, 'r') as f:
            config = json.load(f)
            
        # Store each configuration value
        for key, value in config.items():
            Configuration.set(key, value)
            
        print(f"Loaded configuration from {config_path}")
    except Exception as e:
        print(f"Error loading configuration: {e}")

def start_server():
    """Start the server using current configuration"""
    port = Configuration.get("server.port", 7541)
    host = Configuration.get("server.host", "localhost")
    
    print(f"Starting server on {host}:{port}")
    # Server startup code here...

def stop_server():
    """Stop the running server"""
    print("Stopping server...")
    # Server shutdown code here...

def check_status():
    """Check server status"""
    print("Checking server status...")
    # Status check code here...

if __name__ == "__main__":
    main()
```

**Integration Features:**

1. **Command-Line Arguments**: Override configuration via command line
2. **Configuration Files**: Load settings from external files
3. **Subcommands**: Different operations with specific configurations
4. **Default Values**: Sensible defaults when not explicitly configured
5. **Environment Variables**: Automatic loading from environment

## Conclusion

The Upsonic Configuration Storage module provides a robust foundation for managing application settings, credentials, and state. Its design prioritizes:

1. **Simplicity**: Easy-to-use interface with minimal complexity
2. **Reliability**: Durable storage that persists across restarts
3. **Flexibility**: Support for various data types and structures
4. **Security**: Safe handling of sensitive information
5. **Integration**: Seamless work with environment variables and external systems

By leveraging this module effectively, you can create more configurable, resilient, and user-friendly Upsonic applications while maintaining clean separation between code and configuration.
