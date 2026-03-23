# ChatGPT / OpenAI Setup Guide

Connect your seller agent to ChatGPT or OpenAI Codex using the same MCP endpoint used by Claude Desktop.

## Prerequisites

Same as [Claude Desktop Setup](claude-desktop-setup.md) — your developer must have deployed the seller agent and generated credentials.

## Configuration

The seller agent exposes an MCP SSE endpoint at `/mcp/sse`. ChatGPT can connect to this via its MCP server configuration.

### MCP Server URL

```
https://your-publisher.example.com/mcp/sse
```

### Authentication

Include the operator API key in the request headers:

```
Authorization: Bearer sk-operator-XXXXX
```

## Available Tools

Once connected, ChatGPT can call the same ~62 tools available in Claude Desktop:

- **Inventory**: `list_products`, `sync_inventory`, `list_inventory`
- **Media Kit**: `list_packages`, `create_package`
- **Pricing**: `get_rate_card`, `update_rate_card`, `get_pricing`
- **Deals**: `create_deal_from_template`, `push_deal_to_buyers`, `distribute_deal_via_ssp`
- **Approvals**: `list_pending_approvals`, `approve_or_reject`
- **Buyers**: `list_buyer_agents`, `register_buyer_agent`, `set_agent_trust`
- **Curators**: `list_curators`, `create_curated_deal`
- **Setup**: `get_setup_status`, `health_check`, `get_config`

## REST API Alternative

If your ChatGPT setup doesn't support MCP, you can use the REST API directly:

```
Base URL: https://your-publisher.example.com/api/v1
Auth: Authorization: Bearer sk-operator-XXXXX
```

See the [API Overview](../api/overview.md) for all 57+ REST endpoints.
