# Outgoing Message Logic Diagrams
**Mevrik Channels Platform**  
**Created:** October 27, 2025  
**Last Updated:** October 27, 2025  
**Document Type:** Message Operations & Agent Actions Documentation

---

## Table of Contents

1. [Complete Outgoing Message Overview](#1-complete-outgoing-message-overview)
2. [Agent Reply Flow (message_reply)](#2-agent-reply-flow-message_reply)
3. [Message History Flow (message_history)](#3-message-history-flow-message_history)
4. [Message Closed Flow (message_closed)](#4-message-closed-flow-message_closed)
5. [Bulk Reply Flow (message_bulk_reply)](#5-bulk-reply-flow-message_bulk_reply)
6. [Bulk Close Flow (message_bulk_close)](#6-bulk-close-flow-message_bulk_close)
7. [Video Call Request Flow (video_call_request)](#7-video-call-request-flow-video_call_request)
8. [Video Call Start Flow (video_call_start)](#8-video-call-start-flow-video_call_start)
9. [Post Details Flow (post_details)](#9-post-details-flow-post_details)
10. [Comment Details Flow (comment_details)](#10-comment-details-flow-comment_details)
11. [System Message Closed (system_message_closed)](#11-system-message-closed-system_message_closed)
12. [MessagesController Core Logic](#12-messagescontroller-core-logic)

---

## 1. Complete Outgoing Message Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      OUTGOING MESSAGE OPERATIONS                            │
└─────────────────────────────────────────────────────────────────────────────┘

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ ENTRY POINT                                                                ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                                            ┃
┃                   POST /api/v1/messages                                    ┃
┃                   POST /api/v1/agent/messages                              ┃
┃                                                                            ┃
┃                   ?action={action_name}                                    ┃
┃                                                                            ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                              │
                              ▼
                ┌─────────────────────────┐
                │  MessagesController     │
                │  __invoke()             │
                └─────────────────────────┘
                              │
                              ▼
                      Check Mode?
                              │
                ┌─────────────┴─────────────┐
                │                           │
                ▼                           ▼
        ┌───────────────┐           ┌───────────────┐
        │ AGENT MODE    │           │ CUSTOMER MODE │
        │ /agent/       │           │ /api/v1/      │
        └───────────────┘           └───────────────┘
                │                           │
                ▼                           ▼
        ┌───────────────┐           ┌───────────────┐
        │ Must have     │           │ Must have     │
        │ Agent Login   │           │ Customer      │
        │               │           │ Session       │
        └───────────────┘           └───────────────┘
                │                           │
                └─────────────┬─────────────┘
                              ▼
                ┌──────────────────────────────┐
                │ Create MessagingRequest      │
                │ Helper($request)             │
                └──────────────────────────────┘
                              │
                              ▼
                         Action Type?
                              │
        ┌─────────────────────┼─────────────────────┬──────────────┐
        │                     │                     │              │
        ▼                     ▼                     ▼              ▼
┌──────────────┐      ┌──────────────┐     ┌──────────────┐  ┌────────┐
│ Agent Reply  │      │ Bulk Ops     │     │ History/List │  │ Video  │
│ Operations   │      │              │     │ Operations   │  │ Calls  │
└──────────────┘      └──────────────┘     └──────────────┘  └────────┘
        │                     │                     │              │
        ▼                     ▼                     ▼              ▼

┌──────────────┐      ┌──────────────┐     ┌──────────────┐  ┌────────┐
│message_reply │      │message_bulk_ │     │message_      │  │video_  │
│message_      │      │reply         │     │history       │  │call_   │
│example       │      │message_bulk_ │     │message_      │  │request │
│              │      │close         │     │closed        │  │video_  │
│              │      │              │     │              │  │call_   │
│              │      │              │     │              │  │start   │
└──────────────┘      └──────────────┘     └──────────────┘  └────────┘

┌──────────────┐      ┌──────────────┐
│ Social Media │      │ System Ops   │
│ Details      │      │              │
└──────────────┘      └──────────────┘
        │                     │
        ▼                     ▼
┌──────────────┐      ┌──────────────┐
│post_details  │      │system_       │
│comment_      │      │message_      │
│details       │      │closed        │
└──────────────┘      └──────────────┘
```

---

## 2. Agent Reply Flow (message_reply)

```
┌──────────────────────────────────────────────────────────────────┐
│                    AGENT REPLY FLOW                              │
│                    (message_reply)                               │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│ Agent Dashboard │
│ Selects Message │
│ Types Reply     │
└─────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ POST /api/v1/agent/messages                │
│ ?action=message_reply                      │
│                                            │
│ Payload:                                   │
│ {                                          │
│   "message": {                             │
│     "message_type": "message",             │
│     "body": "Agent response here",         │
│     "is_reply": 67288                      │
│   }                                        │
│ }                                          │
│                                            │
│ Headers:                                   │
│ X-Mevrik-Attending-Id: {agent_id}          │
│ Authorization: Bearer {token}              │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessagesController::__invoke()             │
│ • Extract action = "message_reply"         │
│ • Detect isAgentMode = true                │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Check Agent Authentication                 │
└────────────────────────────────────────────┘
        │
   ┌────┴────┐
   │         │
  No       Yes
   │         │
   ▼         ▼
┌──────┐  ┌────────────────────────────────┐
│Error │  │ Create MessagingRequestHelper  │
│401   │  │ $messageContainer              │
└──────┘  └────────────────────────────────┘
                    │
                    ▼
          ┌──────────────────────────┐
          │ Check hasBulkReplyId()?  │
          └──────────────────────────┘
                    │
            ┌───────┴────────┐
            │                │
          Yes              No
            │                │
            ▼                │
  ┌──────────────────┐       │
  │ firstReplyId()   │       │
  │ (Process first)  │       │
  └──────────────────┘       │
            │                │
            └────────┬───────┘
                     ▼
          ┌──────────────────────────┐
          │ Get Parent Message       │
          │ $parent = parentMessage()│
          └──────────────────────────┘
                     │
             ┌───────┴────────┐
             │                │
           Found           Not Found
             │                │
             ▼                ▼
  ┌──────────────────┐  ┌──────────┐
  │ createSession    │  │ Error    │
  │ FromMessage(     │  │ 403      │
  │ $parent)         │  │ "Failed  │
  │                  │  │ to       │
  │ Restore customer │  │ rehydrate│
  │ session context  │  │ session" │
  └──────────────────┘  └──────────┘
             │
             ▼
  ┌──────────────────────────┐
  │ Check session_path?      │
  └──────────────────────────┘
             │
      ┌──────┴──────┐
      │             │
    Exists        None
      │             │
      ▼             │
  ┌────────┐        │
  │ Set    │        │
  │ Session│        │
  │ Path   │        │
  └────────┘        │
      │             │
      └──────┬──────┘
             ▼
  ┌──────────────────────────┐
  │ validateSessionFor       │
  │ AgentReply()             │
  │ • Check session is LIVE  │
  │ • Verify agent assigned  │
  └──────────────────────────┘
             │
       ┌─────┴─────┐
       │           │
     Valid     Invalid
       │           │
       │           ▼
       │     ┌──────────┐
       │     │ Error    │
       │     │ Session  │
       │     │ not LIVE │
       │     └──────────┘
       │
       ▼
  ┌──────────────────────────┐
  │ app('message_reply')     │
  │ ->run($messageContainer) │
  └──────────────────────────┘
             │
             ▼
  ┌──────────────────────────┐
  │ MessageReply Action      │
  │ • Build outgoing message │
  │ • Determine channel      │
  │ • Format for channel     │
  └──────────────────────────┘
             │
             ▼
       Channel Type?
             │
    ┌────────┼────────┐
    │        │        │
    ▼        ▼        ▼
┌────────┐ ┌──┐ ┌──────┐
│Widget  │ │FB│ │Email │
└────────┘ └──┘ └──────┘
    │        │        │
    └────────┼────────┘
             ▼
  ┌──────────────────────────┐
  │ Send via Channel API     │
  │ • Widget: WebSocket      │
  │ • FB: Graph API          │
  │ • Email: SMTP            │
  └──────────────────────────┘
             │
             ▼
  ┌──────────────────────────┐
  │ Store Outgoing Message   │
  │ in Database              │
  └──────────────────────────┘
             │
             ▼
  ┌──────────────────────────┐
  │ Update Session State     │
  │ • Last agent activity    │
  │ • Message count          │
  └──────────────────────────┘
             │
             ▼
  ┌──────────────────────────┐
  │ Trigger Events           │
  │ • AgentRepliedEvent      │
  │ • Update Analytics       │
  └──────────────────────────┘
             │
             ▼
  ┌──────────────────────────┐
  │ Return Success Response  │
  │ {                        │
  │   "success": true,       │
  │   "message_id": "..."    │
  │ }                        │
  └──────────────────────────┘
```

---

## 3. Message History Flow (message_history)

```
┌──────────────────────────────────────────────────────────────────┐
│                  MESSAGE HISTORY FLOW                            │
│                  (message_history)                               │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│ Agent/Customer  │
│ Requests        │
│ Message History │
└─────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ POST /api/v1/messages                      │
│ ?action=message_history                    │
│                                            │
│ Payload:                                   │
│ {                                          │
│   "session_id": "sess_123",                │
│   "limit": 50,                             │
│   "offset": 0,                             │
│   "filters": {                             │
│     "from_date": "2025-01-01",             │
│     "to_date": "2025-01-31"                │
│   }                                        │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessagesController                         │
│ • Validate action                          │
│ • Create MessageContainer                  │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Check Mode                                 │
└────────────────────────────────────────────┘
        │
   ┌────┴────┐
   │         │
 Agent    Customer
   │         │
   ▼         ▼
┌──────┐  ┌─────────┐
│Check │  │ Check   │
│Agent │  │ Customer│
│Auth  │  │ Session │
└──────┘  └─────────┘
   │         │
   └────┬────┘
        ▼
┌────────────────────────────────────────────┐
│ app('message_history')                     │
│ ->run($messageContainer)                   │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessageHistory Action                      │
│ • Extract filters                          │
│ • Validate session access                  │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Query Database                             │
│ SELECT * FROM messages                     │
│ WHERE session_id = ?                       │
│ AND created_at BETWEEN ? AND ?             │
│ ORDER BY created_at DESC                   │
│ LIMIT ? OFFSET ?                           │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Format Messages                            │
│ • Include sender info                      │
│ • Include attachments                      │
│ • Include metadata                         │
│ • Mask sensitive data                      │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Build Response                             │
│ {                                          │
│   "messages": [...],                       │
│   "pagination": {                          │
│     "total": 150,                          │
│     "limit": 50,                           │
│     "offset": 0,                           │
│     "has_more": true                       │
│   },                                       │
│   "session": {                             │
│     "id": "sess_123",                      │
│     "status": "active"                     │
│   }                                        │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Return Response                            │
└────────────────────────────────────────────┘
```

---

## 4. Message Closed Flow (message_closed)

### 4.1. Main Closure Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                   MESSAGE CLOSED FLOW                            │
│                   (message_closed)                               │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│ Agent Closes    │
│ Conversation    │
└─────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ POST /api/v1/agent/messages                │
│ ?action=message_closed                     │
│                                            │
│ Payload:                                   │
│ {
    "message": {
        "message_type": "message",
        "body": "Hiii",
        "is_reply": 67288
    }
}                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessagesController                         │
│ • Agent mode required                      │
│ • Validate permissions                     │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ app('message_closed')                      │
│ ->run($messageContainer)                   │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ STEP 1: Update Session                     │
│ UPDATE sessions SET                        │
│   status = 'closed',                       │
│   closed_at = NOW(),                       │
│   closed_by = agent_id,                    │
│   close_reason = ?,                        │
│   close_notes = ?                          │
│ WHERE id = ?                               │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ STEP 2: Update Case                        │
│ UPDATE cases SET                           │
│   status = 'closed'                        │
│ WHERE session_id = ?                       │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ STEP 3: Update Queue                       │
│ DELETE FROM queues                         │
│ WHERE session_id = ?                       │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ STEP 4: Create System Message              │
│ "Conversation closed by Agent"             │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ STEP 5: Check Channel Type                │
└────────────────────────────────────────────┘
        │
   ┌────┴──────────────┬──────────────┐
   │                   │              │
   ▼                   ▼              ▼
┌────────┐      ┌──────────┐   ┌──────────┐
│Widget  │      │Facebook  │   │Facebook  │
│        │      │Messenger │   │Comment   │
└────────┘      └──────────┘   └──────────┘
   │                   │              │
   └─────────┬─────────┘              │
             │                        │
             ▼                        │
┌────────────────────────┐            │
│ STEP 6: Send CSAT      │            │
│ Survey                 │            │
│ (Widget/Messenger)     │            │
└────────────────────────┘            │
             │                        │
             └──────────┬─────────────┘
                        ▼
            ┌────────────────────────┐
            │ STEP 7: Generate       │
            │ Reports                │
            └────────────────────────┘
                        │
        ┌───────────────┼──────────────┐
        │                              │
        ▼                              ▼
┌──────────────┐            ┌─────────────────────┐
│report_master │            │customer_social_     │
│              │            │activity_reply_report│
│(Widget/      │            │                     │
│ Messenger)   │            │(Facebook Comment)   │
└──────────────┘            └─────────────────────┘
        │                              │
        └───────────────┬──────────────┘
                        ▼
            ┌────────────────────────┐
            │ STEP 8: Message Object │
            └────────────────────────┘
```

### 4.2. CSAT Survey Flow (Widget & Messenger Only)

```
┌──────────────────────────────────────────────────────────────────┐
│         CSAT SURVEY AFTER CLOSURE (Widget/FB Messenger)          │
└──────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────┐
│ Session Closed (Widget or FB Messenger)    │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Send CSAT Survey Message                   │
│                                            │
│ Text: "Are you satisfied with             │
│        the service?"                       │
│                                            │
│ Quick Reply Buttons:                       │
│ [  Yes  ] [  No  ]                         │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ INSERT INTO csat_survey                    │
│ {                                          │
│   session_id: "sess_123",                  │
│   sent_at: NOW(),                          │
│   attempt_count: 1,                        │
│   status: 'pending'                        │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
    [WAIT FOR CUSTOMER RESPONSE]
        │
        ▼
┌────────────────────────────────────────────┐
│ Customer Response Received                 │
│ (Handled by MessageReceiverEvent)         │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Check if CSAT Survey Pending               │
│ for this session                           │
└────────────────────────────────────────────┘
        │
   ┌────┴────┐
   │         │
  Yes       No
   │         │
   │         ▼
   │    [NORMAL PROCESSING]
   │
   ▼
┌────────────────────────────────────────────┐
│ Parse Response                             │
│ $response = strtolower(trim($body))        │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Match Against Keywords                     │
│ YES: "yes","y","Yes","YES"  │
│ NO:  "no","n","NO","No"     │
└────────────────────────────────────────────┘
        │
   ┌────┴─────────┐
   │              │
 Match        No Match
   │              │
   ▼              ▼
┌─────────┐  ┌──────────────┐
│VALID    │  │INVALID       │
│RESPONSE │  │RESPONSE      │
└─────────┘  └──────────────┘
   │              │
   │              ▼
   │         ┌──────────────┐
   │         │Get attempt_  │
   │         │count from DB │
   │         └──────────────┘
   │              │
   │         ┌────┴────┐
   │         │         │
   │       < 3      >= 3
   │         │         │
   │         ▼         ▼
   │    ┌────────┐ ┌────────┐
   │    │Incr    │ │Stop    │
   │    │attempt │ │Asking  │
   │    │        │ │        │
   │    │Resend  │ │UPDATE  │
   │    │CSAT    │ │status= │
   │    │Quest   │ │'no_    │
   │    │        │ │response│
   │    └────────┘ └────────┘
   │
   ▼
┌────────────────────────────────────────────┐
│ UPDATE csat_survey SET                     │
│   response = 'yes/no',                     │
│   responded_at = NOW(),                    │
│   status = 'completed'                     │
│ WHERE session_id = ?                       │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Send Thank You Message                     │
│ "Thank you for your feedback."             │
└────────────────────────────────────────────┘
        │
        ▼
     [END CSAT FLOW]
```

### 4.3. Report Generation Flow

```
┌──────────────────────────────────────────────────────────────────┐
│              REPORT GENERATION AFTER CLOSURE                     │
└──────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────┐
│ Session Successfully Closed                │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Determine Channel Type                     │
└────────────────────────────────────────────┘
        │
   ┌────┴─────────────────┐
   │                      │
   ▼                      ▼
┌──────────────┐    ┌────────────────┐
│ Widget/FB    │    │ Facebook       │
│ Messenger    │    │ Comment        │
└──────────────┘    └────────────────┘
   │                      │
   ▼                      │
┌──────────────────────────────────────────┐
│ Generate report_master                   │
├──────────────────────────────────────────┤
│ INSERT INTO report_master                │
│ {                                        │
│   session_id: "sess_123",                │
│   channel: "widget|messenger",           │
│   customer_id: "cust_456",               │
│   agent_id: "agent_789",                 │
│   started_at: "2025-01-15 10:00:00",     │
│   closed_at: "2025-01-15 10:45:00",      │
│   duration_seconds: 2700,                │
│   message_count: 15,                     │
│   agent_message_count: 8,                │
│   customer_message_count: 7,             │
│   first_response_time: 45,               │
│   avg_response_time: 120,                │
│   resolution_status: "resolved",         │
│   csat_score: null,  // Updated later    │
│   tags: ["support","technical"],         │
│   created_at: NOW()                      │
│ }                                        │
└──────────────────────────────────────────┘
   │                      │
   │                      ▼
   │        ┌───────────────────────────────┐
   │        │ Generate customer_social_     │
   │        │ activity_reply_report         │
   │        ├───────────────────────────────┤
   │        │ INSERT INTO customer_social_  │
   │        │ activity_reply_report         │
   │        │ {                             │
   │        │   session_id: "sess_123",     │
   │        │   post_id: "fb_post_456",     │
   │        │   comment_id: "fb_comm_789",  │
   │        │   customer_id: "cust_456",    │
   │        │   page_id: "page_123",        │
   │        │   comment_text: "...",        │
   │        │   reply_count: 3,             │
   │        │   agent_id: "agent_789",      │
   │        │   first_reply_at: "...",      │
   │        │   last_reply_at: "...",       │
   │        │   response_time_seconds: 60,  │
   │        │   resolved: true,             │
   │        │   sentiment: "positive",      │
   │        │   created_at: NOW()           │
   │        │ }                             │
   │        └───────────────────────────────┘
   │                      │
   └──────────┬───────────┘
              ▼
┌────────────────────────────────────────────┐
│ Trigger Report Events                      │
│ • ReportGeneratedEvent::dispatch()         │
│ • Update Analytics Dashboard               │
└────────────────────────────────────────────┘
              │
              ▼
┌────────────────────────────────────────────┐
│ Update Aggregate Metrics (Redis Cache)     │
│ • Daily conversation count                 │
│ • Average resolution time                  │
│ • Agent performance metrics                │
└────────────────────────────────────────────┘
```

### 4.4. Complete Integrated Flow

```
┌──────────────────────────────────────────────────────────────────┐
│     COMPLETE MESSAGE CLOSED FLOW (All Steps)                     │
└──────────────────────────────────────────────────────────────────┘

        Agent Closes Conversation
                    │
                    ▼
        ┌────────────────────────────┐
        │ 1. UPDATE sessions         │
        │    status = 'closed'       │
        └────────────────────────────┘
                    │
                    ▼
        ┌────────────────────────────┐
        │ 2. UPDATE cases            │
        │    status = 'closed'       │
        └────────────────────────────┘
                    │
                    ▼
        ┌────────────────────────────┐
        │ 3. DELETE FROM queues      │
        └────────────────────────────┘
                    │
                    ▼
        ┌────────────────────────────┐
        │ 4. Create System Message   │
        └────────────────────────────┘
                    │
                    ▼
            Channel Type?
                    │
        ┌───────────┼────────────┐
        │           │            │
        ▼           ▼            ▼
    ┌──────┐   ┌──────┐    ┌──────┐
    │Widget│   │  FB  │    │  FB  │
    │      │   │Msgr  │    │Comm  │
    └──────┘   └──────┘    └──────┘
        │           │            │
        └─────┬─────┘            │
              │                  │
              ▼                  │
    ┌──────────────────┐         │
    │ 5a. Send CSAT    │         │
    │     Survey       │         │
    │     "Are you     │         │
    │     satisfied?"  │         │
    │     [Yes] [No]   │         │
    └──────────────────┘         │
              │                  │
              ▼                  │
    ┌──────────────────┐         │
    │ 5b. Track Survey │         │
    │     csat_survey  │         │
    │     attempt_     │         │
    │     count        │         │
    └──────────────────┘         │
              │                  │
              ▼                  │
    ┌──────────────────┐         │
    │ 5c. Listen for   │         │
    │     Response     │         │
    │     (up to 3     │         │
    │      times)      │         │
    └──────────────────┘         │
              │                  │
              └────────┬─────────┘
                       ▼
            ┌────────────────────┐
            │ 6. Generate Reports│
            └────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        │                             │
        ▼                             ▼
┌──────────────┐            ┌──────────────────┐
│INSERT report │            │INSERT customer_  │
│_master       │            │social_activity_  │
│              │            │reply_report      │
│(Widget/Msgr) │            │(FB Comment)      │
└──────────────┘            └──────────────────┘
        │                             │
        └──────────────┬──────────────┘
                       ▼
            ┌────────────────────┐
            │ 7. Trigger Events  │
            │    • ReportGen     │
            │    • Analytics     │
            └────────────────────┘
                       │
                       ▼
            ┌────────────────────┐
            │ 8. Update Cache    │
            │    & Metrics       │
            └────────────────────┘
                       │
                       ▼
            ┌────────────────────┐
            │ 9. Return Success  │
            └────────────────────┘
```

---

## 5. Bulk Reply Flow (message_bulk_reply)

```
┌──────────────────────────────────────────────────────────────────┐
│                    BULK REPLY FLOW                               │
│                    (message_bulk_reply)                          │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│ Agent           │
│ Selects Multiple│
│ Conversations   │
│ Sends Same Reply│
└─────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ POST /api/v1/agent/messages                │
│ ?action=message_bulk_reply                 │
│                                            │
│ Payload:                                   │
│ {                                          │
│   "reply_ids": [67288, 67289, 67290],      │
│   "message": {                             │
│     "message_type": "message",             │
│     "body": "Thank you for contacting..."  │
│   }                                        │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessagesController                         │
│ • Detect action = message_bulk_reply       │
│ • Agent mode required                      │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Extract Payload                            │
│ $payload = post()                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessageBulkReply::dispatch($payload)       │
│ • Queue for background processing          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Return Immediate Response                  │
│ {                                          │
│   "success": true,                         │
│   "message": "Processing reply for         │
│               3 messages"                  │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
        [BACKGROUND QUEUE]
        │
        ▼
┌────────────────────────────────────────────┐
│ MessageBulkReply Job Executes              │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Iterate Over reply_ids                     │
└────────────────────────────────────────────┘
        │
        ▼
    For Each ID:
        │
        ▼
┌────────────────────────────────────────────┐
│ 1. Load Parent Message                     │
│ 2. Create Session Context                  │
│ 3. Validate Agent Permission               │
│ 4. Build Reply Message                     │
│ 5. Send via Channel                        │
│ 6. Store in Database                       │
│ 7. Update Session                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Handle Errors                              │
│ • Log failed sends                         │
│ • Retry with backoff                       │
│ • Track success/failure count              │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Send Completion Notification               │
│ • Notify agent via WebSocket               │
│ • Show success/failure summary             │
│   "Sent: 2/3, Failed: 1/3"                 │
└────────────────────────────────────────────┘
```

---

## 6. Bulk Close Flow (message_bulk_close)

```
┌──────────────────────────────────────────────────────────────────┐
│                    BULK CLOSE FLOW                               │
│                    (message_bulk_close)                          │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│ Agent           │
│ Selects Multiple│
│ Conversations   │
│ Closes All      │
└─────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ POST /api/v1/agent/messages                │
│ ?action=message_bulk_close                 │
│                                            │
│ Payload:                                   │
│ {                                          │
│   "session_ids": [                         │
│     "sess_123",                            │
│     "sess_456",                            │
│     "sess_789"                             │
│   ],                                       │
│   "reason": "resolved",                    │
│   "notes": "Issue resolved"                │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessagesController                         │
│ • Detect action = message_bulk_close       │
│ • Agent mode required                      │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Extract Payload                            │
│ $payload = post()                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessageBulkClose::dispatch($payload)       │
│ • Queue for background processing          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Return Immediate Response                  │
│ { "success": true }                        │
└────────────────────────────────────────────┘
        │
        ▼
        [BACKGROUND QUEUE]
        │
        ▼
┌────────────────────────────────────────────┐
│ MessageBulkClose Job Executes              │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Iterate Over session_ids                   │
└────────────────────────────────────────────┘
        │
        ▼
    For Each Session:
        │
        ▼
┌────────────────────────────────────────────┐
│ 1. Load Session                            │
│ 2. Validate Agent Has Permission           │
│ 3. Check Session Status                    │
└────────────────────────────────────────────┘
        │
        ▼
    ┌───────┴────────┐
    │                │
 Valid           Invalid
    │                │
    ▼                ▼
┌─────────┐      ┌─────────┐
│ Close   │      │ Skip &  │
│ Session │      │ Log     │
└─────────┘      │ Error   │
    │            └─────────┘
    ▼
┌────────────────────────────────────────────┐
│ Update Session                             │
│ • status = 'closed'                        │
│ • closed_at = NOW()                        │
│ • closed_by = agent_id                     │
│ • close_reason = reason                    │
│ • close_notes = notes                      │
└────────────────────────────────────────────┘
    │
    ▼
┌────────────────────────────────────────────┐
│ Create System Message                      │
│ "Conversation closed by Agent"             │
└────────────────────────────────────────────┘
    │
    ▼
┌────────────────────────────────────────────┐
│ Trigger Events                             │
│ • SessionClosedEvent                       │
│ • Update Analytics                         │
│ • Notify Customer (optional)               │
└────────────────────────────────────────────┘
    │
    ▼
┌────────────────────────────────────────────┐
│ Release Agent Assignment                   │
│ • Free up agent capacity                   │
│ • Update agent metrics                     │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Send Completion Notification               │
│ • Notify agent via WebSocket               │
│ • Show closure summary                     │
└────────────────────────────────────────────┘
```

---

## 7. Video Call Request Flow (video_call_request)

```
┌──────────────────────────────────────────────────────────────────┐
│                 VIDEO CALL REQUEST FLOW                          │
│                 (video_call_request)                             │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│ Agent/Customer  │
│ Initiates Video │
│ Call Request    │
└─────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ POST /api/v1/messages                      │
│ ?action=video_call_request                 │
│                                            │
│ Payload:                                   │
│ {                                          │
│   "session_id": "sess_123",                │
│   "initiator": "agent|customer",           │
│   "call_type": "audio|video",              │
│   "message": "Request for video support"   │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessagesController                         │
│ • Validate action                          │
│ • Check permissions                        │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ app('video_call_request')                  │
│ ->run($messageContainer)                   │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ VideoCallRequest Action                    │
│ • Load session                             │
│ • Validate participants                    │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Check Session Status                       │
└────────────────────────────────────────────┘
        │
   ┌────┴────┐
   │         │
 Active   Closed
   │         │
   │         ▼
   │    ┌──────────┐
   │    │ Error    │
   │    │ Cannot   │
   │    │ request  │
   │    │ call on  │
   │    │ closed   │
   │    │ session  │
   │    └──────────┘
   │
   ▼
┌────────────────────────────────────────────┐
│ Generate Video Call Token                 │
│ • Create unique call_id                    │
│ • Generate secure tokens for participants  │
│ • Set expiry time (30 minutes)             │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Store Call Request                         │
│ INSERT INTO video_calls                    │
│ • call_id                                  │
│ • session_id                               │
│ • initiator                                │
│ • status = 'pending'                       │
│ • created_at                               │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Send Notification to Recipient             │
└────────────────────────────────────────────┘
        │
   ┌────┴────┐
   │         │
 Agent   Customer
   │         │
   ▼         ▼
┌────────┐ ┌──────────┐
│WebSock │ │ Widget   │
│et to   │ │ Notif +  │
│Agent   │ │ WebSocket│
│Dashboard│ │          │
└────────┘ └──────────┘
   │         │
   └────┬────┘
        ▼
┌────────────────────────────────────────────┐
│ Create System Message                      │
│ "Video call requested"                     │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Return Response                            │
│ {                                          │
│   "success": true,                         │
│   "call_id": "call_abc123",                │
│   "status": "pending",                     │
│   "tokens": {                              │
│     "agent_token": "...",                  │
│     "customer_token": "..."                │
│   },                                       │
│   "expires_at": "2025-01-15T10:30:00Z"     │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
    [WAIT FOR ACCEPTANCE]
        │
        ▼
    Timeout (30 min)?
        │
   ┌────┴────┐
   │         │
  Yes       No
   │         │
   ▼         ▼
┌──────┐  [CALL ACCEPTED]
│Auto  │  → video_call_start
│Close │
│Call  │
└──────┘
```

---

## 8. Video Call Start Flow (video_call_start)

```
┌──────────────────────────────────────────────────────────────────┐
│                  VIDEO CALL START FLOW                           │
│                  (video_call_start)                              │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│ Recipient       │
│ Accepts Call    │
│ Request         │
└─────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ POST /api/v1/messages                      │
│ ?action=video_call_start                   │
│                                            │
│ Payload:                                   │
│ {                                          │
│   "call_id": "call_abc123",                │
│   "participant": "agent|customer"          │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessagesController                         │
│ • Validate action                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ app('video_call_start')                    │
│ ->run($messageContainer)                   │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ VideoCallStart Action                      │
│ • Load call by call_id                     │
│ • Validate tokens                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Check Call Status                          │
└────────────────────────────────────────────┘
        │
   ┌────┴──────────┐
   │               │
 Pending      Expired/Ended
   │               │
   │               ▼
   │          ┌──────────┐
   │          │ Error    │
   │          │ Call not │
   │          │ available│
   │          └──────────┘
   │
   ▼
┌────────────────────────────────────────────┐
│ Update Call Status                         │
│ • status = 'active'                        │
│ • started_at = NOW()                       │
│ • participants_joined = [...]              │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Initialize Video Service                   │
│ (e.g., Twilio, Agora, WebRTC)              │
│ • Create room                              │
│ • Configure media settings                 │
│ • Enable recording (if configured)         │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Notify Other Participant                   │
│ "Call started, joining..."                 │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Create System Message                      │
│ "Video call started"                       │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Return Connection Details                  │
│ {                                          │
│   "success": true,                         │
│   "call_id": "call_abc123",                │
│   "status": "active",                      │
│   "room_url": "https://...",               │
│   "ice_servers": [...],                    │
│   "media_constraints": {...}               │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
    [CALL IN PROGRESS]
        │
        ▼
┌────────────────────────────────────────────┐
│ Monitor Call Status                        │
│ • Track duration                           │
│ • Monitor quality                          │
│ • Handle disconnections                    │
└────────────────────────────────────────────┘
        │
        ▼
    Call Ends
        │
        ▼
┌────────────────────────────────────────────┐
│ Update Call Record                         │
│ • status = 'ended'                         │
│ • ended_at = NOW()                         │
│ • duration = calculate                     │
│ • recording_url (if recorded)              │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Create System Message                      │
│ "Video call ended (Duration: 15:30)"       │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Trigger Analytics Event                    │
│ • VideoCallCompletedEvent                  │
│ • Update agent metrics                     │
└────────────────────────────────────────────┘
```

---

## 9. Post Details Flow (post_details)

```
┌──────────────────────────────────────────────────────────────────┐
│                   POST DETAILS FLOW                              │
│                   (post_details)                                 │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│ Agent           │
│ Views Facebook  │
│ Post Details    │
└─────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ POST /api/v1/agent/messages                │
│ ?action=post_details                       │
│                                            │
│ Payload:                                   │
│ {                                          │
│   "message": {                             │
│     "id": 67920,                           │
│     "is_reply": 67920,                     │
│     "postDetails": true                    │
│   }                                        │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessagesController                         │
│ • Agent mode required                      │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ app('post_details')                        │
│ ->run($messageContainer)                   │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ PostDetails Action                         │
│ • Extract message.id (67920)               │
│ • Load message from database               │
│ • Check postDetails flag                   │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Check Cache                                │
└────────────────────────────────────────────┘
        │
   ┌────┴────┐
   │         │
 Found    Not Found
   │         │
   │         ▼
   │    ┌────────────────────────────┐
   │    │ Fetch from Facebook API    │
   │    │ GET /v18.0/{post_id}       │
   │    │ ?fields=message,created_   │
   │    │ time,reactions,comments    │
   │    └────────────────────────────┘
   │         │
   │         ▼
   │    ┌────────────────────────────┐
   │    │ Store in Cache             │
   │    │ TTL: 5 minutes             │
   │    └────────────────────────────┘
   │         │
   └────┬────┘
        ▼
┌────────────────────────────────────────────┐
│ Load Post & Comments                       │
│ SELECT * FROM messages                     │
│ WHERE id = ? OR parent_id = ?              │
│ • Include reply threads                    │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Load Reactions (if requested)              │
│ • Aggregate reaction counts                │
│ • Group by reaction type                   │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Build Response                             │
│ {                                          │
│   "post": {                                │
│     "id": "fb_post_123",                   │
│     "message": "...",                      │
│     "created_time": "...",                 │
│     "author": {...},                       │
│     "permalink_url": "..."                 │
│   },                                       │
│   "comments": [                            │
│     {                                      │
│       "id": "...",                         │
│       "message": "...",                    │
│       "from": {...},                       │
│       "created_time": "...",               │
│       "replies": [...]                     │
│     }                                      │
│   ],                                       │
│   "reactions": {                           │
│     "like": 45,                            │
│     "love": 12,                            │
│     "total": 57                            │
│   }                                        │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Return Response                            │
└────────────────────────────────────────────┘
```

---

## 10. Comment Details Flow (comment_details)

```
┌──────────────────────────────────────────────────────────────────┐
│                 COMMENT DETAILS FLOW                             │
│                 (comment_details)                                │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│ Agent           │
│ Views Comment   │
│ Thread Details  │
└─────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ POST /api/v1/agent/messages                │
│ ?action=comment_details                    │
│                                            │
│ Payload:                                   │
│ {
  "message": {
    "id": 67920,
    "is_reply": 67920,
    "postDetails": true
  }
}                                         │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessagesController                         │
│ • Agent mode required                      │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ app('comment_details')                     │
│ ->run($messageContainer)                   │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ CommentDetails Action                      │
│ • Extract comment_id                       │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Load Comment from Database                 │
│ SELECT * FROM messages                     │
│ WHERE id = ?                      │
│                   │
└────────────────────────────────────────────┘
        │
        ▼
    ┌───────┴────────┐
    │                │
  Found          Not Found
    │                │
    │                ▼
    │           ┌──────────┐
    │           │ Fetch    │
    │           │ from FB  │
    │           │ API      │
    │           └──────────┘
    │                │
    └────────┬───────┘
             ▼
┌────────────────────────────────────────────┐
│ Load Parent Post (if requested)            │
│ • Get post_id from comment                 │
│ • Load post details                        │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Load Replies (if requested)                │
│ SELECT * FROM messages                     │
│ WHERE parent_id = ?                        │
│ AND type = 'fb_comment'                    │
│ ORDER BY created_at ASC                    │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Load Customer Context                      │
│ • Get customer profile                     │
│ • Get conversation history                 │
│ • Get previous interactions                │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Build Response                             │
│ {                                          │
│   "comment": {                             │
│     "id": "fb_comment_456",                │
│     "message": "...",                      │
│     "from": {...},                         │
│     "created_time": "...",                 │
│     "parent_comment_id": "..."             │
│   },                                       │
│   "parent_post": {                         │
│     "id": "...",                           │
│     "message": "...",                      │
│     "permalink_url": "..."                 │
│   },                                       │
│   "replies": [                             │
│     {                                      │
│       "id": "...",                         │
│       "message": "...",                    │
│       "from": {...}                        │
│     }                                      │
│   ],                                       │
│   "customer_context": {                    │
│     "previous_comments": 5,                │
│     "total_interactions": 12,              │
│     "sentiment": "positive"                │
│   }                                        │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Return Response                            │
└────────────────────────────────────────────┘
```

---

## 11. System Message Closed (system_message_closed)

```
┌──────────────────────────────────────────────────────────────────┐
│              SYSTEM MESSAGE CLOSED FLOW                          │
│              (system_message_closed)                             │
└──────────────────────────────────────────────────────────────────┘

┌─────────────────┐
│ System/Automated│
│ Process         │
│ Closes Session  │
└─────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ POST /api/v1/messages                      │
│ ?action=system_message_closed              │
│                                            │
│ Payload:                                   │
│ {                                          │
│   "session_id": "sess_123",                │
│   "reason": "timeout|auto_resolved|...",   │
│   "triggered_by": "cron|rule|event"        │
│ }                                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ MessagesController                         │
│ • Validate system action                   │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ app('system_message_closed')               │
│ ->run($messageContainer)                   │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ SystemMessageClosed Action                 │
│ • Load session                             │
│ • Validate closure eligibility             │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Check Session Status                       │
└────────────────────────────────────────────┘
        │
   ┌────┴──────┐
   │           │
 Active    Already Closed
   │           │
   │           ▼
   │      ┌──────────┐
   │      │ Skip     │
   │      │ No action│
   │      └──────────┘
   │
   ▼
┌────────────────────────────────────────────┐
│ Determine Closure Reason                   │
└────────────────────────────────────────────┘
        │
   ┌────┴───────────────┬──────────────┐
   │                    │              │
   ▼                    ▼              ▼
┌────────┐        ┌──────────┐   ┌─────────┐
│Timeout │        │ Auto     │   │ Business│
│24h no  │        │ Resolved │   │ Rule    │
│activity│        │ by bot   │   │ Trigger │
└────────┘        └──────────┘   └─────────┘
   │                    │              │
   └────────────┬───────┴──────────────┘
                ▼
┌────────────────────────────────────────────┐
│ Update Session                             │
│ • status = 'closed'                        │
│ • closed_at = NOW()                        │
│ • closed_by = 'system'                     │
│ • close_reason = reason                    │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Create System Message                      │
│ "Conversation automatically closed         │
│  due to [reason]"                          │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Notify Customer (if configured)            │
│ • Send closure notification                │
│ • Include reopening instructions           │
│ • Add satisfaction survey link             │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Release Agent Assignment                   │
│ • Free agent capacity                      │
│ • Update agent metrics                     │
│ • Move to agent's closed queue             │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Trigger Analytics Events                   │
│ • SessionAutoClosedEvent                   │
│ • Update closure statistics                │
│ • Track closure reasons                    │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Archive Conversation (if configured)       │
│ • Move to archive storage                  │
│ • Compress attachments                     │
│ • Update indexes                           │
└────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────┐
│ Return Response                            │
│ {                                          │
│   "success": true,                         │
│   "session_id": "sess_123",                │
│   "closed_at": "...",                      │
│   "reason": "..."                          │
│ }                                          │
└────────────────────────────────────────────┘
```

---

## 12. MessagesController Core Logic

```
┌──────────────────────────────────────────────────────────────────┐
│          MessagesController::__invoke() Core Logic               │
└──────────────────────────────────────────────────────────────────┘

        ┌────────────────────────┐
        │ Request Received       │
        └────────────────────────┘
                    │
                    ▼
        ┌────────────────────────┐
        │ Extract action param   │
        │ $action = post('action')│
        └────────────────────────┘
                    │
                    ▼
        ┌────────────────────────┐
        │ Check URL segment      │
        │ $isAgentMode =         │
        │ segment(3) == 'agent'  │
        └────────────────────────┘
                    │
                    ▼
        ┌────────────────────────┐
        │ Validate action        │
        │ isInvalidAction()?     │
        └────────────────────────┘
                    │
            ┌───────┴────────┐
            │                │
          Valid          Invalid
            │                │
            │                ▼
            │        ┌──────────────┐
            │        │ throwJson    │
            │        │ Error        │
            │        │ "Invalid     │
            │        │  action"     │
            │        └──────────────┘
            │
            ▼
        Mode Check?
            │
    ┌───────┴────────┐
    │                │
 Agent           Customer
 Mode              Mode
    │                │
    ▼                ▼
┌─────────┐    ┌──────────┐
│ Check   │    │ Check    │
│ Agent   │    │ Customer │
│ Login   │    │ Session  │
└─────────┘    └──────────┘
    │                │
    ▼                │
┌─────────┐          │
│ Logged? │          │
└─────────┘          │
    │                │
┌───┴───┐            │
│       │            │
Yes    No            │
│       │            │
│       ▼            │
│  ┌────────┐        │
│  │ Error  │        │
│  │ "Bruhh"│        │
│  └────────┘        │
│                    │
└─────────┬──────────┘
          ▼
┌────────────────────────────┐
│ Create                     │
│ MessagingRequestHelper     │
│ try {                      │
│   $messageContainer =      │
│   new MessagingRequest     │
│   Helper($request)         │
│ } catch {                  │
│   return error             │
│ }                          │
└────────────────────────────┘
          │
          ▼
   ┌──────────────┐
   │ Agent Mode?  │
   └──────────────┘
          │
    ┌─────┴──────┐
    │            │
   Yes          No
    │            │
    ▼            └─────────┐
┌─────────────────────┐    │
│ AGENT MODE LOGIC    │    │
├─────────────────────┤    │
│ 1. hasBulkReplyId()?│    │
│    → firstReplyId() │    │
│                     │    │
│ 2. Get Parent Msg   │    │
│    $parent =        │    │
│    parentMessage()  │    │
│                     │    │
│ 3. Check parent     │    │
│    exists?          │    │
│    No → Error 403   │    │
│                     │    │
│ 4. createSession    │    │
│    FromMessage()    │    │
│                     │    │
│ 5. Check session_   │    │
│    path?            │    │
│    → setSessionPath │    │
│                     │    │
│ 6. validateSession  │    │
│    ForAgentReply()  │    │
└─────────────────────┘    │
          │                │
          └────────┬───────┘
                   ▼
        ┌──────────────────┐
        │ Check Action     │
        │ Type             │
        └──────────────────┘
                   │
      ┌────────────┼────────────┐
      │            │            │
      ▼            ▼            ▼
┌──────────┐ ┌──────────┐ ┌──────────┐
│message_  │ │message_  │ │ Other    │
│bulk_close│ │bulk_reply│ │ Actions  │
└──────────┘ └──────────┘ └──────────┘
      │            │            │
      ▼            ▼            │
┌──────────┐ ┌──────────┐      │
│Message   │ │Message   │      │
│BulkClose │ │BulkReply │      │
│::dispatch│ │::dispatch│      │
└──────────┘ └──────────┘      │
      │            │            │
      ▼            ▼            │
┌──────────┐ ┌──────────┐      │
│Return    │ │Return    │      │
│success   │ │success   │      │
└──────────┘ └──────────┘      │
                                │
                                ▼
                    ┌──────────────────────┐
                    │ app($action)         │
                    │ ->run(               │
                    │  $messageContainer)  │
                    └──────────────────────┘
                                │
                                ▼
                    ┌──────────────────────┐
                    │ Action Class Executes│
                    │ Specific Logic       │
                    └──────────────────────┘
                                │
                                ▼
                    ┌──────────────────────┐
                    │ Return Response      │
                    └──────────────────────┘
```

---

**Valid Actions List:**
```
┌────────────────────────────────────────────────────────────────┐
│ VALID ACTIONS IN CONTROLLER                                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│ Agent Reply Operations:                                        │
│  • message_reply         - Send agent reply to customer        │
│  • message_example       - Send example message                │
│                                                                │
│ Bulk Operations:                                               │
│  • message_bulk_reply    - Reply to multiple conversations     │
│  • message_bulk_close    - Close multiple conversations        │
│                                                                │
│ Query Operations:                                              │
│  • message_history       - Get conversation history            │
│  • message_closed        - Get closed conversations            │
│                                                                │
│ Video Call Operations:                                         │
│  • video_call_request    - Request video call                  │
│  • video_call_start      - Start video call                    │
│                                                                │
│ Social Media Operations:                                       │
│  • post_details          - Get Facebook post details           │
│  • comment_details       - Get comment thread details          │
│  • wall_post_details     - Get wall post details               │
│  • social_feed_tickets   - Social media ticket management      │
│                                                                │
│ System Operations:                                             │
│  • system_message_closed - System-initiated closure            │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

**Document Version:** 1.0  
**Created:** October 27, 2025  
**Style:** ASCII Box Diagrams

