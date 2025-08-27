# Technical Foundation Specification: Gemini CLI VS Code Bridge

## Technology Stack Decisions

### Backend Architecture

- **Framework**: Python 3.8+ (standard library focused)
- **Runtime**: Python 3.8+ (updated for enhanced threading and subprocess support)
- **Key Dependencies**:
  - `watchdog`: File system monitoring with cross-platform event handling
  - `subprocess`: Child process management for Gemini CLI integration
  - `threading`: Two-thread coordination model for concurrent I/O operations
  - `queue`: Thread-safe communication between main and output reader threads
  - `pathlib`: Cross-platform file path handling
- **Development Tools**: pytest framework, pylint for code quality, black for formatting

### Process Architecture

- **Process Management**: Single persistent Gemini CLI subprocess with captured stdin/stdout streams
- **Threading Model**: Two-thread sequential processing (main thread + dedicated output reader)
- **Communication Strategy**: Bounded queue (maxsize=100) with threading.Event for shutdown coordination
- **Error Recovery**: Automatic subprocess restart on failure with graceful degradation modes

### File System Integration

- **File Monitoring**: Debounced watchdog observers (200ms delay) to prevent rapid-fire processing
- **I/O Strategy**: Append-only response logging with UTF-8 encoding support
- **File Handling**: Atomic read operations for prompt.md, buffered writes to response.md
- **Cross-Platform**: Path normalization and encoding handling across Windows, macOS, Linux

## API Contract Specifications

### Process Management Interface

#### Subprocess Initialization

```python
# Gemini CLI process configuration
process_config = {
    "command": ["gemini"],
    "stdin": subprocess.PIPE,
    "stdout": subprocess.PIPE,
    "stderr": subprocess.PIPE,
    "text": True,
    "bufsize": 1,  # Line buffering for real-time response
    "universal_newlines": True
}

# Platform-specific timeout and retry settings
SUBPROCESS_TIMEOUT = 30  # seconds
MAX_RESTART_ATTEMPTS = 3
RESTART_DELAY = 2  # seconds between restart attempts
```

#### Thread Communication Protocol

```python
# Output queue message format
OutputMessage = {
    "type": "response" | "error" | "status",
    "content": str,
    "timestamp": float,
    "source": "stdout" | "stderr"
}

# Shutdown coordination
shutdown_event = threading.Event()
output_queue = queue.Queue(maxsize=100)
```

### File System Event Handling

#### File Watcher Configuration

```python
# Watchdog event handler specification
class PromptHandler(FileSystemEventHandler):
    def on_modified(self, event):
        if event.is_directory:
            return
        if event.src_path.endswith('prompt.md'):
            # Debounce mechanism - 200ms delay
            self.schedule_prompt_processing()

# File processing pipeline
def process_prompt_file():
    """
    Read entire content of prompt.md and send to Gemini CLI
    
    Returns:
        bool: True if successful, False if error occurred
    
    Raises:
        FileNotFoundError: If prompt.md doesn't exist
        UnicodeDecodeError: If file contains invalid UTF-8
        ProcessLookupError: If Gemini CLI process is not running
    """
```

#### Response File Management

```python
def append_response(content: str) -> bool:
    """
    Append Gemini CLI response to response.md
    
    Args:
        content: Raw output from Gemini CLI stdout
        
    Returns:
        bool: True if write successful
        
    Error Handling:
        - Disk space full: Log error, continue operation
        - File permissions: Attempt chmod, fallback to console output
        - Encoding errors: Use 'replace' strategy for invalid characters
    """
    
    try:
        with open('response.md', 'a', encoding='utf-8') as f:
            f.write(f"\n{content}\n")
            f.flush()  # Ensure immediate write to disk
        return True
    except (IOError, OSError) as e:
        logging.error(f"Failed to write response: {e}")
        return False
```

## Data Model Architecture

### Process State Management

```python
class GeminiProcess:
    """
    Manages lifecycle and communication with Gemini CLI subprocess
    """
    def __init__(self):
        self.process: Optional[subprocess.Popen] = None
        self.is_running: bool = False
        self.restart_count: int = 0
        self.last_activity: float = time.time()
        
    def start(self) -> bool:
        """Initialize Gemini CLI subprocess"""
        
    def send_prompt(self, prompt: str) -> bool:
        """Send prompt to Gemini CLI stdin"""
        
    def is_healthy(self) -> bool:
        """Check if process is responsive"""
        
    def restart(self) -> bool:
        """Restart failed Gemini CLI process"""
```

### Thread Coordination Model

```python
class BridgeCoordinator:
    """
    Coordinates file watching and subprocess communication
    """
    def __init__(self):
        self.gemini_process = GeminiProcess()
        self.output_queue = queue.Queue(maxsize=100)
        self.shutdown_event = threading.Event()
        self.output_thread: Optional[threading.Thread] = None
        
    def start_bridge(self):
        """Initialize all components and start monitoring"""
        
    def stop_bridge(self):
        """Graceful shutdown with resource cleanup"""
```

### Error Classification System

```python
from enum import Enum

class ErrorType(Enum):
    RECOVERABLE_FILE_ERROR = "recoverable_file"
    RECOVERABLE_PROCESS_ERROR = "recoverable_process"
    FATAL_CONFIGURATION_ERROR = "fatal_config"
    NETWORK_CONNECTIVITY_ERROR = "network"

class ErrorHandler:
    """
    Centralized error handling with recovery strategies
    """
    @staticmethod
    def handle_error(error_type: ErrorType, exception: Exception) -> bool:
        """
        Process error and attempt recovery
        
        Returns:
            bool: True if recovery successful, False if fatal
        """
```

## Integration Architecture

### Cross-Platform Compatibility Layer

#### Platform-Specific Configurations

```python
import platform
import sys

class PlatformConfig:
    """
    Platform-specific settings for optimal compatibility
    """
    def __init__(self):
        self.platform = platform.system().lower()
        self.config = self._load_platform_config()
        
    def _load_platform_config(self):
        configs = {
            'windows': {
                'subprocess_creation_flags': subprocess.CREATE_NEW_PROCESS_GROUP,
                'file_encoding': 'utf-8-sig',  # Handle BOM
                'path_separator': '\\',
                'gemini_command': ['gemini.exe']
            },
            'darwin': {
                'subprocess_creation_flags': 0,
                'file_encoding': 'utf-8',
                'path_separator': '/',
                'gemini_command': ['gemini']
            },
            'linux': {
                'subprocess_creation_flags': 0,
                'file_encoding': 'utf-8',
                'path_separator': '/',
                'gemini_command': ['gemini']
            }
        }
        return configs.get(self.platform, configs['linux'])
```

#### File System Event Normalization

```python
from pathlib import Path
from watchdog.observers import Observer

class CrossPlatformFileWatcher:
    """
    Normalize file system events across platforms
    """
    def __init__(self, path: str, callback: callable):
        self.path = Path(path).resolve()
        self.callback = callback
        self.observer = Observer()
        self._setup_platform_specific_watching()
        
    def _setup_platform_specific_watching(self):
        """
        Configure file watching for platform-specific behaviors
        
        Windows: Handle file locking and delayed write notifications
        macOS: Manage FSEvents volume-level notifications  
        Linux: Process inotify events with proper debouncing
        """
```

### Configuration Management

#### Environment Variable Schema

```bash
# Core Configuration
GEMINI_CLI_PATH="/usr/local/bin/gemini"        # Custom Gemini CLI location
BRIDGE_WORK_DIR="/path/to/project"             # Working directory for files
BRIDGE_LOG_LEVEL="INFO"                        # DEBUG, INFO, WARNING, ERROR

# File Configuration  
BRIDGE_PROMPT_FILE="prompt.md"                 # Input file name
BRIDGE_RESPONSE_FILE="response.md"             # Output file name
BRIDGE_BACKUP_RESPONSES="true"                 # Enable response file backup

# Process Configuration
GEMINI_SUBPROCESS_TIMEOUT="30"                 # Subprocess timeout in seconds
BRIDGE_MAX_RESTART_ATTEMPTS="3"                # Maximum process restart attempts
BRIDGE_DEBOUNCE_DELAY="200"                    # File watch debounce in milliseconds

# Threading Configuration
BRIDGE_OUTPUT_QUEUE_SIZE="100"                 # Maximum queued output messages
BRIDGE_SHUTDOWN_TIMEOUT="5"                    # Graceful shutdown timeout
```

#### Configuration Validation

```python
class ConfigValidator:
    """
    Validate configuration and provide sensible defaults
    """
    REQUIRED_CONFIGS = ['GEMINI_CLI_PATH']
    DEFAULT_VALUES = {
        'BRIDGE_WORK_DIR': '.',
        'BRIDGE_LOG_LEVEL': 'INFO',
        'BRIDGE_PROMPT_FILE': 'prompt.md',
        'BRIDGE_RESPONSE_FILE': 'response.md',
        'GEMINI_SUBPROCESS_TIMEOUT': '30',
        'BRIDGE_MAX_RESTART_ATTEMPTS': '3',
        'BRIDGE_DEBOUNCE_DELAY': '200'
    }
    
    @classmethod
    def validate_and_load(cls) -> dict:
        """
        Validate environment configuration and return normalized settings
        
        Raises:
            ConfigurationError: If required settings missing or invalid
        """
```

## Development Environment Setup

### Local Development Requirements

```bash
# System Requirements
Python >= 3.8.0
OS: Windows 10+, macOS 12+, or Ubuntu 20+
Memory: 50MB available RAM
Disk: 10MB free space for logs and temporary files

# Installation Steps
1. Clone repository: git clone <repository-url>
2. Create virtual environment: python -m venv venv
3. Activate environment: source venv/bin/activate (Linux/Mac) or venv\Scripts\activate (Windows)
4. Install dependencies: pip install watchdog
5. Verify Gemini CLI: gemini --version
6. Configure environment variables (see Configuration section)
7. Run bridge: python bridge.py
```

### Testing Framework

- **Unit Testing**: pytest framework with subprocess mocking for isolated testing
- **Integration Testing**: Real Gemini CLI process testing with temporary file fixtures
- **Cross-Platform Testing**: Automated testing across Windows, macOS, and Linux environments
- **Stress Testing**: 1000+ rapid file changes without deadlocks or memory leaks
- **Error Scenario Testing**: Process crashes, file permission failures, network interruptions

### Test Data Management

```python
import pytest

# Test fixtures for integration testing
@pytest.fixture
def sample_prompts():
    """Provide sample prompts for testing"""
    return [
        "What is the capital of France?",
        "Explain quantum computing in simple terms",
        "Write a Python function to reverse a string"
    ]

@pytest.fixture
def expected_response_patterns():
    """Expected response patterns for validation"""
    return [
        r"Paris is the capital",
        r"quantum.*superposition",
        r"def.*reverse.*string"
    ]

@pytest.fixture(params=[
    "invalid_gemini_path",
    "readonly_response_file", 
    "missing_prompt_file",
    "process_sudden_termination"
])
def error_scenarios(request):
    """Parametrized fixture for error scenario testing"""
    return request.param
```

### Performance Monitoring

```python
import time

class PerformanceMonitor:
    """
    Track bridge performance and resource usage
    """
    def __init__(self):
        self.start_time = time.time()
        self.prompt_count = 0
        self.response_count = 0
        self.error_count = 0
        self.memory_usage = []
        
    def log_prompt_processed(self, processing_time: float):
        """Record prompt processing metrics"""
        
    def get_performance_summary(self) -> dict:
        """Return comprehensive performance statistics"""
```

## Implementation Validation Checklist

### Pre-Development Validation

- [ ] **Strategic Decision Translation**: All architectural decisions from strategic blueprint correctly implemented in technical specifications
- [ ] **Database Schema Validation**: File-based storage model supports all required conversation persistence features
- [ ] **API Contract Coverage**: Process management and file I/O operations completely specified
- [ ] **External Integration Specs**: Gemini CLI subprocess integration patterns clearly defined
- [ ] **Development Environment**: Complete setup documentation with cross-platform considerations

### Post-Implementation Validation

- [ ] **Process Communication**: Gemini CLI subprocess responds correctly to stdin prompts across all platforms
- [ ] **File System Integration**: File watching triggers immediate and reliable prompt processing
- [ ] **Thread Coordination**: Two-thread model operates without deadlocks through 1000+ prompt cycles
- [ ] **Error Recovery**: Automatic process restart functions correctly after subprocess failures
- [ ] **Cross-Platform Compatibility**: Identical functionality verified on Windows, macOS, and Linux

### Quality Assurance Standards

- [ ] **Memory Stability**: 48-hour continuous operation without memory leaks or performance degradation
- [ ] **Response Time**: Sub-500ms latency from file save to subprocess stdin delivery
- [ ] **Error Handling**: 95% of error scenarios recover automatically without user intervention
- [ ] **Data Integrity**: Zero data loss during normal operation and graceful shutdown procedures
- [ ] **Resource Usage**: Memory consumption under 50MB during 8-hour development sessions

## Next Phase Handoff

**For MVP Prioritization**: Technical foundation supports all Must Have features from MoSCoW analysis. Two-thread architecture provides sufficient complexity management while maintaining real-time responsiveness. File-based I/O model eliminates need for complex state management or database dependencies.

**Implementation Risks**: 
- Threading coordination complexity requires iterative testing and debugging
- Cross-platform subprocess behavior variations need comprehensive validation
- Gemini CLI process stability depends on external network conditions and API availability

**Decision Points**: 
- Sequential vs parallel prompt processing may need adjustment based on performance testing
- Error recovery strategies may require refinement based on real-world failure patterns  
- Configuration management approach could expand if user customization requests increase