# AI Assistant Conversation Analysis

## Background

We build AI assistants for e-commerce brands. Each brand has its own assistant configured with their product catalog, brand voice, and policies. The assistant handles customer queries -- product questions, recommendations, order issues, and more.

Currently, our team reviews conversations manually every week to identify issues and find ways to improve the assistant experience. This is time-consuming and doesn't scale as we onboard more brands.

## Your Task

Build a system that automates this process. Given conversation data, your system should analyze conversations and surface actionable insights that help us improve the AI assistant.

There is no fixed checklist of what to look for. You need to understand the data, explore it, find patterns, and figure out what's useful. The kinds of issues vary by brand, by product category, by the type of question being asked.

Some questions to get you thinking (these are not exhaustive -- you may find things we haven't thought of):

- Where does the assistant struggle? Where does it do well?
- Are there patterns in conversations where users drop off or seem frustrated?
- How does performance differ across brands? Why might that be?
- What kinds of questions does the assistant handle poorly?
- Are there recurring issues that affect multiple brands, or are problems brand-specific?
- Is the assistant hallucinating -- making up information that isn't true?
- Is the assistant suggesting wrong or irrelevant products?

## What We Expect

- A working system that takes in conversation data and produces useful insights.
- How you build this is entirely up to you -- architecture, tooling, methodology, how you present the output.
- We care about the quality of insights your system produces and how you approach the problem.

## Data

You'll receive a MongoDB dump containing conversations and messages from multiple brands. Import it into your local MongoDB instance.

### Import Instructions

```
mongoimport --db helio_intern --collection conversations --file conversations.json --jsonArray

mongoimport --db helio_intern --collection messages --file messages.json --jsonArray
```

### Schema Reference

The data has two collections: `conversations` and `messages`. A conversation contains multiple messages.

#### `conversations` collection

```
_id            ObjectId   Unique conversation ID
widgetId       ObjectId   The brand's assistant ID. Each brand has a unique widgetId.
createdAt      Date       When the conversation started
updatedAt      Date       Last activity timestamp
```

#### `messages` collection

```
_id              ObjectId   Unique message ID
conversationId   ObjectId   Links to the parent conversation
sender           String     "user" (customer) or "agent" (assistant)
text             String     The message content
messageType      String     "text" (actual chat) or "event" (user interaction
                            tracked as event, e.g. clicking a product suggestion)
metadata         Object     Additional message context (see below)
timestamp        Date       When the message was sent
```

Message `metadata` fields:

```
metadata.eventType     Type of event (e.g. "product_view", "quick_action_click")
```

### Relationships

```
Brand (widgetId)
  |-- Conversation
  |     |-- Messages[]
  |           |-- sender: "user" (customer messages)
  |           |-- sender: "agent" (assistant responses)
  |                 |-- metadata.eventType (interaction events)
```

### Tips

- Use `widgetId` to segment data by brand.
- `messageType: "text"` messages are the actual chat. `messageType: "event"` messages are user interactions like clicking a product the assistant suggested.
- Messages are ordered by `timestamp` within a conversation.
