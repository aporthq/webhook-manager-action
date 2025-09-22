# APort Webhook Manager Action

Manage webhook configurations for APort agents to enable real-time notifications and integrations.

## 🚀 Features

- **Webhook Management** - Set, get, and delete webhook configurations
- **GitHub Integration** - Seamless integration with GitHub Actions
- **Flexible Operations** - Support for multiple webhook operations
- **Security** - Secure webhook URL management
- **Real-time Updates** - Keep webhook configurations in sync

## 📋 Usage

### Basic Usage

```yaml
name: APort Webhook Management
on:
  workflow_dispatch:
    inputs:
      webhook_url:
        description: 'Webhook URL to set'
        required: true

jobs:
  webhook-management:
    runs-on: ubuntu-latest
    steps:
      - name: Set Webhook URL
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          webhook-url: ${{ github.event.inputs.webhook_url }}
          operation: 'set'
```

### Advanced Usage

```yaml
name: Advanced Webhook Management
on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  webhook-management:
    runs-on: ubuntu-latest
    steps:
      - name: Get Current Webhook
        id: get-webhook
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          operation: 'get'

      - name: Set New Webhook
        if: steps.get-webhook.outputs.webhook_url != 'https://new-webhook.example.com'
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          webhook-url: 'https://new-webhook.example.com'
          operation: 'set'

      - name: Delete Webhook
        if: github.event_name == 'workflow_dispatch'
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          operation: 'delete'
```

## 🔧 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `agent-id` | The APort Agent ID to manage webhooks for | ✅ | - |
| `webhook-url` | The webhook URL to set for the agent | ✅* | - |
| `api-base` | The base URL for the APort API | ❌ | `https://api.aport.io` |
| `operation` | Operation to perform (set, get, delete) | ❌ | `set` |
| `github-token` | GitHub token for webhook operations | ❌ | `${{ github.token }}` |

*Required for `set` operation only

## 📤 Outputs

| Output | Description |
|--------|-------------|
| `webhook_url` | The current webhook URL for the agent |
| `success` | Boolean indicating if the operation was successful |

## 🏗️ Setup

### 1. Create APort Agent

1. Go to [APort Dashboard](https://aport.io)
2. Create a new agent
3. Note the Agent ID

### 2. Set GitHub Secrets

Add the following secrets to your repository:

```bash
APORT_AGENT_ID=your-agent-id-here
```

### 3. Prepare Webhook Endpoint

Ensure your webhook endpoint is ready to receive APort notifications:

```javascript
// Example webhook endpoint
app.post('/webhook/aport', (req, res) => {
  const { agent_id, event_type, data } = req.body;
  
  console.log(`Received webhook for agent ${agent_id}:`, event_type);
  
  // Process webhook data
  switch (event_type) {
    case 'policy_violation':
      // Handle policy violation
      break;
    case 'assurance_update':
      // Handle assurance update
      break;
    default:
      console.log('Unknown event type:', event_type);
  }
  
  res.status(200).json({ received: true });
});
```

## 🔍 Operations

### Set Webhook

Set a new webhook URL for the agent:

```yaml
- name: Set Webhook
  uses: aporthq/webhook-manager-action@v1
  with:
    agent-id: ${{ secrets.APORT_AGENT_ID }}
    webhook-url: 'https://your-app.com/webhook/aport'
    operation: 'set'
```

### Get Webhook

Retrieve the current webhook URL:

```yaml
- name: Get Webhook
  id: webhook
  uses: aporthq/webhook-manager-action@v1
  with:
    agent-id: ${{ secrets.APORT_AGENT_ID }}
    operation: 'get'

- name: Use Webhook URL
  run: echo "Current webhook: ${{ steps.webhook.outputs.webhook_url }}"
```

### Delete Webhook

Remove the webhook configuration:

```yaml
- name: Delete Webhook
  uses: aporthq/webhook-manager-action@v1
  with:
    agent-id: ${{ secrets.APORT_AGENT_ID }}
    operation: 'delete'
```

## 📚 Examples

### Environment-specific Webhooks

```yaml
name: Environment Webhooks
on:
  push:
    branches: [main, staging, dev]

jobs:
  set-webhook:
    runs-on: ubuntu-latest
    steps:
      - name: Set Production Webhook
        if: github.ref == 'refs/heads/main'
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          webhook-url: 'https://prod.example.com/webhook/aport'
          operation: 'set'

      - name: Set Staging Webhook
        if: github.ref == 'refs/heads/staging'
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          webhook-url: 'https://staging.example.com/webhook/aport'
          operation: 'set'

      - name: Set Development Webhook
        if: github.ref == 'refs/heads/dev'
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          webhook-url: 'https://dev.example.com/webhook/aport'
          operation: 'set'
```

### Webhook Health Check

```yaml
name: Webhook Health Check
on:
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours

jobs:
  webhook-health:
    runs-on: ubuntu-latest
    steps:
      - name: Check Webhook Status
        id: webhook-check
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          operation: 'get'

      - name: Alert if No Webhook
        if: steps.webhook-check.outputs.webhook_url == ''
        run: |
          echo "⚠️ No webhook configured for agent ${{ secrets.APORT_AGENT_ID }}"
          # Add your alerting logic here
```

## 🚨 Troubleshooting

### Common Issues

1. **Agent not found**
   - Verify `APORT_AGENT_ID` is correct
   - Check agent exists in APort dashboard

2. **Invalid webhook URL**
   - Ensure URL is accessible
   - Check URL format and protocol

3. **Permission denied**
   - Verify GitHub token has necessary permissions
   - Check agent ownership

### Debug Mode

Enable debug logging:

```yaml
- name: APort Webhook Manager
  uses: aporthq/webhook-manager-action@v1
  with:
    agent-id: ${{ secrets.APORT_AGENT_ID }}
    webhook-url: 'https://example.com/webhook'
    operation: 'set'
    debug: true
```

## 📚 Examples

See the `example-repo/` directory for complete working examples.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📄 License

MIT License - see LICENSE file for details.

## 🆘 Support

- 📖 [Documentation](https://aport.io/docs)
- 💬 [Discord Community](https://discord.gg/aport)
- 🐛 [Issue Tracker](https://github.com/aporthq/webhook-manager-action/issues)
- 📧 [Email Support](mailto:support@aport.io)
