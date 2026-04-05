# Selkokielelle — Plain Language Converter

Converts complex Finnish text into selkokieli (plain language) for accessibility compliance. Form submission triggers a GPT-4 pipeline that applies Finnish plain-language guidelines and delivers a side-by-side comparison by email.

## Flow

```
Tally form webhook (text + email)
    ↓
Data extraction → original text, sender email
    ↓
OpenRouter LLM (GPT-4) → applies 15+ plain-language rules
    ↓
Markdown → HTML (side-by-side comparison layout)
    ↓
Gmail API → delivers comparison email
```

## APIs

1. **Tally webhook** — form submission trigger
2. **OpenRouter LLM (GPT-4)** — text transformation
3. **Gmail API** — email delivery

## Prompt Design

The LLM prompt encodes 15+ guidelines from Finnish accessibility standards:
- Sentence length limits (8–12 words)
- Vocabulary simplification
- Active voice requirements
- Structured output formatting for side-by-side display

## Relevance

Selkokieli is a recognized standard for accessible Finnish. Relevant to EU Accessibility Directive compliance and public sector communication requirements. The same form → transformation → notification pattern applies to translation, summarization, or tone adjustment workflows.
