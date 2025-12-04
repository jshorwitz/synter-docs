# Quick Start Guide

Get started with Synter in 5 minutes.

## Prerequisites

- Node.js 18+ or Python 3.8+
- A Synter API key ([get one here](https://syntermedia.ai/dashboard))

## Installation

### TypeScript/Node.js

```bash
npm install @synter/sdk
```

### Python

```bash
pip install synter
```

## Get Your API Key

1. Sign up at [syntermedia.ai](https://syntermedia.ai)
2. Go to **Settings** → **API Keys**
3. Create a new API key
4. Copy the key (starts with `sk_live_` or `sk_test_`)

## Your First Campaign

### TypeScript

```typescript
import { Synter } from '@synter/sdk';

const synter = new Synter(process.env.SYNTER_API_KEY);

// Create a campaign with auto-generated UTMs
const campaign = await synter.campaigns.create({
  name: 'Q4 Product Launch',
  platform: 'google',
  budget: { daily: 500, currency: 'USD' },
  targeting: {
    keywords: ['saas analytics', 'data platform'],
    locations: ['US', 'CA'],
  }
});

console.log(`Campaign created: ${campaign.id}`);
```

### Python

```python
import asyncio
from synter import Synter

async def main():
    async with Synter(api_key="sk_live_...") as synter:
        campaign = await synter.campaigns.create({
            "name": "Q4 Product Launch",
            "platform": "google",
            "budget": {"daily": 500, "currency": "USD"},
            "targeting": {
                "keywords": ["saas analytics", "data platform"],
                "locations": ["US", "CA"],
            }
        })
        print(f"Campaign created: {campaign['id']}")

asyncio.run(main())
```

## Track Conversions

Synter auto-detects the ad platform from click IDs (gclid, rdt_cid, twclid, etc.):

### TypeScript

```typescript
// Auto-detects platform from URL
await synter.conversions.create({
  event: 'purchase',
  value: 299.99,
  sendTo: ['google', 'reddit', 'posthog']
});
```

### Python

```python
await synter.conversions.create({
    "event": "purchase",
    "value": 299.99,
    "send_to": ["google", "reddit", "posthog"]
})
```

## Get ROAS Metrics

```typescript
const roas = await synter.analytics.roas({ window: '30d' });
console.log(roas); // { google: 3.2, reddit: 2.8, linkedin: 4.1 }
```

## Connect Ad Platforms

Use OAuth to connect your ad accounts:

```typescript
// Get OAuth URL
const authUrl = await synter.platforms.getAuthUrl('google', 'https://yourapp.com/callback');

// After user authorizes, complete the flow
await synter.platforms.completeAuth('google', authorizationCode);

// List connected platforms
const platforms = await synter.platforms.list();
```

## Run AI Agents

Use transparent AI agents to optimize your campaigns:

```typescript
// Run budget optimizer in dry-run mode
const optimization = await synter.agents.run('budget-optimizer', {
  dryRun: true,
  params: {
    maxBudgetChange: 0.15,  // ±15%
    minConversions: 10
  }
});

// Review proposed changes
console.log(optimization.result.proposedChanges);

// Apply if approved
await synter.agents.apply(optimization.runId);
```

## Sandbox Mode

Test without hitting real ad platforms:

```typescript
const synter = new Synter({
  apiKey: 'sk_test_...',  // Test keys auto-enable sandbox
  environment: 'sandbox'
});

// All operations use mock data
const campaign = await synter.campaigns.create({ /* ... */ });
```

## Next Steps

- [TypeScript SDK Reference](../sdk/typescript/README.md)
- [Python SDK Reference](../sdk/python/README.md)
- [UTM Management Guide](./guides/utm-management.md)
- [Conversion Tracking Guide](./guides/conversion-tracking.md)
- [AI Agents Guide](./guides/ai-agents.md)
