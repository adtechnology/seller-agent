# MCP (Model Context Protocol)

MCP is the **primary agentic interface** for the seller agent. Buyer agents call seller tools directly through structured, typed tool calls over MCP. This is the preferred protocol for automated workflows where the buyer agent knows exactly which operation it needs.

## Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/mcp/sse` | GET | Streamable HTTP transport (SSE) for persistent tool sessions |
| `/mcp/tools` | GET | Tool discovery --- lists all available tools and their schemas |
| `/mcp/call` | POST | Execute a tool call with typed arguments |

## Available Tools

The seller agent exposes the following tools via MCP. Each maps to a core seller operation:

| Tool | Description |
|------|-------------|
| `list_products` | Browse the product catalog with optional filters |
| `get_product` | Get full details for a specific product |
| `get_pricing` | Tiered pricing calculation with buyer context (tier, volume, deal type) |
| `check_availability` | Check inventory avails for a product, date range, and impression volume |
| `create_proposal` | Submit a proposal for seller review |
| `create_order` | Create an order from an accepted proposal |
| `create_line_item` | Add line items to an existing order |
| `book_programmatic_guaranteed` | Book a Programmatic Guaranteed (PG) deal directly |
| `create_pmp_deal` | Create a Private Marketplace (PMP) deal and return a Deal ID |
| `get_deal_status` | Check the status of an existing deal |
| `request_quote` | Request a non-binding price quote (IAB Deals API) |
| `book_deal` | Book a deal from an accepted quote (IAB Deals API) |

## Example Tool Call

```json
{
  "name": "get_pricing",
  "arguments": {
    "product_id": "premium-video",
    "buyer_tier": "agency",
    "volume": 5000000
  }
}
```

The response is a typed JSON object with the pricing breakdown, including base CPM, tier discount, volume discount, and effective CPM.

## When to Use MCP

MCP is the right choice when the buyer agent:

- Knows exactly which operation to perform (e.g., check availability, book a deal)
- Needs deterministic, structured responses
- Is running an automated workflow without human intervention
- Wants the fastest possible execution path

## Protocol Comparison

| | MCP | A2A | REST API |
|---|-----|-----|----------|
| **Interface style** | Structured tool calls | Natural language (JSON-RPC) | HTTP request/response |
| **Best for** | Automated agent workflows | Discovery, negotiation, complex queries | Human operators, dashboards |
| **Response format** | Typed tool results | Mixed text + structured data | JSON |
| **Multi-turn** | Stateless per call | Yes, via `contextId` | Stateless |
| **Speed** | Fastest | Moderate (LLM processing) | Fast |
| **Determinism** | Fully deterministic | Non-deterministic (LLM) | Fully deterministic |
| **Transport** | SSE (Streamable HTTP) | HTTP POST (JSON-RPC 2.0) | HTTP verbs |

## MCP Server Examples

The repository includes two MCP server examples:

- **`examples/publisher_gam_server.py`** --- Publisher seller agent with Google Ad Manager integration. Exposes CTV inventory via MCP, books PG lines in GAM, and creates PMP deals.
- **`examples/dsp_server.py`** --- DSP seller agent. Receives Deal IDs from buyers, activates deals on campaigns, and books performance and mobile app campaigns.

## See Also

- [A2A Protocol](a2a.md) --- conversational agent-to-agent interface
- [Agent Discovery](agent-discovery.md) --- how buyer agents discover this seller
- [API Overview](overview.md) --- full REST API reference
- [Buyer Agent MCP Client](https://iabtechlab.github.io/buyer-agent/) --- buyer-side MCP client documentation
