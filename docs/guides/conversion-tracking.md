# Conversion Tracking

Unified conversion tracking across all ad platforms with automatic click ID detection.

## Overview

Synter provides a single API to track conversions across all your ad platforms. We automatically detect which platform the click came from and send the conversion to the right place.

## How It Works

1. User clicks an ad → arrives at your site with a click ID in the URL
2. You call `synter.conversions.create()` with the event details
3. Synter auto-detects the platform from the click ID
4. Conversion is sent to the ad platform's API
5. Optionally, event is also sent to your analytics platforms

## Click ID Detection

Each platform uses a different URL parameter:

| Platform | Parameter | Example |
|----------|-----------|---------|
| Google Ads | `gclid` | `?gclid=CjwKCAjw...` |
| Reddit | `rdt_cid` | `?rdt_cid=abc123` |
| LinkedIn | `li_fat_id` | `?li_fat_id=xyz789` |
| Microsoft | `msclkid` | `?msclkid=def456` |
| Meta | `fbclid` | `?fbclid=fb123` |
| X/Twitter | `twclid` | `?twclid=tw789` |

## Basic Usage

### TypeScript

```typescript
// Auto-detects platform from URL
await synter.conversions.create({
  event: 'purchase',
  value: 299.99,
  currency: 'USD'
});

// Or provide click ID manually
await synter.conversions.create({
  clickId: req.query.gclid,
  event: 'signup',
  value: 0
});
```

### Python

```python
# Auto-detects platform from click ID
await synter.conversions.create({
    "click_id": request.args.get("gclid"),
    "event": "purchase",
    "value": 299.99,
    "currency": "USD"
})
```

## Fan Out to Multiple Destinations

Send conversions to ad platforms AND analytics tools simultaneously:

```typescript
await synter.conversions.create({
  event: 'purchase',
  value: 299.99,
  sendTo: ['google', 'reddit', 'posthog', 'mixpanel']
});
```

## Conversion Events

Standard events we support:

| Event | Description | Typical Value |
|-------|-------------|---------------|
| `page_view` | Page viewed | 0 |
| `signup` | User signed up | 0 |
| `lead` | Lead captured | 0-50 |
| `add_to_cart` | Item added to cart | Item price |
| `begin_checkout` | Checkout started | Cart value |
| `purchase` | Purchase completed | Order value |
| `subscription` | Subscription started | MRR |

## Custom Events

You can also track custom events:

```typescript
await synter.conversions.create({
  event: 'demo_scheduled',
  value: 500,  // Estimated lead value
  properties: {
    plan: 'enterprise',
    source: 'pricing_page'
  }
});
```

## Server-Side Tracking

For server-side conversion tracking, store the click ID on your backend:

```typescript
// On page load, store click ID in session
app.get('/', (req, res) => {
  if (req.query.gclid) {
    req.session.clickId = req.query.gclid;
    req.session.clickPlatform = 'google';
  }
});

// On conversion, use stored click ID
app.post('/checkout', async (req, res) => {
  await synter.conversions.create({
    clickId: req.session.clickId,
    event: 'purchase',
    value: req.body.orderTotal
  });
});
```

## Offline Conversions

For conversions that happen offline (phone calls, in-person sales):

```typescript
await synter.conversions.create({
  clickId: storedClickId,
  event: 'offline_purchase',
  value: 5000,
  conversionTime: new Date('2025-01-15T14:30:00Z')  // When conversion actually happened
});
```

## Debugging

Use the tracking utilities to debug click detection:

```typescript
import { detectClickId, extractTrackingParams } from '@synter/sdk';

// Detect click ID from URL
const { platform, clickId } = detectClickId(window.location.href);
console.log(platform); // 'google'
console.log(clickId);  // 'CjwKCAjw...'

// Extract all tracking params
const tracking = extractTrackingParams(window.location.href, document.referrer);
console.log(tracking.clickIds);  // { google: 'abc123', reddit: 't3_xyz' }
console.log(tracking.utm);        // { source: 'google', medium: 'cpc', ... }
```

## Best Practices

1. **Store click IDs server-side** for accurate attribution
2. **Use consistent event names** across your codebase
3. **Include transaction IDs** to dedupe conversions
4. **Send conversions as soon as possible** after they occur
5. **Test with sandbox mode** before going live
