---
name: zulk-url-shortener
description: "Premium AI-first URL shortening and management with real-time analytics and team collaboration via MCP. Use when shortening links for marketing, tracking AI interactions, or managing custom domains. Keywords: url, shortener, analytics, link management, mcp."
license: MIT
compatibility: Requires an MCP-compatible client and internet access.
metadata:
  repository: https://github.com/Zu-lk/zulk-short-url-skill
  mcp_url: https://mcp.zu.lk/mcp
  mcp_sse_url: https://mcp.zu.lk/sse
  mcp_command: npx mcp-remote https://mcp.zu.lk/mcp
---

# Zulk URL Shortener Skill

This skill enables AI agents to manage short links, organizations, and analytics using the Zu.lk MCP (Model Context Protocol) server.

## Overview

Zu.lk is an AI-first premium URL shortener designed for blazing-fast performance and seamless AI integration. This skill connects your agent to the Zulk MCP server, allowing it to:

- Create shortened URLs (e.g., `zu.lk/abcd`)
- Manage existing links and campaigns
- Access real-time analytics
- Manage organizations and collaborate with team members

## Installation

To use this skill, add the Zulk MCP server configuration to your AI assistant's settings (e.g., `mcp.json` or equivalent).

### Configuration Options

Choose the transport method that best fits your environment:

#### 1. Streamable HTTP (Recommended)

Fastest and most reliable communication.

```json
{
  "mcpServers": {
    "zulk-url-shortener": { "url": "https://mcp.zu.lk/mcp" }
  }
}
```

#### 2. SSE (Server-Sent Events)

Real-time streaming specialized for certain clients.

```json
{
  "mcpServers": {
    "zulk-url-shortener": { "url": "https://mcp.zu.lk/sse" }
  }
}
```

#### 3. Stdio (via mcp-remote or mcporter)

Uses standard input/output via a remote bridge.

```json
{
  "mcpServers": {
    "zulk-url-shortener": {
      "command": "npx",
      "args": ["mcporter", "https://mcp.zu.lk/mcp"]
    }
  }
}
```

## Step-by-Step Instructions

1.  **Preparation**: Ensure you have a Zu.lk account or are ready to sign in via Google.
2.  **Configuration**: Add one of the JSON configurations above to your agent's MCP settings file.
3.  **Authentication**: When you first run a command like "shorten this link", the agent will present an OAuth URL. Follow the link to authenticate.
4.  **Verification**: Ask the agent "List my recently created links" to verify the connection is active.
5.  **Execution**: Use natural language to create links, e.g., "Create a short link for https://google.com with alias 'my-search'".

## When To Use This Skill

Use this skill when the user asks to:

- shorten a URL
- manage short links
- view or update an existing short link
- manage organizations
- invite or manage organization members
- retrieve link analytics
- list links belonging to an organization

## Available MCP Tools

The Zulk MCP server provides the following tools. Note that tool names use an underscore (`_`) instead of a dot:

### Organization Management

- `zulk_get_organizations`: Returns all organizations that the authenticated user has access to.
- `zulk_create_organization`: Creates a new organization with the authenticated user as owner. Parameters: `name` (string).

### Organization Members

- `zulk_get_organization_members`: Returns all members of a specific organization. Parameters: `orgId` (string).
- `zulk_add_organization_member`: Adds a new member to an organization (requires ADMIN or OWNER role). Parameters: `orgId` (string), `email` (string), `role` (optional: "MANAGER" | "ADMIN" | "OWNER").
- `zulk_update_member_role`: Updates the role of a specific member in an organization. Parameters: `orgId` (string), `memberId` (string), `role` ("MANAGER" | "ADMIN" | "OWNER").
- `zulk_remove_organization_member`: Removes a member from an organization (requires ADMIN or OWNER role). Parameters: `orgId` (string), `memberId` (string).

### Link Management

- `zulk_get_organization_links`: Returns all short links for a specific organization. Parameters: `orgId` (string).
- `zulk_create_link`: Creates a new short link for the given URL in the specified organization. Parameters: `orgId` (string), `url` (string, valid URI), `key` (optional string), `length` (optional number 5-10), `expiresAt` (optional ISO 8601 string, Pro plans only), `password` (optional string, Pro plans only), `utmParams` (optional).
- `zulk_get_link`: Returns details of a specific link by ID from the specified organization. Parameters: `orgId` (string), `linkId` (string).
- `zulk_update_link`: Updates an existing short link for the specified organization. Parameters: `orgId` (string), `linkId` (string), `url` (string, valid URI), `key` (string), `expiresAt` (optional ISO 8601 string, Pro plan feature), `password` (optional string, Pro plan feature).

### Analytics

- `zulk_get_organization_analytics`: Returns click analytics data for an organization's links from PostHog. Parameters: `orgId` (string), `dateFrom` (optional string, default: -7d), `dateTo` (optional string, default: today), `interval` (optional string, default: day).

## Common Workflows

### Creating a Short Link

1. Determine which organization the link should belong to.
2. If the organization is not specified, call `zulk_get_organizations` to list available organizations.
3. Call `zulk_create_link` with the specific `orgId` and `url`.
4. Return the generated short link to the user.

### Listing Links in an Organization

1. Identify the organization.
2. Call `zulk_get_organization_links`.
3. Return the list of links in a readable format.

### Inviting a Member

1. Ensure the user has **ADMIN** or **OWNER** permissions.
2. Call `zulk_add_organization_member` with the organization ID and member details.
3. Confirm the member was successfully added.

### Viewing Analytics

1. Identify the organization.
2. Call `zulk_get_organization_analytics`.
3. Summarize click metrics for the user.

## Best Practices

- Always verify the organization before creating or modifying links.
- Prefer listing organizations if the user did not specify one.
- Provide clean responses when returning links or analytics.
- Clearly report permission errors when they occur.

## Usage Examples

### Creating a Link

**Input**: "Shorten https://github.com/Zu-lk/zulk-short-url-skill for my marketing team"
**Agent reasoning**:

1. Identify the organization belonging to the marketing team.
2. Call `zulk_create_link`.
3. Return the generated short link.
   **Output**: "Generated short link: https://zu.lk/z-skill"

### Checking Analytics

**Input**: "Show analytics for our links"
**Agent reasoning**:

1. Determine the organization.
2. Call `zulk_get_organization_analytics`.
3. Summarize the click data.
   **Output**: "Your links received 1,240 clicks yesterday. Here is the breakdown..."

## Edge Cases & Troubleshooting

- **Auth Failure**: If authentication fails, ensure you are using the correct Google account. You may need to restart the agent to re-trigger the OAuth flow.
- **Alias Taken**: If a custom alias is already in use, the agent should suggest an alternative or append a random string.
- **Rate Limits**: If you exceed your plan's link limit, the MCP server will return an error indicating the limit has been reached.
- **Link Expiration**: Ensure you check if the link has an expiration date if it suddenly stops working.

## References

- [Official Website](https://zu.lk)
- [MCP Documentation](https://zu.lk/-/mcp)
