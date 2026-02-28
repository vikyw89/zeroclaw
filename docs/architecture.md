# ZeroClaw Architecture

This document provides visual diagrams of the ZeroClaw system architecture.
All diagrams use [Mermaid](https://mermaid.js.org/) syntax and render inline on GitHub.

Last updated: **February 28, 2026**.

---

## Diagram 1: High-Level System Architecture

```mermaid
flowchart TD
    CLI["main.rs — CLI Entrypoint"]
    Config["config/ — Schema + Loading"]
    Agent["agent/ — Orchestration Loop"]

    CLI --> Config
    Config --> Agent

    subgraph Providers["providers/ — Model Providers"]
        direction TB
        P1["OpenAI / Codex / Compatible"]
        P2["Anthropic / Gemini / GLM"]
        P3["Copilot / OpenRouter / Ollama"]
        P4["Bedrock / Telnyx"]
        P5["Reliable wrapper + Router"]
    end

    subgraph Channels["channels/ — Messaging Channels"]
        direction TB
        C1["Telegram / Discord / Slack"]
        C2["IRC / Matrix / Nostr"]
        C3["WhatsApp / WhatsApp Web / Signal"]
        C4["Email / iMessage / MQTT"]
        C5["Lark / DingTalk / QQ"]
        C6["Mattermost / WATI / ClawdTalk"]
        C7["LINQ / Nextcloud Talk"]
    end

    subgraph Tools["tools/ — Tool Execution"]
        direction TB
        T1["Shell / FileRead / FileWrite / FileEdit"]
        T2["Browser / WebFetch / WebSearch"]
        T3["MemoryStore / MemoryRecall / MemoryForget"]
        T4["Screenshot / MCP / Delegate / Subagent"]
        T5["Git / Glob / ContentSearch / PDF / DOCX"]
        T6["Schedule / Cron / Hardware / SOP / WASM"]
        T7["ApplyPatch / TaskPlan / Process / HttpRequest"]
    end

    subgraph Memory["memory/ — Memory Backends"]
        direction TB
        M1["Markdown / SQLite / Postgres"]
        M2["Qdrant / Vector / Hybrid / Lucid / None"]
        M3["Embeddings + Chunker"]
    end

    subgraph Security["security/ — Policy + Sandboxing"]
        direction TB
        S1["Policy / Pairing / Secrets / Audit"]
        S2["Bubblewrap / Firejail / Landlock / Docker"]
        S3["Roles / OTP / E-Stop"]
        S4["LeakDetector / PromptGuard / SyscallAnomaly"]
    end

    subgraph Observability["observability/ — Telemetry"]
        direction TB
        O1["Log / Prometheus / OTel"]
        O2["Verbose / Noop / Multi / Cost / RuntimeTrace"]
    end

    subgraph Runtime["runtime/ — Execution Adapters"]
        direction TB
        R1["Native"]
        R2["Docker"]
        R3["WASM"]
    end

    subgraph Gateway["gateway/ — HTTP Gateway"]
        direction TB
        G1["REST API / OpenAI-compat / OpenClaw-compat"]
        G2["SSE / WebSocket / Static Files"]
    end

    subgraph Peripherals["peripherals/ — Hardware"]
        direction TB
        H1["Serial / RPi GPIO"]
        H2["NucleoFlash / ArduinoFlash / ArduinoUpload"]
        H3["UnoQ Bridge / UnoQ Setup"]
    end

    subgraph Supporting["Supporting Modules"]
        direction TB
        SOP["sop/ — SOP Engine + Dispatch"]
        Cron["cron/ — Scheduler + Store"]
        Plugins["plugins/ — Discovery + Registry"]
        Tunnel["tunnel/ — Cloudflare / Ngrok / Tailscale"]
        Hooks["hooks/ — Lifecycle Hooks"]
        RAG["rag/ — Retrieval Augmented Generation"]
        SkillForge["skillforge/ + skills/"]
        Coord["coordination/ + economic/ + goals/"]
        Health["health/ + heartbeat/ + cost/"]
        Identity["identity.rs + util.rs + multimodal.rs"]
    end

    Agent <-->|"ChatRequest / Response"| Providers
    Agent <-->|"ChannelMessage"| Channels
    Agent <-->|"ToolCall / ToolResult"| Tools
    Agent <-->|"store / recall / search"| Memory
    Agent -->|"events"| Observability
    Agent <-->|"exec context"| Runtime
    Agent -->|"enforce"| Security
    Gateway -->|"inbound requests"| Agent
    Peripherals -->|"hardware tools"| Tools
    Agent <-->|"SOP dispatch"| SOP
    Agent <-->|"scheduled tasks"| Cron
    Plugins -->|"extend"| Agent
    Tunnel -->|"expose"| Gateway
    Hooks -->|"lifecycle"| Agent
```

---

## Diagram 2: Trait Extension Points

```mermaid
classDiagram
    class Provider {
        <<trait>>
        +chat(req: ChatRequest) Result~ChatResponse~
        +stream_chat(req: ChatRequest) Stream~ChatChunk~
        +supports_tools() bool
        +supports_vision() bool
        +name() String
    }

    class Channel {
        <<trait>>
        +send(msg: OutboundMessage) Result
        +listen() Stream~ChannelMessage~
        +health_check() Result
        +name() String
    }

    class Tool {
        <<trait>>
        +name() String
        +description() String
        +parameters_schema() Value
        +execute(params: Value) Result~ToolResult~
    }

    class Memory {
        <<trait>>
        +store(key: String, value: String) Result
        +recall(key: String) Result~Option~String~~
        +search(query: String) Result~Vec~MemoryEntry~~
        +delete(key: String) Result
        +list() Result~Vec~String~~
    }

    class Observer {
        <<trait>>
        +on_event(event: AgentEvent)
    }

    class RuntimeAdapter {
        <<trait>>
        +name() String
        +has_shell_access() bool
        +has_filesystem_access() bool
        +storage_path() PathBuf
        +supports_long_running() bool
    }

    class Peripheral {
        <<trait>>
        +name() String
        +board_type() String
        +connect() Result
        +disconnect() Result
        +tools() Vec~Box~dyn Tool~~
    }

    class OpenAiProvider {
        +chat()
        +stream_chat()
    }
    class AnthropicProvider {
        +chat()
        +stream_chat()
    }
    class GeminiProvider {
        +chat()
        +stream_chat()
    }
    class OllamaProvider {
        +chat()
        +stream_chat()
    }
    class ReliableProvider {
        +chat()
        +with_fallback()
    }

    class TelegramChannel {
        +send()
        +listen()
    }
    class DiscordChannel {
        +send()
        +listen()
    }
    class SlackChannel {
        +send()
        +listen()
    }
    class WhatsAppWebChannel {
        +send()
        +listen()
    }

    class ShellTool {
        +execute()
    }
    class FileReadTool {
        +execute()
    }
    class BrowserTool {
        +execute()
    }
    class WebFetchTool {
        +execute()
    }

    class MarkdownMemory {
        +store()
        +recall()
        +search()
    }
    class SqliteMemory {
        +store()
        +recall()
        +search()
    }
    class PostgresMemory {
        +store()
        +recall()
        +search()
    }
    class QdrantMemory {
        +store()
        +recall()
        +search()
    }

    class LogObserver {
        +on_event()
    }
    class PrometheusObserver {
        +on_event()
    }
    class OtelObserver {
        +on_event()
    }

    class NativeRuntime {
        +has_shell_access()
        +has_filesystem_access()
    }
    class DockerRuntime {
        +has_shell_access()
        +has_filesystem_access()
    }
    class WasmRuntime {
        +has_shell_access()
        +has_filesystem_access()
    }

    class SerialPeripheral {
        +connect()
        +tools()
    }
    class RpiPeripheral {
        +connect()
        +tools()
    }

    Provider <|.. OpenAiProvider
    Provider <|.. AnthropicProvider
    Provider <|.. GeminiProvider
    Provider <|.. OllamaProvider
    Provider <|.. ReliableProvider

    Channel <|.. TelegramChannel
    Channel <|.. DiscordChannel
    Channel <|.. SlackChannel
    Channel <|.. WhatsAppWebChannel

    Tool <|.. ShellTool
    Tool <|.. FileReadTool
    Tool <|.. BrowserTool
    Tool <|.. WebFetchTool

    Memory <|.. MarkdownMemory
    Memory <|.. SqliteMemory
    Memory <|.. PostgresMemory
    Memory <|.. QdrantMemory

    Observer <|.. LogObserver
    Observer <|.. PrometheusObserver
    Observer <|.. OtelObserver

    RuntimeAdapter <|.. NativeRuntime
    RuntimeAdapter <|.. DockerRuntime
    RuntimeAdapter <|.. WasmRuntime

    Peripheral <|.. SerialPeripheral
    Peripheral <|.. RpiPeripheral
```

---

## Diagram 3: Message Flow (End-to-End)

```mermaid
sequenceDiagram
    actor User
    participant Channel
    participant Gateway
    participant AgentLoop as Agent Loop
    participant PromptBuilder
    participant Memory
    participant Provider
    participant ToolDispatcher as Tool Dispatcher
    participant Tool
    participant Observer

    User->>Channel: Send message
    Channel->>AgentLoop: ChannelMessage

    Note over Gateway: Alternatively: HTTP request via REST/SSE/WS
    Gateway-->>AgentLoop: Inbound API request

    Observer->>Observer: Record message_received event

    AgentLoop->>Memory: recall(context_key)
    Memory-->>AgentLoop: Memory context entries

    AgentLoop->>PromptBuilder: build_prompt(system, history, context)
    PromptBuilder-->>AgentLoop: Assembled ChatRequest

    AgentLoop->>Provider: chat(ChatRequest)
    Observer->>Observer: Record provider_request event

    alt Response contains ToolCalls
        Provider-->>AgentLoop: ChatResponse { tool_calls: [...] }
        Observer->>Observer: Record tool_calls_received event

        loop For each ToolCall
            AgentLoop->>ToolDispatcher: dispatch(tool_name, params)
            ToolDispatcher->>Tool: execute(params)
            Tool-->>ToolDispatcher: ToolResult
            ToolDispatcher-->>AgentLoop: ToolResult
            Observer->>Observer: Record tool_executed event
        end

        AgentLoop->>Provider: chat(ChatRequest with tool results)
        Provider-->>AgentLoop: Final ChatResponse
    else Direct response
        Provider-->>AgentLoop: ChatResponse { content: "..." }
    end

    Observer->>Observer: Record response_ready event

    AgentLoop->>Memory: store(conversation_turn)

    AgentLoop->>Channel: send(OutboundMessage)
    Channel->>User: Deliver response

    Observer->>Observer: Record message_delivered event
```

---

## Navigation

- Back to docs hub: [README.md](README.md)
- Unified TOC: [SUMMARY.md](SUMMARY.md)
- Provider reference: [providers-reference.md](providers-reference.md)
- Channel reference: [channels-reference.md](channels-reference.md)
- Hardware/peripherals design: [hardware-peripherals-design.md](hardware-peripherals-design.md)
- Security overview: [security/README.md](security/README.md)
