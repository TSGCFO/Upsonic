# Comprehensive Guide to Upsonic Server Manager

The Server Manager in Upsonic provides robust server control capabilities, handling server startup, shutdown, monitoring, and port management. It's designed to ensure reliable server operations even in complex environments.

## Core Functionality

### 1. Server Initialization and Configuration

The `ServerManager` class handles server lifecycle with comprehensive configuration:

```python
from upsonic.server_manager import ServerManager

# Initialize a server manager for main application
main_server = ServerManager(
    app_path="upsonic.server.api:app",  # Path to FastAPI app
    host="0.0.0.0",                     # Listen on all interfaces
    port=7541,                          # Port to use
    name="main"                         # Server identifier
)
```

The initialization parameters include:
- `app_path`: Module path to the FastAPI application
- `host`: Network interface to bind to
- `port`: Port number to listen on
- `name`: Unique identifier for the server

### 2. Starting a Server

The `start` method launches the server process with configurable output:

```python
# Start server with default settings
main_server.start()

# Start with redirected output (for production)
main_server.start(redirect_output=True)

# Force start (kill any existing processes on the port)
main_server.start(force=True)
```

When starting a server, the manager:
1. Checks if the port is available
2. Cleans up any conflicting processes
3. Launches uvicorn with the specified configuration
4. Tracks the process ID for later management
5. Waits for the server to become responsive

### 3. Stopping a Server

The `stop` method gracefully shuts down the server:

```python
# Gracefully stop the server
main_server.stop()
```

The stopping process:
1. Attempts to terminate the process group
2. Falls back to forced termination if needed
3. Cleans up PID files
4. Verifies the server is completely stopped

### 4. Checking Server Status

The `is_running` method verifies if the server is active:

```python
# Check if the server is running
if main_server.is_running():
    print("Server is running")
else:
    print("Server is not running")
    main_server.start()
```

The status check:
1. Verifies the process is still responsive
2. Checks PID files for externally started servers
3. Confirms the process type is a Python process

## Advanced Usage

### 1. Port Management

The server manager includes robust port conflict resolution:

```python
# Creating a custom server manager with port conflict handling
def create_safe_server(app_path, port_range_start, name):
    # Try ports in range until finding an available one
    for port in range(port_range_start, port_range_start + 10):
        server = ServerManager(app_path, "localhost", port, name)
        # Check if port is in use without killing processes
        if not server._is_port_in_use():
            return server
            
    raise RuntimeError(f"No available ports in range {port_range_start}-{port_range_start + 10}")
```

The port management utilities:
- Detect if ports are already in use
- Identify processes using specific ports
- Safely free ports when needed
- Handle complex network configurations

### 2. Process Group Management

The server manager handles process groups to ensure clean shutdowns:

```python
def start_server_cluster(server_configs):
    """Start multiple coordinated servers."""
    servers = []
    
    for config in server_configs:
        server = ServerManager(
            app_path=config["app_path"],
            host=config["host"],
            port=config["port"],
            name=config["name"]
        )
        
        try:
            server.start(redirect_output=True)
            servers.append(server)
        except Exception as e:
            # If any server fails to start, stop all started servers
            for started_server in servers:
                started_server.stop()
            raise RuntimeError(f"Failed to start server cluster: {e}")
            
    return servers
```

Process group features:
- Creates isolated process groups for clean management
- Manages parent-child process relationships
- Ensures all related processes are properly terminated
- Prevents orphaned processes

### 3. Logging and Debugging

Configure server output for production or debugging:

```python
# Development mode with console output
dev_server = ServerManager(app_path, host, port, "dev")
dev_server.start(redirect_output=False)

# Production mode with logs
prod_server = ServerManager(app_path, host, port, "prod")
prod_server.start(redirect_output=True)
```

Logging features:
- Console output for development
- File-based logging for production
- Separate error and standard output
- Automatic log directory creation

## System Integration

### 1. Application Lifecycle Management

Integrate server management into application lifecycle for clean startup and shutdown:

```python
import atexit

class ServerController:
    def __init__(self):
        self.main_server = ServerManager(
            "upsonic.server.api:app", 
            "0.0.0.0", 
            7541, 
            "main"
        )
        self.tools_server = ServerManager(
            "upsonic.tools_server.server.api:app", 
            "0.0.0.0", 
            7542, 
            "tools"
        )
        
        # Register shutdown handler
        atexit.register(self.shutdown_all)
        
    def start_all(self, redirect_output=True):
        self.main_server.start(redirect_output=redirect_output)
        self.tools_server.start(redirect_output=redirect_output)
        
    def shutdown_all(self):
        """Shutdown all servers gracefully"""
        # Stop servers in reverse order
        if self.tools_server.is_running():
            self.tools_server.stop()
            
        if self.main_server.is_running():
            self.main_server.stop()
            
    def check_status(self):
        """Check status of all servers"""
        return {
            "main": self.main_server.is_running(),
            "tools": self.tools_server.is_running()
        }
```

This pattern ensures that:
- All servers start in the correct order
- Shutdown happens gracefully when the application exits
- The application can check server status at any time

### 2. Docker Environment Integration

Using the server manager in containerized environments:

```python
def configure_docker_server():
    """Configure server for Docker environment"""
    # Use environment variables for configuration
    host = os.environ.get("SERVER_HOST", "0.0.0.0")
    port = int(os.environ.get("SERVER_PORT", "7541"))
    
    # Create the server manager
    server = ServerManager(
        app_path="upsonic.server.api:app",
        host=host,
        port=port,
        name="docker_server"
    )
    
    # Configure signal handlers for Docker
    def handle_sigterm(signum, frame):
        server.stop()
        sys.exit(0)
        
    # Register signal handlers
    signal.signal(signal.SIGTERM, handle_sigterm)
    signal.signal(signal.SIGINT, handle_sigterm)
    
    return server
```

This Docker integration:
- Uses environment variables for configuration
- Handles Docker container lifecycle signals
- Ensures clean container shutdown
- Prevents orphaned processes in containers

### 3. Systemd Service Integration

Create a robust service for Linux systems:

```python
def setup_systemd_service():
    """Create configuration for systemd service"""
    service_content = """[Unit]
Description=Upsonic Server
After=network.target

[Service]
User=upsonic
WorkingDirectory=/opt/upsonic
ExecStart=/usr/bin/python3 -m upsonic.server start
ExecStop=/usr/bin/python3 -m upsonic.server stop
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
"""
    
    # Write service file
    with open("/etc/systemd/system/upsonic.service", "w") as f:
        f.write(service_content)
        
    # Enable and start service
    subprocess.run(["systemctl", "daemon-reload"])
    subprocess.run(["systemctl", "enable", "upsonic"])
    subprocess.run(["systemctl", "start", "upsonic"])
```

This creates a production-ready service that:
- Automatically starts on system boot
- Restarts after failures
- Integrates with system service management
- Follows best practices for Linux services

## Error Handling

### 1. Port Conflict Resolution

Advanced port conflict handling:

```python
def ensure_available_port(host, desired_port, max_attempts=5):
    """Find an available port, starting with desired_port"""
    current_port = desired_port
    
    for attempt in range(max_attempts):
        # Try creating a socket
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        try:
            s.bind((host, current_port))
            s.close()
            # If we reach here, port is available
            return current_port
        except socket.error:
            # Port is in use, try killing the process
            manager = ServerManager("dummy", host, current_port, "temp")
            if manager._kill_process_using_port():
                # Process killed, port should be free now
                return current_port
                
            # Try next port
            current_port += 1
        finally:
            s.close()
            
    raise RuntimeError(f"Could not find available port after {max_attempts} attempts")
```

This approach:
- Attempts to find an available port
- Tries to free up the desired port first
- Falls back to alternative ports if needed
- Provides a configurable number of attempts

### 2. Robust Process Management

Handling complex process termination scenarios:

```python
def force_terminate_process(pid, timeout=5):
    """Forcefully terminate a process with proper cleanup"""
    try:
        process = psutil.Process(pid)
        
        # Check if process is a Python process to avoid terminating wrong process
        if not process.name().startswith("python"):
            return False
            
        # Try graceful termination first
        process.terminate()
        
        try:
            # Wait for termination
            process.wait(timeout=timeout)
            return True
        except psutil.TimeoutExpired:
            # Force kill if graceful termination fails
            process.kill()
            process.wait(timeout=1)
            return True
            
    except (psutil.NoSuchProcess, psutil.AccessDenied, ProcessLookupError):
        # Process already gone or can't be accessed
        return False
```

This ensures:
- Processes are properly identified before termination
- Graceful shutdown is attempted first
- Forced termination is used as a fallback
- Proper cleanup happens in all scenarios

## Advanced Deployment

### 1. Multi-Environment Configuration

Configure servers for different environments:

```python
def create_environment_server(environment):
    """Create server configuration for specific environment"""
    env_configs = {
        "development": {
            "app_path": "upsonic.server.api:app",
            "host": "localhost",
            "port": 7541,
            "debug": True,
            "redirect_output": False
        },
        "testing": {
            "app_path": "upsonic.server.api:app",
            "host": "0.0.0.0",
            "port": 7542,
            "debug": True,
            "redirect_output": True
        },
        "production": {
            "app_path": "upsonic.server.api:app",
            "host": "0.0.0.0",
            "port": 7541,
            "debug": False,
            "redirect_output": True
        }
    }
    
    config = env_configs.get(environment)
    if not config:
        raise ValueError(f"Unknown environment: {environment}")
        
    server = ServerManager(
        app_path=config["app_path"],
        host=config["host"],
        port=config["port"],
        name=f"{environment}_server"
    )
    
    return server, config
```

This allows:
- Environment-specific configurations
- Different ports for different environments
- Custom debugging settings per environment
- Consistent server management across environments

### 2. High Availability Setup

Creating a redundant server configuration:

```python
class HighAvailabilityController:
    def __init__(self, app_path, port_range_start, port_range_end):
        self.app_path = app_path
        self.port_range = range(port_range_start, port_range_end)
        self.servers = []
        
    def start_cluster(self, server_count=2):
        """Start multiple server instances for redundancy"""
        available_ports = []
        
        # Find available ports
        for port in self.port_range:
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            try:
                s.bind(("0.0.0.0", port))
                available_ports.append(port)
                if len(available_ports) >= server_count:
                    break
            except socket.error:
                continue
            finally:
                s.close()
                
        if len(available_ports) < server_count:
            raise RuntimeError(f"Not enough available ports. Found {len(available_ports)}, need {server_count}")
            
        # Start servers on available ports
        for i, port in enumerate(available_ports):
            server = ServerManager(
                app_path=self.app_path,
                host="0.0.0.0",
                port=port,
                name=f"ha_server_{i}"
            )
            server.start(redirect_output=True)
            self.servers.append(server)
            
        return [s.port for s in self.servers]
        
    def stop_cluster(self):
        """Stop all servers in the cluster"""
        for server in self.servers:
            server.stop()
        self.servers = []
```

This high-availability setup:
- Starts multiple redundant servers
- Finds available ports automatically
- Provides failure tolerance
- Enables load balancing across instances

## Troubleshooting

### 1. Debugging Server Issues

Functions for diagnosing server problems:

```python
def diagnose_server_issues(server_manager):
    """Diagnose common server issues"""
    results = {
        "is_running": server_manager.is_running(),
        "port_available": not server_manager._is_port_in_use(),
        "pid_file_exists": os.path.exists(server_manager._pid_file),
        "stored_pid": server_manager._read_pid(),
        "system_info": {
            "platform": sys.platform,
            "python_version": sys.version,
            "memory_available": psutil.virtual_memory().available / (1024 * 1024)
        }
    }
    
    # Check if PID from file is actually running
    pid = server_manager._read_pid()
    if pid:
        try:
            process = psutil.Process(pid)
            results["pid_running"] = process.is_running()
            results["process_name"] = process.name()
            results["process_cmdline"] = process.cmdline()
        except psutil.NoSuchProcess:
            results["pid_running"] = False
            
    return results
```

This tool:
- Checks server status comprehensively
- Verifies port availability
- Validates PID file consistency
- Provides system diagnostics
- Examines process details

### 2. Recovery Procedures

Functions for recovering from failure states:

```python
def recover_server(server_manager):
    """Attempt to recover server from failure state"""
    # Stop any potentially running instance
    try:
        server_manager.stop()
    except Exception as e:
        print(f"Warning: Error during stop phase: {e}")
    
    # Clean up PID file
    server_manager._cleanup_pid()
    
    # Force cleanup port
    try:
        server_manager._force_cleanup_port()
    except Exception as e:
        print(f"Warning: Error cleaning up port: {e}")
        
    # Try to start server with forced mode
    try:
        server_manager.start(force=True)
        return True
    except Exception as e:
        print(f"Recovery failed: {e}")
        return False
```

This recovery process:
- Attempts to clean up existing processes
- Removes stale PID files
- Forcefully frees required ports
- Uses forced start mode
- Provides detailed error information