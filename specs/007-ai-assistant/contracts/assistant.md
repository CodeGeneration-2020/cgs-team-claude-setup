# API Contract: AI Assistant

**Feature**: 007-ai-assistant | **Date**: 2026-03-17

## POST /v1/assistant/start

Starts a new assistant session and returns the first question.

**Authentication**: Optional (works for visitors and authenticated users)

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "sessionId": "clx_asst_001",
    "question": {
      "id": "q1",
      "text": "What areas of climate finance interest you most?",
      "displayOrder": 1,
      "isSkippable": true,
      "placeholderText": "e.g., renewable energy, clean water, biodiversity..."
    },
    "totalQuestions": 4,
    "currentQuestion": 1
  }
}
```

---

## POST /v1/assistant/:sessionId/answer

Submits an answer to the current question. Returns a contextual response and the next question (or results if this was the last question).

**Authentication**: Optional

### Request

```json
{
  "questionId": "q1",
  "answer": "I'm interested in renewable energy and sustainable agriculture projects"
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| questionId | string | Yes | Must match current question |
| answer | string | Yes | Max 2000 chars, non-empty |

### Response (200 OK — next question)

```json
{
  "success": true,
  "data": {
    "contextualResponse": "Great choices! Renewable energy and sustainable agriculture are among the most impactful sectors. Let me ask a few more questions to narrow things down.",
    "question": {
      "id": "q2",
      "text": "Are there specific regions or countries you're focused on?",
      "displayOrder": 2,
      "isSkippable": true,
      "placeholderText": "e.g., East Africa, Southeast Asia, global..."
    },
    "currentQuestion": 2,
    "totalQuestions": 4,
    "isComplete": false
  }
}
```

### Response (200 OK — flow complete, results ready)

```json
{
  "success": true,
  "data": {
    "contextualResponse": "Thank you! Based on your interests in renewable energy in East Africa, I've found some projects that might be a great fit.",
    "isComplete": true,
    "searchQuery": "renewable energy sustainable agriculture East Africa",
    "resultCount": 12,
    "redirectUrl": "/search?q=renewable+energy+sustainable+agriculture+East+Africa&source=assistant"
  }
}
```

---

## POST /v1/assistant/:sessionId/skip

Skips the current question and moves to the next one.

**Authentication**: Optional

### Request

```json
{
  "questionId": "q2"
}
```

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "contextualResponse": "No problem, let's move on.",
    "question": {
      "id": "q3",
      "text": "What type of projects do you prefer?",
      "displayOrder": 3,
      "isSkippable": true,
      "placeholderText": "e.g., early-stage, community-driven, large-scale..."
    },
    "currentQuestion": 3,
    "totalQuestions": 4,
    "isComplete": false
  }
}
```

---

## GET /v1/assistant/questions

Returns the predefined question list (for debugging/admin purposes).

**Authentication**: None (public)

### Response (200 OK)

```json
{
  "success": true,
  "data": [
    {
      "id": "q1",
      "text": "What areas of climate finance interest you most?",
      "displayOrder": 1,
      "isSkippable": true,
      "placeholderText": "e.g., renewable energy, clean water..."
    },
    {
      "id": "q2",
      "text": "Are there specific regions or countries you're focused on?",
      "displayOrder": 2,
      "isSkippable": true,
      "placeholderText": "e.g., East Africa, Southeast Asia..."
    }
  ]
}
```

---

## POST /v1/assistant/:sessionId/transfer

Transfers a visitor's session to an authenticated user (called after login/register).

**Authentication**: Required

### Request

```json
{
  "sessionId": "clx_asst_001"
}
```

### Response (200 OK)

```json
{
  "success": true,
  "data": {
    "sessionId": "clx_asst_001",
    "userId": "clx_user_123",
    "transferred": true
  }
}
```

### Error (404)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "NOT_FOUND",
    "message": "Session not found or already expired."
  }
}
```
