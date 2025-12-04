# @synter/sdk

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. One SDK for Google Ads, Reddit Ads, X/Twitter Ads, and beyond.

## Features

✅ **Infrastructure-first**: UTM management, conversion tracking, analytics integration  
✅ **Multi-platform**: Google, Reddit, LinkedIn, Microsoft, Meta, X  
✅ **Transparent AI agents**: Run, inspect, and control optimization agents programmatically  
✅ **Bring your own AI**: Export raw data for custom ML models  
✅ **Analytics integrations**: PostHog, Heap, Mixpanel, Segment  
✅ **TypeScript-native**: Full type safety and IntelliSense  

## Installation

```bash
npm install @synter/sdk
```

## Quick Start

```typescript
import { Synter } from '@synter/sdk';

const synter = new Synter(process.env.SYNTER_API_KEY);

// Create a campaign (UTMs auto-generated)
const campaign = await synter.campaigns.create({
  name: 'Q4 Product Launch',
  platform: 'google',
  budget: { daily: 500, currency: 'USD' },
  targeting: {
    keywords: ['saas analytics', 'data platform'],
    locations: ['US', 'CA'],
  }
});

// Track conversions (platform auto-detected)
await synter.conversions.create({
  event: 'purchase',
  value: 299.99
});

// Get ROAS
const roas = await synter.analytics.roas({ window: '30d' });
console.log(roas); // { google: 3.2, reddit: 2.8 }
```

## API Reference

### `synter.campaigns`

```typescript
await synter.campaigns.create(params)   // Create campaign with auto-generated UTMs
await synter.campaigns.list(params)     // List campaigns
await synter.campaigns.get(id)          // Get campaign details
await synter.campaigns.update(id, params) // Update campaign
await synter.campaigns.launch(id)       // Launch campaign
await synter.campaigns.pause(id)        // Pause campaign
```

### `synter.conversions`

```typescript
await synter.conversions.create(params) // Track conversion (auto-detects platform)
await synter.conversions.list(params)   // List conversions
```

### `synter.analytics`

```typescript
await synter.analytics.roas(params)        // Get ROAS metrics
await synter.analytics.cac(params)         // Get CAC metrics
await synter.analytics.attribution(params) // Get attribution data
await synter.analytics.query(params)       // Custom analytics query
await synter.analytics.sendTo(platform, event) // Send event to analytics platform
```

### `synter.agents`

```typescript
await synter.agents.list()              // List available agents
await synter.agents.run(agent, params)  // Trigger agent
await synter.agents.runs(agent, params) // Get agent history
await synter.agents.getRun(runId)       // Get run details
await synter.agents.apply(runId)        // Apply dry-run changes
await synter.agents.updateConfig(agent, config) // Update agent config
```

### `synter.platforms`

```typescript
await synter.platforms.list()                     // List connected platforms
await synter.platforms.getAuthUrl(platform, uri)  // Get OAuth URL
await synter.platforms.completeAuth(platform, code) // Complete OAuth
await synter.platforms.disconnect(platform)       // Disconnect platform
```

### `synter.data`

```typescript
await synter.data.export(params)  // Export raw ad data
await synter.data.stream(params)  // Stream large datasets
```

## UTM Management

Auto-generated UTM parameters with platform-specific macros:

```typescript
await synter.campaigns.create({
  name: 'Product Launch',
  platform: 'google',
  utmTemplate: {
    source: 'google',
    medium: 'cpc',
    campaign: 'product-launch',
    content: '{{ad_id}}',     // → {creative}
    term: '{{keyword}}'       // → {keyword}
  }
});
```

## Conversion Tracking

Unified conversion tracking across all platforms:

```typescript
// Auto-detects platform from URL (gclid, rdt_cid, twclid)
await synter.conversions.create({
  event: 'purchase',
  value: 299.99,
  sendTo: ['google', 'reddit', 'posthog', 'mixpanel']
});

// Or provide click ID manually
await synter.conversions.create({
  clickId: req.query.gclid,
  event: 'signup',
  sendTo: ['google', 'heap']
});
```

## AI Agents

Full programmatic control over AI optimization:

```typescript
// Run budget optimizer in dry-run mode
const optimization = await synter.agents.run('budget-optimizer', {
  dryRun: true,
  params: {
    maxBudgetChange: 0.15,  // ±15%
    minConversions: 10
  }
});

// Inspect proposed changes
console.log(optimization.result.proposedChanges);
// [
//   { campaignId: 'cmp_123', currentBudget: 500, proposedBudget: 575, reason: 'CAC below target' },
//   { campaignId: 'cmp_456', currentBudget: 300, proposedBudget: 240, reason: 'CAC above threshold' }
// ]

// Apply if approved
if (userApproved) {
  await synter.agents.apply(optimization.runId);
}
```

## Bring Your Own AI

Export raw data for custom ML models:

```typescript
const data = await synter.data.export({
  platforms: ['google', 'reddit', 'x'],
  window: '30d',
  metrics: ['spend', 'clicks', 'conversions', 'revenue'],
  dimensions: ['platform', 'campaign_id', 'date']
});

// Feed to your own model
const predictions = await yourMLModel.predict(data.data);

// Or send to GPT-4
const analysis = await openai.chat.completions.create({
  model: 'gpt-4',
  messages: [{
    role: 'user',
    content: `Analyze this ad performance:\n${JSON.stringify(data.data)}`
  }]
});
```

## Analytics Integration

Send ad events to PostHog, Heap, Mixpanel automatically:

```typescript
const synter = new Synter({
  apiKey: process.env.SYNTER_API_KEY,
  analytics: {
    posthog: { apiKey: process.env.POSTHOG_KEY, enabled: true },
    mixpanel: { token: process.env.MIXPANEL_TOKEN, enabled: true }
  }
});

// All conversions automatically flow to analytics platforms
await synter.conversions.create({ event: 'purchase', value: 99 });
```

## Sandbox Mode

Test without hitting real ad platforms:

```typescript
const synter = new Synter({
  apiKey: 'sk_test_...',  // Test keys auto-enable sandbox
  environment: 'sandbox'
});

const campaign = await synter.campaigns.create({ /* ... */ });
console.log(campaign.id);  // 'cmp_test_123'
```

## TypeScript Support

Full type safety out of the box:

```typescript
import type { Campaign, CampaignCreateParams, Platform } from '@synter/sdk';

const params: CampaignCreateParams = {
  name: 'Launch',
  platform: 'google' as Platform,
  budget: { daily: 500, currency: 'USD' }
};
```

## Error Handling

```typescript
import { SynterError } from '@synter/sdk';

try {
  await synter.campaigns.create({ /* ... */ });
} catch (error) {
  if (error instanceof SynterError) {
    console.error(error.message);  // Human-readable message
    console.error(error.status);   // HTTP status code
    console.error(error.code);     // Error code (e.g., 'invalid_utm')
  }
}
```

## Links

- [Documentation](https://syntermedia.ai/docs)
- [Quick Start](../../docs/quickstart.md)
- [Guides](../../docs/guides/README.md)

## License

MIT
