# ADK AGUI Middleware

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/trendmicro/adk-agui-middleware)
[![CI](https://github.com/trendmicro/adk-agui-middleware/actions/workflows/ci.yml/badge.svg)](https://github.com/trendmicro/adk-agui-middleware/actions/workflows/ci.yml)
[![CodeQL](https://github.com/trendmicro/adk-agui-middleware/actions/workflows/codeql.yml/badge.svg)](https://github.com/trendmicro/adk-agui-middleware/actions/workflows/codeql.yml)
[![Semgrep](https://github.com/trendmicro/adk-agui-middleware/actions/workflows/semgrep.yml/badge.svg)](https://github.com/trendmicro/adk-agui-middleware/actions/workflows/semgrep.yml)
[![Gitleaks](https://github.com/trendmicro/adk-agui-middleware/actions/workflows/gitleaks.yml/badge.svg)](https://github.com/trendmicro/adk-agui-middleware/actions/workflows/gitleaks.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code style: Ruff](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json)](https://github.com/astral-sh/ruff)
[![Security: Bandit](https://img.shields.io/badge/security-bandit-yellow.svg)](https://github.com/PyCQA/bandit)
[![Type Checker: mypy](https://img.shields.io/badge/type_checker-mypy-blue.svg)](https://github.com/python/mypy)

**Enterprise-grade Python 3.13+ middleware that seamlessly bridges Google's Agent Development Kit (ADK) with AGUI protocol, providing high-performance Server-Sent Events streaming and Human-in-the-Loop (HITL) workflow orchestration.**

## Overview

ADK AGUI Middleware is a production-ready Python 3.13+ library engineered for enterprise-scale integration between Google's Agent Development Kit and AGUI (Agent User Interface) protocol. The middleware provides a robust foundation for building AI agent applications with real-time streaming capabilities, concurrent session management, and sophisticated Human-in-the-Loop (HITL) workflows.

### Key Features

- **🏗️ Enterprise Architecture**: Modular design with dependency injection, abstract base classes, and clean separation of concerns
- **⚡ High-Performance SSE**: Asynchronous Server-Sent Events streaming with bidirectional event translation pipeline
- **🔒 Session Management**: Thread-safe session locking with configurable timeout and retry mechanisms
- **🤝 HITL Workflows**: Complete orchestration of Human-in-the-Loop tool call workflows with state persistence
- **🔄 Event Translation**: Real-time ADK ↔ AGUI event conversion with streaming message management
- **🛡️ Production-Ready**: Comprehensive error handling, structured logging, and graceful shutdown mechanisms
- **🎯 Type Safety**: Full Python 3.13 type annotations with strict mypy validation and Pydantic data models

### Highlights

- **Modernized foundation**: The core was redesigned from the ground up, replacing the legacy implementation and closing logic gaps that previously prevented outbound data from being delivered.
- **Conversation lifecycle APIs**: Fresh helpers&mdash;`get_agui_thread_list`, `delete_agui_thread`, `patch_agui_state`, `get_agui_message_snapshot`, and `get_agui_state_snapshot`&mdash;simplify querying, updating, and auditing multi-thread conversations.
- **Lifecycle-aware middleware plugins**: Each request instantiates a stateful middleware class that can coordinate message pipelines, convert accumulated events, and execute custom workflows across ADK↔ADK, ADK↔AGUI, and AGUI↔AGUI translations while handling timeouts gracefully.
- **Pluggable concurrency control**: A lock abstraction (memory-backed by default) allows swapping in providers such as Redis and tweaking retry behavior, enabling reliable operation across multiple Kubernetes pods without leaking in-memory state.
- **Observability extensions**: Input/output logging plugins capture request context, store conversation histories, and reshape final payloads&mdash;for example, mapping unexpected errors to standard error codes before responses reach clients.
- **Adaptive context extraction**: `app_name`, `user_id`, `session_id`, and `extract_initial_state` now accept request objects so headers and other metadata can populate runtime context beyond the standard `RunAgentInput` fields.
- **AGUI input transformers**: Custom plugins can modify inbound AGUI payloads, including promoting queued names and IDs and remapping selected user messages into tool invocations for long-running ADK tools.
- **Extension-ready craftsmanship**: The codebase follows SOLID principles, keeps functions compact, and exposes extensible base classes, making overrides straightforward when bespoke behavior is required.
- **Strict static analysis**: Comprehensive typing paired with rigorous mypy enforcement keeps the codebase close to statically checked reliability.
- **Utility-rich toolkit**: Dedicated utility modules streamline complex conversion logic, including support for ThinkingMessage and Thinking mode plus standards-compliant SSE encoding.
- **Revamped HITL pipeline**: The Human-in-the-Loop experience was rebuilt to leverage long-running tools for high-throughput, operator-friendly workflows.

## Installation

```bash
pip install adk-agui-middleware
```

### Requirements

- Python 3.13+ (recommended 3.13.3+)
- Google ADK >= 1.9.0
- AGUI Protocol >= 0.1.7
- FastAPI >= 0.104.0

## Architecture Overview

### High-Level System Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Web Applications<br/>React/Vue/Angular]
        MOBILE[Mobile Apps<br/>iOS/Android/Flutter]
        API[API Clients<br/>REST/GraphQL]
    end

    subgraph "Middleware Layer"
        ENDPOINT[FastAPI Endpoint<br/>register_agui_endpoint]
        SSE_SVC[SSE Service<br/>Event Orchestration]
        LOCK[Session Lock Handler<br/>Concurrency Control]
    end

    subgraph "Core Processing"
        AGUI_USER[AGUI User Handler<br/>Workflow Orchestrator]
        RUNNING[Running Handler<br/>Agent Execution]
        USER_MSG[User Message Handler<br/>Input Processing]
        SESSION[Session Handler<br/>State Management]
    end

    subgraph "Event Translation"
        TRANSLATOR[Event Translator<br/>ADK ↔ AGUI]
        CONVERTER[SSE Converter<br/>Protocol Formatter]
    end

    subgraph "Session Management"
        SESS_MGR[Session Manager<br/>ADK Operations]
        HIST_SVC[History Service<br/>Conversation Persistence]
    end

    subgraph "Google ADK"
        RUNNER[ADK Runner<br/>Agent Container]
        AGENT[Base Agent<br/>Custom Implementation]
        ADK_SESS[ADK Session Service<br/>State Persistence]
    end

    subgraph "Infrastructure"
        SHUTDOWN[Shutdown Handler<br/>Graceful Cleanup]
        LOGGER[Structured Logging<br/>Event Tracking]
        UTILS[Utility Functions<br/>Helper Components]
    end

    %% Client connections
    WEB --> ENDPOINT
    MOBILE --> ENDPOINT
    API --> ENDPOINT

    %% Core processing flow
    ENDPOINT --> SSE_SVC
    SSE_SVC --> LOCK
    LOCK --> AGUI_USER

    %% AGUI workflow orchestration
    AGUI_USER --> RUNNING
    AGUI_USER --> USER_MSG
    AGUI_USER --> SESSION

    %% Agent execution
    RUNNING --> RUNNER
    RUNNER --> AGENT
    RUNNING --> TRANSLATOR

    %% Event processing
    TRANSLATOR --> CONVERTER
    SSE_SVC --> CONVERTER

    %% Session operations
    SESSION --> SESS_MGR
    SESS_MGR --> ADK_SESS
    ENDPOINT --> HIST_SVC
    HIST_SVC --> SESS_MGR
    HIST_SVC --> RUNNING

    %% Infrastructure
    SSE_SVC --> SHUTDOWN
    RUNNING --> LOGGER
    SESS_MGR --> LOGGER

    classDef client fill:#e3f2fd,color:#000,stroke:#1976d2
    classDef middleware fill:#e8f5e8,color:#000,stroke:#388e3c
    classDef core fill:#fff3e0,color:#000,stroke:#f57c00
    classDef translation fill:#fce4ec,color:#000,stroke:#c2185b
    classDef session fill:#f3e5f5,color:#000,stroke:#7b1fa2
    classDef adk fill:#e1f5fe,color:#000,stroke:#0288d1
    classDef infra fill:#f1f8e9,color:#000,stroke:#689f38

    class WEB,MOBILE,API client
    class ENDPOINT,SSE_SVC,LOCK middleware
    class AGUI_USER,RUNNING,USER_MSG,SESSION core
    class TRANSLATOR,CONVERTER translation
    class SESS_MGR,HIST_SVC session
    class RUNNER,AGENT,ADK_SESS adk
    class SHUTDOWN,LOGGER,UTILS infra
```

### Event Translation Pipeline

```mermaid
graph LR
    subgraph "ADK Events"
        ADK_TEXT[Text Content<br/>Streaming]
        ADK_FUNC[Function Calls<br/>Tool Invocation]
        ADK_RESP[Function Response<br/>Tool Results]
        ADK_STATE[State Delta<br/>Session Updates]
    end

    subgraph "Translation Engine"
        TRANSLATOR[Event Translator<br/>Core Engine]
        MSG_UTIL[Message Utils<br/>Text Processing]
        FUNC_UTIL[Function Utils<br/>Tool Processing]
        STATE_UTIL[State Utils<br/>Delta Management]
    end

    subgraph "AGUI Events"
        AGUI_START[Text Message Start]
        AGUI_CONTENT[Text Message Content]
        AGUI_END[Text Message End]
        AGUI_TOOL[Tool Call Event]
        AGUI_RESULT[Tool Result Event]
        AGUI_DELTA[State Delta Event]
        AGUI_SNAPSHOT[State Snapshot Event]
    end

    subgraph "SSE Output"
        SSE_DATA[data: JSON payload]
        SSE_EVENT[event: Event type]
        SSE_ID[id: Unique identifier]
    end

    %% ADK to Translation
    ADK_TEXT --> TRANSLATOR
    ADK_FUNC --> TRANSLATOR
    ADK_RESP --> TRANSLATOR
    ADK_STATE --> TRANSLATOR

    %% Translation processing
    TRANSLATOR --> MSG_UTIL
    TRANSLATOR --> FUNC_UTIL
    TRANSLATOR --> STATE_UTIL

    %% Translation to AGUI
    MSG_UTIL --> AGUI_START
    MSG_UTIL --> AGUI_CONTENT
    MSG_UTIL --> AGUI_END
    FUNC_UTIL --> AGUI_TOOL
    FUNC_UTIL --> AGUI_RESULT
    STATE_UTIL --> AGUI_DELTA
    STATE_UTIL --> AGUI_SNAPSHOT

    %% AGUI to SSE
    AGUI_START --> SSE_DATA
    AGUI_CONTENT --> SSE_DATA
    AGUI_END --> SSE_DATA
    AGUI_TOOL --> SSE_DATA
    AGUI_RESULT --> SSE_DATA
    AGUI_DELTA --> SSE_DATA
    AGUI_SNAPSHOT --> SSE_DATA

    AGUI_START --> SSE_EVENT
    AGUI_CONTENT --> SSE_EVENT
    AGUI_END --> SSE_EVENT
    AGUI_TOOL --> SSE_EVENT
    AGUI_RESULT --> SSE_EVENT
    AGUI_DELTA --> SSE_EVENT
    AGUI_SNAPSHOT --> SSE_EVENT

    classDef adk fill:#e1f5fe,color:#000,stroke:#0288d1
    classDef translation fill:#fff3e0,color:#000,stroke:#f57c00
    classDef agui fill:#fce4ec,color:#000,stroke:#c2185b
    classDef sse fill:#e8f5e8,color:#000,stroke:#388e3c

    class ADK_TEXT,ADK_FUNC,ADK_RESP,ADK_STATE adk
    class TRANSLATOR,MSG_UTIL,FUNC_UTIL,STATE_UTIL translation
    class AGUI_START,AGUI_CONTENT,AGUI_END,AGUI_TOOL,AGUI_RESULT,AGUI_DELTA,AGUI_SNAPSHOT agui
    class SSE_DATA,SSE_EVENT,SSE_ID sse
```

## Quick Start

### Basic Enterprise Implementation

```python
from fastapi import FastAPI, Request
from google.adk.agents import BaseAgent
from adk_agui_middleware import SSEService, register_agui_endpoint
from adk_agui_middleware.data_model.config import RunnerConfig
from adk_agui_middleware.data_model.context import ConfigContext

# Initialize FastAPI application
app = FastAPI(
    title="AI Agent Service",
    description="Enterprise ADK-AGUI middleware service",
    version="1.0.0"
)

# Define your custom ADK agent
class EnterpriseAgent(BaseAgent):
    """Custom enterprise agent with HITL capabilities."""

    def __init__(self):
        super().__init__()
        self.instructions = """
        You are an enterprise AI assistant with access to various tools.

        Key behaviors:
        - Always ask for human approval before executing high-impact operations
        - Provide clear explanations for tool usage and reasoning
        - Handle errors gracefully and inform users of any issues
        - Maintain conversation context across multiple interactions
        """

# Context extraction functions for multi-tenant deployment
async def extract_user_id(content, request: Request) -> str:
    """Extract user ID from JWT token or API headers."""
    # Production: Implement JWT token validation
    auth_header = request.headers.get("Authorization", "")
    if auth_header.startswith("Bearer "):
        # Decode JWT and extract user_id
        pass

    # Fallback to header-based user identification
    return request.headers.get("X-User-ID", "anonymous")

async def extract_app_name(content, request: Request) -> str:
    """Extract application name from subdomain or headers."""
    host = request.headers.get("Host", "localhost")
    if "." in host:
        subdomain = host.split(".")[0]
        return f"enterprise-{subdomain}"
    return "enterprise-default"

# Configure middleware context
config_context = ConfigContext(
    app_name=extract_app_name,
    user_id=extract_user_id,
    session_id=lambda content, req: content.thread_id,
)

# Configure runner with production settings
runner_config = RunnerConfig(
    use_in_memory_services=True  # Set to False for production persistence
)

# Initialize and register services
agent = EnterpriseAgent()
sse_service = SSEService(agent, runner_config, config_context)
register_agui_endpoint(app, sse_service)


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        app,
        host="0.0.0.0",
        port=8000,
        log_level="info",
        access_log=True
    )

```


## API Reference

### Core Endpoints

| Method | Endpoint | Description | Request Body | Response Type |
|--------|----------|-------------|--------------|---------------|
| `POST` | `/` | Execute agent with streaming response | `RunAgentInput` | `EventSourceResponse` |
| `GET` | `/thread/list` | List user's conversation threads | - | `List[Dict[str, str]]` |
| `DELETE` | `/thread/{thread_id}` | Delete conversation thread | - | `Dict[str, str]` |
| `GET` | `/message_snapshot/{thread_id}` | Get conversation history | - | `MessagesSnapshotEvent` |
| `GET` | `/state_snapshot/{thread_id}` | Get session state snapshot | - | `StateSnapshotEvent` |
| `PATCH` | `/state/{thread_id}` | Update session state | `List[JSONPatch]` | `Dict[str, str]` |

### Event Types

The middleware supports comprehensive event translation between ADK and AGUI formats:

#### AGUI Event Types
- `TEXT_MESSAGE_START` - Begin streaming text response
- `TEXT_MESSAGE_CONTENT` - Streaming text content chunk
- `TEXT_MESSAGE_END` - Complete streaming text response
- `TOOL_CALL` - Agent tool/function invocation
- `TOOL_RESULT` - Tool execution result
- `STATE_DELTA` - Incremental state update
- `STATE_SNAPSHOT` - Complete state snapshot
- `RUN_STARTED` - Agent execution began
- `RUN_FINISHED` - Agent execution completed
- `ERROR` - Error event with details

#### SSE Format
All events are converted to Server-Sent Events format:
```javascript
{
  "data": "{...}",        // JSON-serialized event data
  "event": "event_type",  // AGUI event type
  "id": "unique_id"       // UUID for event correlation
}
```
### Security Best Practices

- **Authentication**: JWT token validation and RBAC integration
- **Session Isolation**: Proper tenant isolation for multi-tenant deployments
- **Audit Logging**: Comprehensive audit trails for compliance requirements
- **Error Handling**: Secure error handling without information leakage

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

## Security

See [SECURITY.md](SECURITY.md) for our security policy and vulnerability reporting process.
