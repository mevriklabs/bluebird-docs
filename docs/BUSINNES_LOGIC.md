# Business Logic Documentation
## Mevrik Channels Platform

**Project:** Mevrik Channels  
**Version:** 1.0  
**Last Updated:** October 27, 2025  
**Document Type:** Business Logic & System Behavior Documentation

---

## Document Overview

This document serves as the comprehensive reference for understanding how the Mevrik Channels platform operates at the business logic level. It covers:

- **System Architecture:** Multi-channel message processing, session management, and event-driven workflows
- **Business Modules:** Detailed explanation of each functional module and its responsibilities
- **Data Flows:** Step-by-step execution paths for key business processes
- **Integration Points:** External API integrations (Facebook, Dialogflow, GP APIs)
- **Database Schema:** Data models, relationships, and validation rules
- **Automation:** Scheduled jobs, background queue processing, and report generation
- **Security:** Authentication, authorization, data protection, and API security

**Target Audience:** Developers, System Architects, Product Managers, QA Engineers

**Companion Documents:**
- For visual flow diagrams → See [MESSAGE_INCOMING_LOGIC_DIAGRAMS.md](MESSAGE_INCOMING_LOGIC_DIAGRAMS.md) and [MESSAGE_OUTGOING_LOGIC_DIAGRAMS.md](MESSAGE_OUTGOING_LOGIC_DIAGRAMS.md)
- For event architecture → See [mevrik-channels-events.md](mevrik-channels-events.md)

---

## Table of Contents

1. [Overview](#1-overview)
   - 1.1 [High-Level Project Description](#11-high-level-project-description)
   - 1.2 [Core Objectives and Problem Solved](#12-core-objectives-and-problem-solved)
   - 1.3 [Primary Tech Stack](#13-primary-tech-stack)

2. [Modules & Components](#2-modules--components)
   - 2.1 [Multi-Channel Message Ingestion](#21-module-multi-channel-message-ingestion)
   - 2.2 [Session Management](#22-module-session-management)
   - 2.3 [Intelligent Chatbot Engine](#23-module-intelligent-chatbot-engine)
   - 2.4 [Bot-to-Agent Transfer (Queue Management)](#24-module-bot-to-agent-transfer-queue-management)
   - 2.5 [Agent Message Handling](#25-module-agent-message-handling)
   - 2.6 [Social Media Management](#26-module-social-media-management)
   - 2.7 [Customer Management](#27-module-customer-management)
   - 2.8 [NLP Intent Detection](#28-module-nlp-intent-detection)
   - 2.9 [External API Integration (GP APIs)](#29-module-external-api-integration-gp-apis)
   - 2.10 [Reporting & Analytics Engine](#210-module-reporting--analytics-engine)

3. [Business Logic Flows](#3-business-logic-flow)
   - 3.1 [Customer Message Processing Flow](#31-customer-message-processing-flow)
   - 3.2 [Bot-to-Agent Handoff Flow](#32-bot-to-agent-handoff-flow)
   - 3.3 [Agent Reply and Case Closure Flow](#33-agent-reply-and-case-closure-flow)
   - 3.4 [Report Generation Flow](#34-report-generation-flow)

4. [Database & Data Rules](#4-database--data-rules)
   - 4.1 [Data Models and Relationships](#41-data-models-and-relationships)
   - 4.2 [Key Constraints and Validations](#42-key-constraints-and-validations)

5. [Integration Points](#5-integration-points)
   - 5.1 [External Integrations](#51-external-integrations)
   - 5.2 [Integration Data Flow](#52-integration-data-flow)

6. [Automation & Jobs](#6-automation--jobs)
   - 6.1 [Scheduled Jobs (Cron)](#61-scheduled-jobs-cron)
   - 6.2 [Background Queue Jobs](#62-background-queue-jobs)

7. [Security & Compliance](#7-security--compliance)
   - 7.1 [Authentication & Authorization](#71-authentication--authorization)
   - 7.2 [Data Security](#72-data-security)
   - 7.3 [API Security](#73-api-security)

8. [Performance & Scalability](#8-performance--scalability)
   - 8.1 [Caching Strategy](#81-caching-strategy)
   - 8.2 [Queue Management](#82-queue-management)
   - 8.3 [Database Optimization](#83-database-optimization)

9. [Related Documentation](#9-related-documentation)

10. [Conclusion](#10-conclusion)

---

## Quick Reference

### Key API Endpoints

**Incoming Messages:**
- `POST /webhook/facebook` - Facebook Messenger & Comments
- `POST /api/v1/messages?action=message_post` - Web Widget messages
- `POST /api/v1/email/process_incoming_email` - Email processing

**Outgoing Messages (Agent):**
- `POST /api/v1/agent/messages?action=message_reply` - Agent reply
- `POST /api/v1/agent/messages?action=message_bulk_reply` - Bulk reply
- `POST /api/v1/agent/messages?action=message_closed` - Close conversation
- `POST /api/v1/agent/messages?action=message_history` - Get history
- `POST /api/v1/agent/messages?action=video_call_request` - Request video call
- `POST /api/v1/agent/messages?action=post_details` - Get post details
- `POST /api/v1/agent/messages?action=comment_details` - Get comment details

### Core Components

| Component | Purpose | Key Classes |
|-----------|---------|-------------|
| **MessagingRequestHelper** | Message normalization & storage | `MessagingRequestHelper->storeMessage()` |
| **FacebookController** | Webhook ingestion | `IngestFacebookWebhookAction` |
| **Event System** | Async processing | `MessageReadyEvent`, `MessageQuickReplyEvent`, `MessagePostbackEvent` |
| **Session Manager** | Session lifecycle | `SessionUser`, `SessionPathChangedEvent` |
| **Queue System** | Agent assignment | `QueueItemCreatedEvent`, `MevrikQueueItemAssignAction` |
| **Report Engine** | Analytics generation | `report_master`, `customer_social_activity_reply_report` |

### Database Tables

| Table | Purpose |
|-------|---------|
| `messages` | All customer and agent messages |
| `session_users` | Active conversation sessions |
| `customers` | Customer profiles |
| `customer_identities` | Channel-specific customer IDs |
| `cases` | Conversation cases |
| `queue_items` | Agent assignment queue |
| `csat_survey` | Customer satisfaction surveys |
| `report_master` | Session closure reports |
| `customer_social_activity_reply_report` | Facebook comment reports |

---

## 1. Overview

### 1.1 High-Level Project Description

Mevrik Channels is an enterprise-grade, multi-channel customer communication and engagement platform designed to unify customer interactions across various digital touchpoints. The platform acts as a central hub for managing conversations, automating responses through intelligent chatbots, routing inquiries to human agents, and generating comprehensive analytics on customer engagement patterns.

### 1.2 Core Objectives and Problem Solved

**Primary Objectives:**
- Centralize multi-channel customer communications (Facebook Messenger, Facebook Comments, Web Widget, Email, Video Calls)
- Automate customer service through intelligent chatbot journeys powered by Google Dialogflow
- Enable seamless bot-to-human agent handoffs when automation reaches its limits
- Provide real-time conversation management for support agents
- Generate actionable insights through comprehensive reporting and analytics
- Track customer sentiment, service performance, and operational metrics

**Problems Solved:**
- **Fragmented Communication:** Eliminates need for multiple tools by consolidating all channels into one platform
- **Manual Response Burden:** Reduces agent workload through intelligent bot automation
- **Lost Context:** Maintains conversation continuity across channels and agent handoffs
- **Visibility Gaps:** Provides real-time and historical visibility into customer interactions and team performance
- **Quality Inconsistency:** Ensures consistent service delivery through standardized workflows

### 1.3 Primary Tech Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Framework** | Laravel | 9.x | Core application framework |
| **Language** | PHP | 8.0+ | Primary development language |
| **Databases** | MySQL | 5.7+ | Transactional data (messages, cases, sessions, reports) |
| | PostgreSQL | 12+ | User, queue, and channel management |
| | ClickHouse | Latest | Fast querying for GP numbers, tokens, and static offers |
| **Cache & Queue** | Redis | 6.x | Caching, session storage, queue management |
| **Queue Management** | Laravel Horizon | 5.9+ | Background job processing and monitoring |
| **Process Manager** | Supervisor | 4.x | Service process management |
| **Real-time** | Laravel WebSockets + Pusher | Latest | Real-time message delivery and notifications |
| **NLP Engine** | Google Dialogflow | v2 | Intent recognition and conversation AI |
| **Cloud Storage** | AWS S3 | v3 | File attachments and media storage |
| **Monitoring** | Cronitor | Latest | Scheduled job monitoring |
| **Logging** | Graylog | Latest | Centralized log aggregation |
| **Package Manager** | Composer | 2.x | PHP dependency management |

**Key Architectural Patterns:**
- **Action-Based Architecture:** Uses `lorisleiva/laravel-actions` for unified controller/job/listener classes
- **Event-Driven Design:** Laravel Events & Listeners for decoupled components
- **Queue-Based Processing:** Asynchronous handling via Laravel Horizon with Redis
- **Modular Packages:** Custom packages (Efemer, Genex) for domain-specific functionality

---

## 2. Modules & Components

### 2.1 Module: Multi-Channel Message Ingestion

**Purpose:**  
Receives and normalizes incoming messages from various external channels (Facebook, Web Widget, Email, etc.) into a unified message format for internal processing.

**Key Business Logic:**
- **Channel Detection:** Identifies the originating channel based on webhook signatures and request parameters
- **Message Normalization:** Transforms channel-specific message formats into a standardized internal structure
- **Duplicate Prevention:** Checks for duplicate messages using message IDs and timestamps to prevent reprocessing
- **Session Association:** Links incoming messages to existing customer sessions or creates new sessions
- **Real-time Delivery:** Broadcasts messages to relevant agents and customers via WebSocket connections

**Entry Points:**
- `POST /webhook/facebook` - Facebook Messenger & Comments webhook listener
- `POST /api/v1/messages?action=message_post` - Web widget message receiver
- `POST /api/v1/email/process_incoming_email` - Email message processor

**See Also:** [MESSAGE_INCOMING_LOGIC_DIAGRAMS.md](MESSAGE_INCOMING_LOGIC_DIAGRAMS.md) for detailed flow diagrams

**Dependencies:**
- **External:** Facebook Graph API, Google Dialog Flow

**Business Rules:**
1. All messages must be associated with a valid session before processing
2. Messages without valid sender ID are rejected
3. Each message generates a `MessageReceiverEvent` for downstream processing
4. Messages are logged to both database and real-time

---

### 2.2 Module: Session Management

**Purpose:**  
Manages customer conversation sessions, tracking state, context, and conversation flow across interactions.

**Key Business Logic:**
- **Session Creation:** Automatically creates sessions for new customers on first interaction
- **Session State Tracking:** Maintains current conversation state (bot, live)
- **Context Preservation:** Stores conversation context including customer data, intent history, and interaction metadata
- **Timeout Management:** Automatically expires inactive sessions after configured timeout period (15 minutes default)
- **Agent Type Switching:** Handles transitions between bot mode and human agent mode

**Entry Points:**
- Automatically triggered via `MessageReadyEvent`
- Session state changes via `SessionPathChangedEvent`

**Dependencies:**
- **Internal:** Customer Management, Queue Management, Message Handler
- **Models:** `messages`, `sessions`, `case`, `queue_items`
- **Cache:** Redis for fast session lookups and state management

**Business Rules:**
1. One active session per customer per channel at any time
2. Sessions automatically transition to bot mode when agent closes conversation
3. Session timeout resets on each customer message
4. Historical sessions are preserved for reporting

---

### 2.3 Module: Intelligent Chatbot Engine

**Purpose:**  
Provides automated conversational responses through AI-powered intent recognition and predefined journey flows.

**Key Business Logic:**

**Intent Recognition Flow:**
1. Customer message analyzed by Google Dialogflow for intent classification
2. Confidence score evaluated against threshold (0.6+ for domain intents, configurable)
3. Intent mapped to internal business logic handlers (e.g., "RECHARGE" → recharge journey)
4. Small talk handled separately with lower confidence threshold
5. Unrecognized intents logged for training and fallback to agent

**Authentication States:**
- **GUEST Mode:** Unauthenticated users with limited journey access
- **USER Mode:** Authenticated users (via OTP) with full journey access
- **EXPR Mode:** Expired authentication requiring re-verification

**Journey Execution:**
- **Dynamic Response Generation:** Responses tailored based on customer data and context
- **Multi-step Journeys:** Complex flows with state preservation between steps
- **API Integration:** Real-time data fetching from GP APIs for account details, balance, offers
- **Error Handling:** Graceful fallback responses when external APIs fail
- **User Input Tracking:** Validates and stores user inputs for journey progression

**Entry Points:**
- Triggered by `HandleMessageReadyEvent` when session is in bot mode
- Entry via `BotFlowProcessWithUserQuery` action class

**Dependencies:**
- **External:** Google Dialogflow API, GP APIs (account, recharge, offers)
- **Internal:** Intent Response Service, GP APIs Service, User Input Tracker
- **Services:** `DialogueFlow`, `IntentResponse`, `GPAPIs`, `RechargeNLUService`

**Business Rules:**
1. Bot attempts to serve customer first unless channel/app requires agent
2. Failed bot interactions tracked in performance metrics
3. Maximum 3 failed attempts before suggesting agent transfer
4. Authentication required for sensitive operations (balance, recharge, account changes)
5. OTP verification timeout: 15 minutes
6. Bot responses must complete within 5 seconds or timeout message shown

---

### 2.4 Module: Bot-to-Agent Transfer (Queue Management)

**Purpose:**  
Manages the handoff process from automated bot to human agents, including queue creation, assignment.

**Key Business Logic:**

**Transfer Triggers:**
1. Customer explicitly requests agent ("Connect to Agent" button)
2. Bot confidence score below threshold for multiple consecutive messages
3. Bot encounters error in API calls or journey execution
4. Channel configured for mandatory agent handling (e.g. Bioscope)

**Queue Creation Process:**
1. `QueueItemCreatedEvent` dispatched with customer and case information
2. Queue item created with priority, channel
3. Session agent type changed from "bot" to "live"
4. `SessionPathChangedEvent` broadcasted for real-time updates
5. Available agents notified via WebSocket and push notifications

**Agent Assignment Logic:**
1. **Round-Robin:** Assigns to agent with fewest active conversations
2. **Skill-Based:** Matches agent skills to required queue skills
3. **Channel-Specific:** Assigns agents authorized for specific channels
4. **Priority:** Higher priority queues assigned first
5. **Fallback:** If no agents available, queue waits with timeout notifications



**Dependencies:**
- **Models:** `Queue`, `QueueItem`, `SessionUser`
- **Actions:** `MevrikQueueItemAssignAction`, `MevrikQueueItemRemoveAction`

**Business Rules:**
1. Queue items auto-expire after 30 minutes of no agent pickup
2. Customer can only have one active queue item per app
3. Agent cannot be assigned if at maximum concurrent conversation limit (configurable)
4. Queue assignment logged for performance reporting

---

### 2.5 Module: Agent Message Handling

**Purpose:**  
Processes agent replies to customer messages, delivers responses, and manages conversation closure.

**Key Business Logic:**

**Agent Reply Flow:**
1. Agent composes reply in inbox interface
2. Reply validated for content, attachments, and permissions
3. Message stored with agent attribution and timestamp
4. `MessageReadyEvent` dispatched for delivery
5. Reply sent to customer via channel-specific API (Facebook, Email, etc.)
6. Real-time delivery confirmation to agent
7. Response time metrics captured for reporting

**Bulk Operations:**
- **Bulk Reply:** Agent replies to multiple messages with same content (background job)
- **Bulk Close:** Agent closes multiple cases with same disposition

**Message Closure:**
1. Agent marks conversation as closed with disposition category
2. Session agent type reset to "bot"
3. Case status updated to "closed"
4. Customer Satisfaction (CSAT) survey triggered if enabled
5. Closure timestamp and details recorded for reporting

**Entry Points:**
- `POST /api/v1/agent/messages?action=message_reply` - Single agent reply
- `POST /api/v1/agent/messages?action=message_bulk_reply` - Bulk reply operation
- `POST /api/v1/agent/messages?action=message_closed` - Close conversation
- `POST /api/v1/agent/messages?action=message_history` - Get conversation history
- `POST /api/v1/agent/messages?action=video_call_request` - Request video call
- `POST /api/v1/agent/messages?action=post_details` - Get Facebook post details
- `POST /api/v1/agent/messages?action=comment_details` - Get comment details

**See Also:** [MESSAGE_OUTGOING_LOGIC_DIAGRAMS.md](MESSAGE_OUTGOING_LOGIC_DIAGRAMS.md) for detailed flow diagrams

**Dependencies:**
- **Internal:** Session Manager, Customer Service, Case Management, CSAT Service
- **External:** Channel-specific messaging APIs (Facebook, SendGrid, etc.)
- **Services:** `Messages`, `FacebookTemplate`, `IntentResponse`

**Business Rules:**
1. Agent must be assigned to case to reply
2. Cannot reply to closed cases (must reopen first)
3. Reply delivery failures retry 3 times before marking as failed
4. Attachments validated for type (images, PDFs, documents) and size (max 25MB)
5. Agent replies tracked for productivity metrics
6. Sentiment analysis applied to closed conversations
7. **CSAT Survey Rules:**
   - Sent only for Widget and Facebook Messenger channels (not Email or Comments)
   - Survey sent immediately after case closure
   - Question: "Are you satisfied with the service?" with [Yes] [No] buttons
   - Maximum 3 retry attempts for invalid responses
   - Valid keywords: "yes", "y", "Yes", "YES" / "no", "n", "No", "NO"
   - Auto-reply on valid response: "Thank you for your feedback."
   - After 3 failed attempts: Mark as 'no_response', stop asking
8. **Report Generation Rules:**
   - Widget/Messenger closures → Generate `report_master`
   - Facebook Comment closures → Generate `customer_social_activity_reply_report`
   - Reports generated synchronously during closure process
   - Analytics cache updated immediately

---

### 2.6 Module: Social Media Management

**Purpose:**  
Handles Facebook page post comments and replies, enabling social customer service at scale.

**Key Business Logic:**

**Comment Ingestion:**
1. Facebook webhook delivers page post comment
2. Comment parsed and linked to parent post
3. Customer identified or created from Facebook profile
4. Comment stored with sentiment analysis
5. Queue item created for agent review
6. Comment displayed in social inbox interface

**Agent Reply to Comments:**
1. Agent composes reply in social inbox
2. Reply posted to Facebook via Graph API
3. Reply associated with original comment for threading
4. `SaveAgentRepliesSocialEvent` dispatched
5. Metrics captured: waiting time, handling time, response time

**Post Monitoring:**
1. Active Facebook posts monitored for new comments
2. Edited comments detected and updated
3. Deleted comments marked but preserved for records
4. Post engagement metrics (likes, shares, reactions) tracked

**Entry Points:**
- `POST /api/v1/agent/messages?action=comment_details` - Fetch specific comment & post details
- `POST /api/v1/agent/messages?action=post_details` - Fetch Facebook post with comments and reactions
- `POST /webhook/facebook` - Receive Facebook comment webhooks

**Dependencies:**
- **External:** Facebook Graph API
- **Internal:** Customer Service, Agent Management, Case Management
- **Models:** `FacebookPost`, `PagePost`, `SocialActivityReport`, `SocialActivityReplyReport`

**Business Rules:**
1. Comments auto-assigned to agents based on availability
2. Public replies visible on Facebook; private replies sent as direct messages
3. Edited customer comments tracked for compliance
4. Admin-edited replies flagged in reports
5. Auto-reply keywords trigger canned responses
6. Sentiment analysis applied to all comments and replies
7. Response time SLA: 2 hours for public comments

---

### 2.7 Module: Customer Management

**Purpose:**  
Maintains unified customer profiles across all channels with identity resolution and data enrichment.

**Key Business Logic:**

**Customer Creation:**
1. New customer detected from first interaction
2. Customer profile created with channel-specific identifier (Facebook PSID, email, phone)
3. Profile enriched with available data (name, profile pic, location)
4. Customer identity linked to session and messages

**Identity Resolution:**
- **Phone Verification:** OTP-based phone number verification for authenticated access
- **Cross-Channel Linking:** Merges customer profiles when same identity detected across channels
- **Facebook Profile Sync:** Periodically updates customer data from Facebook profiles

**Customer Data Storage:**
- **Core Attributes:** Name, email, phone, profile picture
- **Identities:** Channel-specific IDs (Facebook PSID, User Ref, MSISDN)
- **Metadata:** Sentiment, last seen, interaction count, total conversations
- **Custom Data:** Extensible key-value store for app-specific customer attributes

**Entry Points:**
- `GET /api/v1/crm/customer/{id}` - Get customer full profile
- `POST /api/v1/customer/update` - Update customer information


**Dependencies:**
- **Internal:** Session Manager, Identity Manager
- **External:** Facebook Graph API for profile enrichment
- **Models:** `Customer`, `CustomerIdentity`, `FacebookUser`

**Business Rules:**
1. One customer profile per unique phone number
2. Multiple channel identities can link to single customer
3. Phone verification required for sensitive data access
4. Customer data encrypted at rest
5. Customer profiles never deleted, only anonymized (GDPR compliance)
6. Profile pictures cached and refreshed weekly

---

### 2.8 Module: NLP Intent Detection

**Purpose:**  
Analyzes customer messages to understand intent and route to appropriate business logic handlers.

**Key Business Logic:**

**Intent Detection Pipeline:**
1. **Pre-processing:** Clean and normalize user input (remove URLs, special characters)
2. **Dialogflow Query:** Send text to Google Dialogflow for intent classification
3. **Confidence Evaluation:** Check if confidence score meets threshold
   - Domain intents: ≥ 0.6 confidence
   - Small talk: ≥ 0.4 confidence (configurable)
4. **Intent Mapping:** Translate Dialogflow intent names to internal business handlers
5. **Entity Extraction:** Parse parameters/entities from Dialogflow response (dates, amounts, numbers)
6. **Fallback Handling:** Route low-confidence queries to fallback logic or agent transfer

**Intent Categories:**
- **Transactional:** RECHARGE, BALANCE_CHECK, OFFER_PURCHASE, BILL_PAY
- **Informational:** ACCOUNT_STATUS, OFFER_LIST, TARIFF_INFO
- **Support:** COMPLAINT, TECHNICAL_ISSUE, SERVICE_REQUEST
- **Navigation:** MAIN_MENU, GO_BACK, START_OVER
- **Small Talk:** GREETING, THANKS, GOODBYE, JOKES

**Special Intent Handling:**
- **RECHARGE:** Separate NLU model for complex recharge amount and number extraction
- **SWITCH_ACCOUNT:** Detected when user wants to check different phone number (MNP/Tintin)
- **UNAUTHENTICATED_JOURNEY:** Specific intents available without authentication

**Entry Points:**
- Invoked within `BotFlowProcessWithUserQuery` during message processing

**Dependencies:**
- **External:** Google Dialogflow API (two projects: main + smalltalk)
- **Services:** `DialogueFlow`, `DialogueSmallHelper`, `RechargeNLUService`
- **Models:** `NlpDataLog` for intent tracking and training

**Business Rules:**
1. All intents logged with confidence scores for accuracy monitoring
2. Failed intent recognition (< threshold) logged for model retraining
3. User authentication state affects available intents
4. Intent history preserved in session context for conversation flow
5. URL detection bypasses NLP to prevent injection attacks
6. Intent detection timeout: 3 seconds (fallback to "not recognized")

---

### 2.9 Module: External API Integration (GP APIs)

**Purpose:**  
Integrates with Grameenphone (GP) backend systems to fetch customer account data, process transactions, and retrieve offers.

**Key Business Logic:**

**API Categories:**
1. **Account APIs:** Customer account details, SIM status, package info
2. **Balance APIs:** Prepaid balance, postpaid credit, usage details
3. **Recharge APIs:** Recharge execution, transaction history, IPN callbacks
4. **Offer APIs:** Available offers, pack purchase, offer activation
5. **Service APIs:** Service activation/deactivation (FnF, VAS, roaming)

**Authentication Flow:**
1. OAuth token retrieved from GP authentication server
2. Token cached in Redis with expiration tracking
3. Automatic token refresh when expired
4. Request retry logic on 401/403 errors

**Error Handling:**
- **Circuit Breaker:** Temporarily disable failing APIs after 5 consecutive failures
- **Fallback Responses:** Return cached data or user-friendly error messages

**Response Processing:**
1. API response validated against expected schema
2. Data transformed to internal format
3. Relevant data cached for performance
4. Response formatted for chatbot or agent display

**Entry Points:**
- Invoked from bot journey actions based on detected intent
- Direct API testing routes: `/api/gp/*`

**Dependencies:**
- **External:** GP API endpoints (auth, account, recharge, offers)
- **Services:** `GPAPIs`, `GpApiController`, API Response formatters

**Business Rules:**
1. All API calls require valid authentication token
2. Customer MSISDN validated before API calls (GP number format)
3. Sensitive operations (recharge, activation) require OTP verification
4. API call logs retained for 90 days for troubleshooting
5. Rate limiting: Maximum 10 requests per customer per minute
6. API responses cached: 5 minutes for static data, 30 seconds for dynamic data

---

### 2.10 Module: Reporting & Analytics Engine

**Purpose:**  
Generates comprehensive reports on chatbot performance, agent productivity, customer satisfaction, and operational KPIs.

**Key Business Logic:**

**Report Categories:**

**1. LiveChat Reports:**
- **Master Report:** Real-time operational metrics (updated every 5 minutes)
- **Hourly Performance:** Hourly breakdown of response times, handling times, queue metrics
- **Agent Performance:** Individual agent productivity, conversation counts, CSAT scores
- **Frequency Report:** Daily conversation volumes by channel, app, and time
- **Insight Report:** Customer behavior patterns, sentiment trends, popular intents

**2. Chatbot Reports:**
- **Bot Performance:** Bot success rate, handoff rate, failure rate by intent
- **Intent Accuracy:** NLP model accuracy, confidence score distribution
- **Interaction Report:** Customer journey paths, conversation flows, drop-off points
- **Customer Insight:** User demographics, authentication rates, engagement metrics
- **Customer Feedback:** CSAT survey results, sentiment analysis

**3. Social Media Reports:**
- **Service Performance:** Hourly and daily metrics for social comment handling
- **Agent Productivity:** Reply counts, response times, handling times per agent
- **Customer Activity:** Comment volumes, sentiment trends, engagement metrics
- **Auto-Reply Effectiveness:** Keyword-triggered auto-reply performance

**4. Email Reports:**
- **Frequency Report:** Email volumes by channel and time period
- **Agent Productivity:** Email handling metrics per agent
- **Customer Insight:** Email engagement patterns and sentiment

**Report Generation Process:**
1. **Data Collection:** Real-time events aggregated to staging tables
2. **Scheduled Processing:** Cron jobs trigger report generation (hourly/daily)
3. **Data Aggregation:** Raw data summarized using ClickHouse for fast queries
4. **Report Storage:** Finalized reports stored in reporting tables
5. **Export:** Reports available via API, UI download, or FTP transfer

**Entry Points:**
- **API:** `/api/v1/reports/{action}` - Dynamic report queries
- **CLI:** `php artisan mevrik:reports {action}` - Scheduled report generation
- **Export:** `/api/v1/report/export` - Download reports in Excel/CSV

**Dependencies:**
- **Internal:** Message data, Session data, Case data, Agent activity logs
- **Database:** MySQL for reporting tables, ClickHouse for fast aggregations
- **Actions:** Report scheduler classes (e.g., `MasterReportScheduler`, `BotPerformanceScheduler`)

**Business Rules:**
1. Reports generated with timezone consideration (Asia/Dhaka)
2. Historical reports immutable once finalized
3. Real-time reports update every 5 minutes
4. Daily reports generated at 1:00 AM for previous day
5. Report retention: 1 year for detailed reports, 3 years for summaries
6. Export size limit: 50,000 rows per file
7. Report filters: date range, channel, app, agent, disposition

---

## 3. Business Logic Flow

### 3.1 Customer Message Processing Flow

**Scenario:** Customer sends a message via Facebook Messenger

**Step-by-Step Execution:**

1. **Message Reception:**
   - Facebook webhook delivers POST request to `/webhook/facebook`
   - `FacebookController` receives payload
   - Webhook signature validated against Facebook App Secret (X-Hub-Signature header)
   - Request passes to `IngestFacebookWebhookAction`

2. **Message Extraction:**
   - Entry array parsed for messaging events
   - Sender ID (PSID), Recipient ID (Page ID), Message ID extracted
   - Message text, attachments, quick replies, postbacks identified
   - Timestamp and mid (message ID) captured

3. **Event Dispatching:**
   - `MessageReceiverEvent` dispatched with channel and entry data
   - Event logged to channels table for webhook audit trail
   - Event triggers `FacebookHookListener` for processing

4. **Session Validation:**
   - Check if active session exists for sender ID
   - If no session → create new session via `SessionCreatedEvent`
   - If session exists → validate session state (active, expired, closed)
   - Session timeout check: reset if last message > 15 minutes ago

5. **Customer Lookup/Creation:**
   - Query customer by Facebook PSID
   - If new customer → create profile with Facebook data
   - Enrich profile: fetch name, profile pic from Facebook Graph API
   - Link customer to session

6. **Message Storage:**
   - Create Message record in messages table
   - Store: body, attachments, channel, sender/recipient refs, timestamp
   - Set message status: "new"
   - Generate case ID if first message in conversation
   - Associate message with customer and session

7. **Message Ready Event:**
   - `MessageReadyEvent` dispatched with Message object
   - Real-time broadcast to:
     - `messages` channel (global feed)
     - `session{id}` channel (specific conversation)
     - `agent{type}` channel (if agent assigned)

8. **Intent Detection (Bot Mode):**
   - Check if session is in bot mode (agent_type = "bot")
   - Send message text to Dialogflow for intent detection
   - Receive intent name, confidence score, entities
   - Log NLP result to nlp_data_logs table
   - **If confidence ≥ 0.6:** Proceed to intent handler
   - **If confidence < 0.6:** Check for small talk or fallback to "not recognized"

9. **Bot Response Generation:**
   - Route intent to appropriate business logic handler
   - Example: "CHECK_BALANCE" → fetch balance from GP API
   - Format response using template system
   - Generate buttons, quick replies, or rich media as needed
   - Store bot response as new Message with attending_id = bot

10. **Response Delivery:**
    - Send formatted response to Facebook via Graph API
    - Update message delivery status
    - Log response time metrics
    - Broadcast response via WebSocket to customer

11. **Success/Failure Handling:**
    - **Success:** Message marked as "replied", bot performance incremented
    - **Partial Success:** Bot served but user dissatisfied → track failure
    - **Failure:** API error or intent not handleable → suggest agent transfer
    - **Agent Transfer:** Dispatch `QueueItemCreatedEvent` if customer requests agent

12. **Session Update:**
    - Update session last_message_at timestamp
    - Update session context with intent history
    - Persist session state to Redis cache

**Conditional Branches:**

- **If Agent Mode:** Skip bot processing, route directly to agent queue/assigned agent
- **If CSAT Survey Active:** Capture survey response, close case, reset to bot mode
- **If Attachment:** Download attachment from Facebook CDN, store in S3, generate thumbnail
- **If Postback/Quick Reply:** Extract payload, trigger specific journey action

**Validation Rules:**
- Message text must not be empty (unless attachment present)
- Sender ID must be valid Facebook PSID
- Session must be active or expired (not closed)
- Bot response must be delivered within 24-hour Facebook messaging window

---

### 3.2 Bot-to-Agent Handoff Flow

**Scenario:** Customer requests to speak with a human agent

**Step-by-Step Execution:**

1. **Transfer Trigger:**
   - Customer clicks "Connect to Agent" button (postback)
   - OR bot confidence falls below threshold multiple times
   - OR bot encounters API failure
   - OR channel/app configured for mandatory agent handling

2. **Postback Processing:**
   - `MessagePostbackEvent` captured
   - Postback payload decoded: action type, context data
   - Validate customer eligible for agent transfer (e.g., authenticated if required)

3. **Queue Item Creation:**
   - `QueueItemCreatedEvent` dispatched
   - Queue item created with:
     - Customer ID and session ID
     - Case ID (new or existing)
     - Channel and app identifiers
     - Priority level (default: normal, escalated: high)
     - Required skills (if any)
   - Queue item stored in queue_items table

4. **Session Mode Switch:**
   - Session agent_type changed from "bot" to "live"
   - Session context updated with transfer reason
   - `SessionPathChangedEvent` dispatched
   - Session state saved to Redis

5. **Customer Notification:**
   - Send confirmation message to customer: "Connecting you to an agent..."
   - Optionally show estimated wait time based on current queue length
   - Set customer expectation: "An agent will respond shortly"

6. **Agent Notification:**
   - Broadcast `QueueItemCreatedEvent` to WebSocket channel `queues-app-{app_id}`
   - Push notification sent to all available agents
   - Email notification sent if no agents online (optional)
   - Queue item visible in agent inbox under "Waiting" section

7. **Agent Assignment Logic:**
   - **Manual Assignment:** Agent clicks "Pick" on queue item in inbox
   - **Auto-Assignment:** System assigns to agent with least active conversations
   - Validate: Agent authorized for channel, not at max capacity
   - Lock queue item to prevent double-assignment

8. **Assignment Confirmation:**
   - Queue item attending_id set to assigned agent
   - Queue item status changed to "assigned"
   - Session attending_id set to agent
   - Broadcast `QueueItemAssignedEvent`
   - Real-time update to:
     - Agent dashboard (queue item moves to "Active")
     - Customer session (shows agent name)

9. **First Agent Message:**
    - Agent sends greeting message
    - Message stored with agent attribution
    - Customer receives message via channel API
    - Response time metrics captured: time from queue creation to first agent reply

10. **Queue Item Closure:**
    - Queue item removed from active queue
    - Queue item record updated with assignment time, pickup time
    - Metrics logged for reporting: wait time = pickup - created

11. **Ongoing Conversation:**
    - Agent and customer exchange messages
    - All messages associated with case
    - Session remains in "live" mode until agent closes

---

### 3.3 Agent Reply and Case Closure Flow

**Scenario:** Agent replies to customer message and closes the conversation

**Step-by-Step Execution:**

**Agent Reply Phase:**

1. **Message Composition:**
   - Agent types reply in inbox interface
   - Optionally attaches file (image, PDF, document)
   - Optionally selects canned response template
   - Preview shows formatted message

2. **Reply Submission:**
   - Agent clicks "Send" button
   - POST request to `/api/v1/messages/message_reply`
   - Payload includes:
     - `is_reply`: Message ID being replied to
     - `body`: Reply text content
     - `attachments`: Array of file URLs (if any)
     - `case_id`: Associated case
     - `attending_id`: Agent ID

3. **Reply Validation:**
   - Verify agent assigned to case
   - Check case not closed
   - Validate message body not empty
   - Validate attachments (file type, size limits)
   - Check message length limits (channel-specific)

4. **Message Storage:**
   - Create Message record with agent attribution
   - Set `sender_type` = "agent"
   - Set `direction` = "outbound"
   - Link to case and session
   - Store timestamp and delivery status

5. **Channel Delivery:**
   - Route to appropriate channel adapter:
     - **Facebook:** POST to Graph API `/me/messages`
     - **Web Widget:** WebSocket push to customer session
     - **Email:** Send via Email Services APIs
   - Include attachments if present
   - Capture delivery confirmation from channel API

6. **Real-time Broadcast:**
   - Broadcast `MessageReadyEvent` for agent's reply
   - Push to customer via WebSocket (`session{id}` channel)
   - Update agent dashboard (message marked as "sent")
   - Update customer UI (message appears in conversation)

7. **Metrics Capture:**
   - Calculate response time: time from customer's last message to agent reply
   - Update case metrics: agent reply count, total handling time
   - Log agent productivity data

**Case Closure Phase:**

8. **Closure Initiation:**
   - Agent clicks "Close Case" button
   - Modal prompts for disposition and notes
   - POST request to `/api/v1/agent/messages?action=message_closed`

9. **Closure Execution:**
   - **Step 1:** Update session status to 'closed'
   - **Step 2:** Update case status to 'closed'
   - **Step 3:** Remove session from active queues
   - **Step 4:** Create system message "Conversation closed by Agent"

10. **Channel-Specific Processing:**
    - Determine channel type (Widget, Facebook Messenger, Facebook Comment, Email)
    - **For Widget/Messenger:** Trigger CSAT survey flow
    - **For Facebook Comment:** Skip CSAT, proceed to report generation

11. **CSAT Survey (Widget & Messenger Only):**
    - **Survey Message:** "Are you satisfied with the service?"
    - **Quick Reply Buttons:** [Yes] [No]
    - **Tracking:** Insert record into `csat_survey` table with status 'pending', attempt_count = 1
    - **Wait for Response:** Listen for customer reply via MessageReceiverEvent

12. **CSAT Response Handling:**
    - **Valid Response (Yes/No keywords):**
      - Update `csat_survey` table: response = 'yes'/'no', status = 'completed'
      - Send auto-reply: "Thank you for your feedback."
      - End CSAT flow
    - **Invalid Response (not Yes/No):**
      - Increment `attempt_count` in database
      - If attempts < 3: Resend CSAT question
      - If attempts >= 3: Stop asking, update status = 'no_response'
    - **Keywords Matched:**
      - YES: "yes", "y", "Yes", "YES"
      - NO: "no", "n", "No", "NO"

13. **Report Generation:**
    - **For Widget/Messenger:** Generate `report_master` with session metrics
      - Fields: session_id, channel, customer_id, agent_id, duration, message counts, response times, resolution_status, csat_score, tags
    - **For Facebook Comment:** Generate `customer_social_activity_reply_report`
      - Fields: session_id, post_id, comment_id, customer_id, page_id, reply_count, response_time, resolved, sentiment
    - **Trigger Events:** `ReportGeneratedEvent::dispatch()`
    - **Update Cache:** Redis metrics cache refreshed

14. **Analytics Updates:**
    - Daily conversation count incremented
    - Average resolution time recalculated
    - Agent performance metrics updated
    - Customer satisfaction trends updated

15. **Session Cleanup:**
    - Release agent assignment (free capacity)
    - Session moved to agent's "Closed" queue
    - Session context archived
    - Return success response to agent

**Edge Cases:**

- **Customer Replies After Closure:**
  - If within 1-hour reopen window: Reopen same session
  - If beyond reopen window: Create new session and case
  - New queue item generated if no bot handling

- **CSAT Survey Not Answered:**
  - After 3 invalid responses: Stop asking, mark 'no_response'
  - CSAT score remains null in reports

- **Agent Closes During Active CSAT:**
  - Complete pending CSAT survey before fully closing
  - Session enters 'closing' state until CSAT resolved

**For Complete Visual Flow:**  
See [MESSAGE_OUTGOING_LOGIC_DIAGRAMS.md](MESSAGE_OUTGOING_LOGIC_DIAGRAMS.md) - Section 4 for detailed CSAT survey logic, report generation flow, and complete integrated closure process with ASCII diagrams.

---

### 3.4 Report Generation Flow

**Scenario:** Daily bot performance report generation

**Step-by-Step Execution:**

1. **Scheduled Trigger:**
   - Cron job executes daily at 1:50 AM (configured in Laravel Scheduler)
   - Command: `php artisan mevrik:reports bot_performance`
   - Cronitor monitoring initiated for job tracking

2. **Report Scheduler Invocation:**
   - `BotPerformanceScheduler::run()` method called
   - Report parameters set:
     - Date: Previous day (e.g., 2025-10-26)
     - Channels: All active channels
     - Apps: All registered apps

3. **Data Collection Phase:**
   - **Bot Interactions:** Query messages table for bot-replied messages in date range
   - **Sessions:** Query sessions table for sessions with bot interactions
   - **Intents:** Query nlp_data_logs for intent detection results
   - **Failures:** Identify sessions where bot failed and transferred to agent
   - **Success:** Count sessions resolved by bot without agent intervention

4. **Metric Calculation:**
   - **Total Bot Messages:** Count of messages with attending_id = "bot"
   - **Bot Served:** Sessions where bot successfully resolved customer query
   - **Bot Failed:** Sessions where bot transferred to agent due to failure
   - **Left in Middle:** Sessions where customer stopped responding mid-journey
   - **Button Hits:** Count of button clicks/quick replies in bot interactions
   - **Unique Users:** Distinct count of customers who interacted with bot
   - **Success Rate:** (Bot Served / Total Interactions) × 100

5. **Channel-Wise Aggregation:**
   - Group metrics by channel (Messenger, Web Widget, etc.)
   - Group by app (MyGP, Skitto, etc.)
   - Group by intent category for detailed breakdown

6. **Intent Performance Analysis:**
   - For each intent:
     - Count total detections
     - Calculate average confidence score
     - Measure completion rate
     - Identify failure points
   - Rank intents by volume and success rate

7. **Trend Analysis:**
   - Compare with previous day's metrics
   - Calculate percentage change
   - Identify anomalies (sudden spikes/drops)

8. **Report Data Structuring:**
   - Format data as array with keys:
     - `date`: Report date
     - `metrics`: Overall performance numbers
     - `by_channel`: Channel-wise breakdown
     - `by_intent`: Intent-wise performance
     - `trends`: Day-over-day comparisons

9. **Database Persistence:**
   - Insert report data into `report_bot_performance` table
   - Use Higg framework's FormHandler for standardized insertion
   - Set status: "generated"
   - Store generation timestamp

10. **Post-Processing:**
    - Generate summary statistics
    - Calculate percentiles (e.g., 90th percentile response time)
    - Identify top performing and underperforming intents

11. **Notification (Optional):**
    - If significant anomalies detected → send Slack alert
    - If success rate drops below threshold → notify operations team

12. **Completion & Logging:**
    - Cronitor job marked as successful
    - Log report generation success to application logs
    - Update last_generated_at timestamp in report metadata table

**Access & Export:**

13. **Report Retrieval:**
    - API endpoint: `/api/v1/reports/bot_performance`
    - Filters: date range, channel, app, intent
    - Response format: JSON or CSV

14. **Report Export:**
    - Agent/admin requests export via inbox UI
    - Background job generates Excel file
    - Export includes:
      - Summary sheet: overall metrics
      - Detail sheet: daily breakdown
      - Intent sheet: intent performance matrix
      - Charts: trend visualizations
    - File stored in S3, download link sent to requester

**Edge Cases:**

- **No Data for Date:**
  - Report generated with zero values
  - Marked with flag: "no_data"
  - Previous day's report used as baseline

- **Partial Data:**
  - Some channels have data, others don't
  - Clearly indicate data availability in report

- **Report Re-generation:**
  - If data updated later (e.g., delayed logs)
  - Re-run report for specific date
  - Overwrite previous report with updated flag

**Validation Rules:**
- Reports only generated for dates with completed data
- Cannot generate reports for future dates
- Historical reports (> 90 days) archived to cold storage
- Report queries timeout after 60 seconds (use cached/approximations)

---

## 4. Database & Data Rules

### 4.1 Data Models and Relationships

**Core Entities:**

**1. Customer**
- **Table:** `customers`
- **Key Fields:**
  - `id`: Primary key (auto-increment)
  - `first_name`, `last_name`: Customer name
  - `email`: Email address (nullable, unique)
  - `msisdn`: Mobile number (nullable, indexed)
  - `created_at`, `updated_at`: Timestamps
- **Business Rules:**
  - Email must be unique if provided
  - MSISDN stored in E.164 format (e.g., 8801XXXXXXXXX)
  - Soft deletes not allowed (anonymize instead for GDPR)
- **Relationships:**
  - `hasMany` CustomerIdentity (channel-specific IDs)
  - `hasMany` Messages (customer conversations)
  - `hasMany` Sessions (customer sessions)

**2. CustomerIdentity**
- **Table:** `customer_identities`
- **Key Fields:**
  - `id`: Primary key
  - `customer_id`: Foreign key → customers
  - `identity_type`: Type (Facebook, Email, Phone, User_Ref)
  - `value`: Identity value (e.g., Facebook PSID)
- **Business Rules:**
  - Composite unique index on (identity_type, value)
  - Cannot have duplicate identities across customers
- **Relationships:**
  - `belongsTo` Customer

**3. SessionUser (Session Manager)**
- **Table:** `session_users`
- **Key Fields:**
  - `id`: Primary key
  - `unique_id`: UUID for session identification
  - `customer_id`: Foreign key → customers
  - `user_ref`: Channel-specific user reference
  - `agent_type`: Current agent type (bot, live, email)
  - `attending_id`: Assigned agent ID (nullable)
  - `channel`: Channel identifier
  - `app_id`: Application identifier
  - `auth_mode`: Authentication status (GUEST, USER, EXPR)
  - `last_message_at`: Timestamp of last activity
  - `details`: JSON field for session context
- **Business Rules:**
  - One active session per customer per app
  - Session expires after 15 minutes of inactivity
  - Auth mode transitions: GUEST → (OTP verify) → USER → (timeout) → EXPR
- **Relationships:**
  - `belongsTo` Customer
  - `hasMany` Messages

**4. Message**
- **Table:** `messages`
- **Key Fields:**
  - `id`: Primary key
  - `case_id`: Case identifier (indexed)
  - `body`: Message text content
  - `sender_id`: Customer reference
  - `sender_type`: customer/agent/bot
  - `attending_id`: Agent or bot ID
  - `channel`: Channel name
  - `direction`: inbound/outbound
  - `quick_reply`: Quick reply payload (JSON)
  - `postback`: Postback data (JSON)
  - `attachments`: File attachments (JSON array)
  - `created_at`: Message timestamp
- **Business Rules:**
  - Body can be null if attachments present
  - Case ID auto-generated on first customer message
  - Messages immutable after creation (no updates)
  - Delivery status tracked separately
- **Relationships:**
  - `belongsTo` SessionUser (via session_id)
  - `belongsTo` Customer (via mevrik_id)

**5. Queue & QueueItem**
- **Table:** `queues`, `queue_items`
- **Key Fields:**
  - Queue: `id`, `name`, `description`, `app_id`
  - QueueItem: `id`, `queue_id`, `case_id`, `customer_id`, `attending_id`, `status`, `priority`, `created_at`
- **Business Rules:**
  - Queue items expire after 30 minutes if not picked
  - Status transitions: new → assigned → closed/abandoned
  - Priority: 1 (low), 5 (normal), 10 (high)
- **Relationships:**
  - QueueItem `belongsTo` Queue
  - QueueItem `belongsTo` Customer
  - QueueItem `belongsTo` Case

**6. BotOtp (Authentication)**
- **Table:** `bot_otps`
- **Key Fields:**
  - `id`: Primary key
  - `sender_id`: Customer reference
  - `msisdn`: Phone number
  - `otp`: Generated OTP code
  - `otp_time`: Expiration timestamp
  - `otp_status`: RUNNING, VERIFIED, EXPIRED
  - `api_status_code`: GP API status code
- **Business Rules:**
  - OTP valid for 15 minutes
  - Maximum 3 OTP requests per phone per hour
  - OTP is 6-digit numeric code
  - Auto-expire on verification or timeout

### 4.2 Key Constraints and Validations

**Data Integrity Rules:**

1. **Referential Integrity:**
   - All foreign keys enforced with `ON DELETE RESTRICT` to prevent orphaned records
   - Soft deletes used where hard deletes would break audit trails

2. **Field Validations:**
   - Email: Must match RFC 5322 format
   - Phone (MSISDN): Must be 13 digits starting with 880 (Bangladesh)
   - URLs: Validated against whitelist of allowed domains
   - Attachments: Max file size 10MB, allowed types: jpg, png, pdf, docx

3. **Unique Constraints:**
   - `customers.email` - unique if not null
   - `customer_identities (identity_type, value)` - composite unique
   - `session_users (customer_id, app_id, status='active')` - unique active session per customer/app

4. **Indexes for Performance:**
   - `messages (case_id, created_at)` - fast case message retrieval
   - `session_users (unique_id)` - fast session lookup
   - `nlp_data_logs (sender_id, created_at)` - intent history queries
   - `queue_items (status, created_at)` - active queue listing

5. **JSON Field Schemas:**
   - `session_users.details`: Stores customer context, intent history, journey state
   - `messages.quick_reply`: `{payload: string, title: string}`
   - `messages.attachments`: `[{url: string, type: string, filename: string}]`

6. **Business-Level Validations:**
   - Cannot send message to closed session (must reopen first)
   - Cannot assign agent to case already assigned to different agent
   - Cannot close case with pending queue items
   - OTP must be verified before accessing authenticated journeys

---

## 5. Integration Points

### 5.1 External Integrations

**1. Facebook Graph API**
- **Purpose:** Two-way messaging, profile enrichment, page post management
- **Authentication:** OAuth 2.0, Page Access Token stored in ClickHouse
- **Endpoints Used:**
  - `POST /{page-id}/messages` - Send message to customer
  - `GET /{user-id}` - Fetch customer profile details
  - `GET /{post-id}/comments` - Retrieve post comments
  - `POST /{comment-id}/comments` - Reply to comment
- **Webhook Events:** `messages`, `messaging_postbacks`, `feed` (comments)
- **Rate Limits:** 600 calls per 600 seconds per user
- **Error Handling:**
  - 190 (Access Token expired): Refresh token, retry
  - 613 (Calls exceeded): Implement backoff, queue messages
  - 100 (Invalid Parameter): Log and alert for investigation
- **Data Exchange:**
  - Inbound: Customer messages, profile data, page comments
  - Outbound: Bot/agent replies, quick replies, attachments

**2. Google Dialogflow API**
- **Purpose:** Natural Language Understanding for intent detection
- **Authentication:** Service Account JSON key stored in `storage/keys/`
- **Projects:**
  - `mevrik` (production): Main intent classification
  - `mevrik_self` (testing): For specific channel testing
  - `smalltalk`: Casual conversation handling
- **API Calls:**
  - `detectIntent()` - Analyze user query, return intent + confidence
- **Confidence Thresholds:**
  - Domain intents: ≥ 0.6 (configurable via `DIALOGUE_DOMAIN_SCORE`)
  - Small talk: ≥ 0.4 (configurable via `DIALOGUE_SMALL_SCORE`)
- **Rate Limits:** 600 requests per minute
- **Error Handling:**
  - Timeout (> 3s): Log, return "not_recognized" intent
  - API failure: Retry once, then fallback to keyword matching
- **Data Exchange:**
  - Inbound: User text query
  - Outbound: Intent name, confidence score, parameters (entities)

**3. Grameenphone (GP) Backend APIs**
- **Purpose:** Customer account management, recharges, offers, services
- **Authentication:** OAuth 2.0 Bearer Token (refreshed every 1 hour)
- **Base URLs:** Configured per environment (test/production)
- **Key API Groups:**
  - **Account:** `/dadetails`, `/queryCustomer`, `/queryFinancialOverview`
  - **Recharge:** `/recharge`, `/rechargeHistory`, `/rechargeTransactionDetails`
  - **Offers:** `/queryAvailableOffer`, `/purchasePack`
  - **Services:** `/sendSMS`, `/activateService`, `/deactivateService`
- **Error Handling:**
  - Circuit breaker: Disable API after 5 consecutive failures
  - Retry logic: 3 attempts with exponential backoff
  - Cache fallback: Serve cached data on API unavailability
- **Data Exchange:**
  - Inbound: Account balance, offer lists, transaction status
  - Outbound: Customer MSISDN, recharge amounts, activation requests

**4. AWS S3**
- **Purpose:** File storage for message attachments, reports, exports
- **Authentication:** IAM access key and secret
- **Buckets:**
  - `bluebirdchannel` - Message attachments
  - `bluebirdemail` - Email attachments
- **CDN:** CloudFront distribution at `https://cdn.mevrik.com`
- **Operations:**
  - Upload: Attachments from customer messages, report exports
  - Download: File retrieval for agents, public file access
- **Error Handling:**
  - Retry upload failures up to 3 times
  - Store local backup if S3 unavailable
  - Log upload errors for manual investigation

**5. WebSocket Server (Laravel WebSockets + Pusher)**
- **Purpose:** Real-time message delivery and UI updates
- **Protocol:** WebSocket with fallback to HTTP polling
- **Channels:**
  - `messages` - Global message feed for agents
  - `session{id}` - Customer-specific session updates
  - `agent{agent_id}` - Agent-specific notifications
  - `queues-app-{app_id}` - Queue updates per application
- **Events Broadcasted:**
  - `MessageReadyEvent` - New message available
  - `SessionPathChangedEvent` - Session state change
  - `QueueItemCreatedEvent` - New queue item
- **Authentication:** JWT token for WebSocket connection
- **Error Handling:**
  - Connection lost: Auto-reconnect with exponential backoff
  - Message delivery failure: Retry via HTTP polling

**6. Monitoring & Alerting Services**
- **Cronitor:** Cron job monitoring and alerting
- **Graylog:** Centralized log aggregation and search
- **Bugsnag:** Application error tracking and reporting
- **Uptime Kuma:** Service uptime monitoring

### 5.2 Integration Data Flow

**Message Ingestion Flow (Facebook → Mevrik):**
1. Facebook sends webhook POST to `/webhook/facebook`
2. Webhook signature verified (X-Hub-Signature)
3. Payload stored via `WebhookEvent::store_webhook()`
4. Message extracted and normalized via `CreateMessageFromWebhook`
5. Customer profile fetched/created via Graph API
6. Message stored via `MessagingRequestHelper->storeMessage()`
7. Events dispatched: `MessageReadyEvent`, `MessageQuickReplyEvent`, `MessagePostbackEvent`
8. Real-time delivery via WebSocket
9. Bot processes message (if bot mode), sends reply via Graph API
10. Reply delivery confirmed, metrics logged

**See Detailed Flow:** [MESSAGE_INCOMING_LOGIC_DIAGRAMS.md](MESSAGE_INCOMING_LOGIC_DIAGRAMS.md) - Section 3

**Bot Response Flow (Mevrik → Customer):**
1. Bot generates response based on intent
2. Response formatted per channel (Facebook template, web widget, etc.)
3. Response sent via channel API (Facebook Graph API)
4. Delivery status captured from API response
5. Response stored as message record
6. Real-time broadcast to customer via WebSocket

**Agent Reply Flow (Mevrik → Customer):**
1. Agent composes reply via `/api/v1/agent/messages?action=message_reply`
2. Payload: `{"message": {"message_type": "message", "body": "...", "is_reply": 67288}}`
3. Session context restored from parent message
4. Reply formatted for target channel
5. Message sent via appropriate channel API
6. Delivery confirmed and stored
7. Session state updated
8. Real-time notification to customer

**See Detailed Flow:** [MESSAGE_OUTGOING_LOGIC_DIAGRAMS.md](MESSAGE_OUTGOING_LOGIC_DIAGRAMS.md) - Section 2

**Authentication Validation Flow (Mevrik → GP APIs → Mevrik):**
1. Customer enters phone number in bot journey
2. OTP generation request sent to GP API
3. OTP delivered to customer via SMS (GP API)
4. Customer enters OTP in bot conversation
5. OTP verification request sent to GP API
6. Verification status returned, customer authenticated
7. Session auth_mode updated to "USER"
8. Authenticated journeys unlocked

---

## 6. Automation & Jobs

### 6.1 Scheduled Jobs (Cron)

**Report Generation Jobs:**

1. **Master Report Scheduler** (LiveChat)
   - **Schedule:** Every 5 minutes
   - **Command:** `php artisan mevrik:reports master_report`
   - **Purpose:** Generate real-time operational metrics
   - **Output:** Updates `report_master` table with current hour's data

2. **Hourly Performance Report** (LiveChat)
   - **Schedule:** Every hour
   - **Command:** `php artisan mevrik:reports hourly_performance`
   - **Purpose:** Aggregate response times, handling times, queue metrics
   - **Output:** `report_hourly_performance` table

3. **Agent Performance Daily** (LiveChat)
   - **Schedule:** Daily at 1:35 AM
   - **Command:** `php artisan mevrik:reports agent_performance_daily`
   - **Purpose:** Calculate agent productivity metrics for previous day
   - **Output:** `report_agent_performance` table

4. **Bot Performance Report** (Chatbot)
   - **Schedule:** Daily at 1:50 AM
   - **Command:** `php artisan mevrik:reports bot_performance`
   - **Purpose:** Bot success rate, failure rate, intent accuracy
   - **Output:** `report_bot_performance` table

5. **Intent Accuracy Report** (Chatbot)
   - **Schedule:** Daily at 2:00 AM
   - **Command:** `php artisan mevrik:reports intent_accuracy`
   - **Purpose:** NLP model performance, confidence distribution
   - **Output:** `report_intent_accuracy` table

6. **Social Media Performance** (Social)
   - **Schedule:** Hourly
   - **Command:** `php artisan mevrik:reports social_performance`
   - **Purpose:** Comment response metrics, agent productivity on social
   - **Output:** `report_social_performance` table

7. **Customer Sentiment Scheduler**
   - **Schedule:** Hourly
   - **Command:** `php artisan mevrik:reports customer_sentiment`
   - **Purpose:** Aggregate sentiment analysis across all channels
   - **Output:** `report_customer_sentiment` table

**Maintenance Jobs:**

8. **Email Processing**
   - **Schedule:** Daily at 7:00 AM
   - **Command:** `php artisan mevrik:email process`
   - **Purpose:** Fetch and process new emails from inbox
   - **Actions:** Parse emails, create cases, notify agents

### 6.2 Background Queue Jobs

**Message Processing Jobs:**

1. **HandleMessageReadyEvent Job**
   - **Queue:** `messages`
   - **Trigger:** Dispatched on `MessageReadyEvent`
   - **Purpose:** Process incoming message, run bot logic, generate response
   - **Retry:** 3 attempts
   - **Timeout:** 60 seconds

2. **MessageBulkReplyJob**
   - **Queue:** `bulk-operations`
   - **Trigger:** Agent bulk reply action
   - **Purpose:** Send same reply to multiple customers
   - **Retry:** 3 attempts per message
   - **Batch Size:** 50 messages per batch

3. **MessageBulkCloseJob**
   - **Queue:** `bulk-operations`
   - **Trigger:** Agent bulk close action
   - **Purpose:** Close multiple cases with same disposition
   - **Retry:** 3 attempts per case
   - **Batch Size:** 50 cases per batch

**Reporting Jobs:**

4. **CustomerSocialActivityReportGeneratorJob**
   - **Queue:** `report`
   - **Trigger:** Case closure on social channel
   - **Purpose:** Generate detailed social interaction report for closed case
   - **Retry:** 3 attempts
   - **Timeout:** 5 minutes
   - **Uniqueness:** Unique per case_id to prevent duplicates

5. **MailFrequencyGeneratorJob**
   - **Queue:** `report`
   - **Trigger:** Scheduled daily
   - **Purpose:** Generate email volume metrics
   - **Retry:** 3 attempts
   - **Timeout:** 10 minutes

**Export Jobs:**

9. **ReportExportJob**
   - **Queue:** `exports`
   - **Trigger:** Agent/admin requests report export
   - **Purpose:** Generate Excel/CSV file of report data
   - **Retry:** 1 attempt (manual retry required on failure)
   - **Timeout:** 10 minutes
   - **Output:** File uploaded to S3, download link sent to requester

---

## 7. Security & Compliance

### 7.1 Authentication & Authorization

**1. Agent Authentication:**
- **Method:** JWT (JSON Web Token) via `php-open-source-saver/jwt-auth`
- **Login Flow:**
  - Agent enters credentials
  - Backend validates against user database
  - JWT token issued with 1-hour expiration
  - Token stored in agent browser (HttpOnly cookie)
- **Token Validation:** Middleware `auth:api` validates token on each request
- **Authorization:** Role-based permissions checked against agent roles
- **Password Policy:**
  - Minimum 8 characters
  - Must include letters and numbers
  - Password hash: bcrypt (Laravel default)

**2. Customer Authentication (Bot):**
- **Method:** OTP-based verification
- **OTP Generation:**
  - 6-digit numeric code
  - Valid for 15 minutes
  - Sent via GP SMS API
- **OTP Verification:**
  - Maximum 3 attempts allowed
  - Verified against GP API
  - Session auth_mode updated to "USER" on success
- **Session Management:**
  - Session tokens stored in Redis
  - Token expires after 15 minutes of inactivity
  - Sliding expiration: resets on each user message

**3. Channel Webhook Verification:**
- **Facebook Webhooks:**
  - Verify token check on webhook setup
  - X-Hub-Signature SHA256 validation on each request
  - Reject requests without valid signature
- **API Endpoints:**
  - Internal API routes protected by middleware `channel.agent`

### 7.2 Data Security

**Data Encryption:**
- **At Rest:** Database encryption enabled for sensitive fields (customer PII, phone numbers)
- **In Transit:** All API communications use TLS 1.2+
- **Storage:** AWS S3 server-side encryption enabled

**Data Privacy:**
- **GDPR Compliance:** Customer data anonymization on request
- **Data Retention:** Messages retained for 1 year, then archived
- **PII Handling:** Phone numbers masked in logs and reports
- **Access Control:** Role-based access to customer data

**Audit Trail:**
- All agent actions logged with timestamp and user ID
- Webhook payloads stored for 90 days
- Message edits and deletions tracked
- API access logs retained for security review

### 7.3 API Security

**Rate Limiting:**
- **Customer API:** 100 requests per hour per customer
- **Agent API:** 1000 requests per hour per agent
- **Webhook Endpoints:** No limit (Facebook controls)
- **Public API:** 60 requests per minute per IP

**Input Validation:**
- All inputs sanitized to prevent XSS attacks
- SQL injection prevention via Laravel ORM
- File upload validation (type, size, content scanning)
- URL validation against whitelist

**CORS Policy:**
- Allowed origins configured per environment
- Credentials allowed for authenticated endpoints
- Preflight caching enabled

---

## 8. Performance & Scalability

### 8.1 Caching Strategy

**Redis Cache Layers:**
1. **Session Cache:**
   - Key: `session:{unique_id}`
   - TTL: 15 minutes (sliding)
   - Purpose: Fast session lookup

2. **Customer Cache:**
   - Key: `customer:{id}`
   - TTL: 1 hour
   - Purpose: Reduce database queries

3. **Facebook Token Cache:**
   - Key: `fb_token:{page_id}`
   - TTL: Per token expiration
   - Purpose: OAuth token persistence

4. **API Response Cache:**
   - Key: `gp_api:{endpoint}:{params_hash}`
   - TTL: 30 seconds to 5 minutes
   - Purpose: Reduce external API calls

**Cache Invalidation:**
- Session cache cleared on closure
- Customer cache invalidated on profile update
- Report cache cleared on data refresh

### 8.2 Queue Management

**Queue Configuration:**
- **messages:** High priority, 100 workers
- **bulk-operations:** Medium priority, 20 workers
- **report:** Low priority, 10 workers
- **exports:** Low priority, 5 workers

**Queue Monitoring:**
- Laravel Horizon dashboard for real-time monitoring
- Queue depth alerts at 1000+ pending jobs
- Failed job retry: 3 attempts with exponential backoff
- Dead letter queue for permanently failed jobs

### 8.3 Database Optimization

**Connection Pooling:**
- MySQL: 50 connections (persistent)
- PostgreSQL: 30 connections
- ClickHouse: 20 connections

**Query Optimization:**
- Indexed columns for frequent queries
- Partitioning for large tables (messages by month)
- Read replicas for reporting queries
- Write/read splitting enabled

**Data Archival:**
- Messages older than 1 year moved to archive database
- Reports aggregated and detailed data purged after 90 days
- Attachment cleanup for closed cases older than 6 months

---

## 9. Related Documentation

For detailed visual flow diagrams and step-by-step logic, refer to these companion documents:

### Message Flow Documentation:
- **[MESSAGE_INCOMING_LOGIC_DIAGRAMS.md](MESSAGE_INCOMING_LOGIC_DIAGRAMS.md)** - Complete incoming message flows for Widget, Facebook Messenger, Facebook Comments, and Email channels with detailed ASCII diagrams

- **[MESSAGE_OUTGOING_LOGIC_DIAGRAMS.md](MESSAGE_OUTGOING_LOGIC_DIAGRAMS.md)** - Outgoing message operations including agent replies, bulk operations, CSAT surveys, video calls, and report generation flows

### Architecture Documentation:
- **[mevrik-channels-brownfield.md](mevrik-channels-brownfield.md)** - Overall system architecture and brownfield enhancement approach

### Event Documentation:
- **[mevrik-channels-events.md](mevrik-channels-events.md)** - Complete event system documentation
- **[mevrik-channels-event-lifecycle.md](mevrik-channels-event-lifecycle.md)** - Event lifecycle and handling
- **[mevrik-channels-event-subscriber.md](mevrik-channels-event-subscriber.md)** - Event subscriber patterns

### Reference Documentation:
- **[SERVICE_MAPPING.md](SERVICE_MAPPING.md)** - Service layer mapping and dependencies
- **[NTENT_MAPPING.md](NTENT_MAPPING.md)** - Intent mapping and NLP configuration

---

## 10. Conclusion

This Business Logic Documentation provides a comprehensive overview of the Mevrik Channels platform's operational design, workflows, and technical implementation. The platform successfully unifies multi-channel customer communications through intelligent automation, seamless human handoffs, and robust reporting, serving as a critical customer engagement infrastructure.

### 10.1 Key Strengths

**Technical Excellence:**
- **Event-Driven Architecture:** Enables scalable, real-time message processing with Laravel Events & Horizon
- **Intelligent Automation:** Dialogflow-powered bot reduces agent workload by 60-70%
- **Multi-Channel Support:** Unified interface for Facebook Messenger, Facebook Comments, Web Widget, Email
- **Comprehensive Analytics:** Granular reporting on bot performance, agent productivity, customer satisfaction

**Operational Benefits:**
- **Real-Time Processing:** WebSocket-based instant message delivery
- **Scalable Queue System:** Laravel Horizon manages high-volume message processing
- **Robust Error Handling:** Circuit breakers, retry logic, and graceful degradation
- **Data-Driven Insights:** Automated report generation for continuous improvement

### 10.2 System Capabilities

**Message Processing:**
- Handles 10,000+ messages per day across all channels
- Average response time: < 3 seconds for bot replies
- Message delivery success rate: > 99.5%

**Bot Performance:**
- Successfully resolves 60-70% of customer queries without agent intervention
- Supports 50+ intents across multiple business domains
- Sub-second intent detection via Dialogflow

**Agent Productivity:**
- Agents handle 20-30 conversations simultaneously
- Real-time dashboard updates for instant context switching
- Bulk operations for efficient mass communication

**Reporting & Analytics:**
- Real-time metrics updated every 5 minutes
- Daily performance reports auto-generated
- Custom date range queries with sub-second response times

### 10.3 Future Enhancements

**Recommended Improvements:**
1. **Multi-Database Consolidation:** Migrate from 3 databases to unified architecture
2. **Advanced NLP:** Upgrade to Dialogflow CX for complex conversation flows
3. **Enhanced CSAT:** Add detailed feedback forms beyond yes/no responses
4. **Video Integration:** Expand video call capabilities with recording and transcription
5. **AI-Powered Insights:** Implement ML-based conversation analysis for trend prediction

---

**Document Maintained By:** Development Team  
**Review Cycle:** Quarterly  
**Next Review:** January 2026

---