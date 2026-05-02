# AI Assistant Chat Audit - Message Flagging

## Background

We build AI shopping assistants for e-commerce brands. Each assistant is configured with the brand's product catalog, policies, and tone. It handles customer queries - product recommendations, order tracking, FAQs, and more.

Every week our team manually reads through conversations to find where the assistant is struggling. It doesn't scale. We want a system to do this automatically.

## Your Task

Build a system that reads conversation data and **flags specific messages that need attention**.

For each flagged exchange, the system must produce:

1. **The conversation context** - enough of the thread to understand what happened
2. **The flagged message(s)** - the specific user message, agent response, or both
3. **A flag label** - a short category name (e.g. `IRRELEVANT_PRODUCT`)
4. **An explanation** - one or two sentences describing exactly what went wrong

### Example output

```
CONVERSATION 69f2c448...  [brand: 680a0a8b...]
------------------------------------------------
[user]   When will my order be delivered?

[agent]  To view your order details, please sign in to
         your account first.

[event]  user clicked: /account/login

[user]   Already signed in

[agent]  To view your order details, please sign in to
         your account first.

         ^ CONTEXT_IGNORED
         User followed the agent's instruction and confirmed they
         signed in. The agent repeated the identical response without
         acknowledging the updated context.

[user]   After sign in can't view order

[agent]  To view your order details, please sign in to
         your account first.

         ^ UNANSWERED_QUESTION
         The user's actual problem - unable to view orders after
         signing in - was stated twice and never addressed. The agent
         looped on the same instruction regardless of user feedback.
------------------------------------------------
```

## What We Evaluate

- **Accuracy.** Flags must reflect real problems. Only flag things that are genuinely wrong - a false positive is as bad as a miss.
- **Precision over recall.** A system that surfaces 50 accurate flags is more useful than one that flags 500 messages indiscriminately.
- **Speed.** The system should process the full dataset in a reasonable time. Design with throughput in mind.
- **Cost efficiency.** If you use LLM APIs, token usage matters. Avoid unnecessary calls, over-prompting, or processing messages that don't need analysis.
- **Explainability.** Every flag needs a clear reason. If you can't write a crisp explanation, the flag shouldn't exist.
- **Presentation.** Output must be readable by a non-technical reviewer. They should be able to act on it without digging into code.

Architecture, tooling, and methodology are entirely up to you.

## Data

Two datasets are provided. Both have the same schema.

| File | Conversations | Messages | Period |
|------|--------------|----------|--------|
| `conversations.json` + `messages.json` | 300 | ~1,500 | March 2026 |
| `conversations_v2.json` + `messages_v2.json` | 300 | ~2,200 | April 2026 |

### Import Instructions

```bash
# Dataset 1 (March 2026)
mongoimport --db helio_intern --collection conversations --file conversations.json --jsonArray
mongoimport --db helio_intern --collection messages --file messages.json --jsonArray

# Dataset 2 (April 2026)
mongoimport --db helio_intern --collection conversations_v2 --file conversations_v2.json --jsonArray
mongoimport --db helio_intern --collection messages_v2 --file messages_v2.json --jsonArray
```

### Schema

#### `conversations`

| Field | Type | Description |
|-------|------|-------------|
| `_id` | string | Unique conversation ID |
| `widgetId` | string | Brand identifier - each brand has a unique `widgetId` |
| `createdAt` | Date | When the conversation started |
| `updatedAt` | Date | Last activity timestamp |

#### `messages`

| Field | Type | Description |
|-------|------|-------------|
| `_id` | string | Unique message ID |
| `conversationId` | string | Links to the parent conversation |
| `sender` | string | `"user"` (customer) or `"agent"` (assistant) |
| `text` | string | Message content |
| `messageType` | string | `"text"` for actual chat; `"event"` for user interactions |
| `metadata` | object | Additional context |
| `timestamp` | Date | When the message was sent |

### Notes

- **Focus on `messageType: "text"` messages** for your analysis - those are the actual chat exchanges.
- **`messageType: "event"` messages** record user interactions such as clicking a product suggestion (`metadata.eventType: "product_view"`). Use them as supporting signals - e.g. a user clicking products after an agent message is a positive signal; no clicks after product recommendations may indicate the recommendations were off.
- Use `widgetId` to segment analysis by brand. Problems are often brand-specific.
- Messages are in chronological order by `timestamp` within a conversation.
