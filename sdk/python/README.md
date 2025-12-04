# synter

**The developer-first SDK for multi-platform ad management.**

Ship ads like you ship code. One SDK for Google Ads, Reddit Ads, X/Twitter Ads, and beyond.

## Features

✅ **Infrastructure-first**: UTM management, conversion tracking, analytics integration  
✅ **Multi-platform**: Google, Reddit, LinkedIn, Microsoft, Meta, X  
✅ **Transparent AI agents**: Run, inspect, and control optimization agents programmatically  
✅ **Bring your own AI**: Export raw data for custom ML models  
✅ **Analytics integrations**: PostHog, Heap, Mixpanel, Segment  
✅ **Type-safe**: Full type hints for Python 3.8+  
✅ **Async-first**: Built on httpx for high-performance async I/O  

## Installation

```bash
pip install synter
```

## Quick Start

```python
import asyncio
from synter import Synter

async def main():
    async with Synter(api_key="sk_live_...") as synter:
        # Create a campaign (UTMs auto-generated)
        campaign = await synter.campaigns.create({
            "name": "Q4 Product Launch",
            "platform": "google",
            "budget": {"daily": 500, "currency": "USD"},
            "targeting": {
                "keywords": ["saas analytics", "data platform"],
                "locations": ["US", "CA"],
            }
        })

        # Track conversions (platform auto-detected)
        await synter.conversions.create({
            "click_id": request.args.get("gclid"),
            "event": "purchase",
            "value": 299.99
        })

        # Get ROAS
        roas = await synter.analytics.roas({"window": "30d"})
        print(roas)  # {"google": 3.2, "reddit": 2.8}

asyncio.run(main())
```

## API Reference

### `synter.campaigns`

```python
await synter.campaigns.create(params)      # Create campaign with auto-generated UTMs
await synter.campaigns.list(params)        # List campaigns
await synter.campaigns.get(id)             # Get campaign details
await synter.campaigns.update(id, params)  # Update campaign
await synter.campaigns.launch(id)          # Launch campaign
await synter.campaigns.pause(id)           # Pause campaign
```

### `synter.conversions`

```python
await synter.conversions.create(params)    # Track conversion (auto-detects platform)
await synter.conversions.list(params)      # List conversions
```

### `synter.analytics`

```python
await synter.analytics.roas(params)        # Get ROAS metrics
await synter.analytics.cac(params)         # Get CAC metrics
await synter.analytics.attribution(params) # Get attribution data
await synter.analytics.query(params)       # Custom analytics query
await synter.analytics.send_to(platform, event)  # Send event to analytics platform
```

### `synter.agents`

```python
await synter.agents.list()                 # List available agents
await synter.agents.run(agent, params)     # Trigger agent
await synter.agents.runs(agent, params)    # Get agent history
await synter.agents.get_run(run_id)        # Get run details
await synter.agents.apply(run_id)          # Apply dry-run changes
await synter.agents.update_config(agent, config)  # Update agent config
```

### `synter.platforms`

```python
await synter.platforms.list()                           # List connected platforms
await synter.platforms.get_auth_url(platform, redirect) # Get OAuth URL
await synter.platforms.complete_auth(platform, code)    # Complete OAuth
await synter.platforms.disconnect(platform)             # Disconnect platform
```

### `synter.data`

```python
await synter.data.export(params)           # Export raw ad data
async for batch in synter.data.stream(params):  # Stream large datasets
    process(batch)
```

## UTM Management

Auto-generated UTM parameters with platform-specific macros:

```python
await synter.campaigns.create({
    "name": "Product Launch",
    "platform": "google",
    "utm_template": {
        "source": "google",
        "medium": "cpc",
        "campaign": "product-launch",
        "content": "{{ad_id}}",     # → {creative}
        "term": "{{keyword}}"       # → {keyword}
    }
})
```

## Conversion Tracking

Unified conversion tracking across all platforms:

```python
# Auto-detects platform from click ID (gclid, rdt_cid, twclid)
await synter.conversions.create({
    "click_id": gclid,
    "event": "purchase",
    "value": 299.99,
    "send_to": ["google", "reddit", "posthog", "mixpanel"]
})
```

## AI Agents

Full programmatic control over AI optimization:

```python
# Run budget optimizer in dry-run mode
optimization = await synter.agents.run("budget-optimizer", {
    "dry_run": True,
    "window": {"start": "2025-11-01", "end": "2025-11-08"},
    "params": {
        "max_budget_change": 0.15,  # ±15%
        "min_conversions": 10
    }
})

# Inspect proposed changes
print(optimization["result"]["proposed_changes"])
# [
#   {"campaign_id": "cmp_123", "current_budget": 500, "proposed_budget": 575, "reason": "CAC below target"},
#   {"campaign_id": "cmp_456", "current_budget": 300, "proposed_budget": 240, "reason": "CAC above threshold"}
# ]

# Apply if approved
if user_approved:
    await synter.agents.apply(optimization["run_id"])
```

## Bring Your Own AI

Export raw data for custom ML models:

```python
# Export normalized ad data
data = await synter.data.export({
    "platforms": ["google", "reddit", "x"],
    "window": "30d",
    "metrics": ["spend", "clicks", "conversions", "revenue"],
    "dimensions": ["platform", "campaign_id", "date"]
})

# Feed to your own model
predictions = your_ml_model.predict(data["data"])

# Or send to GPT-4
import openai
analysis = await openai.ChatCompletion.create(
    model="gpt-4",
    messages=[{
        "role": "user",
        "content": f"Analyze this ad performance:\n{data['data']}"
    }]
)
```

## Analytics Integration

Send ad events to PostHog, Heap, Mixpanel automatically:

```python
from synter.utils import AnalyticsIntegrationManager

manager = AnalyticsIntegrationManager([
    {"platform": "posthog", "api_key": os.getenv("POSTHOG_KEY"), "enabled": True},
    {"platform": "mixpanel", "api_key": os.getenv("MIXPANEL_TOKEN"), "enabled": True}
])

await manager.track({
    "event": "purchase",
    "properties": {"value": 99, "campaign": "launch"}
})
```

## Sandbox Mode

Test without hitting real ad platforms:

```python
synter = Synter({
    "api_key": "sk_test_...",  # Test keys auto-enable sandbox
    "environment": "sandbox"
})

campaign = await synter.campaigns.create({...})
print(campaign["id"])  # "cmp_test_123"
```

## Type Safety

Full type hints for Python 3.8+:

```python
from synter.types import Campaign, CampaignCreateParams, Platform

params: CampaignCreateParams = {
    "name": "Launch",
    "platform": "google",
    "budget": {"daily": 500, "currency": "USD"}
}

campaign: Campaign = await synter.campaigns.create(params)
```

## Error Handling

```python
from synter import SynterError

try:
    await synter.campaigns.create({...})
except SynterError as e:
    print(e.message)  # Human-readable message
    print(e.status)   # HTTP status code
    print(e.code)     # Error code (e.g., "invalid_utm")
```

## Context Manager

Use async context manager for automatic cleanup:

```python
async with Synter(api_key="sk_live_...") as synter:
    campaign = await synter.campaigns.create({...})
    # HTTP client automatically closed on exit
```

## Development

```bash
# Install dev dependencies
pip install -e ".[dev]"

# Run tests
pytest

# Type checking
mypy synter

# Format code
black synter
ruff check synter
```

## Links

- [Documentation](https://syntermedia.ai/docs)
- [Quick Start](../../docs/quickstart.md)
- [Guides](../../docs/guides/README.md)

## License

MIT
