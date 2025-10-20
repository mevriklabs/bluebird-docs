# Mevrik Channels Events - Product Requirements Document

## Document Information
- **Version**: 1.0
- **Last Updated**: [Date]
- **Author**: [Author Name]
- **Reviewers**: [Reviewer Names]
- **Status**: [Draft/Review/Approved]

## Table of Contents
1. [Overview](#overview)
2. [System Context](#system-context)
3. [Event Architecture](#event-architecture)
4. [Message Events](#message-events)
5. [Session Events](#session-events)
6. [Case Events](#case-events)
7. [Customer Events](#customer-events)
8. [Queue Events](#queue-events)
9. [Social Media Events](#social-media-events)
10. [Event Processing](#event-processing)
11. [Broadcasting & Real-time Communication](#broadcasting--real-time-communication)
12. [Event Handlers & Listeners](#event-handlers--listeners)
13. [Performance & Scalability](#performance--scalability)
14. [Security & Reliability](#security--reliability)
15. [Monitoring & Debugging](#monitoring--debugging)
16. [Future Enhancements](#future-enhancements)

## Overview

### Purpose
The Mevrik Channels Events system provides a comprehensive event-driven architecture for the omnichannel customer experience platform. It enables real-time communication, asynchronous processing, and decoupled system components through a robust event system built on Laravel's event broadcasting capabilities.

### Scope
- Message lifecycle events (creation, replies, delivery)
- Session management events (creation, updates, state changes)
- Case management events (creation, closure, status changes)
- Customer profile events (creation, updates, verification)
- Queue management events (creation, assignment, processing)
- Social media integration events (webhook processing, content updates)
- Real-time broadcasting to connected clients
- Event-driven business logic processing

### Key Stakeholders
- **Customers**: Real-time message delivery and status updates
- **Customer Service Agents**: Live notifications and queue updates
- **System Administrators**: Event monitoring and system health
- **Developers**: Event-driven integration and extension points

## System Context

### Event System Role
```
┌─────────────────────────────────────────────────────────────┐
│                    External Triggers                        │
├─────────────────────────────────────────────────────────────┤
│  Web Widgets  │  Facebook  │  Email  │  Social Media       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                Event System Core                            │
├─────────────────────────────────────────────────────────────┤
│  Event Dispatch  │  Event Broadcasting  │  Event Processing │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Event Consumers                          │
├─────────────────────────────────────────────────────────────┤
│  Real-time Clients │  Background Jobs │  External Systems   │
└─────────────────────────────────────────────────────────────┘
```

### Integration Points
- **Laravel Events**: Core event dispatching and handling
- **Laravel Broadcasting**: Real-time event broadcasting
- **Queue System**: Asynchronous event processing
- **WebSocket Connections**: Real-time client communication
- **External APIs**: Third-party service integrations

## Event Architecture

### Core Components

#### 1. Event Classes
- **Base Event Structure**: Laravel event classes with broadcasting capabilities
- **Event Data**: Structured event payloads with relevant model data
- **Broadcasting Channels**: Dynamic channel assignment based on context

#### 2. Event Dispatching
- **Synchronous Dispatch**: Immediate event processing for critical operations
- **Asynchronous Dispatch**: Background job processing for non-critical events
- **Conditional Dispatch**: Environment-based event dispatching

#### 3. Event Broadcasting
- **Channel Management**: Dynamic channel creation and management
- **Real-time Delivery**: WebSocket-based real-time event delivery
- **Channel Security**: Secure channel access and authentication

### Event Categories

The system supports 15+ distinct event types organized into categories:

#### Message Events (6)
1. **MessageReadyEvent** - New customer message ready for processing
2. **MessageReplyReadyEvent** - Agent reply ready for delivery
3. **MessagePostbackEvent** - Postback interaction received
4. **MessageQuickReplyEvent** - Quick reply interaction received
5. **NewMessageReceivedEvent** - New message received from external source
6. **MessageDeliveryFailedEvent** - Message delivery failure

#### Session Events (2)
7. **SessionCreatedEvent** - New customer session created
8. **SessionPathChangedEvent** - Session context/path changed

#### Case Events (3)
9. **MevrikCaseCreatedEvent** - New support case created
10. **MevrikCaseClosedEvent** - Support case closed
11. **MevrikCSATStartEvent** - CSAT survey initiated

#### Customer Events (1)
12. **NewCustomerProfileCreated** - New customer profile created

#### Queue Events (1)
13. **QueueItemCreatedEvent** - New queue item created for agent assignment

#### Social Media Events (2)
14. **NewFacebookEventReceived** - New Facebook webhook event received
15. **SaveAgentRepliesSocialEvent** - Agent reply to social media content

## Message Events

### 1. MessageReadyEvent
**Purpose**: Signal that a new customer message is ready for processing

**Trigger Conditions**:
- Customer sends message via web widget, Facebook, or email
- Message successfully stored in database
- Message is not an agent reply

**Event Data**:
```php
public Message $message;
```

**Broadcasting Channels**:
- `messages` - Global message channel
- `session{id}` - Session-specific channel
- `agent{agent_type}` - Agent-specific channel

**Broadcast Event Name**: `message_post`

**Processing Flow**:
1. Message stored in database
2. Event dispatched with message data
3. Real-time notification sent to connected clients
4. Background processing triggered for bot/agent response

### 2. MessageReplyReadyEvent
**Purpose**: Signal that an agent reply is ready for delivery to customer

**Trigger Conditions**:
- Agent sends reply to customer message
- Reply message successfully stored
- Message has `is_reply` flag set

**Event Data**:
```php
public Message $message;
```

**Broadcasting Channels**:
- `messages` - Global message channel
- `session{id}` - Session-specific channel
- `agent{agent_type}` - Agent-specific channel

**Broadcast Event Name**: `message_reply`

**Processing Flow**:
1. Agent reply stored in database
2. Event dispatched with reply data
3. Real-time notification sent to customer
4. Delivery confirmation processed

### 3. MessagePostbackEvent
**Purpose**: Handle postback interactions from message buttons

**Trigger Conditions**:
- Customer clicks postback button in message
- Postback data received and validated

**Event Data**:
```php
public Message $message;
public array $postback;
```

**Processing Flow**:
1. Postback interaction received
2. Event dispatched with message and postback data
3. Postback handler processes interaction
4. Appropriate response generated

### 4. MessageQuickReplyEvent
**Purpose**: Handle quick reply interactions from message options

**Trigger Conditions**:
- Customer selects quick reply option
- Quick reply data received and validated

**Event Data**:
```php
public Message $message;
public array $quick_reply;
```

**Processing Flow**:
1. Quick reply interaction received
2. Event dispatched with message and reply data
3. Quick reply handler processes selection
4. Contextual response generated

### 5. NewMessageReceivedEvent
**Purpose**: Process new messages received from external sources

**Trigger Conditions**:
- Webhook receives new message from external channel
- Message data validated and parsed

**Event Data**:
```php
public MessagingRequestHelper $messageContainer;
```

**Processing Flow**:
1. External message received via webhook
2. Message container created with payload
3. Event dispatched for processing
4. Message routing and handling initiated

### 6. MessageDeliveryFailedEvent
**Purpose**: Handle message delivery failures

**Trigger Conditions**:
- Message delivery attempt fails
- Retry attempts exhausted
- Delivery error detected

**Event Data**:
```php
public Message $message;
public string $error;
public array $context;
```

**Processing Flow**:
1. Delivery failure detected
2. Event dispatched with error details
3. Failure handling initiated
4. Alternative delivery methods attempted

## Session Events

### 1. SessionCreatedEvent
**Purpose**: Signal creation of new customer session

**Trigger Conditions**:
- New customer starts interaction
- Session data successfully created
- Customer authentication completed

**Event Data**:
```php
public SessionUser $sessionUser;
```

**Processing Flow**:
1. Customer session created
2. Event dispatched with session data
3. Session initialization completed
4. Welcome message or bot interaction initiated

### 2. SessionPathChangedEvent
**Purpose**: Handle session context or path changes

**Trigger Conditions**:
- Session agent type changes (bot ↔ live)
- Session context updates
- Case assignment changes

**Event Data**:
```php
public SessionUser $sessionUser;
public string $oldPath;
public string $newPath;
```

**Processing Flow**:
1. Session path/context changes
2. Event dispatched with change details
3. UI updates triggered for connected clients
4. Agent handover or context switch processed

## Case Events

### 1. MevrikCaseCreatedEvent
**Purpose**: Signal creation of new support case

**Trigger Conditions**:
- Customer requires support assistance
- Case data successfully created
- Case assigned to appropriate queue

**Event Data**:
```php
public MevrikCase $case;
```

**Processing Flow**:
1. Support case created
2. Event dispatched with case data
3. Case routing and assignment initiated
4. Agent notification sent if applicable

### 2. MevrikCaseClosedEvent
**Purpose**: Signal closure of support case

**Trigger Conditions**:
- Case resolution completed
- Agent closes case with disposition
- Case timeout or automatic closure

**Event Data**:
```php
public MevrikCase $case;
```

**Broadcasting Channels**:
- `messages` - Global message channel
- `session{id}` - Session-specific channel
- `agent{agent_type}` - Agent-specific channel

**Broadcast Event Name**: `case_closed`

**Processing Flow**:
1. Case marked as closed
2. Event dispatched with case data
3. CSAT survey triggered
4. Session reset to bot mode

### 3. MevrikCSATStartEvent
**Purpose**: Signal initiation of customer satisfaction survey

**Trigger Conditions**:
- Case successfully closed
- CSAT survey triggered
- Customer eligible for survey

**Event Data**:
```php
public MevrikCase $case;
```

**Broadcasting Channels**:
- `messages` - Global message channel
- `session{id}` - Session-specific channel
- `agent{agent_type}` - Agent-specific channel

**Broadcast Event Name**: `case_start`

**Processing Flow**:
1. CSAT survey initiated
2. Event dispatched with case data
3. Survey sent to customer
4. Response tracking initiated

## Customer Events

### 1. NewCustomerProfileCreated
**Purpose**: Signal creation of new customer profile

**Trigger Conditions**:
- New customer registers or is identified
- Customer profile data created
- Customer verification completed

**Event Data**:
```php
public MevrikCustomer $customer;
```

**Processing Flow**:
1. Customer profile created
2. Event dispatched with customer data
3. Customer onboarding initiated
4. Profile enrichment triggered

## Queue Events

### 1. QueueItemCreatedEvent
**Purpose**: Signal creation of new queue item for agent assignment

**Trigger Conditions**:
- Customer requires human agent assistance
- Queue item created for agent assignment
- Agent transfer initiated

**Event Data**:
```php
public QueueItem $queueItem;
```

**Broadcasting Channels**:
- `queues` - Global queue channel
- `queues-app-{app_id}` - App-specific queue channel

**Broadcast Event Name**: `queue_created`

**Processing Flow**:
1. Queue item created
2. Event dispatched with queue data
3. Agent notification sent
4. Queue management updated

## Social Media Events

### 1. NewFacebookEventReceived
**Purpose**: Process new Facebook webhook events

**Trigger Conditions**:
- Facebook webhook receives new event
- Event data validated and logged
- Event ready for processing

**Event Data**:
```php
public WebhookEventLog $webhookEventLog;
```

**Processing Flow**:
1. Facebook webhook event received
2. Event logged in webhook event log
3. Event dispatched for processing
4. Social media interaction handled

### 2. SaveAgentRepliesSocialEvent
**Purpose**: Handle agent replies to social media content

**Trigger Conditions**:
- Agent replies to social media post/comment
- Reply content validated
- Social media API integration ready

**Event Data**:
```php
public Message $message;
public array $socialData;
```

**Processing Flow**:
1. Agent reply to social media
2. Event dispatched with reply data
3. Social media API called
4. Reply posted to social platform

## Event Processing

### HandleMessageReadyEvent Action

The `HandleMessageReadyEvent` action serves as the central message processing hub:

#### Core Functionality
1. **Message Validation**: Validates message data and session context
2. **Session Management**: Creates or updates customer sessions
3. **Channel-Specific Processing**: Handles different channel types (Bioscope, Sign Line, Prelab)
4. **Bot vs Agent Routing**: Determines whether to route to bot or human agent
5. **CSAT Processing**: Handles customer satisfaction survey interactions
6. **Queue Management**: Creates queue items for agent assignment

#### Processing Logic
```php
function handle(Message $message) {
    // 1. Message post-processing
    $message->postRead();
    
    // 2. Session validation and creation
    if (mevrik()->invalidSession()) {
        mevrik()->createSessionFromMessage($message);
    }
    
    // 3. Channel-specific ticket creation
    if ($message->isMessageType('video_call_request')) {
        $this->create_ticket_for_sign_line($message);
        return null;
    }
    
    // 4. Bot mode processing
    if (!$message->isAgentReply() && $session->isAgentBot()) {
        // Handle bot interactions, CSAT, and agent transfers
    }
}
```

#### Channel-Specific Processing
- **Bioscope Channel**: Automatic queue creation for all messages
- **Sign Line Channel**: Video call request handling
- **Prelab App (ID: 13)**: Automatic agent transfer
- **Standard Channels**: Bot processing with agent transfer on demand

## Broadcasting & Real-time Communication

### Broadcasting Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Event Broadcasting                       │
├─────────────────────────────────────────────────────────────┤
│  Laravel Broadcasting  │  WebSocket Server  │  Redis Pub/Sub │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Connected Clients                        │
├─────────────────────────────────────────────────────────────┤
│  Customer Widgets  │  Agent Dashboards  │  Admin Panels    │
└─────────────────────────────────────────────────────────────┘
```

### Channel Types

#### 1. Global Channels
- **`messages`**: All message-related events
- **`queues`**: All queue-related events
- **`sessions`**: All session-related events

#### 2. Session-Specific Channels
- **`session{id}`**: Events for specific customer session
- **`user{id}`**: Events for specific customer
- **`case{id}`**: Events for specific support case

#### 3. Agent-Specific Channels
- **`agent{agent_type}`**: Events for specific agent type
- **`agent{agent_id}`**: Events for specific agent
- **`queues-app-{app_id}`**: App-specific queue events

### Real-time Features
1. **Live Message Updates**: Real-time message delivery to customers
2. **Agent Notifications**: Instant notifications for new messages and queue items
3. **Session State Updates**: Real-time session context changes
4. **Queue Management**: Live queue updates for agents
5. **Case Status Updates**: Real-time case status changes

## Event Handlers & Listeners

### Event Handler Structure
```php
class EventHandler {
    public function handle(Event $event) {
        // Event processing logic
        // Database updates
        // External API calls
        // Notification sending
    }
}
```

### Listener Registration
```php
// In EventServiceProvider
protected $listen = [
    MessageReadyEvent::class => [
        HandleMessageReadyEvent::class,
        LogMessageEvent::class,
    ],
    SessionCreatedEvent::class => [
        InitializeCustomerSession::class,
        SendWelcomeMessage::class,
    ],
];
```

### Event Processing Patterns
1. **Synchronous Processing**: Immediate event handling for critical operations
2. **Asynchronous Processing**: Background job processing for non-critical events
3. **Event Chaining**: Events triggering other events in sequence
4. **Conditional Processing**: Environment-based event handling

## Performance & Scalability

### Optimization Strategies
1. **Event Batching**: Batch multiple events for efficient processing
2. **Selective Broadcasting**: Only broadcast to relevant channels
3. **Event Caching**: Cache frequently accessed event data
4. **Queue Optimization**: Optimize background job processing

### Scalability Measures
1. **Horizontal Scaling**: Multiple event processing workers
2. **Channel Partitioning**: Distribute channels across multiple servers
3. **Load Balancing**: Distribute event processing load
4. **Database Optimization**: Optimize event-related database queries

### Performance Metrics
- **Event Processing Time**: < 100ms for simple events
- **Broadcasting Latency**: < 50ms for real-time delivery
- **Throughput**: Support 10,000+ events per minute
- **Channel Capacity**: Support 1,000+ concurrent channels

## Security & Reliability

### Security Measures
1. **Channel Authentication**: Secure channel access and validation
2. **Event Data Validation**: Validate all event data before processing
3. **Access Control**: Role-based access to event channels
4. **Audit Logging**: Log all event processing activities

### Reliability Features
1. **Event Retry Logic**: Automatic retry for failed event processing
2. **Dead Letter Queues**: Handle failed events gracefully
3. **Event Ordering**: Maintain event processing order where critical
4. **Failure Recovery**: Graceful handling of system failures

### Error Handling
1. **Event Validation**: Validate event data before processing
2. **Exception Handling**: Comprehensive exception handling
3. **Fallback Mechanisms**: Fallback processing for critical events
4. **Monitoring**: Real-time monitoring of event processing health

## Monitoring & Debugging

### Event Monitoring
1. **Event Metrics**: Track event processing performance
2. **Channel Monitoring**: Monitor channel health and usage
3. **Error Tracking**: Track and alert on event processing errors
4. **Performance Monitoring**: Monitor event processing performance

### Debugging Tools
1. **Event Logging**: Comprehensive event logging
2. **Event Tracing**: Trace event flow through system
3. **Channel Inspection**: Inspect channel state and connections
4. **Performance Profiling**: Profile event processing performance

### Health Checks
1. **Event Processing Health**: Monitor event processing system health
2. **Channel Health**: Monitor WebSocket channel health
3. **Queue Health**: Monitor background job queue health
4. **Database Health**: Monitor event-related database performance

## Future Enhancements

### Short-term Improvements
1. **Event Analytics**: Enhanced event analytics and reporting
2. **Event Replay**: Ability to replay events for debugging
3. **Event Filtering**: Advanced event filtering and routing
4. **Performance Optimization**: Further performance optimizations

### Medium-term Features
1. **Event Sourcing**: Implement event sourcing for audit trails
2. **Event Versioning**: Support for event schema versioning
3. **Event Compression**: Compress event data for efficiency
4. **Advanced Routing**: More sophisticated event routing logic

### Long-term Vision
1. **Microservices Events**: Event-driven microservices architecture
2. **Event Streaming**: Real-time event streaming capabilities
3. **Machine Learning**: ML-powered event processing and routing
4. **Global Distribution**: Multi-region event distribution

---

## Appendices

### Appendix A: Event Reference
| Event | Purpose | Broadcasting | Data | Trigger |
|-------|---------|--------------|------|---------|
| MessageReadyEvent | New message ready | Yes | Message | Customer message |
| MessageReplyReadyEvent | Agent reply ready | Yes | Message | Agent reply |
| MessagePostbackEvent | Postback interaction | No | Message, Postback | Button click |
| MessageQuickReplyEvent | Quick reply interaction | No | Message, QuickReply | Quick reply |
| NewMessageReceivedEvent | External message | No | MessageContainer | Webhook |
| MessageDeliveryFailedEvent | Delivery failure | No | Message, Error | Delivery fail |
| SessionCreatedEvent | New session | No | SessionUser | Session creation |
| SessionPathChangedEvent | Session change | No | SessionUser | Path change |
| MevrikCaseCreatedEvent | New case | No | MevrikCase | Case creation |
| MevrikCaseClosedEvent | Case closed | Yes | MevrikCase | Case closure |
| MevrikCSATStartEvent | CSAT started | Yes | MevrikCase | CSAT initiation |
| NewCustomerProfileCreated | New customer | No | MevrikCustomer | Profile creation |
| QueueItemCreatedEvent | New queue item | Yes | QueueItem | Queue creation |
| NewFacebookEventReceived | Facebook event | No | WebhookEventLog | Webhook |
| SaveAgentRepliesSocialEvent | Social reply | No | Message, SocialData | Social reply |

### Appendix B: Broadcasting Channels
| Channel Pattern | Purpose | Access | Example |
|-----------------|---------|--------|---------|
| messages | Global messages | Public | messages |
| session{id} | Session-specific | Authenticated | session123 |
| agent{type} | Agent-specific | Authenticated | agentlive |
| queues | Global queues | Authenticated | queues |
| queues-app-{id} | App-specific queues | Authenticated | queues-app-13 |
| user{id} | User-specific | Authenticated | user456 |
| case{id} | Case-specific | Authenticated | case789 |

### Appendix C: Event Processing Flow
```
1. Event Triggered
   ↓
2. Event Validation
   ↓
3. Event Dispatch
   ↓
4. Event Broadcasting (if applicable)
   ↓
5. Event Handler Execution
   ↓
6. Database Updates
   ↓
7. External API Calls
   ↓
8. Notification Sending
   ↓
9. Event Completion
```

### Appendix D: Configuration Parameters
| Parameter | Description | Default | Environment |
|-----------|-------------|---------|-------------|
| EVENT_BROADCAST_ENABLED | Enable event broadcasting | true | Production |
| EVENT_QUEUE_ENABLED | Enable event queuing | true | Production |
| EVENT_RETRY_ATTEMPTS | Max retry attempts | 3 | Production |
| EVENT_TIMEOUT | Event processing timeout | 30 | Production |
| CHANNEL_CLEANUP_INTERVAL | Channel cleanup interval | 3600 | Production |
| MAX_CONCURRENT_EVENTS | Max concurrent events | 1000 | Production |

---

*This document is a living document and should be updated as the event system evolves.*
