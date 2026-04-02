# Local Development and Testing

This guide explains how to run and test the M-PESA Express MCP server locally.

## Prerequisites

- Python 3.13+
- uv package manager (install with `pip install uv`)
- Safaricom DARAJA sandbox credentials

## Setup

1. **Clone or navigate to the project directory:**
   ```bash
   cd mpesa-mcp-server
   ```

2. **Install dependencies:**
   ```bash
   uv sync
   ```

3. **Set up environment variables:**
   ```bash
   cp .env.example .env
   # Edit .env with your actual DARAJA credentials
   ```

   Or export them directly:
   ```bash
   export MPESA_CONSUMER_KEY="your_key_here"
   export MPESA_CONSUMER_SECRET="your_secret_here"
   ```

## Running Locally

Start the MCP server:
```bash
uv run server.py
```

The server will start on `http://localhost:8080` (or the PORT specified in environment).

## Testing the Tools

You can test the MCP tools using various clients:

### Using MCP CLI (if installed)
```bash
mcp dev server.py
```

### Using Python Script
Create a test script:

```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def test_tools():
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "server.py"]
    )

    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # Test list_products
            result = await session.call_tool("list_products", {})
            print("Products:", result)

            # Test get_product
            result = await session.call_tool("get_product", {"product_id": "coffee-001"})
            print("Coffee:", result)

asyncio.run(test_tools())
```

### Manual Testing with HTTP

The server exposes an HTTP endpoint. You can test with curl:

```bash
# List products
curl -X POST http://localhost:8080/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc": "2.0", "id": 1, "method": "tools/call", "params": {"name": "list_products", "arguments": {}}}'
```

## Available Tools

- `list_products` - Returns all products in catalog
- `get_product(product_id)` - Get specific product details
- `calculate_order_total(items)` - Calculate total for order items
- `generate_access_token_request` - Get DARAJA access token
- `validate_stk_push_payload(...)` - Validate STK push request
- `initiate_stk_push(...)` - Send actual STK push (requires credentials)
- `parse_stk_callback(payload)` - Parse callback response
- `explain_stk_error(code)` - Explain error codes

## Testing STK Push

⚠️ **Important:** STK Push testing requires valid DARAJA credentials and will make real API calls to Safaricom sandbox.

For testing without real credentials, use the validation tool:
```python
# This validates without making API calls
result = await session.call_tool("validate_stk_push_payload", {
    "phone_number": "254712345678",
    "amount": 100
})
```

## Troubleshooting

- **Port already in use:** Change PORT environment variable
- **Missing credentials:** Ensure MPESA_CONSUMER_KEY and MPESA_CONSUMER_SECRET are set
- **Timeout errors:** Safaricom sandbox can be slow; increase MPESA_HTTP_TIMEOUT_SECONDS
- **Validation errors:** Check phone number format (2547XXXXXXXX) and amount (>=1)