# MessagesController - Product Requirements Document

## Document Information
- **Version**: 1.0
- **Last Updated**: [Date]
- **Author**: [Author Name]
- **Reviewers**: [Reviewer Names]
- **Status**: [Draft/Review/Approved]

## Table of Contents
1. [Overview](#overview)
2. [System Context](#system-context)
3. [Controller Architecture](#controller-architecture)
4. [Message Actions](#message-actions)
5. [Authentication & Authorization](#authentication--authorization)
6. [Message Processing Flow](#message-processing-flow)
7. [API Specifications](#api-specifications)
8. [Error Handling](#error-handling)
9. [Performance Considerations](#performance-considerations)
10. [Security Requirements](#security-requirements)
11. [Testing Requirements](#testing-requirements)
12. [Future Enhancements](#future-enhancements)

## Overview

### Purpose
The MessagesController serves as the central entry point for all messaging functionality in the Mevrik Channels PHP platform. It handles both customer-initiated messages from web widgets and agent-initiated messages from the agent dashboard, providing a unified interface for message processing, routing, and management.

### Scope
- Customer message posting and retrieval
- Agent message replies and management
- Bulk message operations
- Message history and queue management
- CSAT survey initiation
- Video call functionality
- Social media message handling
- System message operations

### Key Stakeholders
- **Customers**: End users interacting through web widgets, Facebook Messenger, email
- **Customer Service Agents**: Human agents responding to customer messages
- **System Administrators**: Managing message flows and system operations
- **Business Operations**: Monitoring message metrics and performance

## System Context

### Controller Role
```
┌─────────────────────────────────────────────────────────────┐
│                    External Channels                        │
├─────────────────────────────────────────────────────────────┤
│  Web Widgets  │  Facebook Messenger  │  Email  │  Social    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                MessagesController                           │
│              (Central Message Entry Point)                 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Message Actions                          │
├─────────────────────────────────────────────────────────────┤
│  MessagePost │ MessageReply │ MessageHistory │ MessageCSAT  │
│  MessageQueue│ MessageClosed│ Bulk Operations│ Video Calls  │
└─────────────────────────────────────────────────────────────┘
```

### Integration Points
- **MessagingRequestHelper**: Message container and validation
- **Session Management**: Customer and agent session handling
- **Message Storage**: Database persistence and retrieval
- **Event System**: Asynchronous message processing
- **External APIs**: Channel-specific integrations

## Controller Architecture

### Core Components

#### 1. Request Processing
```php
public function __invoke(Request $request)
{
    $action = post('action');
    $isAgentMode = $request->segment(3) == 'agent';
    
    // Validation and authentication
    // Message container creation
    // Action routing
}
```

#### 2. Authentication Modes
- **Customer Mode**: Requires valid customer session
- **Agent Mode**: Requires agent authentication via `x-mevrik-attending-id` header

#### 3. Message Container
- **MessagingRequestHelper**: Centralized message data handling
- **Payload Processing**: Request data normalization and validation
- **Session Integration**: Customer/agent session management

### Supported Actions

The controller supports 20 distinct message actions organized into categories:

#### Core Message Actions
1. **message_post** - Customer message posting
2. **message_reply** - Agent message replies
3. **message_send** - Direct message sending
4. **message_history** - Message retrieval and history
5. **message_example** - Message template examples

#### Bulk Operations
6. **message_bulk_reply** - Bulk agent replies
7. **message_bulk_close** - Bulk message closure

#### Queue & Case Management
8. **message_queue** - Queue management and agent transfer
9. **message_closed** - Case closure with disposition
10. **system_message_closed** - System-initiated closure

#### Customer Satisfaction
11. **message_csat** - CSAT survey initiation

#### Video Communication
12. **video_call_request** - Video call initiation
13. **video_call_start** - Video call session start

#### Social Media Integration
14. **post_details** - Social media post details
15. **comment_details** - Social media comment details
16. **wall_post_details** - Wall post information
17. **social_feed_tickets** - Social media ticket management

#### Utility Actions
18. **message_json** - JSON message processing
19. **message_agent** - Agent-specific message operations

## Message Actions

### 1. Core Message Actions

#### message_post
**Purpose**: Handle customer-initiated messages from web widgets and external channels

**Functionality**:
- Store customer message in database
- Initialize message processing pipeline
- Trigger response generation (bot or agent)
- Update session state and case information

**Input Parameters**:
- `body`: Message content (text, JSON, or structured data)
- `message_type`: Type of message (message, template, etc.)
- `encoding`: Content encoding (text, json)
- `received_from`: Customer identifier
- `case_id`: Associated case ID (optional)

**Output**:
- Message stored successfully
- Response generated and sent
- Session updated

#### message_reply
**Purpose**: Handle agent replies to customer messages

**Functionality**:
- Store agent reply message
- Link reply to original customer message
- Update case status and session state
- Trigger response delivery to customer

**Input Parameters**:
- `is_reply`: ID of message being replied to
- `body`: Agent reply content
- `case_id`: Case ID for context
- `attending_id`: Agent identifier

**Output**:
- Reply message stored
- Customer notified of response
- Case status updated

#### message_history
**Purpose**: Retrieve message history for customers and agents

**Functionality**:
- Fetch messages based on session or case ID
- Support pagination and filtering
- Transform messages to appropriate format
- Handle message ordering and limits

**Input Parameters**:
- `from_message_id`: Starting message ID for pagination
- `limit`: Maximum number of messages to return
- `order`: Sort order (ASC/DESC)
- `case_id`: Specific case ID (optional)

**Output**:
- Array of message objects
- Pagination metadata
- Message templates and formatting

### 2. Bulk Operations

#### message_bulk_reply
**Purpose**: Allow agents to reply to multiple messages simultaneously

**Functionality**:
- Process multiple message IDs
- Create individual replies for each message
- Dispatch bulk operation as background job
- Provide progress feedback

**Input Parameters**:
- `message_ids`: Array of message IDs to reply to
- `body`: Reply content for all messages
- `case_id`: Case context

**Output**:
- Background job dispatched
- Success confirmation
- Processing status

#### message_bulk_close
**Purpose**: Close multiple messages/cases in bulk

**Functionality**:
- Process multiple message IDs for closure
- Apply disposition and sentiment data
- Trigger CSAT surveys if applicable
- Update case statuses

**Input Parameters**:
- `message_ids`: Array of message IDs to close
- `disposition`: Closure reason/category
- `sentiment`: Customer sentiment analysis

**Output**:
- Bulk closure job dispatched
- Case statuses updated
- CSAT surveys triggered

### 3. Queue & Case Management

#### message_queue
**Purpose**: Handle agent transfer and queue management

**Functionality**:
- Transfer customer from bot to human agent
- Create queue items for agent assignment
- Update session agent type
- Notify agents of new queue items

**Input Parameters**:
- `message_id`: Message triggering transfer
- `queue_type`: Type of queue assignment
- `priority`: Queue priority level

**Output**:
- Queue item created
- Agent transfer initiated
- Session updated to live agent mode

#### message_closed
**Purpose**: Close customer cases with proper disposition

**Functionality**:
- Store closure message with disposition
- Update case status to closed
- Reset session agent to bot
- Trigger CSAT survey
- Log closure details

**Input Parameters**:
- `is_reply`: Message ID being closed
- `disposition`: Closure reason/category
- `sentiment`: Customer sentiment
- `case_id`: Case to close

**Output**:
- Case closed successfully
- CSAT survey initiated
- Session reset to bot mode

### 4. Customer Satisfaction

#### message_csat
**Purpose**: Initiate customer satisfaction surveys

**Functionality**:
- Create CSAT survey for completed case
- Send survey to customer through appropriate channel
- Track survey responses
- Update case with CSAT data

**Input Parameters**:
- `message_id`: Message associated with case
- `case_id`: Case to survey
- `survey_type`: Type of CSAT survey

**Output**:
- CSAT survey created and sent
- Response tracking initiated
- Case updated with survey status

### 5. Video Communication

#### video_call_request
**Purpose**: Initiate video call requests between customers and agents

**Functionality**:
- Create video call session
- Generate call room and credentials
- Notify both customer and agent
- Handle call state management

**Input Parameters**:
- `case_id`: Associated case
- `agent_id`: Assigned agent
- `customer_id`: Customer identifier

**Output**:
- Video call session created
- Call credentials generated
- Participants notified

#### video_call_start
**Purpose**: Start active video call session

**Functionality**:
- Initialize video call room
- Connect participants
- Handle call state transitions
- Record call metadata

**Input Parameters**:
- `call_id`: Video call session ID
- `participant_id`: Participant identifier

**Output**:
- Call session started
- Participants connected
- Call state active

### 6. Social Media Integration

#### post_details
**Purpose**: Retrieve social media post information

**Functionality**:
- Fetch post details from social platforms
- Store post metadata
- Link posts to customer cases
- Handle post interactions

**Input Parameters**:
- `post_id`: Social media post ID
- `platform`: Social media platform
- `case_id`: Associated case

**Output**:
- Post details retrieved
- Metadata stored
- Case updated

#### comment_details
**Purpose**: Handle social media comment interactions

**Functionality**:
- Process social media comments
- Create message entries for comments
- Link comments to posts and cases
- Handle comment responses

**Input Parameters**:
- `comment_id`: Social media comment ID
- `post_id`: Parent post ID
- `body`: Comment content

**Output**:
- Comment processed
- Message created
- Case updated

### 7. Utility Actions

#### message_example
**Purpose**: Provide message template examples

**Functionality**:
- Generate example messages for different types
- Support template rendering
- Provide formatting examples
- Help with message structure

**Input Parameters**:
- `type`: Message type for example
- `format`: Output format (json, template)

**Output**:
- Example message structure
- Template formatting
- Usage guidelines

#### message_json
**Purpose**: Process JSON-formatted messages

**Functionality**:
- Parse and validate JSON message content
- Transform JSON to message format
- Handle structured message data
- Support complex message types

**Input Parameters**:
- `json_data`: JSON message content
- `message_type`: Type of JSON message

**Output**:
- Parsed message data
- Validation results
- Processed message

## Authentication & Authorization

### Customer Mode Authentication
- **Session Validation**: Valid customer session required
- **Session Creation**: Automatic session creation for new customers
- **Session Context**: Customer session provides user context and preferences

### Agent Mode Authentication
- **Agent Login**: Valid agent authentication required
- **Header Validation**: `x-mevrik-attending-id` header for agent identification
- **Session Rehydration**: Customer session recreated from message context
- **Session Path**: Agent can switch between customer sessions

### Authorization Levels
1. **Customer**: Can post messages, view own history
2. **Agent**: Can reply, close cases, access all customer data
3. **Admin**: Full system access and management

## Message Processing Flow

### Customer Message Flow
```
1. Customer sends message via widget/channel
2. MessagesController receives request
3. Session validation/creation
4. Message container creation
5. Message storage
6. Response generation (bot/agent)
7. Response delivery
8. Case/session updates
```

### Agent Message Flow
```
1. Agent initiates action from dashboard
2. Agent authentication validation
3. Customer session rehydration
4. Message container creation
5. Action-specific processing
6. Database updates
7. Customer notification
8. Case/session updates
```

### Bulk Operation Flow
```
1. Agent selects multiple messages
2. Bulk action request sent
3. Background job dispatched
4. Individual operations processed
5. Progress tracking
6. Completion notification
7. Results aggregation
```

## API Specifications

### Request Format
```json
{
  "action": "message_post",
  "message": {
    "body": "Customer message content",
    "message_type": "message",
    "encoding": "text",
    "received_from": "customer_id",
    "case_id": 12345
  },
  "session_path": "optional_session_context"
}
```

### Response Format
```json
{
  "success": true,
  "data": {
    "message_id": 67890,
    "status": "processed",
    "response": "Bot response content"
  },
  "error": null
}
```

### Error Response Format
```json
{
  "success": false,
  "error": "Error message description",
  "code": "ERROR_CODE",
  "details": {}
}
```

## Error Handling

### Validation Errors
- **Invalid Action**: Action not in allowed list
- **Missing Parameters**: Required parameters not provided
- **Invalid Format**: Malformed request data

### Authentication Errors
- **Unauthorized Access**: Invalid or missing authentication
- **Session Expired**: Customer session timeout
- **Agent Not Logged In**: Agent authentication required

### Processing Errors
- **Message Storage Failed**: Database operation failure
- **Session Creation Failed**: Session management error
- **External API Error**: Third-party service failure

### Error Recovery
- **Graceful Degradation**: Fallback to basic functionality
- **Retry Logic**: Automatic retry for transient failures
- **User Notification**: Clear error messages to users

## Performance Considerations

### Optimization Strategies
1. **Message Caching**: Cache frequently accessed messages
2. **Bulk Operations**: Background processing for bulk actions
3. **Database Indexing**: Optimized queries for message retrieval
4. **Session Management**: Efficient session storage and retrieval

### Scalability Measures
1. **Queue Processing**: Asynchronous message processing
2. **Load Balancing**: Distribute message processing load
3. **Database Sharding**: Partition message data by channel/customer
4. **Caching Strategy**: Redis caching for session and message data

### Performance Metrics
- **Response Time**: < 200ms for simple operations
- **Throughput**: Support 1000+ messages per minute
- **Availability**: 99.9% uptime target
- **Error Rate**: < 0.1% error rate

## Security Requirements

### Data Protection
1. **Message Encryption**: Encrypt sensitive message content
2. **Access Control**: Role-based access to message data
3. **Audit Logging**: Log all message operations
4. **Data Retention**: Implement message retention policies

### Input Validation
1. **Sanitization**: Sanitize all input data
2. **XSS Prevention**: Prevent cross-site scripting attacks
3. **SQL Injection**: Parameterized queries only
4. **Rate Limiting**: Prevent message flooding

### Authentication Security
1. **Session Security**: Secure session management
2. **Token Validation**: Validate authentication tokens
3. **Agent Verification**: Verify agent permissions
4. **Customer Verification**: Validate customer identity

## Testing Requirements

### Unit Testing
- **Action Classes**: Test individual message actions
- **Helper Classes**: Test MessagingRequestHelper functionality
- **Validation Logic**: Test input validation and sanitization
- **Error Handling**: Test error scenarios and recovery

### Integration Testing
- **End-to-End Flows**: Test complete message processing flows
- **Database Operations**: Test message storage and retrieval
- **External Integrations**: Test channel-specific integrations
- **Session Management**: Test session creation and management

### Performance Testing
- **Load Testing**: Test under high message volume
- **Stress Testing**: Test system limits and breaking points
- **Concurrent Users**: Test multiple simultaneous users
- **Bulk Operations**: Test bulk message processing

### Security Testing
- **Authentication**: Test authentication mechanisms
- **Authorization**: Test access control and permissions
- **Input Validation**: Test malicious input handling
- **Data Protection**: Test data encryption and security

## Future Enhancements

### Short-term Improvements
1. **Message Templates**: Enhanced template system
2. **Rich Media Support**: Better media message handling
3. **Message Search**: Advanced search capabilities
4. **Analytics Integration**: Enhanced message analytics

### Medium-term Features
1. **AI Integration**: Enhanced AI-powered responses
2. **Multi-language Support**: Better internationalization
3. **Message Scheduling**: Scheduled message delivery
4. **Advanced Filtering**: Sophisticated message filtering

### Long-term Vision
1. **Microservices Architecture**: Break into focused services
2. **Real-time Processing**: WebSocket-based real-time messaging
3. **Machine Learning**: ML-powered message classification
4. **Advanced Analytics**: Predictive analytics and insights

---

## Appendices

### Appendix A: Message Action Reference
| Action | Purpose | Input | Output | Auth Required |
|--------|---------|-------|--------|---------------|
| message_post | Customer message | body, type | stored message | Customer |
| message_reply | Agent reply | is_reply, body | reply message | Agent |
| message_history | Get messages | filters, pagination | message list | Customer/Agent |
| message_queue | Agent transfer | message_id | queue item | Agent |
| message_closed | Close case | disposition, sentiment | closed case | Agent |
| message_csat | Start CSAT | message_id | CSAT survey | Agent |
| message_bulk_reply | Bulk replies | message_ids, body | bulk job | Agent |
| message_bulk_close | Bulk closure | message_ids, disposition | bulk job | Agent |
| video_call_request | Video call | case_id, participants | call session | Agent |
| video_call_start | Start call | call_id | active call | Agent |
| post_details | Social post | post_id, platform | post data | Agent |
| comment_details | Social comment | comment_id, post_id | comment data | Agent |
| message_example | Template example | type, format | example | Any |
| message_json | JSON processing | json_data | parsed data | Any |

### Appendix B: Error Codes
| Code | Description | Resolution |
|------|-------------|------------|
| INVALID_ACTION | Action not supported | Use valid action |
| AUTH_REQUIRED | Authentication needed | Provide valid auth |
| SESSION_INVALID | Invalid session | Re-authenticate |
| MESSAGE_STORE_FAILED | Database error | Retry or contact support |
| BULK_PROCESSING_FAILED | Bulk operation failed | Check individual items |
| EXTERNAL_API_ERROR | Third-party service error | Retry or use fallback |

### Appendix C: Configuration Parameters
| Parameter | Description | Default | Environment |
|-----------|-------------|---------|-------------|
| MESSAGE_CACHE_TTL | Message cache time | 3600 | Production |
| BULK_JOB_TIMEOUT | Bulk operation timeout | 300 | Production |
| SESSION_TTL | Session timeout | 1800 | Production |
| MAX_MESSAGE_SIZE | Maximum message size | 10000 | Production |
| RATE_LIMIT_PER_MINUTE | Rate limit | 100 | Production |

---

*This document is a living document and should be updated as the messaging system evolves.*
