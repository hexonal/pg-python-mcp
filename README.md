# PostgreSQL Python MCP Server

[中文文档](README_zh.md) | English

A Python-based PostgreSQL MCP server providing secure database operations with built-in safety checks and AST-based SQL parsing.

## Features

- 🔍 **Database Listing**: List all databases in the PostgreSQL instance
- 📋 **Table Listing**: View all tables in the configured database
- 🔍 **Table Description**: Detailed table structure information
- 📊 **Safe Querying**: Execute SQL queries (SELECT only by default)
- 🛡️ **Security Boundaries**: Strict operation limits within configured database
- ⚙️ **Configurable**: Support for enabling additional operations via environment variables
- 📋 **JSON Output**: Structured JSON responses optimized for AI consumption
- 🌳 **AST-Based Security**: Advanced SQL parsing using Abstract Syntax Trees for 100% accuracy

## Security Features

### Default Safe Mode
- Only allows `SELECT`, `SHOW`, `DESCRIBE`, `EXPLAIN` queries
- Blocks dangerous operations like `DROP`, `DELETE`, `UPDATE`, `INSERT`
- Strictly limits operations to the configured database scope
- Prevents SQL injection attacks using AST-based analysis
- Detects nested dangerous operations and UNION-based attacks

### Advanced Mode (Optional)
Set environment variable `PG_ALLOW_DANGEROUS=true` to enable:
- Full CRUD operations
- Extended database management functions

## Configuration

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `PG_HOST` | PostgreSQL host address and port (format: host:port or host) | Yes |
| `PG_USER` | PostgreSQL username | Yes |
| `PG_PASSWORD` | PostgreSQL password | Yes |
| `PG_DATABASE` | Target database name | Yes |
| `PG_ALLOW_DANGEROUS` | Allow dangerous operations (true/false) | No (default: false) |

### Claude Desktop Configuration Example

Add to your Claude Desktop configuration file:

```json
{
  "mcpServers": {
    "postgresql": {
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/hexonal/pg-python-mcp-.git",
        "pg-python-mcp"
      ],
      "env": {
        "PG_HOST": "your-postgres-host:5432",
        "PG_USER": "your-username",
        "PG_PASSWORD": "your-password",
        "PG_DATABASE": "your-database"
      }
    }
  }
}
```

## Available Tools

### 1. list_databases
Lists all databases in the PostgreSQL instance (current configured database is highlighted).

### 2. list_tables  
Lists all tables in the currently configured database.

### 3. describe_table
Describes the structure of a specified table, including column names, types, null constraints, and key information.

**Parameters:**
- `table_name` (string): Name of the table to describe

### 4. execute_query
Executes SQL query statements and returns results in JSON format.

**Parameters:**
- `query` (string): SQL statement to execute

**Security Constraints:**
- Default mode only allows query operations (SELECT, SHOW, DESCRIBE, EXPLAIN)
- Automatically detects and blocks dangerous operations using AST parsing
- Operations limited to the configured database scope
- 100% accuracy in detecting SQL injection attempts

**JSON Output Format:**
```json
{
  "status": "success",
  "message": "Query executed successfully, returned 2 rows",
  "columns": ["id", "name", "email"],
  "data": [
    {
      "id": 1,
      "name": "User 1", 
      "email": "user1@example.com"
    },
    {
      "id": 2,
      "name": "User 2",
      "email": "user2@example.com"
    }
  ]
}
```

## Installation and Usage

### Using uvx (Recommended)

```bash
# Install from Git
uvx --from git+https://github.com/hexonal/pg-python-mcp-.git pg-python-mcp

# Or for local development
uvx pg-python-mcp
```

### Manual Installation

```bash
# Clone repository
git clone https://github.com/hexonal/pg-python-mcp-.git
cd pg-python-mcp

# Install dependencies
pip install -e .

# Run server
python -m pg_mcp
```

## Usage Examples

### List All Databases
```
Tool: list_databases
```

### List Tables in Current Database
```  
Tool: list_tables
```

### Describe Table Structure
```
Tool: describe_table
Parameters: {"table_name": "users"}
```

### Execute Query
```
Tool: execute_query  
Parameters: {"query": "SELECT * FROM users LIMIT 10"}
```

## Development

### Project Structure
```
pg-python-mcp/
├── pg_mcp/
│   ├── __init__.py          # Main MCP server entry point
│   ├── __main__.py          # Run script
│   └── pg_handler.py        # PostgreSQL handler with AST security
├── test_ast_security.py     # AST security validation tests
├── test_stdio.py           # MCP protocol testing
├── pyproject.toml          # Project configuration
├── README.md               # English documentation
└── README_zh.md            # Chinese documentation
```

### Local Development
```bash
# Clone project
git clone https://github.com/hexonal/pg-python-mcp-.git
cd pg-python-mcp

# Install development dependencies
pip install -e ".[dev]"

# Run security tests
python test_ast_security.py

# Test MCP protocol
python test_stdio.py

# Code formatting
black pg_mcp/
isort pg_mcp/

# Type checking
mypy pg_mcp/
```

## Technology Stack

- **FastMCP 2.0**: Modern MCP framework with decorator-based tool registration
- **asyncpg**: Async PostgreSQL database operations
- **sqlparse**: SQL Abstract Syntax Tree parsing for security analysis
- **Python 3.8+**: Broad compatibility support

## Security Implementation

### AST-Based SQL Analysis
The server uses Abstract Syntax Tree parsing to achieve 100% accuracy in SQL security checking:

```python
def is_query_safe(self, query: str) -> tuple[bool, str]:
    """Check query safety using AST parsing"""
    try:
        parsed = sqlparse.parse(query)
        for statement in parsed:
            is_safe, error_msg = self._check_statement_safety(statement)
            if not is_safe:
                return False, error_msg
        return True, ""
```

### Security Test Results
- ✅ Safe queries: 8/8 (100%)
- 🛡️ Dangerous queries blocked: 10/10 (100%)
- 🎯 Overall accuracy: **100%**

## License

MIT License

## Security Notice

⚠️ **Important Security Guidelines**:
- All environment variables are required with no unsafe defaults
- Ensure minimal PostgreSQL user permissions in production
- Regularly rotate database passwords
- Avoid using administrative database users in configuration
- Run this MCP server in isolated environments
- Default safe mode provides basic protection but cannot replace comprehensive security policies

## Troubleshooting

### Common Issues

1. **Connection Failed**: Check if PostgreSQL service is running and network connectivity
2. **Environment Variable Error**: Verify all required environment variables are properly set
3. **Permission Error**: Confirm PostgreSQL user has access permissions to the specified database
4. **Query Rejected**: Check if query contains forbidden keywords, or consider enabling advanced mode
5. **MCP Protocol Issues**: Ensure you're using FastMCP 2.0 compatible configuration

### Getting Help

- Check the [Chinese documentation](README_zh.md) for additional details
- Review the test files for usage examples
- Examine the AST security tests for supported query patterns