# UTM Management

Stop manually managing UTM parameters. Synter handles platform-specific macros automatically.

## The Problem

Each ad platform has its own dynamic parameter syntax:

| Platform | Syntax | Example |
|----------|--------|---------|
| Google Ads | `{keyword}` | `{creative}`, `{keyword}`, `{campaignid}` |
| Reddit | `{{AD_ID}}` | `{{AD_ID}}`, `{{CAMPAIGN_ID}}` |
| LinkedIn | `{creative}` | `{creative}`, `{campaign_id}` |
| Microsoft | `{keyword}` | `{AdId}`, `{keyword}` |

Managing these manually is error-prone and tedious.

## The Solution

Use Synter's universal template syntax and we'll translate to each platform:

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

Your landing page URLs automatically get:
```
?utm_source=google&utm_medium=cpc&utm_campaign=product-launch
&utm_content={creative}&utm_term={keyword}
```

## Universal Template Variables

| Variable | Google | Reddit | LinkedIn | Microsoft |
|----------|--------|--------|----------|-----------|
| `{{ad_id}}` | `{creative}` | `{{AD_ID}}` | `{creative}` | `{AdId}` |
| `{{campaign_id}}` | `{campaignid}` | `{{CAMPAIGN_ID}}` | `{campaign_id}` | `{CampaignId}` |
| `{{keyword}}` | `{keyword}` | N/A | N/A | `{keyword}` |
| `{{placement}}` | `{placement}` | `{{SUBREDDIT}}` | N/A | `{placement}` |
| `{{device}}` | `{device}` | `{{DEVICE}}` | N/A | `{device}` |

## Standalone Usage

You can also use the UTM builder directly:

### TypeScript

```typescript
import { buildUTMParameters } from '@synter/sdk';

const utm = buildUTMParameters(
  {
    source: 'google',
    campaign: 'launch',
    content: '{{ad_id}}_{{keyword}}'
  },
  'google'
);
// → { content: '{creative}_{keyword}' }
```

### Python

```python
from synter.utils import build_utm_parameters

utm = build_utm_parameters(
    {"source": "google", "campaign": "launch", "content": "{{ad_id}}_{{keyword}}"},
    "google"
)
# {"content": "{creative}_{keyword}"}
```

## Best Practices

1. **Use consistent campaign names** across platforms for easier cross-platform analysis
2. **Include ad_id and campaign_id** in UTMs for granular attribution
3. **Use term for keywords** on search platforms (Google, Microsoft)
4. **Use content for creative variants** to A/B test different ad copy
