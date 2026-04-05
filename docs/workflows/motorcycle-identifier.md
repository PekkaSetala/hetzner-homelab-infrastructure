# AI Motorcycle Identifier

Webhook-triggered image analysis pipeline. Receives a motorcycle photo, identifies it via OpenAI Vision, enriches with market data, and delivers results by email.

## Flow

```
Webhook POST (image upload)
    ↓
Binary → Base64 conversion
    ↓
OpenAI GPT-4o Vision → make, model, year, features
    ↓
OpenRouter LLM + SerpAPI → recommendations, similar bikes, pricing
    ↓
Markdown → HTML formatting
    ↓
Gmail API → delivers analysis email
```

## APIs

1. **Webhook** — event trigger
2. **OpenAI GPT-4o Vision** — image analysis
3. **OpenRouter LLM** — recommendation generation
4. **SerpAPI** — web search for market data
5. **Gmail API** — email delivery (OAuth2)

## Data Transformation

`binary image → base64 → JSON (Vision API response) → markdown → HTML → email`

Five APIs chained in sequence. Vision API prompts are structured to return consistent fields (make, model, year, condition). The LLM step uses that structured output as context for recommendations.

## Pattern

Webhook → analysis → enrichment → notification. Same pattern applies to order processing, support ticket routing, or form submission pipelines.
