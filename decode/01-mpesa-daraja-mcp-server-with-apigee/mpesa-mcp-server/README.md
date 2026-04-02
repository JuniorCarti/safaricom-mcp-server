# M-PESA Express MCP Server

A Model Context Protocol (MCP) server that provides tools for interacting with Safaricom's M-PESA Express (STK Push) API and a static product catalog.

## Features

### Catalog Tools
- `list_products` - List all available products
- `get_product(product_id)` - Get details for a specific product
- `calculate_order_total(items)` - Calculate total cost for order items

### Payment Tools
- `generate_access_token_request` - Generate DARAJA API access token
- `validate_stk_push_payload(...)` - Validate STK Push request parameters
- `initiate_stk_push(...)` - Initiate M-PESA STK Push payment
- `parse_stk_callback(payload)` - Parse payment callback response
- `explain_stk_error(code)` - Explain error codes

## Quick Start

### Local Development

1. **Install dependencies:**
   ```bash
   uv sync
   # or
   pip install -r requirements.txt
   ```

2. **Set environment variables:**
   ```bash
   export MPESA_CONSUMER_KEY="your_key"
   export MPESA_CONSUMER_SECRET="your_secret"
   ```

3. **Run the server:**
   ```bash
   uv run server.py
   # or
   python server.py
   ```

### Deployment to Cloud Run

See [DEPLOY_NOTES.md](DEPLOY_NOTES.md) for detailed deployment instructions.

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `MPESA_CONSUMER_KEY` | Yes | DARAJA Consumer Key |
| `MPESA_CONSUMER_SECRET` | Yes | DARAJA Consumer Secret |
| `PORT` | No | Server port (default: 8080) |
| `MPESA_HTTP_TIMEOUT_SECONDS` | No | API timeout (default: 30.0) |

## Security

- Credentials are read from environment variables only
- No hardcoded secrets in source code
- Designed for deployment behind authentication (IAM/IAP)
- Uses HTTPS for all external API calls

## Architecture

```
Gemini CLI / ADK Agent
        |
        v
   Apigee (future)
        |
        v
  Cloud Run MCP Server
        |
        +--> Static Product Catalog
        |
        v
Safaricom DARAJA APIs
```

## Testing

See [LOCAL_RUN.md](LOCAL_RUN.md) for local testing instructions.

## License

This project is part of the DECODE workshop materials.