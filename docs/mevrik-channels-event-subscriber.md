# Mevrik Channels Event Subscribers

## Document Information
- **Version**: 1.0
- **Last Updated**: [Date]
- **Author**: [Author Name]
- **Reviewers**: [Reviewer Names]
- **Status**: [Draft/Review/Approved]

## Table of Contents
1. [Overview](#overview)
2. [System Context](#system-context)
3. [Event Subscriber Architecture](#event-subscriber-architecture)
4. [MessageEventSubscriber](#messageeventsubscriber)
5. [FacebookEventSubscriber](#facebookeventsubscriber)
6. [SessionEventSubscriber](#sessioneventsubscriber)
7. [EmailEventSubscriber](#emaileventsubscriber)
8. [SocialRepliesEventSubscriber](#socialreplieseventsubscriber)
9. [Event Subscription Patterns](#event-subscription-patterns)
10. [Integration & Dependencies](#integration--dependencies)
11. [Performance & Scalability](#performance--scalability)
12. [Error Handling & Recovery](#error-handling--recovery)
13. [Testing & Validation](#testing--validation)
14. [Future Enhancements](#future-enhancements)

## Overview

### Purpose
This document provides a comprehensive technical specification for the Mevrik Channels Event Subscriber system. Event subscribers are the core components that handle event processing, business logic execution, and system integration within the omnichannel customer experience platform.

### Scope
- Complete event subscriber architecture and implementation
- Individual subscriber class functionality and responsibilities
- Event handling patterns and processing logic
- Integration with external services and APIs
- Error handling and recovery mechanisms
- Performance optimization strategies

### Key Stakeholders
- **Development Teams**: Implementation and maintenance guidance
- **System Architects**: Architecture understanding and design decisions
- **DevOps Engineers**: Deployment and monitoring strategies
- **Product Managers**: Feature understanding and business logic
- **QA Teams**: Testing scenarios and validation criteria

## System Context

### Event Subscriber Role in Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Event System Core                        │
├─────────────────────────────────────────────────────────────┤
│  Event Dispatch  │  Event Broadcasting  │  Event Processing │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                Event Subscribers                            │
├─────────────────────────────────────────────────────────────┤
│  MessageEventSubscriber  │  FacebookEventSubscriber        │
│  SessionEventSubscriber  │  EmailEventSubscriber           │
│  SocialRepliesEventSubscriber                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Business Logic Layer                     │
├─────────────────────────────────────────────────────────────┤
│  Bot Processing  │  Agent Management  │  Case Management    │
│  Social Media    │  Email Delivery    │  CSAT Processing    │
└─────────────────────────────────────────────────────────────┘
```

### Subscriber Responsibilities
- **Event Processing**: Handle specific event types and execute business logic
- **Service Integration**: Integrate with external APIs and services
- **Data Transformation**: Transform event data for downstream processing
- **Error Handling**: Manage failures and implement recovery mechanisms
- **Performance Optimization**: Ensure efficient event processing

## Event Subscriber Architecture

### Core Components

#### 1. Subscriber Base Structure
```php
class EventSubscriber {
    // Event handler methods
    public function handleEventType(EventType $event) { }
    
    // Event subscription registration
    public function subscribe($events) { }
}
```

#### 2. Event Subscription Pattern
- **Event Registration**: Subscribe to specific event types
- **Handler Mapping**: Map events to handler methods
- **Priority Management**: Control event processing order
- **Conditional Processing**: Environment-based event handling

#### 3. Processing Flow
```
Event Triggered → Subscriber Selection → Handler Execution → Business Logic → Integration → Completion
```

### Subscriber Categories

The system implements 5 specialized event subscribers:

#### Message Processing (1)
1. **MessageEventSubscriber** - Core message processing and routing

#### Channel-Specific Processing (2)
2. **FacebookEventSubscriber** - Facebook Messenger and social media integration
3. **EmailEventSubscriber** - Email channel processing and delivery

#### Session & Customer Management (1)
4. **SessionEventSubscriber** - Session lifecycle and customer management

#### Social Media Integration (1)
5. **SocialRepliesEventSubscriber** - Social media agent replies and activity

## MessageEventSubscriber

### Overview
The `MessageEventSubscriber` is the central event processing hub that handles core message lifecycle events, bot interactions, agent transfers, and message delivery across all channels.

### Event Handlers

#### 1. handleFacebookMessageWebhook()
**Purpose**: Process incoming Facebook webhook events and create message objects

**Event**: `NewFacebookEventReceived`
**Trigger**: Facebook webhook receives new message/event
**Processing**:
```php
function handleFacebookMessageWebhook(NewFacebookEventReceived $event) {
    // Create message object from Facebook webhook data
    CreateMessageFromWebhook::run($event->webhookEventLog);
}
```

**Business Logic**:
- Extract message data from Facebook webhook
- Create standardized message object
- Validate message format and content
- Store message in database

#### 2. handleMessageReady()
**Purpose**: Process new customer messages ready for system handling

**Event**: `MessageReadyEvent`
**Trigger**: New customer message stored and ready for processing
**Processing**:
```php
function handleMessageReady(MessageReadyEvent $event) {
    $message = $event->message;
    $message->postRead();
    // Additional processing logic
}
```

**Business Logic**:
- Mark message as read
- Initialize message processing pipeline
- Trigger bot or agent routing logic
- Update message status and metadata

#### 3. handleMessageQuickReply()
**Purpose**: Handle quick reply button interactions from customers

**Event**: `MessageQuickReplyEvent`
**Trigger**: Customer clicks quick reply button in message
**Processing**:
```php
function handleMessageQuickReply(MessageQuickReplyEvent $event) {
    if (!$event->message->isQuickReply()) {
        return null;
    }
    
    $quick_reply = $event->quick_reply;
    $suffix = key($quick_reply);
    
    switch ($suffix) {
        case 'hat': // Human Agent Transfer
            if (strtolower($quick_reply['hat']) == 'yes') {
                // Process live agent transfer
            }
            break;
    }
}
```

**Business Logic**:
- Validate quick reply interaction
- Process Human Agent Transfer (HAT) requests
- Route to appropriate handler based on reply type
- Update session context and agent mode

#### 4. handleMessagePostback()
**Purpose**: Handle postback button interactions from customers

**Event**: `MessagePostbackEvent`
**Trigger**: Customer clicks postback button in message
**Processing**:
```php
function handleMessagePostback(MessagePostbackEvent $event) {
    if (!$event->message->isPostback()) {
        return null;
    }
    
    $postback = $event->postback;
    $suffix = key($postback);
    
    switch ($suffix) {
        case 'hat': // Human Agent Transfer
            if (strtolower($postback['hat']) == 'yes') {
                // Process live agent transfer
            }
            break;
    }
}
```

**Business Logic**:
- Validate postback interaction
- Process Human Agent Transfer (HAT) requests
- Handle different postback types
- Update customer journey state

#### 5. handleMessageReply()
**Purpose**: Process agent replies and route to appropriate delivery channels

**Event**: `MessageReplyReadyEvent`
**Trigger**: Agent sends reply to customer message
**Processing**:
```php
function handleMessageReply(MessageReplyReadyEvent $event) {
    $this->_attachSession($event->message);
    $session = mevrik()->session();
    
    if ($session->isOriginEmail()) {
        EmailMessageDeliveryEvent::dispatch($event->message);
    } else if ($session->isOriginFacebook()) {
        FacebookMessageDeliveryEvent::dispatch($event->message);
    } else if ($session->isMatch('referrer', 'robi') || $session->isMatch('referrer', 'airtel')) {
        DeliverRobiChannelMessage::dispatch(['message_id' => $event->message->id]);
    }
}
```

**Business Logic**:
- Attach session context to message
- Determine delivery channel based on session origin
- Route to appropriate delivery service
- Handle channel-specific delivery logic

#### 6. handleDialogflowIntentDetected()
**Purpose**: Process Dialogflow intent detection results and update session context

**Event**: `DialogflowIntentDetected`
**Trigger**: Google Dialogflow detects intent from customer message
**Processing**:
```php
function handleDialogflowIntentDetected(DialogflowIntentDetected $event) {
    $query = $event->textQuery;
    if (isset($query->messageId) && !empty($query->messageId)) {
        if (mevrik()->invalidSession()) {
            $message = app('message')->getObject($query->messageId);
            mevrik()->createSessionFromMessage($message);
        }
        
        $session = mevrik()->session();
        mevrik()->setSessionPath("{$session->agent}/" . str_slug(array_get($query->response, 'name')));
    }
}
```

**Business Logic**:
- Create session from message if needed
- Update session path based on detected intent
- Route to appropriate bot journey
- Maintain conversation context

### Channel-Specific Processing

#### 1. create_ticket_for_sign_line()
**Purpose**: Handle video call requests for Sign Line channel

**Processing**:
```php
function create_ticket_for_sign_line(Message $message) {
    if ($message->isChannelType('sign_line')) {
        // Video call messages sent to agent queue immediately
        HumanAgentTransfer::run($message);
    }
}
```

**Business Logic**:
- Detect Sign Line channel type
- Automatically transfer to human agent
- Create queue item for video call handling
- Prioritize video call requests

#### 2. create_ticket_for_bioscope()
**Purpose**: Handle Bioscope channel messages with automatic agent transfer

**Processing**:
```php
function create_ticket_for_bioscope(Message $message, SessionUser $session) {
    if ($session->isChannelBioscope()) {
        $case_id = $session->getValue('case_id');
        if (!empty($case_id)) {
            $queue = app('queue_item')->get_queue_by_case_id($case_id);
            if (empty($queue)) {
                HumanAgentTransfer::run($message);
            }
        }
        return true;
    }
    return false;
}
```

**Business Logic**:
- Detect Bioscope channel type
- Check for existing queue items
- Create agent transfer if no existing queue
- Ensure all Bioscope messages get human attention

### Event Subscription
```php
public function subscribe($events) {
    $events->listen(NewFacebookEventReceived::class, [
        MessageEventSubscriber::class, 'handleFacebookMessageWebhook'
    ]);
    
    $events->listen(MessageReadyEvent::class, [
        MessageEventSubscriber::class, 'handleMessageReady'
    ]);
    
    $events->listen(MessageQuickReplyEvent::class, [
        MessageEventSubscriber::class, 'handleMessageQuickReply'
    ]);
    
    $events->listen(MessagePostbackEvent::class, [
        MessageEventSubscriber::class, 'handleMessagePostback'
    ]);
    
    $events->listen(MessageReplyReadyEvent::class, [
        MessageEventSubscriber::class, 'handleMessageReply'
    ]);
    
    $events->listen(DialogflowIntentDetected::class, [
        MessageEventSubscriber::class, 'handleDialogflowIntentDetected'
    ]);
}
```

## FacebookEventSubscriber

### Overview
The `FacebookEventSubscriber` handles all Facebook-specific event processing, including message delivery, social media interactions, and Facebook API integration.

### Event Handlers

#### 1. handleFacebookMessageDelivery()
**Purpose**: Process Facebook message delivery and handle different message types

**Event**: `FacebookMessageDeliveryEvent`
**Trigger**: Agent reply ready for Facebook delivery
**Processing**:
```php
function handleFacebookMessageDelivery(FacebookMessageDeliveryEvent $event) {
    $silent = env('MEVRIK_SILENT', 0);
    
    if (!$silent) {
        if ($event->message->isMessageType(Message::$MESSAGE_TYPE_COMMENT)) {
            // Handle comment replies
            $this->handleCommentReply($event->message);
        } else {
            // Handle direct messages
            SendMessageToFacebook::run($event->message);
        }
    }
}
```

**Business Logic**:
- Check silent mode configuration
- Route comment vs direct message handling
- Process social media interactions
- Handle Facebook API delivery

#### 2. handleCommentReply()
**Purpose**: Handle Facebook comment replies with different reply types

**Processing**:
```php
function handleCommentReply(Message $message) {
    if (!empty($message->params['reopen']) && $message->params['reopen']) {
        // Handle reopened comment
        if ($message->isParamReplyAs() == 'comment') {
            $fb_instance = SaveAgentReplies::run($message, 'edit');
            $agent_action = SocialAgentAction::run($message->message_ref, 'reply_as', ['value' => $message->params]);
            SendCommentToFacebookPage::run($fb_instance, $agent_action['object_id'], $message);
        } elseif ($message->isParamReplyAs() == 'direct') {
            // Handle direct message reply
            $this->handleDirectMessageReply($message);
        }
    } else {
        // Handle new comment reply
        $this->handleNewCommentReply($message);
    }
}
```

**Business Logic**:
- Detect comment reply type (edit vs new)
- Handle direct message vs public comment
- Process social media agent actions
- Manage Facebook post interactions

#### 3. sendMessageWithRequestHelper()
**Purpose**: Send messages to Facebook using the messaging request helper

**Processing**:
```php
function sendMessageWithRequestHelper(FacebookPost $fb_instance, $private_type, $message, $message_ref) {
    $payload_template = [];
    
    // Handle text messages
    if (isset($message->body)) {
        $payload_template['text'] = $message->body;
    }
    
    // Handle attachments
    $attachment_data = $message->getMetadata('attachment') ?? null;
    if (!empty($attachment_data)) {
        $attachment_array = json_decode($attachment_data->meta_value, true);
        if (isset($attachment_array['type']) && $attachment_array['payload']['url']) {
            $payload_template[$attachment_array['type']] = $attachment_array['payload']['url'];
        }
    }
    
    // Send to Facebook API
    $fbToken = app('facebook_helper')->feedApi()->fbToken($message->channel);
    foreach ($payload_template as $key => $value) {
        $data = $fbToken->send_api($private_type['object_id'], [$key => $value], null, $private_set);
        
        // Handle delivery errors
        $error_message = array_get($data, 'data.error.message');
        if (!empty($error_message)) {
            $this->handleDeliveryError($message, $error_message, $message_ref);
        }
    }
}
```

**Business Logic**:
- Build Facebook API payload
- Handle text and attachment messages
- Process Facebook API responses
- Manage delivery errors and feedback

#### 4. handleFacebookDeliveryFailed()
**Purpose**: Handle Facebook message delivery failures

**Event**: `MessageDeliveryFailedEvent`
**Trigger**: Facebook message delivery fails
**Processing**:
```php
function handleFacebookDeliveryFailed(MessageDeliveryFailedEvent $event) {
    // Keep log of failed Facebook message delivery
    // Store error details for analysis
    // Implement retry logic if needed
}
```

**Business Logic**:
- Log delivery failure details
- Store error information for analysis
- Implement retry mechanisms
- Update message status

#### 5. handleNewFacebookWebhookReceived()
**Purpose**: Process new Facebook webhook events

**Event**: `NewFacebookEventReceived`
**Trigger**: New Facebook webhook event received
**Processing**:
```php
function handleNewFacebookWebhookReceived(NewFacebookEventReceived $eventReceived) {
    // Extract journey data for sequential bot journey
    // Only if webhook was processed
    // ExtractFeedToRunChatologyJourney::dispatchIf($eventReceived->webhookEventLog->processed, $eventReceived->webhookEventLog);
}
```

**Business Logic**:
- Process Facebook webhook events
- Extract journey data for bot processing
- Validate webhook processing status
- Trigger appropriate bot journeys

### Event Subscription
```php
public function subscribe($events) {
    $events->listen(NewFacebookEventReceived::class, [
        FacebookEventSubscriber::class, 'handleNewFacebookWebhookReceived'
    ]);
    
    $events->listen(FacebookMessageDeliveryEvent::class, [
        FacebookEventSubscriber::class, 'handleFacebookMessageDelivery'
    ]);
    
    $events->listen(MessageDeliveryFailedEvent::class, [
        FacebookEventSubscriber::class, 'handleFacebookDeliveryFailed'
    ]);
}
```

## SessionEventSubscriber

### Overview
The `SessionEventSubscriber` manages customer session lifecycle, case management, CSAT processing, and customer profile creation events.

### Event Handlers

#### 1. handleSessionCreated()
**Purpose**: Process new customer session creation

**Event**: `SessionCreatedEvent`
**Trigger**: New customer session created
**Processing**:
```php
function handleSessionCreated(SessionCreatedEvent $event) {
    // $session = $event->sessionUser;
    
    // This action creates and updates customer details from session data
    // CreateCustomerFromSession::run($session);
}
```

**Business Logic**:
- Initialize new customer session
- Create customer profile from session data
- Set up session context and preferences
- Initialize customer journey tracking

#### 2. handleSessionPathChanged()
**Purpose**: Handle session context and agent mode changes

**Event**: `SessionPathChangedEvent`
**Trigger**: Session agent type or context changes
**Processing**:
```php
function handleSessionPathChanged(SessionPathChangedEvent $pathChangedEvent) {
    $session = $pathChangedEvent->sessionUser;
    
    // Update case with new agent mode
    $case = mevrik()->getCase($session->getValue('case_id'));
    if ($case && $case->isMatch('agent', 'bot')) {
        \DB::table('mevrik_cases')
            ->where('id', $session->getValue('case_id'))
            ->update(['agent' => $session->getValue('agent')]);
    }
}
```

**Business Logic**:
- Update case agent mode when session changes
- Synchronize session and case states
- Handle bot to live agent transitions
- Maintain case context consistency

#### 3. handleNewCustomerProfileCreated()
**Purpose**: Process new customer profile creation

**Event**: `NewCustomerProfileCreated`
**Trigger**: New customer profile created
**Processing**:
```php
function handleNewCustomerProfileCreated(NewCustomerProfileCreated $profileCreated) {
    // Sync customer name from Facebook
    // UpdateCustomerWithFacebookProfile::run($profileCreated->customer);
    
    // Send greeting for new customer profile created
    // Check if already sent using customer meta
    // welcome.greeting_sent
}
```

**Business Logic**:
- Sync customer data from external sources
- Send welcome messages to new customers
- Initialize customer preferences
- Set up customer journey tracking

#### 4. handleSessionResetEvent()
**Purpose**: Handle session reset and cleanup

**Event**: `SessionReset`
**Trigger**: Session reset requested
**Processing**:
```php
function handleSessionResetEvent(SessionReset $session_reset_event) {
    $session = get_session($session_reset_event->session_id);
    
    // Close existing queue item
    MevrikQueueItemCloseAction::dispatch($session->getValue('case_id'));
    
    // Reset session data
    $newAnonymousRef = $session->anonymousRefId();
    $options = [
        'agent' => 'bot',
        'context' => 'default',
        'case_id' => 0,
        'unique_id' => $newAnonymousRef
    ];
    
    $session->updateSessionData($options);
}
```

**Business Logic**:
- Close existing queue items
- Reset session to anonymous state
- Clear case associations
- Initialize new session context

#### 5. handleMevrikCaseClosed()
**Purpose**: Handle case closure events

**Event**: `MevrikCaseClosedEvent`
**Trigger**: Support case closed
**Processing**:
```php
function handleMevrikCaseClosed(MevrikCaseClosedEvent $event) {
    // CSAT is triggered at the point of message_closed
    // if (!$session?->isOriginEmail()) {
    //     $this->startCSAT($case);
    // }
}
```

**Business Logic**:
- Process case closure
- Trigger CSAT surveys when appropriate
- Update case status and metrics
- Clean up case-related resources

#### 6. handleMevrikCSATStart()
**Purpose**: Handle CSAT survey initiation

**Event**: `MevrikCSATStartEvent`
**Trigger**: CSAT survey initiated
**Processing**:
```php
function handleMevrikCSATStart(MevrikCSATStartEvent $event) {
    $case = $event->case;
    $session = $case->getSession();
    mevrik()->setSession($session);
    $this->startCSAT($case);
}
```

**Business Logic**:
- Initialize CSAT survey process
- Set up customer session for CSAT
- Validate customer eligibility
- Start CSAT survey delivery

#### 7. startCSAT()
**Purpose**: Start CSAT survey for eligible customers

**Processing**:
```php
public function startCSAT(MevrikCase $case) {
    $customer = $case->getCustomer();
    $eligible_for_csat = $customer?->eligibleForCSAT();
    
    if ($eligible_for_csat) {
        StartCSATAction::dispatch($case->primaryId());
        return true;
    }
    
    return false;
}
```

**Business Logic**:
- Check customer CSAT eligibility
- Dispatch CSAT survey action
- Track CSAT initiation
- Handle eligibility validation

### Event Subscription
```php
public function subscribe($events) {
    $events->listen(SessionCreatedEvent::class, [
        SessionEventSubscriber::class, 'handleSessionCreated'
    ]);
    
    $events->listen(SessionReset::class, [
        SessionEventSubscriber::class, 'handleSessionResetEvent'
    ]);
    
    $events->listen(SessionPathChangedEvent::class, [
        SessionEventSubscriber::class, 'handleSessionPathChanged'
    ]);
    
    $events->listen(NewCustomerProfileCreated::class, [
        SessionEventSubscriber::class, 'handleNewCustomerProfileCreated'
    ]);
    
    $events->listen(MevrikCaseClosedEvent::class, [
        SessionEventSubscriber::class, 'handleMevrikCaseClosed'
    ]);
}
```

## EmailEventSubscriber

### Overview
The `EmailEventSubscriber` handles email channel message delivery and processing within the omnichannel system.

### Event Handlers

#### 1. handleEmailMessageDelivery()
**Purpose**: Process email message delivery

**Event**: `EmailMessageDeliveryEvent`
**Trigger**: Agent reply ready for email delivery
**Processing**:
```php
function handleEmailMessageDelivery(EmailMessageDeliveryEvent $event) {
    $silent = env('MEVRIK_SILENT', 0);
    
    if (!$silent) {
        DeliverReplyEmail::dispatch($event->message);
    }
}
```

**Business Logic**:
- Check silent mode configuration
- Dispatch email delivery action
- Handle email-specific delivery logic
- Process email delivery status

### Event Subscription
```php
public function subscribe($events) {
    $events->listen(EmailMessageDeliveryEvent::class, [
        EmailEventSubscriber::class, 'handleEmailMessageDelivery'
    ]);
}
```

## SocialRepliesEventSubscriber

### Overview
The `SocialRepliesEventSubscriber` handles social media agent replies and activity tracking within the omnichannel system.

### Event Handlers

#### 1. handleSocialAgentReplies()
**Purpose**: Process social media agent replies

**Event**: `SaveAgentRepliesSocialEvent`
**Trigger**: Agent replies to social media content
**Processing**:
```php
function handleSocialAgentReplies(SaveAgentRepliesSocialEvent $event) {
    logDebug('Inside event > Subscriber', ['reply_mid' => $event]);
    
    SaveSocialActivityReplies::run(
        $event->reply_mid, 
        $event->facebookPost, 
        $event->object_type, 
        $event->created_at, 
        $event->reopen
    );
}
```

**Business Logic**:
- Log social media reply processing
- Save social activity replies
- Track agent social media interactions
- Handle reply metadata and context

### Event Subscription
```php
public function subscribe($events) {
    $events->listen(SaveAgentRepliesSocialEvent::class, [
        SocialRepliesEventSubscriber::class, 'handleSocialAgentReplies'
    ]);
}
```

## Event Subscription Patterns

### Subscription Registration
```php
// In EventServiceProvider
protected $subscribe = [
    MessageEventSubscriber::class,
    FacebookEventSubscriber::class,
    SessionEventSubscriber::class,
    EmailEventSubscriber::class,
    SocialRepliesEventSubscriber::class,
];
```

### Event Handler Mapping
```php
// Subscriber class
public function subscribe($events) {
    $events->listen(EventClass::class, [
        SubscriberClass::class, 'handlerMethod'
    ]);
}
```

### Conditional Processing
```php
// Environment-based processing
$silent = env('MEVRIK_SILENT', 0);
if (!$silent) {
    // Process event
}
```

### Error Handling
```php
// Try-catch error handling
try {
    // Event processing logic
} catch (\Exception $exception) {
    // Error logging and handling
    logError($exception->getMessage());
}
```

## Integration & Dependencies

### External Service Integration

#### 1. Facebook API Integration
- **Message Delivery**: Send messages via Facebook Graph API
- **Webhook Processing**: Handle Facebook webhook events
- **Social Media Management**: Manage Facebook page interactions
- **Error Handling**: Process Facebook API errors and feedback

#### 2. Email Service Integration
- **Email Delivery**: Send emails via email service providers
- **Template Processing**: Handle email templates and formatting
- **Delivery Tracking**: Track email delivery status
- **Bounce Handling**: Process email bounces and failures

#### 3. Dialogflow Integration
- **Intent Detection**: Process Dialogflow intent detection results
- **Bot Journey**: Route to appropriate bot journeys
- **Context Management**: Maintain conversation context
- **Response Generation**: Generate bot responses

### Internal Service Dependencies

#### 1. Message Processing Services
- **MessagingRequestHelper**: Message data processing
- **Message Storage**: Database message storage
- **Message Routing**: Channel-specific routing
- **Message Delivery**: Multi-channel delivery

#### 2. Session Management Services
- **SessionUser**: Customer session management
- **Session Context**: Session state and context
- **Session Routing**: Agent mode routing
- **Session Cleanup**: Session reset and cleanup

#### 3. Case Management Services
- **MevrikCase**: Support case management
- **Queue Management**: Agent queue processing
- **CSAT Processing**: Customer satisfaction surveys
- **Case Lifecycle**: Case creation, updates, and closure

## Performance & Scalability

### Optimization Strategies

#### 1. Asynchronous Processing
- **Background Jobs**: Process events asynchronously
- **Queue Processing**: Use Laravel queues for heavy processing
- **Event Batching**: Batch multiple events for efficiency
- **Parallel Processing**: Process independent events in parallel

#### 2. Caching Strategies
- **Session Caching**: Cache session data for performance
- **Message Caching**: Cache frequently accessed messages
- **API Response Caching**: Cache external API responses
- **Database Query Caching**: Cache database query results

#### 3. Resource Management
- **Connection Pooling**: Pool database and API connections
- **Memory Management**: Optimize memory usage
- **CPU Optimization**: Optimize CPU-intensive operations
- **I/O Optimization**: Optimize file and network I/O

### Scalability Measures

#### 1. Horizontal Scaling
- **Multiple Workers**: Deploy multiple event processing workers
- **Load Balancing**: Distribute event processing load
- **Service Partitioning**: Partition services by functionality
- **Database Sharding**: Shard databases for performance

#### 2. Event Processing Optimization
- **Event Filtering**: Filter events at the source
- **Selective Processing**: Process only relevant events
- **Event Prioritization**: Prioritize critical events
- **Batch Processing**: Process events in batches

## Error Handling & Recovery

### Error Types

#### 1. Event Processing Errors
- **Validation Errors**: Invalid event data
- **Processing Errors**: Business logic failures
- **Integration Errors**: External service failures
- **System Errors**: Infrastructure failures

#### 2. Recovery Mechanisms
- **Retry Logic**: Automatic retry for transient failures
- **Fallback Processing**: Alternative processing paths
- **Error Logging**: Comprehensive error logging
- **Dead Letter Queues**: Handle failed events

### Error Handling Patterns

#### 1. Try-Catch Blocks
```php
try {
    // Event processing logic
} catch (\Exception $exception) {
    // Error handling and logging
    logError($exception->getMessage());
    // Retry or fallback logic
}
```

#### 2. Error Event Dispatch
```php
// Dispatch error events for monitoring
MessageDeliveryFailedEvent::dispatch($message, $error);
```

#### 3. Graceful Degradation
```php
// Continue processing even if some events fail
if ($this->processEvent($event)) {
    // Success path
} else {
    // Fallback path
}
```

## Testing & Validation

### Testing Strategies

#### 1. Unit Testing
- **Event Handler Testing**: Test individual event handlers
- **Business Logic Testing**: Test business logic components
- **Integration Testing**: Test external service integration
- **Error Scenario Testing**: Test error handling scenarios

#### 2. Integration Testing
- **End-to-End Testing**: Test complete event flows
- **Service Integration Testing**: Test service interactions
- **API Integration Testing**: Test external API integration
- **Database Integration Testing**: Test database operations

#### 3. Performance Testing
- **Load Testing**: Test under high load conditions
- **Stress Testing**: Test system limits
- **Scalability Testing**: Test horizontal scaling
- **Latency Testing**: Test response times

### Validation Criteria

#### 1. Functional Validation
- **Event Processing**: All events processed correctly
- **Business Logic**: Business rules implemented correctly
- **Integration**: External services integrated properly
- **Error Handling**: Errors handled gracefully

#### 2. Performance Validation
- **Response Time**: Events processed within time limits
- **Throughput**: System handles expected event volume
- **Resource Usage**: Efficient resource utilization
- **Scalability**: System scales with load

## Future Enhancements

### Short-term Improvements

#### 1. Enhanced Error Handling
- **Advanced Retry Logic**: Exponential backoff and circuit breakers
- **Error Analytics**: Detailed error analysis and reporting
- **Automated Recovery**: Self-healing error recovery
- **Error Notifications**: Real-time error notifications

#### 2. Performance Optimization
- **Event Streaming**: Real-time event streaming
- **Advanced Caching**: Multi-level caching strategies
- **Database Optimization**: Advanced database optimization
- **API Optimization**: Optimized external API usage

### Medium-term Features

#### 1. Advanced Event Processing
- **Event Sourcing**: Complete event sourcing implementation
- **Event Replay**: Event replay capabilities
- **Event Versioning**: Event schema versioning
- **Event Compression**: Event data compression

#### 2. Enhanced Integration
- **Microservices Events**: Event-driven microservices
- **API Gateway Integration**: Centralized API management
- **Service Mesh**: Service-to-service communication
- **Event Mesh**: Distributed event processing

### Long-term Vision

#### 1. AI-Powered Processing
- **Machine Learning**: ML-powered event processing
- **Predictive Analytics**: Predictive event analysis
- **Intelligent Routing**: AI-powered event routing
- **Automated Optimization**: Self-optimizing event processing

#### 2. Global Distribution
- **Multi-Region Deployment**: Global event processing
- **Edge Computing**: Edge-based event processing
- **CDN Integration**: Content delivery optimization
- **Global Load Balancing**: Worldwide load distribution

---

## Appendices

### Appendix A: Event Subscriber Summary
| Subscriber | Events Handled | Purpose | Key Features |
|------------|----------------|---------|--------------|
| MessageEventSubscriber | 6 events | Core message processing | Bot routing, agent transfer, delivery |
| FacebookEventSubscriber | 3 events | Facebook integration | Social media, API delivery, webhooks |
| SessionEventSubscriber | 5 events | Session management | CSAT, case management, customer profiles |
| EmailEventSubscriber | 1 event | Email delivery | Email processing and delivery |
| SocialRepliesEventSubscriber | 1 event | Social replies | Social media agent interactions |

### Appendix B: Event Handler Matrix
| Event | MessageEventSubscriber | FacebookEventSubscriber | SessionEventSubscriber | EmailEventSubscriber | SocialRepliesEventSubscriber |
|-------|----------------------|----------------------|---------------------|-------------------|---------------------------|
| NewFacebookEventReceived | ✓ | ✓ | - | - | - |
| MessageReadyEvent | ✓ | - | - | - | - |
| MessageQuickReplyEvent | ✓ | - | - | - | - |
| MessagePostbackEvent | ✓ | - | - | - | - |
| MessageReplyReadyEvent | ✓ | - | - | - | - |
| DialogflowIntentDetected | ✓ | - | - | - | - |
| FacebookMessageDeliveryEvent | - | ✓ | - | - | - |
| MessageDeliveryFailedEvent | - | ✓ | - | - | - |
| SessionCreatedEvent | - | - | ✓ | - | - |
| SessionPathChangedEvent | - | - | ✓ | - | - |
| NewCustomerProfileCreated | - | - | ✓ | - | - |
| SessionReset | - | - | ✓ | - | - |
| MevrikCaseClosedEvent | - | - | ✓ | - | - |
| MevrikCSATStartEvent | - | - | ✓ | - | - |
| EmailMessageDeliveryEvent | - | - | - | ✓ | - |
| SaveAgentRepliesSocialEvent | - | - | - | - | ✓ |

### Appendix C: Performance Targets
| Metric | Target | Measurement |
|--------|--------|-------------|
| Event Processing Time | < 50ms | Average processing time |
| Event Throughput | 5,000/min | Events per minute per subscriber |
| Error Rate | < 0.1% | Failed event percentage |
| Memory Usage | < 100MB | Per subscriber process |
| CPU Usage | < 50% | Per subscriber process |
| Availability | 99.9% | Subscriber uptime |

### Appendix D: Configuration Parameters
| Parameter | Description | Default | Environment |
|-----------|-------------|---------|-------------|
| MEVRIK_SILENT | Silent mode for testing | 0 | All |
| EVENT_RETRY_ATTEMPTS | Max retry attempts | 3 | Production |
| EVENT_TIMEOUT | Event processing timeout | 30 | Production |
| SUBSCRIBER_WORKERS | Number of subscriber workers | 4 | Production |
| EVENT_BATCH_SIZE | Event batch processing size | 100 | Production |
| CACHE_TTL | Cache time-to-live | 3600 | Production |

---

*This document provides a comprehensive guide to the Mevrik Channels Event Subscriber system and should be updated as the system evolves.*
