# Deployment Notes

## Required Environment Variables

The MCP server requires the following environment variables to be set:

| Variable | Description | Required | Example |
|----------|-------------|----------|---------|
| `MPESA_CONSUMER_KEY` | DARAJA Consumer Key from developer portal | Yes | `7WS02XptTqkWBUl1mPWn4Vj0tMxjyWF1MwAneRRGxwl2d2lq` |
| `MPESA_CONSUMER_SECRET` | DARAJA Consumer Secret from developer portal | Yes | `2oNVkVPDebg0NiBteUUbjRlLEtnbHHkGKDyqLDbuAxHJ8Ax5M9K2NWrwzBH5zwDH` |
| `PORT` | Port for the server (set automatically by Cloud Run) | No | `8080` |
| `MPESA_HTTP_TIMEOUT_SECONDS` | Timeout for Safaricom API calls | No | `30.0` |

## Getting DARAJA Credentials

1. Go to [Safaricom Developer Portal](https://developer.safaricom.co.ke/dashboard/myapps)
2. Log in or create an account
3. Create a sandbox app with M-PESA Express product
4. Copy Consumer Key and Consumer Secret

For this workshop, you can use the shared `decodegemini` app credentials (see .env.example).

## Cloud Run Deployment Command

```bash
# Set your project
export GOOGLE_CLOUD_PROJECT="your-project-id"
gcloud config set project $GOOGLE_CLOUD_PROJECT

# Set credentials
export MPESA_CONSUMER_KEY="your_key"
export MPESA_CONSUMER_SECRET="your_secret"

# Create service account
gcloud iam service-accounts create mcp-server-sa --display-name="MCP Server Service Account"

# Deploy
gcloud run deploy safaricom-mpesa-mcp-server \
    --service-account=mcp-server-sa@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
    --no-allow-unauthenticated \
    --region=europe-west1 \
    --source=. \
    --set-env-vars="MPESA_CONSUMER_KEY=${MPESA_CONSUMER_KEY},MPESA_CONSUMER_SECRET=${MPESA_CONSUMER_SECRET}" \
    --labels=dev-tutorial=codelab-mcp
```

## IAM Authentication Requirements

The deployment uses `--no-allow-unauthenticated` which means:
- Only authenticated requests are allowed
- Use Google Cloud identity tokens for authentication
- Service account `mcp-server-sa` is used for the service

## Connecting to Gemini CLI

After deployment, connect using:

```bash
# Get project number and identity token
export PROJECT_NUMBER=$(gcloud projects describe $GOOGLE_CLOUD_PROJECT --format="value(projectNumber)")
export ID_TOKEN=$(gcloud auth print-identity-token)

# Create Gemini settings
mkdir -p ~/.gemini
cat > ~/.gemini/settings.json << EOF
{
  "mcpServers": {
    "safaricom-mpesa-remote": {
      "httpUrl": "https://safaricom-mpesa-mcp-server-${PROJECT_NUMBER}.europe-west1.run.app/mcp",
      "headers": {
        "Authorization": "Bearer ${ID_TOKEN}"
      }
    }
  }
}
EOF

# Start Gemini CLI
gemini
```

## Security Considerations

- **Never commit credentials** to version control
- **Use Secret Manager** for production deployments instead of environment variables
- **Rotate credentials** regularly
- **Monitor logs** for unauthorized access attempts
- **Use VPC** for additional network security

## Production Architecture

For production, consider:
- Apigee API Management as a front door
- Secret Manager for credential storage
- VPC Service Controls
- Cloud Armor for additional security
- Load balancing for high availability

## Cost Estimation

- **Cloud Run:** ~$0.10/hour for active instances
- **Artifact Registry:** Minimal storage costs
- **Cloud Build:** Free tier covers most builds
- **Total:** <$1 for workshop completion

## Troubleshooting Deployment

- **Build failures:** Check Dockerfile and dependencies
- **Authentication errors:** Verify identity token is fresh (<1 hour)
- **Timeout errors:** Increase MPESA_HTTP_TIMEOUT_SECONDS
- **Permission denied:** Ensure service account has proper IAM roles