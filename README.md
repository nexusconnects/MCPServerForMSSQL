# SQL Server MCP for Claude

This repository contains the SQL Server MCP (Model Context Protocol) integration for Claude. This integration allows Claude to directly query and analyze data from your SQL Server database using natural language.

## Overview

The SQL MCP integration works by:

1. Setting up a local server that Claude can communicate with via the Model Context Protocol
2. Connecting to your SQL Server database using pymssql directly (no ODBC required)
3. Translating Claude's natural language requests into SQL queries
4. Returning the query results back to Claude for interpretation and presentation

## Key Features

- **No ODBC Required**: Uses pymssql to connect directly to SQL Server
- **Simplified Architecture**: Minimal dependencies and straightforward design
- **Protocol Support**: Works with Claude's MCP system
- **Cross-Platform**: Works on macOS and Linux consistently
- **Easy Installation**: One-step installation process

## Installation

### Prerequisites

- macOS or Ubuntu Linux
- Python 3.7+
- FreeTDS (installed automatically on macOS via Homebrew)
- Claude CLI (install from [claude.ai/docs/installation](https://claude.ai/docs/installation))
- SQL Server credentials with appropriate permissions

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

1. Start Claude with MCP support:

```bash
claude mcp
```

2. Ask Claude questions about your database:

```
@sql execute_sql
SELECT TOP 5 * FROM YourTable
```

```
@sql execute_sql
SELECT COUNT(*) FROM Users WHERE IsActive = 1
```

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

## Files and Components

Created during installation:
- `simple_sql.py` - Python script that handles SQL queries directly
- `run_simple_sql.sh` - Wrapper script that invokes the Python script
- `.mcp.json` - Configuration file for the MCP server
- `CLAUDE.md` - Instructions for Claude on how to use the SQL integration

## Team Distribution

To distribute this SQL MCP integration to your team:

1. Share this repository with your team members

2. Team members should:
   - Make the scripts executable: `chmod +x *.sh`
   - Run the installation script: `./install_sql_mcp.sh`
   - Follow the prompts to enter their SQL Server details
   - Start using Claude with: `claude mcp`

## Credits

This SQL MCP integration is based on the following open-source projects:

- [FreeTDS](https://www.freetds.org/) - TDS protocol library for communicating with SQL Server
- [pymssql](https://github.com/pymssql/pymssql) - Python DB-API interface for SQL Server
- [Model Context Protocol](https://github.com/anthropics/anthropic-cookbook/tree/main/model-context-protocol) - Anthropic's protocol for tool use with Claude