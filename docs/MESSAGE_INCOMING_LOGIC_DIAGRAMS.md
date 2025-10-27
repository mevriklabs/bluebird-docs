# Incoming Message Logic Diagrams
**Mevrik Channels Platform**  
**Created:** October 27, 2025  
**Last Updated:** October 27, 2025  
**Document Type:** Message Ingestion Flow Documentation

---

## Table of Contents

1. [Complete Message Ingestion Overview](#1-complete-message-ingestion-overview)
2. [Widget Message Flow (Detailed)](#2-widget-message-flow-detailed)
3. [Facebook Webhook Flow (Detailed)](#3-facebook-webhook-flow-detailed)
4. [Email Processing Flow (Detailed)](#4-email-processing-flow-detailed)
5. [MessagingRequestHelper Logic](#5-messagingrequesthelper-logic)
6. [Event Dispatch Logic](#6-event-dispatch-logic)
7. [Session State Management](#7-session-state-management)
8. [HandleMessageReadyEvent Processing](#8-handlemessagereadyevent-processing)
9. [Message Type Detection](#9-message-type-detection)
10. [Complete System Architecture (ASCII)](#10-complete-system-architecture-ascii)

---

## 1. Complete Message Ingestion Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          MESSAGE SOURCES                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│   Web Widget   │   Facebook Messenger   │   Facebook Comment   │   Email    │
└─────────────────────────────────────────────────────────────────────────────┘
        │                      │                      │                  │
        │                      │                      │                  │
        ▼                      ▼                      ▼                  ▼
┌──────────────┐      ┌──────────────────┐      ┌──────────────────────────┐
│  Widget API  │      │ Facebook Webhook │      │    Email Processor       │
│              │      │                  │      │                          │
│ /api/v1/     │      │ /webhook/        │      │ /api/v1/email/           │
│ messages     │      │ facebook         │      │ process_incoming_email   │
│              │      │                  │      │                          │
│ action=      │      │ • Messenger      │      │ • email_id: 5816         │
│ message_post │      │ • Comments       │      │ • dry_run: 0/1           │
│              │      │                  │      │ • sync: 0/1              │
└──────────────┘      └──────────────────┘      └──────────────────────────┘
        │                      │                              │
        │                      │                              │
        └──────────────────────┴──────────────────────────────┘
                               │
                               ▼
        ┌──────────────────────────────────────────────────┐
        │     MessagingRequestHelper (MRH)                 │
        │     • Normalize message format                   │
        │     • Validate payload                           │
        │     • Session management                         │
        └──────────────────────────────────────────────────┘
                               │
                               ▼
        ┌──────────────────────────────────────────────────┐
        │          storeMessage()                          │
        │          Database Storage                        │
        │          Return: $model (Message Model)          │
        └──────────────────────────────────────────────────┘
                               │
                               ▼
                     ┌─────────────────┐
                     │   Environment   │
                     │     Check?      │
                     └─────────────────┘
                       │             │
            Local ◄────┘             └────► Production
               │                              │
               ▼                              ▼
    ┌──────────────────────┐      ┌──────────────────────────┐
    │ HandleMessageReady   │      │ MessageReadyEvent        │
    │ Event::run()         │      │ ::dispatch() [Queue]     │
    │                      │      │         +                │
    │ [Synchronous]        │      │ HandleMessageReady       │
    │                      │      │ Event::run() [Sync]      │
    └──────────────────────┘      └──────────────────────────┘
               │                              │
               └──────────────┬───────────────┘
                              ▼
                   ┌──────────────────────┐
                   │  Check Meta State    │
                   └──────────────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
    ┌──────────┐        ┌──────────┐       ┌──────────┐
    │ Has      │        │ Has      │       │ Continue │
    │ quick_   │        │ postback?│       │ Normal   │
    │ reply?   │        │          │       │ Flow     │
    └──────────┘        └──────────┘       └──────────┘
          │                   │                   │
          ▼                   ▼                   │
    ┌──────────┐        ┌──────────┐             │
    │ Message  │        │ Message  │             │
    │ Quick    │        │ Postback │             │
    │ Reply    │        │ Event    │             │
    │ Event    │        │ dispatch │             │
    │ dispatch │        └──────────┘             │
    └──────────┘                                 │
          │                   │                  │
          └───────────────────┴──────────────────┘
                              ▼
                    ┌──────────────────┐
                    │  Processing      │
                    │  Complete        │
                    └──────────────────┘
```

---

## 2. Widget Message Flow (Detailed)

```
┌─────────────────────────────────────────────────────────────────┐
│                    WIDGET MESSAGE FLOW                          │
└─────────────────────────────────────────────────────────────────┘

        ┌──────────────────┐
        │   Customer       │
        │   Types Message  │
        └──────────────────┘
                │
                ▼
        ┌──────────────────┐
        │   STEP 1:        │
        │   Perform KYC    │
        │   Get Token      │
        └──────────────────┘
                │
                ▼
        ┌──────────────────────────────────────────────┐
        │   STEP 2: POST to API                        │
        │   /api/v1/messages?action=message_post       │
        │                                              │
        │   Payload:                                   │
        │   {                                          │
        │     "message": {                             │
        │       "type": "message",                     │
        │       "body": "Get started"                  │
        │     }                                        │
        │   }                                          │
        │                                              │
        │   Headers:                                   │
        │   Authorization: Bearer {token}              │
        └──────────────────────────────────────────────┘
                │
                ▼
        ┌──────────────────┐
        │   Validate       │
        │   Token          │
        └──────────────────┘
                │
       ┌────────┴────────┐
       │                 │
    Invalid           Valid
       │                 │
       ▼                 ▼
┌─────────────┐   ┌──────────────────────┐
│ Return 401  │   │ Initialize           │
│ Unauthorized│   │ MessagingRequest     │
└─────────────┘   │ Helper               │
                  └──────────────────────┘
                           │
                           ▼
                  ┌──────────────────────┐
                  │ Normalize Message    │
                  │ Structure            │
                  └──────────────────────┘
                           │
                           ▼
                  ┌──────────────────────┐
                  │ Get/Create           │
                  │ Session              │
                  └──────────────────────┘
                           │
                           ▼
                  ┌──────────────────────┐
                  │ storeMessage()       │
                  │ to Database          │
                  └──────────────────────┘
                           │
                           ▼
                  ┌──────────────────────┐
                  │ Dispatch Events      │
                  │ (based on env)       │
                  └──────────────────────┘
                           │
                           ▼
                  ┌──────────────────────┐
                  │ Return 200 OK        │
                  │ Message Confirmation │
                  └──────────────────────┘
```

---

## 3. Facebook Webhook Flow (Detailed)

```
┌──────────────────────────────────────────────────────────────────────┐
│                   FACEBOOK WEBHOOK FLOW                              │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│   Facebook Platform      │
│   • Messenger Messages   │
│   • Page Comments        │
└──────────────────────────┘
            │
            ▼
┌────────────────────────────────────────────┐
│   POST /webhook/facebook                   │
│                                            │
│   Headers:                                 │
│   X-Hub-Signature: sha1={signature}        │
└────────────────────────────────────────────┘
            │
            ▼
┌────────────────────────────────────────────┐
│   FacebookController                       │
│   • Verify webhook signature               │
│   • Route to appropriate handler           │
└────────────────────────────────────────────┘
            │
            ▼
┌────────────────────────────────────────────┐
│   IngestFacebookWebhookAction              │
│   • Parse payload                          │
│   • Detect message type                    │
└────────────────────────────────────────────┘
            │
            ▼
┌────────────────────────────────────────────┐
│   WebhookEvent::store_webhook($payload)    │
│   • Store raw payload for audit           │
│   • Return: $message_key                   │
└────────────────────────────────────────────┘
            │
            ▼
        Payload Type?
            │
    ┌───────┴────────┐
    │                │
    ▼                ▼
┌─────────┐    ┌──────────┐
│Messenger│    │ Comment  │
│ Message │    │          │
└─────────┘    └──────────┘
    │                │
    └────────┬───────┘
             ▼
┌────────────────────────────────────────────┐
│   CreateMessageFromWebhook                 │
│   • Extract sender/recipient               │
│   • Extract message content                │
│   • Detect quick_reply or postback         │
└────────────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────┐
│   new MessagingRequestHelper($message)     │
│   • Normalize Facebook format              │
└────────────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────┐
│   $message_body->storeMessage()            │
│   • Save to database                       │
│   • Return: $model                         │
└────────────────────────────────────────────┘
             │
             ▼
        Environment?
             │
    ┌────────┴────────┐
    │                 │
  Local          Production
    │                 │
    ▼                 ▼
┌─────────┐      ┌──────────────────────┐
│HandleMsg│      │MessageReadyEvent     │
│ReadyEvt │      │::dispatch() [Async]  │
│::run()  │      │        +             │
│         │      │HandleMsgReadyEvt     │
│[Sync]   │      │::run() [Sync]        │
└─────────┘      └──────────────────────┘
    │                 │
    └────────┬────────┘
             ▼
    Check Meta State
             │
    ┌────────┼────────┐
    │        │        │
    ▼        ▼        ▼
┌─────┐  ┌─────┐  ┌─────┐
│quick│  │post │  │none │
│reply│  │back │  │     │
└─────┘  └─────┘  └─────┘
    │        │        │
    ▼        ▼        │
┌─────┐  ┌─────┐     │
│QR   │  │PB   │     │
│Event│  │Event│     │
└─────┘  └─────┘     │
    │        │        │
    └────────┴────────┘
             ▼
    ┌────────────────┐
    │  Return 200 OK │
    │  to Facebook   │
    └────────────────┘
```

---

## 4. Email Processing Flow (Detailed)

```
┌──────────────────────────────────────────────────────────────────┐
│                   EMAIL PROCESSING FLOW                          │
└──────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────┐
│   Email Server                     │
│   Incoming Email Received          │
└────────────────────────────────────┘
                │
                ▼
┌────────────────────────────────────────────────────┐
│   POST /api/v1/email/process_incoming_email        │
│                                                    │
│   Payload:                                         │
│   {                                                │
│     "data": {                                      │
│       "email_id": 5816,                            │
│       "dry_run": 0,      // 0=process, 1=test     │
│       "sync": 1          // 0=async, 1=sync       │
│     }                                              │
│   }                                                │
└────────────────────────────────────────────────────┘
                │
                ▼
        ┌───────────────┐
        │ Email Parser  │
        └───────────────┘
                │
                ▼
┌───────────────────────────────────────┐
│   Fetch Email by email_id             │
│   • Extract headers (From, To)        │
│   • Parse body (HTML/Text)            │
│   • Extract attachments               │
└───────────────────────────────────────┘
                │
                ▼
           dry_run?
                │
        ┌───────┴────────┐
        │                │
      = 1              = 0
        │                │
        ▼                ▼
┌──────────────┐  ┌─────────────────────┐
│ Return       │  │ Email Processor     │
│ Parsed Data  │  │ • Validate sender   │
│ (No Storage) │  │ • Normalize data    │
└──────────────┘  └─────────────────────┘
                           │
                           ▼
                  ┌─────────────────────┐
                  │ Initialize          │
                  │ MessagingRequest    │
                  │ Helper with email   │
                  └─────────────────────┘
                           │
                           ▼
                  ┌─────────────────────┐
                  │ Convert email to    │
                  │ message format:     │
                  │ • subject → title   │
                  │ • body → message    │
                  │ • from → sender     │
                  └─────────────────────┘
                           │
                           ▼
                  ┌─────────────────────┐
                  │ Get/Create          │
                  │ Customer Session    │
                  └─────────────────────┘
                           │
                           ▼
                  ┌─────────────────────┐
                  │ storeMessage()      │
                  │ to Database         │
                  └─────────────────────┘
                           │
                           ▼
                       sync flag?
                           │
                  ┌────────┴────────┐
                  │                 │
                = 1               = 0
                  │                 │
                  ▼                 ▼
          ┌──────────────┐  ┌──────────────┐
          │HandleMessage │  │MessageReady  │
          │ReadyEvent    │  │Event         │
          │::run()       │  │::dispatch()  │
          │              │  │              │
          │[Sync - Wait] │  │[Async Queue] │
          └──────────────┘  └──────────────┘
                  │                 │
                  ▼                 ▼
          ┌──────────────┐  ┌──────────────┐
          │ Return 200   │  │ Return 202   │
          │ OK           │  │ Accepted     │
          │ Complete     │  │ Queued       │
          └──────────────┘  └──────────────┘
```

---

## 5. MessagingRequestHelper Logic

```
┌──────────────────────────────────────────────────────────────────┐
│           MessagingRequestHelper Core Logic                      │
└──────────────────────────────────────────────────────────────────┘

        ┌────────────────────────┐
        │ Initialize with        │
        │ raw message data       │
        └────────────────────────┘
                    │
                    ▼
        ┌────────────────────────┐
        │ Detect Channel Type    │
        └────────────────────────┘
                    │
        ┌───────────┼───────────┬───────────┐
        │           │           │           │
        ▼           ▼           ▼           ▼
    ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
    │Widget│   │ FB   │   │Email │   │Other │
    └──────┘   └──────┘   └──────┘   └──────┘
        │           │           │           │
        └───────────┴───────────┴───────────┘
                    │
                    ▼
        ┌────────────────────────────┐
        │ Normalize to Standard      │
        │ Internal Format            │
        │ • type                     │
        │ • body                     │
        │ • sender_id                │
        │ • recipient_id             │
        │ • timestamp                │
        │ • metadata                 │
        └────────────────────────────┘
                    │
                    ▼
        ┌────────────────────────────┐
        │ Validate Required Fields   │
        └────────────────────────────┘
                    │
            ┌───────┴────────┐
            │                │
          Valid          Invalid
            │                │
            │                ▼
            │        ┌──────────────┐
            │        │ Throw        │
            │        │ Validation   │
            │        │ Exception    │
            │        └──────────────┘
            │
            ▼
┌───────────────────────────┐
│ Get Customer by sender_id │
└───────────────────────────┘
            │
    ┌───────┴────────┐
    │                │
 Exists          Not Found
    │                │
    │                ▼
    │        ┌──────────────┐
    │        │ Create New   │
    │        │ Customer     │
    │        └──────────────┘
    │                │
    └────────┬───────┘
             ▼
┌────────────────────────────┐
│ Find Active Session        │
│ for Channel                │
└────────────────────────────┘
             │
     ┌───────┴────────┐
     │                │
  Exists           None
     │                │
     ▼                ▼
┌─────────┐    ┌─────────────┐
│Check    │    │ Create New  │
│Timeout  │    │ Session     │
└─────────┘    └─────────────┘
     │                │
  Not Expired    ┌────┘
     │           │
     ▼           ▼
┌─────────┐  ┌─────────┐
│Update   │  │Use New  │
│Session  │  │Session  │
└─────────┘  └─────────┘
     │           │
     └─────┬─────┘
           ▼
┌────────────────────────────┐
│ Build Insert Data          │
│ • message content          │
│ • session_id               │
│ • customer_id              │
│ • channel type             │
│ • metadata                 │
│ • timestamps               │
└────────────────────────────┘
           │
           ▼
┌────────────────────────────┐
│ INSERT INTO messages       │
└────────────────────────────┘
           │
           ▼
┌────────────────────────────┐
│ Retrieve Message Model     │
│ with ID                    │
└────────────────────────────┘
           │
           ▼
┌────────────────────────────┐
│ Update Redis Cache         │
└────────────────────────────┘
           │
           ▼
┌────────────────────────────┐
│ Broadcast to WebSocket     │
│ (Real-time delivery)       │
└────────────────────────────┘
           │
           ▼
┌────────────────────────────┐
│ Return Message Model       │
└────────────────────────────┘
```

---

## 6. Event Dispatch Logic

```
┌──────────────────────────────────────────────────────────────────┐
│                    EVENT DISPATCH LOGIC                          │
└──────────────────────────────────────────────────────────────────┘

            ┌────────────────────┐
            │ Message Model      │
            │ Created ($model)   │
            └────────────────────┘
                      │
                      ▼
            ┌────────────────────┐
            │ Check Environment  │
            │ is_local()?        │
            └────────────────────┘
                      │
          ┌───────────┴───────────┐
          │                       │
        TRUE                    FALSE
    (Local Dev)              (Production)
          │                       │
          ▼                       ▼
┌──────────────────┐    ┌────────────────────────┐
│ LOCAL PATH       │    │ PRODUCTION PATH        │
├──────────────────┤    ├────────────────────────┤
│                  │    │                        │
│ HandleMessage    │    │ ┌────────────────────┐ │
│ ReadyEvent       │    │ │MessageReadyEvent   │ │
│ ::run($model)    │    │ │::dispatch($model)  │ │
│                  │    │ │                    │ │
│ [Sync]           │    │ │Queue: "messages"   │ │
│ [Blocks]         │    │ │Priority: High      │ │
│ [Immediate]      │    │ │[Async Processing]  │ │
│                  │    │ └────────────────────┘ │
│                  │    │          +             │
│                  │    │ ┌────────────────────┐ │
│                  │    │ │HandleMessageReady  │ │
│                  │    │ │Event::run($model)  │ │
│                  │    │ │                    │ │
│                  │    │ │[Sync]              │ │
│                  │    │ │[Immediate]         │ │
│                  │    │ └────────────────────┘ │
└──────────────────┘    └────────────────────────┘
          │                       │
          └───────────┬───────────┘
                      ▼
            ┌────────────────────┐
            │ Check Message      │
            │ Meta State         │
            └────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ Has          │ │ Has          │ │ No Special   │
│ quick_reply  │ │ postback     │ │ Metadata     │
│ in meta?     │ │ in meta?     │ │              │
└──────────────┘ └──────────────┘ └──────────────┘
        │             │                   │
      YES           YES                  NO
        │             │                   │
        ▼             ▼                   │
┌──────────────┐ ┌──────────────┐        │
│ Message      │ │ Message      │        │
│ QuickReply   │ │ Postback     │        │
│ Event        │ │ Event        │        │
│ ::dispatch   │ │ ::dispatch   │        │
│              │ │              │        │
│ ($model,     │ │ ($model,     │        │
│  $quick_     │ │  $postback)  │        │
│  reply)      │ │              │        │
└──────────────┘ └──────────────┘        │
        │             │                   │
        └─────────────┴───────────────────┘
                      │
                      ▼
            ┌────────────────────┐
            │ Event Dispatch     │
            │ Complete           │
            └────────────────────┘
```

---

## 7. Session State Management

```
┌──────────────────────────────────────────────────────────────────┐
│                   SESSION STATE FLOW                             │
└──────────────────────────────────────────────────────────────────┘

    ┌────────────────┐
    │ First Message  │
    │ from Customer  │
    └────────────────┘
            │
            ▼
    ┌────────────────┐
    │  Create New    │
    │  Session       │
    │  state: "new"  │
    └────────────────┘
            │
            ▼
    Bot Enabled for Channel?
            │
    ┌───────┴────────┐
    │                │
  YES              NO
    │                │
    ▼                ▼
┌─────────┐    ┌──────────────┐
│ state:  │    │ state:       │
│ "bot_   │    │ "pending"    │
│ active" │    │              │
│         │    │ Queue for    │
│ Route to│    │ Agent        │
│ Bot     │    │ Assignment   │
└─────────┘    └──────────────┘
    │                │
    ├────────────────┤
    │                │
    │    Agent picks up / Auto-assign
    │                │
    └────────┬───────┘
             ▼
    ┌────────────────┐
    │  state:        │
    │  "agent_active"│
    │                │
    │  • Assigned    │
    │    agent_id    │
    │  • Real-time   │
    │    notify      │
    └────────────────┘
             │
             ▼
    Conversation continues...
             │
    ┌────────┴────────┐
    │                 │
  Resolved      Transferred
    │                 │
    ▼                 ▼
┌─────────┐    ┌──────────────┐
│ state:  │    │ state:       │
│"resolved│    │ "pending"    │
│         │    │              │
│Wait for │    │ Queue for    │
│customer │    │ different    │
│response │    │ agent        │
└─────────┘    └──────────────┘
    │
    │ No response for 24h
    │
    ▼
┌─────────────┐
│ state:      │
│ "closed"    │
│             │
│ Session     │
│ Ended       │
└─────────────┘
    │
    │ Customer sends new message
    │
    ▼
┌─────────────────────────┐
│ Within reopen window?   │
│ (1 hour)               │
└─────────────────────────┘
    │
┌───┴────┐
│        │
YES      NO
│        │
▼        ▼
┌────┐  ┌────┐
│Re  │  │New │
│open│  │Ses │
│Same│  │sion│
│Sess│  │    │
│ion │  │    │
└────┘  └────┘
```

---

## 8. HandleMessageReadyEvent Processing

```
┌──────────────────────────────────────────────────────────────────┐
│            HandleMessageReadyEvent PROCESSING                    │
└──────────────────────────────────────────────────────────────────┘

        ┌────────────────────────┐
        │ Event Triggered        │
        │ with $model            │
        └────────────────────────┘
                    │
                    ▼
        ┌────────────────────────┐
        │ Load Session           │
        │ from Message           │
        └────────────────────────┘
                    │
                    ▼
            Session State?
                    │
    ┌───────────────┼───────────────┬───────────┐
    │               │               │           │
    ▼               ▼               ▼           ▼
┌────────┐    ┌──────────┐    ┌────────┐  ┌────────┐
│  new   │    │bot_active│    │agent_  │  │pending │
│        │    │          │    │active  │  │        │
└────────┘    └──────────┘    └────────┘  └────────┘
    │               │               │           │
    ▼               ▼               ▼           ▼
┌────────┐    ┌──────────────────────────┐  ┌────────────┐
│Create  │    │  BOT PROCESSING          │  │ QUEUE      │
│Case    │    ├──────────────────────────┤  │ ASSIGNMENT │
│        │    │                          │  ├────────────┤
│Assign  │    │ 1. Load Bot Journey      │  │            │
│to      │    │ 2. Get Current Step      │  │ Get Queue  │
│Default │    │ 3. Send to Dialogflow    │  │ Rules      │
│Queue   │    │                          │  │            │
└────────┘    │ ┌──────────────────────┐ │  │ Calculate  │
    │         │ │ Intent Matched?      │ │  │ Priority   │
    │         │ └──────────────────────┘ │  │            │
    │         │    │             │       │  │ Find       │
    │         │    YES           NO      │  │ Available  │
    │         │    │             │       │  │ Agent      │
    │         │    ▼             ▼       │  │            │
    │         │ ┌────┐       ┌────────┐ │  │ ┌────────┐ │
    │         │ │Proc│       │Fallback│ │  │ │ Found? │ │
    │         │ │ess │       │Intent  │ │  │ └────────┘ │
    │         │ │Int │       │        │ │  │   │    │   │
    │         │ │ent │       └────────┘ │  │  YES  NO   │
    │         │ └────┘           │      │  │   │    │   │
    │         │    │              │      │  │   ▼    ▼   │
    │         │    └──────┬───────┘      │  │ ┌──┐ ┌──┐ │
    │         │           ▼              │  │ │As│ │Qu│ │
    │         │    ┌────────────┐        │  │ │si│ │eu│ │
    │         │    │Generate    │        │  │ │gn│ │e │ │
    │         │    │Bot Response│        │  │ └──┘ └──┘ │
    │         │    └────────────┘        │  └────────────┘
    │         │           │              │       │
    │         │           ▼              │       │
    │         │    ┌────────────┐        │       │
    │         │    │Handoff?    │        │       │
    │         │    └────────────┘        │       │
    │         │      │        │          │       │
    │         │     YES      NO          │       │
    │         │      │        │          │       │
    │         │      ▼        ▼          │       │
    │         │   ┌────┐  ┌──────┐       │       │
    │         │   │To  │  │Send  │       │       │
    │         │   │Agt │  │Resp  │       │       │
    │         │   └────┘  └──────┘       │       │
    │         └──────────────────────────┘       │
    │                      │                     │
    │                      ▼                     │
    │             ┌─────────────────┐            │
    │             │ AGENT HANDLING  │            │
    │             ├─────────────────┤            │
    │             │                 │            │
    │             │ Get Assigned    │            │
    │             │ Agent           │            │
    │             │                 │            │
    │             │ Check Agent     │            │
    │             │ Online Status   │            │
    │             │                 │            │
    │             │ ┌─────────────┐ │            │
    │             │ │Online?      │ │            │
    │             │ └─────────────┘ │            │
    │             │   │        │    │            │
    │             │  YES      NO    │            │
    │             │   │        │    │            │
    │             │   ▼        ▼    │            │
    │             │ ┌───┐   ┌────┐ │            │
    │             │ │Ntf│   │Que │ │            │
    │             │ │Agt│   │for │ │            │
    │             │ │RT │   │Lat │ │            │
    │             │ └───┘   └────┘ │            │
    │             └─────────────────┘            │
    │                      │                     │
    └──────────────────────┴─────────────────────┘
                           │
                           ▼
                ┌──────────────────┐
                │ Log Analytics    │
                │ Update Metrics   │
                │ Broadcast Update │
                └──────────────────┘
                           │
                           ▼
                ┌──────────────────┐
                │ Processing       │
                │ Complete         │
                └──────────────────┘
```

---

## 9. Message Type Detection

```
┌──────────────────────────────────────────────────────────────────┐
│                MESSAGE TYPE DETECTION                            │
└──────────────────────────────────────────────────────────────────┘

            ┌────────────────┐
            │ Raw Message    │
            │ Payload        │
            └────────────────┘
                    │
                    ▼
            Entry Point?
                    │
    ┌───────────────┼───────────────┬────────────────┐
    │               │               │                │
    ▼               ▼               ▼                ▼
┌────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│Widget  │    │Facebook  │    │Facebook  │    │  Email   │
│API     │    │Messenger │    │Comment   │    │Processor │
└────────┘    └──────────┘    └──────────┘    └──────────┘
    │               │               │                │
    ▼               ▼               ▼                ▼

┌──────────────────────────────────────────────────────────────────┐
│ WIDGET MESSAGE TYPES                                             │
├──────────────────────────────────────────────────────────────────┤
│  Check: payload.message.type                                     │
│                                                                  │
│  • "message" | "text"     → widget_text                          │
│  • "image"                → widget_image                         │
│  • "video"                → widget_video                         │
│  • "file" | "document"    → widget_file                          │
│  • "location"             → widget_location                      │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│ FACEBOOK MESSENGER TYPES                                         │
├──────────────────────────────────────────────────────────────────┤
│  Check: payload.entry[0].messaging[0]                            │
│                                                                  │
│  Has "message.text"?       → fb_text                             │
│                                                                  │
│  Has "message.quick_reply"? → fb_quick_reply                     │
│    └─ Set meta: quick_reply = payload.message.quick_reply       │
│                                                                  │
│  Has "postback"?           → fb_postback                         │
│    └─ Set meta: postback = payload.postback                     │
│                                                                  │
│  Has "message.attachments"?                                      │
│    ├─ type = "image"       → fb_image                            │
│    ├─ type = "video"       → fb_video                            │
│    ├─ type = "audio"       → fb_audio                            │
│    └─ type = "file"        → fb_file                             │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│ FACEBOOK COMMENT TYPES                                           │
├──────────────────────────────────────────────────────────────────┤
│  Check: payload.entry[0].changes[0]                              │
│                                                                  │
│  • value.verb = "add"      → fb_comment_new                      │
│  • value.verb = "edited"   → fb_comment_edited                   │
│  • value.verb = "remove"   → fb_comment_deleted                  │
│                                                                  │
│  Has "value.photo"?        → fb_comment_photo                    │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│ EMAIL MESSAGE TYPES                                              │
├──────────────────────────────────────────────────────────────────┤
│  Check: Email body content                                       │
│                                                                  │
│  • HTML only               → email_html                          │
│  • Text only               → email_text                          │
│  • Both HTML and Text      → email_mixed                         │
│                                                                  │
│  Has attachments?          → Add: email_with_attachment          │
└──────────────────────────────────────────────────────────────────┘

                           │
                           ▼
            ┌──────────────────────────┐
            │ Store Message Type       │
            │ in Database:             │
            │ messages.type            │
            │ messages.channel         │
            └──────────────────────────┘
```

---

## 10. Complete System Architecture (ASCII)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        MEVRIK CHANNELS SYSTEM                           │
│                     Message Ingestion Architecture                      │
└─────────────────────────────────────────────────────────────────────────┘

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ EXTERNAL CHANNELS                                                      ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                                        ┃
┃  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  ┃
┃  │ Web Widget  │  │   Facebook   │  │   Facebook   │  │  Email   │  ┃
┃  │   (Chat)    │  │  Messenger   │  │   Comments   │  │  Server  │  ┃
┃  └─────────────┘  └──────────────┘  └──────────────┘  └──────────┘  ┃
┃         │                │                  │                │        ┃
┗━━━━━━━━━┼━━━━━━━━━━━━━━━━┼━━━━━━━━━━━━━━━━━━┼━━━━━━━━━━━━━━━━┼━━━━━━━━┛
          │                │                  │                │
          │                │                  │                │
          ▼                ▼                  ▼                ▼
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ API ENTRY POINTS                                                       ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                                        ┃
┃  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────────┐  ┃
┃  │ /api/v1/messages │  │ /webhook/        │  │ /api/v1/email/     │  ┃
┃  │ ?action=         │  │ facebook         │  │ process_incoming_  │  ┃
┃  │ message_post     │  │                  │  │ email              │  ┃
┃  │                  │  │ FacebookContr    │  │                    │  ┃
┃  │ [Auth: Token]    │  │ [Auth: Sig]      │  │ [Auth: API Key]    │  ┃
┃  └──────────────────┘  └──────────────────┘  └────────────────────┘  ┃
┃         │                      │                        │             ┃
┗━━━━━━━━━┼━━━━━━━━━━━━━━━━━━━━━━┼━━━━━━━━━━━━━━━━━━━━━━━━┼━━━━━━━━━━━━━┛
          │                      │                        │
          └──────────────────────┴────────────────────────┘
                                 │
                                 ▼
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ PREPROCESSING LAYER                                                    ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                                        ┃
┃  ┌────────────────────────────────────────────────────────────────┐  ┃
┃  │ IngestFacebookWebhookAction / Email Parser                     │  ┃
┃  │ • Validate webhook signature / Parse email                     │  ┃
┃  │ • WebhookEvent::store_webhook($payload)                        │  ┃
┃  │ • CreateMessageFromWebhook                                     │  ┃
┃  └────────────────────────────────────────────────────────────────┘  ┃
┃                                 │                                     ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┼━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                                  ▼
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ CORE MESSAGE PROCESSING ENGINE                                        ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                                        ┃
┃  ┌────────────────────────────────────────────────────────────────┐  ┃
┃  │           MessagingRequestHelper                               │  ┃
┃  ├────────────────────────────────────────────────────────────────┤  ┃
┃  │  1. Validate Input                                             │  ┃
┃  │  2. Detect Channel (widget/facebook/email)                     │  ┃
┃  │  3. Normalize to Standard Format                               │  ┃
┃  │  4. Extract Metadata (sender, recipient, type)                 │  ┃
┃  │  5. Get/Create Customer Record                                 │  ┃
┃  │  6. Find/Create Session                                        │  ┃
┃  │  7. Build Message Object                                       │  ┃
┃  │  8. storeMessage() → Database                                  │  ┃
┃  │  9. Update Redis Cache                                         │  ┃
┃  │ 10. Broadcast to WebSocket                                     │  ┃
┃  │ 11. Return Message Model                                       │  ┃
┃  └────────────────────────────────────────────────────────────────┘  ┃
┃                                 │                                     ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┼━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                                  ▼
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ EVENT DISPATCHER                                                       ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                    is_local() ?                                        ┃
┃        ┌──────────────┴──────────────┐                                ┃
┃        │                             │                                ┃
┃       YES                           NO                                ┃
┃  (Local Dev)                  (Production)                            ┃
┃        │                             │                                ┃
┃        ▼                             ▼                                ┃
┃  ┌──────────────┐         ┌──────────────────────┐                   ┃
┃  │ Synchronous  │         │ Dual Processing:     │                   ┃
┃  │ Only         │         │ • Async (Queue)      │                   ┃
┃  │              │         │ • Sync (Immediate)   │                   ┃
┃  │ HandleMsg    │         │                      │                   ┃
┃  │ ReadyEvent   │         │ MessageReadyEvent    │                   ┃
┃  │ ::run()      │         │ ::dispatch()         │                   ┃
┃  │              │         │    +                 │                   ┃
┃  │              │         │ HandleMsgReadyEvent  │                   ┃
┃  │              │         │ ::run()              │                   ┃
┃  └──────────────┘         └──────────────────────┘                   ┃
┃        │                             │                                ┃
┃        └─────────────┬───────────────┘                                ┃
┃                      │                                                ┃
┃              Check Meta State                                         ┃
┃                      │                                                ┃
┃       ┌──────────────┼──────────────┐                                ┃
┃       │              │              │                                ┃
┃       ▼              ▼              ▼                                ┃
┃  ┌─────────┐  ┌──────────┐  ┌───────────┐                           ┃
┃  │ quick_  │  │postback? │  │  Normal   │                           ┃
┃  │ reply?  │  │          │  │   Flow    │                           ┃
┃  └─────────┘  └──────────┘  └───────────┘                           ┃
┃       │              │                                                ┃
┃       ▼              ▼                                                ┃
┃  ┌─────────┐  ┌──────────┐                                           ┃
┃  │  QR     │  │  PB      │                                           ┃
┃  │  Event  │  │  Event   │                                           ┃
┃  └─────────┘  └──────────┘                                           ┃
┃                                                                       ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━┬━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                           │
                           ▼
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ DOWNSTREAM PROCESSING (HandleMessageReadyEvent)                        ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                                        ┃
┃                       Session State?                                   ┃
┃            ┌──────────────┼──────────────┬──────────┐                 ┃
┃            │              │              │          │                 ┃
┃            ▼              ▼              ▼          ▼                 ┃
┃      ┌─────────┐    ┌─────────┐    ┌─────────┐ ┌─────────┐          ┃
┃      │   new   │    │   bot   │    │  agent  │ │ pending │          ┃
┃      │         │    │  active │    │  active │ │         │          ┃
┃      └─────────┘    └─────────┘    └─────────┘ └─────────┘          ┃
┃            │              │              │          │                 ┃
┃            ▼              ▼              ▼          ▼                 ┃
┃   ┌──────────────┐ ┌────────────┐ ┌───────────┐ ┌──────────┐        ┃
┃   │ Create Case  │ │    BOT     │ │   AGENT   │ │  QUEUE   │        ┃
┃   │ Assign Queue │ │ PROCESSING │ │  HANDLING │ │ ASSIGN   │        ┃
┃   └──────────────┘ ├────────────┤ ├───────────┤ └──────────┘        ┃
┃                    │• Dialogflow│ │• Notify   │                      ┃
┃                    │• Journey   │ │  Agent    │                      ┃
┃                    │• Intent    │ │• Update   │                      ┃
┃                    │  Match     │ │  Dashboard│                      ┃
┃                    │• Response  │ │• Real-time│                      ┃
┃                    │• Handoff?  │ │  Deliver  │                      ┃
┃                    └────────────┘ └───────────┘                      ┃
┃                                                                       ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━┬━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                           │
                           ▼
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ DATA LAYER                                                             ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                                        ┃
┃  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌─────────┐  ┃
┃  │   MySQL      │  │  PostgreSQL  │  │  ClickHouse  │  │  Redis  │  ┃
┃  ├──────────────┤  ├──────────────┤  ├──────────────┤  ├─────────┤  ┃
┃  │• messages    │  │• users       │  │• analytics   │  │• cache  │  ┃
┃  │• sessions    │  │• queues      │  │• tokens      │  │• queues │  ┃
┃  │• cases       │  │• channels    │  │• reports     │  │• locks  │  ┃
┃  │• customers   │  │              │  │              │  │         │  ┃
┃  └──────────────┘  └──────────────┘  └──────────────┘  └─────────┘  ┃
┃                                                                       ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ SUPPORTING SERVICES                                                    ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃                                                                        ┃
┃  ┌─────────────┐  ┌────────────┐  ┌──────────┐  ┌──────────────┐    ┃
┃  │ WebSocket   │  │ Google     │  │  AWS S3  │  │  Graylog     │    ┃
┃  │ (Pusher)    │  │ Dialogflow │  │  Files   │  │  Logging     │    ┃
┃  │ Real-time   │  │ NLP/AI     │  │ Storage  │  │  Monitoring  │    ┃
┃  └─────────────┘  └────────────┘  └──────────┘  └──────────────┘    ┃
┃                                                                       ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

---
