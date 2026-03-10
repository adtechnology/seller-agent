# Ad Seller Agent

The Ad Seller Agent is an **IAB OpenDirect 2.1 compliant** programmatic advertising seller system. It enables automated ad selling through a RESTful API, supporting the full lifecycle from product discovery through deal execution and post-deal modifications.

Part of the IAB Tech Lab Agent Ecosystem --- see also the [Buyer Agent](https://iabtechlab.github.io/buyer-agent/).

## Access Methods

The seller agent supports three protocols for different use cases:

| Protocol | Endpoint | Best For |
|----------|----------|----------|
| **[MCP](api/mcp.md)** | `/mcp/sse` | Automated agent workflows --- structured tool calls, deterministic |
| **[A2A](api/a2a.md)** | `/a2a/seller/jsonrpc` | Conversational agent interactions --- natural language, multi-turn |
| **[REST API](api/overview.md)** | Various endpoints | Human operators, dashboards, non-agent clients |

For agent-to-agent communication, MCP is the primary protocol. A2A adds conversational capabilities for discovery and negotiation.

## Key Capabilities

- **58 endpoints** across **19 categories** covering the complete ad selling workflow
- Tiered pricing engine with buyer-context-aware discounts
- Multi-round automated negotiation with configurable strategies
- Formal order state machine with 12 states and 20 transitions
- Post-deal change request management with severity-based routing
- Event bus for full observability of system activity
- Agent-to-agent (A2A) discovery and trust management
- Human-in-the-loop approval gates for high-value decisions

## Documentation Sections

### Getting Started

- [Quickstart](getting-started/quickstart.md) --- install, run, and make your first API call

### API Reference

- [API Overview](api/overview.md) --- all 58 endpoints grouped by tag
- [MCP Protocol](api/mcp.md) --- primary agentic interface (structured tool calls)
- [A2A Protocol](api/a2a.md) --- conversational agent-to-agent interface
- [Agent Discovery](api/agent-discovery.md) --- `/.well-known/agent.json` and trust registry
- [Authentication](api/authentication.md) --- API keys, access tiers, and agent trust
- [Quotes](api/quotes.md) --- non-binding price quotes
- [Orders](api/orders.md) --- order creation and state machine transitions
- [Change Requests](api/change-requests.md) --- post-deal modifications

### Architecture

- [System Overview](architecture/overview.md) --- components and how they connect
- [Data Flow](architecture/data-flow.md) --- sequence diagrams for key workflows
- [Storage](architecture/storage.md) --- backend interface and key conventions

### State Machines

- [Order Lifecycle](state-machines/order-lifecycle.md) --- 12 states, 20 transitions
- [Change Request Flow](state-machines/change-request-flow.md) --- validation and approval pipeline

### Event Bus

- [Event Bus Overview](event-bus/overview.md) --- all event types and usage

### Integration

- [Buyer Agent Integration](integration/buyer-agent.md) --- discovery, auth, and transaction flows
- [Negotiation Protocol](integration/negotiation.md) --- multi-round negotiation mechanics
