# Mevrik Channels Event Lifecycle

## Document Information
- **Version**: 1.0
- **Last Updated**: [Date]
- **Author**: [Author Name]
- **Reviewers**: [Reviewer Names]
- **Status**: [Draft/Review/Approved]

## Table of Contents
1. [Overview](#overview)
2. [System Context](#system-context)
3. [Event Lifecycle Architecture](#event-lifecycle-architecture)
4. [Complete Message Flow](#complete-message-flow)
5. [Event Sequence Diagrams](#event-sequence-diagrams)
6. [Channel-Specific Flows](#channel-specific-flows)
7. [Broadcasting Strategy](#broadcasting-strategy)
8. [Data Flow & State Transitions](#data-flow--state-transitions)
9. [Error Handling & Recovery](#error-handling--recovery)
10. [Performance Considerations](#performance-considerations)
11. [Monitoring & Observability](#monitoring--observability)
12. [Implementation Guidelines](#implementation-guidelines)

## Overview

### Purpose
This document defines the complete event lifecycle for the Mevrik Channels PHP omnichannel system, providing a comprehensive view of how events flow through the system from initial customer interaction to final CSAT collection. It serves as a technical specification for understanding, implementing, and maintaining the event-driven architecture.

### Scope
- Complete message flow from customer to case closure
- Event sequencing and dependencies
- Broadcasting logic and real-time communication
- Channel-specific processing variations
- Error handling and recovery mechanisms
- Performance optimization strategies

### Key Stakeholders
- **Development Teams**: Implementation and maintenance guidance
- **System Architects**: Architecture understanding and design decisions
- **DevOps Engineers**: Deployment and monitoring strategies
- **Product Managers**: Feature understanding and business logic
- **QA Teams**: Testing scenarios and validation criteria

## System Context

### Event-Driven Architecture Overview
```
┌─────────────────────────────────────────────────────────────┐
│                    Customer Touchpoints                     │
├─────────────────────────────────────────────────────────────┤
│  Web Widgets  │  Facebook Messenger  │  Email  │  Social    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                Event Processing Engine                      │
├─────────────────────────────────────────────────────────────┤
│  Event Dispatch  │  Event Broadcasting  │  Event Handlers   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Business Logic Layer                     │
├─────────────────────────────────────────────────────────────┤
│  Bot Processing  │  Agent Management  │  Case Management    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Real-time Delivery                       │
├─────────────────────────────────────────────────────────────┤
│  WebSocket Clients │  Agent Dashboards │  Admin Panels      │
└─────────────────────────────────────────────────────────────┘
```

### Event Categories & Flow
- **Input Events**: Customer interactions and external webhooks
- **Processing Events**: Internal system processing and routing
- **Output Events**: Agent notifications and customer responses
- **Lifecycle Events**: Case creation, updates, and closure
- **Feedback Events**: CSAT surveys and customer satisfaction

## Event Lifecycle Architecture

### Core Event Flow Principles
1. **Event-Driven Processing**: All system interactions triggered by events
2. **Asynchronous Processing**: Non-blocking event handling for performance
3. **Real-time Broadcasting**: Immediate notification to relevant parties
4. **State Management**: Consistent state transitions through events
5. **Error Recovery**: Graceful handling of failures and retries

### Event Processing Pipeline
```
Event Trigger → Validation → Dispatch → Processing → Broadcasting → Completion
     ↓              ↓           ↓           ↓            ↓            ↓
  Webhook      Data Check   Event Queue   Handler    Real-time    Logging
  Customer     Format       Background    Execution  Delivery    Analytics
  Agent        Security     Job           Business   WebSocket   Monitoring
```

## Complete Message Flow

### Phase 1: Customer Message Initiation

#### 1.1 Customer Sends Message
**Trigger**: Customer interaction via any channel (web widget, Facebook, email, social media)

**Event Sequence**:
```
Customer Action → Channel Webhook → NewMessageReceivedEvent → MessageReadyEvent
```

**Detailed Flow**:
1. **Customer Interaction**
   - Customer types message in web widget
   - Customer sends Facebook message
   - Customer sends email
   - Customer posts social media comment

2. **Webhook Processing**
   - External channel sends webhook to system
   - `NewMessageReceivedEvent` dispatched
   - Message data validated and parsed
   - `MessagingRequestHelper` container created

3. **Message Storage**
   - Message stored in database
   - Session validation/creation
   - `MessageReadyEvent` dispatched
   - Real-time broadcasting initiated

**Event Data Flow**:
```php
// NewMessageReceivedEvent
MessagingRequestHelper $messageContainer

// MessageReadyEvent
Message $message
```

**Broadcasting Channels**:
- `messages` (global)
- `session{id}` (customer session)
- `agent{type}` (agent notifications)

#### 1.2 Session Management
**Trigger**: New customer or existing customer interaction

**Event Sequence**:
```
Session Validation → SessionCreatedEvent (if new) → SessionPathChangedEvent (if context change)
```

**Detailed Flow**:
1. **Session Check**
   - Validate existing customer session
   - Create new session if needed
   - Update session context if required

2. **Session Events**
   - `SessionCreatedEvent` for new customers
   - `SessionPathChangedEvent` for context changes
   - Session state synchronized across system

### Phase 2: Message Processing & Routing

#### 2.1 Bot Processing (Default Path)
**Trigger**: `MessageReadyEvent` with bot agent context

**Event Sequence**:
```
MessageReadyEvent → HandleMessageReadyEvent → Bot Processing → Response Generation
```

**Detailed Flow**:
1. **Message Analysis**
   - `HandleMessageReadyEvent` processes message
   - Intent recognition via Google Dialogflow
   - Context analysis and routing decision

2. **Bot Response**
   - AI bot generates response
   - Response stored as new message
   - `MessageReadyEvent` dispatched for response
   - Customer receives bot response

3. **Fallback to Agent**
   - If bot cannot handle request
   - Customer clicks "Connect Agent" button
   - `MessagePostbackEvent` triggered
   - Agent transfer initiated

#### 2.2 Agent Transfer Process
**Trigger**: Bot fallback or direct agent request

**Event Sequence**:
```
MessagePostbackEvent → QueueItemCreatedEvent → Agent Assignment → SessionPathChangedEvent
```

**Detailed Flow**:
1. **Queue Creation**
   - `QueueItemCreatedEvent` dispatched
   - Queue item created for agent assignment
   - Agent notification sent

2. **Session Update**
   - Session agent type changed to "live"
   - `SessionPathChangedEvent` dispatched
   - Customer notified of agent connection

3. **Agent Assignment**
   - Available agent assigned to queue
   - Agent dashboard updated
   - Real-time notification sent

**Broadcasting Channels**:
- `queues` (global queue updates)
- `queues-app-{app_id}` (app-specific queues)
- `agent{agent_id}` (specific agent)
- `session{id}` (customer session)

### Phase 3: Agent Interaction

#### 3.1 Agent Reply Workflow
**Trigger**: Agent sends reply to customer message

**Event Sequence**:
```
Agent Reply → MessageReplyReadyEvent → Delivery Processing → Customer Notification
```

**Detailed Flow**:
1. **Agent Action**
   - Agent types reply in dashboard
   - Reply validated and stored
   - `MessageReplyReadyEvent` dispatched

2. **Message Delivery**
   - Reply sent to customer via appropriate channel
   - Delivery confirmation processed
   - Customer receives agent response

3. **Session Updates**
   - Session activity updated
   - Case status maintained
   - Real-time sync across system

**Event Data Flow**:
```php
// MessageReplyReadyEvent
Message $message (with is_reply flag)
```

**Broadcasting Channels**:
- `messages` (global)
- `session{id}` (customer session)
- `agent{agent_type}` (agent notifications)

#### 3.2 Case Management
**Trigger**: Agent interaction or customer escalation

**Event Sequence**:
```
Case Creation → MevrikCaseCreatedEvent → Case Updates → MevrikCaseClosedEvent
```

**Detailed Flow**:
1. **Case Creation**
   - Case created when customer needs support
   - `MevrikCaseCreatedEvent` dispatched
   - Case assigned to appropriate queue
   - Agent notification sent

2. **Case Updates**
   - Case status updated during interaction
   - Case context maintained
   - Agent progress tracked

3. **Case Closure**
   - Agent resolves customer issue
   - Case marked as closed with disposition
   - `MevrikCaseClosedEvent` dispatched
   - CSAT survey triggered

### Phase 4: Case Closure & CSAT

#### 4.1 Case Closure Process
**Trigger**: Agent closes case with resolution

**Event Sequence**:
```
Case Closure → MevrikCaseClosedEvent → CSAT Trigger → MevrikCSATStartEvent
```

**Detailed Flow**:
1. **Case Resolution**
   - Agent provides final resolution
   - Case marked as closed
   - Disposition and sentiment recorded
   - `MevrikCaseClosedEvent` dispatched

2. **Session Reset**
   - Session agent type reset to "bot"
   - Session context cleared
   - Customer returned to bot mode

3. **CSAT Initiation**
   - CSAT survey triggered automatically
   - `MevrikCSATStartEvent` dispatched
   - Survey sent to customer

**Event Data Flow**:
```php
// MevrikCaseClosedEvent
MevrikCase $case

// MevrikCSATStartEvent
MevrikCase $case
```

**Broadcasting Channels**:
- `messages` (global)
- `session{id}` (customer session)
- `agent{agent_type}` (agent notifications)

#### 4.2 CSAT Survey Process
**Trigger**: Case closure completion

**Event Sequence**:
```
CSAT Survey → Customer Response → CSAT Processing → Analytics Update
```

**Detailed Flow**:
1. **Survey Delivery**
   - CSAT survey sent to customer
   - Survey questions presented
   - Customer response collected

2. **Response Processing**
   - CSAT response stored
   - Analytics updated
   - Customer satisfaction tracked

3. **Feedback Loop**
   - CSAT data used for improvement
   - Agent performance metrics updated
   - System optimization based on feedback

## Event Sequence Diagrams

### Complete Customer Journey
```
Customer → Webhook → NewMessageReceivedEvent → MessageReadyEvent → Bot Processing
    ↓
Bot Response → MessageReadyEvent → Customer Receives Response
    ↓
Customer Requests Agent → MessagePostbackEvent → QueueItemCreatedEvent
    ↓
Agent Assignment → SessionPathChangedEvent → Agent Reply → MessageReplyReadyEvent
    ↓
Case Resolution → MevrikCaseClosedEvent → CSAT Survey → MevrikCSATStartEvent
    ↓
CSAT Response → Analytics Update → Journey Complete
```

### Agent Workflow
```
QueueItemCreatedEvent → Agent Notification → Agent Assignment → SessionPathChangedEvent
    ↓
Agent Dashboard Update → Agent Reply → MessageReplyReadyEvent → Customer Notification
    ↓
Case Management → Case Updates → Case Closure → MevrikCaseClosedEvent
    ↓
CSAT Trigger → MevrikCSATStartEvent → Performance Analytics
```

### Real-time Broadcasting Flow
```
Event Dispatch → Broadcasting Engine → Channel Routing → WebSocket Delivery
    ↓
Customer Widget ← session{id} channel ← MessageReadyEvent
    ↓
Agent Dashboard ← agent{type} channel ← QueueItemCreatedEvent
    ↓
Admin Panel ← messages channel ← All Events
```

## Channel-Specific Flows

### Web Widget Flow
```
Customer Widget → WebSocket → NewMessageReceivedEvent → MessageReadyEvent
    ↓
Bot Response → WebSocket → Customer Widget Update
    ↓
Agent Transfer → QueueItemCreatedEvent → Agent Dashboard
    ↓
Agent Reply → MessageReplyReadyEvent → Customer Widget
```

### Facebook Messenger Flow
```
Facebook Webhook → NewFacebookEventReceived → NewMessageReceivedEvent
    ↓
Message Processing → MessageReadyEvent → Bot Response
    ↓
Facebook API → Customer Receives Response
    ↓
Agent Transfer → QueueItemCreatedEvent → Agent Dashboard
```

### Email Flow
```
Email Webhook → NewMessageReceivedEvent → MessageReadyEvent
    ↓
Bot Processing → Email Response → Customer Receives Email
    ↓
Agent Transfer → QueueItemCreatedEvent → Agent Dashboard
    ↓
Agent Reply → Email Delivery → Customer Receives Reply
```

### Social Media Flow
```
Social Webhook → NewMessageReceivedEvent → MessageReadyEvent
    ↓
Social Processing → SaveAgentRepliesSocialEvent → Social API
    ↓
Public Response → Social Platform → Customer Sees Reply
```

## Broadcasting Strategy

### Broadcasting Categories

#### 1. Real-time Customer Events
**Events**: `MessageReadyEvent`, `MessageReplyReadyEvent`
**Channels**: `session{id}`, `messages`
**Purpose**: Immediate customer notification
**Delivery**: WebSocket to customer widget

#### 2. Agent Notification Events
**Events**: `QueueItemCreatedEvent`, `MessageReadyEvent`
**Channels**: `agent{type}`, `queues`
**Purpose**: Agent dashboard updates
**Delivery**: WebSocket to agent dashboard

#### 3. System Management Events
**Events**: `SessionCreatedEvent`, `SessionPathChangedEvent`
**Channels**: `sessions`, `messages`
**Purpose**: System state synchronization
**Delivery**: Internal processing + selective broadcasting

#### 4. Case Lifecycle Events
**Events**: `MevrikCaseCreatedEvent`, `MevrikCaseClosedEvent`, `MevrikCSATStartEvent`
**Channels**: `messages`, `session{id}`, `agent{type}`
**Purpose**: Case management and CSAT tracking
**Delivery**: Multi-channel broadcasting

### Broadcasting Logic
```php
// Event Broadcasting Decision Tree
if (event.requiresRealTimeNotification) {
    broadcastToChannels(event.getRelevantChannels());
}

if (event.affectsAgentWorkflow) {
    broadcastToAgentChannels(event.getAgentChannels());
}

if (event.requiresCustomerNotification) {
    broadcastToCustomerChannels(event.getCustomerChannels());
}
```

## Data Flow & State Transitions

### Message State Transitions
```
New → Processing → Stored → Delivered → Read → Archived
 ↓         ↓         ↓         ↓        ↓        ↓
Event   Event    Event    Event   Event   Event
```

### Session State Transitions
```
Anonymous → Guest → Verified → Bot → Live Agent → Closed
    ↓         ↓        ↓       ↓        ↓         ↓
  Event    Event    Event   Event    Event    Event
```

### Case State Transitions
```
Created → Assigned → In Progress → Resolved → Closed → CSAT
   ↓         ↓           ↓           ↓         ↓       ↓
 Event    Event      Event       Event     Event   Event
```

### Customer Journey States
```
Discovery → Engagement → Support → Resolution → Satisfaction
    ↓           ↓          ↓          ↓           ↓
  Event      Event      Event      Event       Event
```

## Error Handling & Recovery

### Error Event Flow
```
Error Detection → MessageDeliveryFailedEvent → Error Analysis → Recovery Action
```

### Recovery Mechanisms
1. **Message Delivery Failures**
   - Automatic retry with exponential backoff
   - Alternative delivery channels
   - Manual intervention alerts

2. **Event Processing Failures**
   - Dead letter queue for failed events
   - Event replay capabilities
   - System health monitoring

3. **Broadcasting Failures**
   - WebSocket reconnection logic
   - Fallback notification methods
   - Channel health monitoring

### Error Event Types
- `MessageDeliveryFailedEvent`: Message delivery issues
- System errors: Database, API, or service failures
- Network errors: WebSocket, HTTP, or connectivity issues
- Validation errors: Data format or security issues

## Performance Considerations

### Event Processing Optimization
1. **Asynchronous Processing**
   - Background job processing for non-critical events
   - Queue-based event handling
   - Parallel processing where possible

2. **Event Batching**
   - Batch multiple events for efficient processing
   - Reduce database round trips
   - Optimize broadcasting operations

3. **Caching Strategy**
   - Cache frequently accessed event data
   - Session state caching
   - Channel state optimization

### Scalability Measures
1. **Horizontal Scaling**
   - Multiple event processing workers
   - Load-balanced broadcasting
   - Distributed queue processing

2. **Channel Management**
   - Channel partitioning across servers
   - Dynamic channel creation/cleanup
   - Connection pooling optimization

3. **Database Optimization**
   - Event data indexing
   - Query optimization
   - Connection pooling

## Monitoring & Observability

### Event Metrics
1. **Processing Metrics**
   - Event processing time
   - Event throughput
   - Error rates
   - Queue depth

2. **Broadcasting Metrics**
   - Broadcasting latency
   - Channel connection counts
   - WebSocket performance
   - Delivery success rates

3. **Business Metrics**
   - Customer satisfaction scores
   - Agent response times
   - Case resolution rates
   - Channel performance

### Monitoring Tools
1. **Event Tracking**
   - Event flow visualization
   - Performance dashboards
   - Error alerting
   - Health checks

2. **Real-time Monitoring**
   - Live event processing
   - Channel health monitoring
   - System performance tracking
   - User experience metrics

## Implementation Guidelines

### Event Implementation Checklist
1. **Event Definition**
   - [ ] Define event class with proper data structure
   - [ ] Implement broadcasting channels
   - [ ] Add event validation
   - [ ] Include error handling

2. **Event Processing**
   - [ ] Implement event handler
   - [ ] Add processing logic
   - [ ] Include state transitions
   - [ ] Add logging and monitoring

3. **Broadcasting Setup**
   - [ ] Configure broadcasting channels
   - [ ] Implement WebSocket delivery
   - [ ] Add fallback mechanisms
   - [ ] Test real-time delivery

4. **Testing & Validation**
   - [ ] Unit tests for event processing
   - [ ] Integration tests for event flow
   - [ ] Performance testing
   - [ ] Error scenario testing

### Best Practices
1. **Event Design**
   - Keep events focused and atomic
   - Include all necessary data
   - Use consistent naming conventions
   - Implement proper validation

2. **Processing Logic**
   - Handle errors gracefully
   - Implement retry mechanisms
   - Add comprehensive logging
   - Monitor performance metrics

3. **Broadcasting Strategy**
   - Only broadcast when necessary
   - Use appropriate channels
   - Implement security measures
   - Monitor delivery success

---

## Appendices

### Appendix A: Event Flow Summary
| Phase | Events | Purpose | Broadcasting |
|-------|--------|---------|--------------|
| Initiation | NewMessageReceivedEvent, MessageReadyEvent | Customer message processing | Yes |
| Processing | MessageReadyEvent, HandleMessageReadyEvent | Bot/agent routing | Yes |
| Agent Transfer | MessagePostbackEvent, QueueItemCreatedEvent | Agent assignment | Yes |
| Agent Interaction | MessageReplyReadyEvent | Agent replies | Yes |
| Case Management | MevrikCaseCreatedEvent, MevrikCaseClosedEvent | Case lifecycle | Yes |
| CSAT | MevrikCSATStartEvent | Customer satisfaction | Yes |

### Appendix B: Broadcasting Channel Matrix
| Event | Global | Session | Agent | Queue | Case |
|-------|--------|---------|-------|-------|------|
| MessageReadyEvent | ✓ | ✓ | ✓ | - | - |
| MessageReplyReadyEvent | ✓ | ✓ | ✓ | - | - |
| QueueItemCreatedEvent | - | - | ✓ | ✓ | - |
| MevrikCaseClosedEvent | ✓ | ✓ | ✓ | - | ✓ |
| MevrikCSATStartEvent | ✓ | ✓ | ✓ | - | ✓ |
| SessionCreatedEvent | - | ✓ | - | - | - |

### Appendix C: Performance Targets
| Metric | Target | Measurement |
|--------|--------|-------------|
| Event Processing Time | < 100ms | Average processing time |
| Broadcasting Latency | < 50ms | WebSocket delivery time |
| Event Throughput | 10,000/min | Events per minute |
| Channel Capacity | 1,000+ | Concurrent channels |
| Error Rate | < 0.1% | Failed event percentage |
| Availability | 99.9% | System uptime |

### Appendix D: Error Recovery Procedures
| Error Type | Detection | Recovery Action | Escalation |
|------------|-----------|-----------------|------------|
| Message Delivery Failure | MessageDeliveryFailedEvent | Retry with backoff | Manual intervention |
| Event Processing Failure | System monitoring | Dead letter queue | Alert operations |
| Broadcasting Failure | Channel monitoring | WebSocket reconnection | System restart |
| Database Error | Exception handling | Transaction rollback | Database team |
| API Error | Response validation | Alternative endpoint | Service team |

---

*This document provides a comprehensive guide to the Mevrik Channels event lifecycle and should be updated as the system evolves.*
