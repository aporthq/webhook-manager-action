# APort Webhook Manager Example

This repository demonstrates how to use the APort Webhook Manager GitHub Action to manage webhook configurations for APort agents.

## 🚀 Quick Start

### 1. Fork this Repository

Click the "Fork" button to create your own copy of this repository.

### 2. Set up APort Agent

1. Go to [APort Dashboard](https://aport.io)
2. Create a new agent
3. Note the Agent ID

### 3. Configure GitHub Secrets

Add the following secrets to your repository:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add the following secrets:

| Secret Name | Description | Example Value |
|-------------|-------------|---------------|
| `APORT_AGENT_ID` | Your APort Agent ID | `agent_1234567890abcdef` |

### 4. Prepare Webhook Endpoint

Create a webhook endpoint to receive APort notifications:

```javascript
// Example webhook endpoint (Node.js/Express)
const express = require('express');
const app = express();

app.use(express.json());

app.post('/webhook/aport', (req, res) => {
  const { agent_id, event_type, data } = req.body;
  
  console.log(`Received webhook for agent ${agent_id}:`, event_type);
  
  // Process webhook data
  switch (event_type) {
    case 'policy_violation':
      console.log('Policy violation detected:', data);
      break;
    case 'assurance_update':
      console.log('Assurance updated:', data);
      break;
    default:
      console.log('Unknown event type:', event_type);
  }
  
  res.status(200).json({ received: true });
});

app.listen(3000, () => {
  console.log('Webhook server running on port 3000');
});
```

### 5. Test the Action

1. Push changes to different branches (main, staging, dev)
2. The action will automatically set appropriate webhook URLs
3. Or trigger manually via **Actions** → **APort Webhook Management** → **Run workflow**

## 🔧 Configuration

### Webhook Operations

| Operation | Description |
|-----------|-------------|
| `set` | Set a new webhook URL for the agent |
| `get` | Retrieve the current webhook URL |
| `delete` | Remove the webhook configuration |

### Environment-specific Webhooks

The workflow is configured to set different webhook URLs based on the branch:

- **main** → `https://prod.example.com/webhook/aport`
- **staging** → `https://staging.example.com/webhook/aport`
- **dev** → `https://dev.example.com/webhook/aport`

## 📊 Usage Examples

### Basic Webhook Management

```yaml
name: Basic Webhook Management
on:
  workflow_dispatch:
    inputs:
      webhook_url:
        description: "Webhook URL to set"
        required: true

jobs:
  webhook-management:
    runs-on: ubuntu-latest
    steps:
      - name: Set Webhook
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          webhook-url: ${{ github.event.inputs.webhook_url }}
          operation: 'set'
```

### Get Current Webhook

```yaml
name: Get Current Webhook
on:
  workflow_dispatch:

jobs:
  get-webhook:
    runs-on: ubuntu-latest
    steps:
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

```yaml
name: Delete Webhook
on:
  workflow_dispatch:

jobs:
  delete-webhook:
    runs-on: ubuntu-latest
    steps:
      - name: Delete Webhook
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          operation: 'delete'
```

## 🚨 Troubleshooting

### Common Issues

1. **"Agent not found" error**
   - Verify `APORT_AGENT_ID` secret is correct
   - Check agent exists in APort dashboard

2. **"Invalid webhook URL" error**
   - Ensure URL is accessible
   - Check URL format and protocol

3. **"Permission denied" error**
   - Verify GitHub token has necessary permissions
   - Check agent ownership

### Debug Mode

To enable debug logging, modify the workflow:

```yaml
- name: APort Webhook Manager
  uses: aporthq/webhook-manager-action@v1
  with:
    agent-id: ${{ secrets.APORT_AGENT_ID }}
    webhook-url: 'https://example.com/webhook'
    operation: 'set'
    debug: true  # Add this line
```

## 📚 Advanced Examples

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

### Conditional Webhook Updates

```yaml
name: Conditional Webhook Updates
on:
  push:
    branches: [main, staging, dev]

jobs:
  webhook-update:
    runs-on: ubuntu-latest
    steps:
      - name: Get Current Webhook
        id: current-webhook
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          operation: 'get'

      - name: Set New Webhook
        if: steps.current-webhook.outputs.webhook_url != 'https://new-webhook.example.com'
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          webhook-url: 'https://new-webhook.example.com'
          operation: 'set'
```

### Webhook Rotation

```yaml
name: Webhook Rotation
on:
  schedule:
    - cron: '0 0 1 * *'  # First day of every month

jobs:
  rotate-webhook:
    runs-on: ubuntu-latest
    steps:
      - name: Generate New Webhook URL
        id: new-webhook
        run: |
          # Generate a new webhook URL with timestamp
          TIMESTAMP=$(date +%s)
          NEW_URL="https://webhook.example.com/aport/$TIMESTAMP"
          echo "new_webhook_url=$NEW_URL" >> $GITHUB_OUTPUT

      - name: Set New Webhook
        uses: aporthq/webhook-manager-action@v1
        with:
          agent-id: ${{ secrets.APORT_AGENT_ID }}
          webhook-url: ${{ steps.new-webhook.outputs.new_webhook_url }}
          operation: 'set'
```

## 🤝 Contributing

1. Fork this repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📄 License

MIT License - see LICENSE file for details.

## 🆘 Support

- 📖 [APort Documentation](https://aport.io/docs)
- 💬 [Discord Community](https://discord.gg/aport)
- 🐛 [Issue Tracker](https://github.com/aporthq/webhook-manager-action/issues)
- 📧 [Email Support](mailto:support@aport.io)

## 🔗 Related

- [APort Dashboard](https://aport.io)
- [Webhook Manager Action](https://github.com/aporthq/webhook-manager-action)
- [APort Documentation](https://aport.io/docs)
