# SQL Server MCP for Claude

This repository contains the SQL Server MCP (Model Context Protocol) integration for Claude. This integration allows Claude to directly query and analyze data from your SQL Server database using natural language.

**This MCP server is certified with MCP Review**: https://mcpreview.com/mcp-servers/shakunvohradeltek/mcpserverformssql

## Overview

The SQL MCP integration works by:

1. Setting up a local server that Claude can communicate with via the Model Context Protocol
2. Connecting to your SQL Server database using pymssql directly (no ODBC required)
3. Translating Claude's natural language requests into SQL queries
4. Returning the query results back to Claude for interpretation and presentation

**Important Protocol Note**: This integration uses Claude's required MCP protocol version "2025-03-26" for compatibility. The integration includes a fixed MCP implementation that handles the protocol correctly, including proper JSON-RPC format and Content-Length headers.

## Key Features

- **No ODBC Required**: Uses pymssql to connect directly to SQL Server
- **Simplified Architecture**: Minimal dependencies and straightforward design
- **Protocol Support**: Works with Claude's MCP system
- **Cross-Platform**: Works on macOS and Linux consistently
- **Easy Installation**: One-step installation process
- **Smart Configuration**: Preserves existing MCP settings and Claude instructions when adding SQL support

## Installation

### Prerequisites

- macOS or Ubuntu Linux
- Python 3.7+
- FreeTDS (installed automatically on macOS via Homebrew)
- Claude CLI (install from [claude.ai/docs/installation](https://claude.ai/docs/installation))
- SQL Server credentials with appropriate permissions (preferably read-only, see [Security Caution](#-security-caution))

### Installation Steps

1. Run the installation script:

```bash
./install_sql_mcp.sh
```

2. Follow the prompts to enter your SQL Server connection details:
   - SQL Server hostname
   - SQL Server port (default: 1433)
   - Database name
   - Username
   - Password
   - Trust server certificate (default: true)

3. The installer will:
   - Install required dependencies
   - Configure FreeTDS for SQL Server connectivity
   - Set up a Python virtual environment
   - Install required Python packages
   - Create the necessary scripts
   - Configure Claude to recognize the SQL MCP server

4. Verify the installation:

```bash
./run_simple_sql.sh "SELECT 1 AS TestQuery"
```

## Usage

1. Start Claude:

```bash
claude
```

You can also:
- Check MCP status: `claude /mcp`
- Debug MCP: `claude --mcp-debug`

2. Ask Claude questions about your database:

```
@sql execute_sql
SELECT TOP 5 * FROM YourTable
```

```
@sql execute_sql
SELECT COUNT(*) FROM Users WHERE IsActive = 1
```

## ⚠️ Security Caution

**Important:** For security reasons, use a database account with **restricted permissions**:

- Create a dedicated read-only user for Claude's database access
- Avoid using accounts with DROP, DELETE, or table creation privileges
- Never use sa or admin accounts in production environments
- Consider restricting access to only specific tables/views needed for analysis

Using overly privileged accounts could result in accidental data loss or database structure changes when Claude generates SQL queries based on natural language requests.

## Best Practices for Prompting Claude

When working with Claude to query your SQL Server database, follow these best practices:

1. **Ask Claude to explore the database schema first**: Before constructing queries, prompt Claude to examine the database structure to understand tables, columns, and relationships.
   ```
   @sql execute_sql
   SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_SCHEMA = 'dbo' ORDER BY TABLE_NAME, ORDINAL_POSITION
   ```

2. **Provide context about your data model**: Explain what kind of data your database contains and how the tables relate to each other.

3. **Start with schema exploration**: Ask Claude to first explore table structures before writing complex queries.

4. **Verify query results**: Have Claude explain what the query results mean and verify they match your expectations.

5. **Build complexity gradually**: Start with simple queries and gradually add complexity as Claude demonstrates understanding of your schema.

## Troubleshooting

If you encounter issues with the SQL MCP integration:

1. Test direct connectivity:

```bash
./run_simple_sql.sh "SELECT 1 AS TestQuery"
```

2. Ensure FreeTDS is configured correctly:

```bash
tsql -C
```

3. Verify Claude can find the MCP server:

```bash
claude mcp list
```

4. If you see "Connection failed" or "Connection closed" messages but queries still work, this is normal behavior and can be ignored.

5. Protocol version compatibility: This installation uses protocol version "2025-03-26" which is required for Claude's MCP. If you encounter MCP transport closed errors, check that the protocol version hasn't changed.

6. JSON-RPC format: Claude's MCP requires proper JSON-RPC format with non-null IDs and Content-Length headers. The installation script configures this correctly.

## MCP Tools Exposed

The MCP server exposes the following tools to Claude:

### `execute_sql`
- **Description**: Execute SQL queries on the connected database
- **Input Schema**: 
  - `query` (string, required): The SQL query to execute
- **Usage**: Use `@sql execute_sql` in Claude followed by your SQL query
- **Returns**: JSON object with query results, row count, and success status

## Files and Components

Created during installation:
- `simple_sql.py` - Python script that handles SQL queries directly
- `run_simple_sql.sh` - Wrapper script that invokes the Python script
- `mcp_fixed.py` - MCP protocol implementation with proper tool exposure
- `run_mcp_fixed.sh` - Wrapper script for the MCP server
- `.mcp.json` - Configuration file for the MCP server
- `CLAUDE.md` - Instructions for Claude on how to use the SQL integration

## Team Distribution

To distribute this SQL MCP integration to your team:

1. Share this repository with your team members

2. Team members should:
   - Make the scripts executable: `chmod +x *.sh`
   - Run the installation script: `./install_sql_mcp.sh`
   - Follow the prompts to enter their SQL Server details
   
### Latest Updates (May 2024)

The installation script has been updated with these important fixes:

1. Fixed MCP protocol version to "2025-03-26" (previously "2024-11-05") to match Claude's requirements
2. Added improved MCP implementation that properly handles JSON-RPC format
3. Properly handles Content-Length headers in MCP communication
4. Fixed notification handling in the protocol
5. Added continuous loop for persistent MCP connections
6. Improved error handling and debugging logs
7. **NEW: Exposed SQL tools through MCP protocol** - The MCP server now properly exposes the `execute_sql` tool through the `tools/list` and `tools/call` methods, making it compatible with Claude's tool discovery and execution system

## Credits

This SQL MCP integration is based on the following open-source projects:

- [FreeTDS](https://www.freetds.org/) - TDS protocol library for communicating with SQL Server
- [pymssql](https://github.com/pymssql/pymssql) - Python DB-API interface for SQL Server
- [Model Context Protocol](https://github.com/anthropics/anthropic-cookbook/tree/main/model-context-protocol) - Anthropic's protocol for tool use with Claude