# Mevrik Channels PHP - Brownfield Product Requirements Document

## Executive Summary

Mevrik Channels PHP (BlueBird) is a comprehensive omnichannel customer experience platform built on Laravel 9, serving as the backend service for digital customer engagement across multiple communication channels. The system seamlessly integrates website widgets, Facebook Messenger, email, and social media comments, allowing businesses to engage customers efficiently and consistently. The platform features an AI-powered bot service using Google Dialogflow for automated customer interactions, with intelligent handover to human agents when the bot cannot resolve customer requests.

## Current System Overview

### Architecture
- **Framework**: Laravel 9 (PHP 8.0+)
- **Database**: MySQL/PostgreSQL with Redis caching and ClickHouse analytics
- **Queue System**: Laravel Horizon for job processing
- **Real-time Communication**: WebSockets (Pusher)
- **External Integrations**: Facebook Graph API, Google Cloud Dialogflow, Grameenphone APIs
- **Package Architecture**: Modular design with custom Laravel packages

### Core Packages
1. **Genex\Channels** - Core channel management and messaging
2. **Genex\Mevrik** - Customer relationship management
3. **Genex\Facebook** - Facebook integration and social media handling
4. **Genex\Email** - Email channel management
5. **Genex\Chatbot** - AI chatbot functionality
6. **Genex\Grameenphone** - Telecom service integration
7. **Genex\Reports** - Analytics and reporting
8. **Efemer\Higg** - Core framework utilities

## Current Features & Capabilities

### 1. Multi-Channel Communication
- **Website Widgets**: Embedded chat interfaces on websites with customizable branding
- **Facebook Messenger**: Webhook integration, message handling, profile management
- **Facebook Comments**: Monitoring and responding to comments on Facebook posts
- **Email**: Incoming email processing and response management
- **Webhook Processing**: Generic webhook handling for external integrations

### 2. Session Management
- **Session Types**: Anonymous, Guest, Verified User, Expired
- **Authentication Modes**: 
  - ANON: Anonymous users
  - GUST: Guest users (Facebook, email)
  - USER: Verified users (mobile OTP)
  - EXPR: Expired sessions
  - EXIT: Terminated sessions
- **Session State**: Redis-cached session data with 30-day TTL
- **Device Tracking**: Multi-device session management

### 3. Customer Management
- **Customer Profiles**: Unified customer identity across channels
- **Identity Management**: Multiple identity types (mobile, email, Facebook)
- **Customer Data**: Profile information, preferences, interaction history
- **Verification**: OTP-based mobile verification system

### 4. Message Processing
- **Message Types**: Text, image, video, audio, file, gallery, buttons
- **Message Status**: New, pending, replied, closed, waiting for response
- **Message Routing**: Channel-specific message handling
- **Template System**: Structured message templates for different channels

### 5. Agent System
- **Agent Types**: AI Bot, Live Agent, Email Agent
- **AI-Powered Bot**: Google Dialogflow integration for automated customer interactions
- **Connect Agent Feature**: Automatic handover to human agents when bot cannot resolve requests
- **Queue Management**: Customer queue for live agent assignment
- **Agent Dashboard**: Message management and response interface
- **Transfer Logic**: Intelligent bot to live agent escalation

### 6. AI & NLP Integration
- **Google Dialogflow Integration**: AI-powered bot service for automated customer interactions
- **Intent Recognition**: Advanced intent detection and classification
- **Intent Management**: Custom intent handling and training
- **Language Detection**: Multi-language support
- **Response Generation**: Automated response based on intents
- **Fallback Handling**: Intelligent escalation to human agents when AI cannot resolve

### 7. CSAT (Customer Satisfaction)
- **Survey System**: Post-interaction satisfaction surveys
- **Question Types**: Rating, text, multiple choice
- **Survey Triggers**: Automatic survey sending after case closure
- **Analytics**: CSAT score tracking and reporting

### 8. Case Management
- **Case Lifecycle**: Creation, assignment, resolution, closure
- **Case Types**: Support, sales, general inquiry
- **Case Categories**: Hierarchical categorization system
- **Case Routing**: Automatic and manual case assignment

### 9. Reporting & Analytics
- **Performance Metrics**: Agent performance, response times, resolution rates
- **Channel Analytics**: Channel-specific usage and performance data
- **Customer Insights**: Customer behavior and satisfaction trends
- **Real-time Dashboards**: Live monitoring of system performance

### 10. Integration Capabilities
- **Grameenphone APIs**: Telecom service integration (recharge, balance, offers)
- **Facebook Graph API**: Profile management, message sending, page management
- **Email Services**: SMTP integration for email responses (SendGrid, Mailgun)
- **Webhook System**: Generic webhook processing for external systems
- **Google Cloud Dialogflow**: AI service integration for bot functionality

## Technical Architecture

### Database Schema
- **Core Tables**: 32+ models including users, messages, sessions, customers
- **Key Entities**:
  - `session_users`: User session management
  - `channel_messages`: Message storage and routing
  - `facebook_users`: Facebook-specific user data
  - `csat_surveys`: Customer satisfaction tracking
  - `mevrik_cases`: Case management
  - `customers`: Customer profile management

### API Endpoints
- **Channel Endpoints**: `/facebook`, `/web-widget`, `/listen`
- **Customer Management**: `/customer/*`, `/v1/crm/*`
- **Agent Interface**: `/v1/agent/*`
- **Reporting**: Various report generation endpoints
- **Grameenphone Integration**: `/gp/*` endpoints

### Event System
- **Message Events**: `MessageReceiverEvent`, `ChannelResponseEvent`
- **Session Events**: `SessionCreatedEvent`, `SessionPathChangedEvent`
- **Case Events**: `MevrikCaseCreatedEvent`, `MevrikCaseClosedEvent`
- **CSAT Events**: `MevrikCSATStartEvent`

### Queue System
- **Job Processing**: Laravel Horizon for background job management
- **Message Queuing**: Asynchronous message processing
- **Report Generation**: Scheduled report generation jobs
- **Data Synchronization**: Background data sync operations

## Current Limitations & Technical Debt

### 1. Code Organization
- **Mixed Responsibilities**: Some controllers handle multiple concerns
- **Legacy Code**: Obsolete methods and endpoints still present
- **Inconsistent Naming**: Mixed naming conventions across packages
- **Documentation**: Limited inline documentation

### 2. Performance Concerns
- **Database Queries**: Some N+1 query issues in relationships
- **Caching Strategy**: Inconsistent caching implementation
- **Session Management**: Heavy Redis usage for session data
- **Message Processing**: Synchronous processing in some areas

### 3. Scalability Issues
- **Single Database**: All data in one database instance
- **Session Storage**: Redis dependency for session management
- **File Storage**: Local file storage without CDN integration
- **Queue Processing**: Single queue worker configuration

### 4. Security Considerations
- **API Authentication**: Mixed authentication methods
- **Data Validation**: Inconsistent input validation
- **Rate Limiting**: Limited rate limiting implementation
- **Data Encryption**: Sensitive data encryption needs review

### 5. Monitoring & Observability
- **Logging**: Basic logging without structured logging
- **Metrics**: Limited application metrics collection
- **Error Tracking**: Basic error handling without comprehensive tracking
- **Performance Monitoring**: No APM integration

## Integration Points

### External Systems
1. **Facebook Graph API**: Profile management, messaging, page management
2. **Google Cloud Dialogflow**: AI-powered bot service for automated interactions
3. **Grameenphone APIs**: Telecom service integration
4. **Email Services**: SMTP for email channel (SendGrid, Mailgun)
5. **Redis**: Session caching and temporary data storage
6. **ClickHouse**: Analytics data storage

### Internal Dependencies
1. **Laravel Framework**: Core application framework
2. **Laravel Horizon**: Queue management
3. **Laravel WebSockets**: Real-time communication
4. **Laravel Sanctum**: API authentication
5. **Custom Packages**: Modular functionality packages

## Data Flow Architecture

### Message Processing Flow
1. **Incoming Message**: Webhook receives message from external channel
2. **Message Ingestion**: `MessageReceiverEvent` processes incoming data
3. **Session Management**: Session lookup/creation based on user identity
4. **AI Intent Recognition**: Google Dialogflow processes customer intent
5. **Bot Response Generation**: AI-powered bot generates automated response
6. **Connect Agent Check**: If bot cannot resolve, triggers "Connect Agent" option
7. **Agent Assignment**: Human agent assignment for complex requests
8. **Message Delivery**: Response sent back through appropriate channel
9. **Case Management**: Case creation/update based on interaction
10. **Analytics**: Message and interaction data stored for reporting

### Session Lifecycle
1. **Session Creation**: Anonymous session created on first interaction
2. **Authentication**: User verification via OTP or social login
3. **Session Upgrade**: Anonymous → Guest → Verified User
4. **AI Bot Interaction**: Google Dialogflow processes customer requests
5. **Connect Agent Trigger**: Automatic handover when bot cannot resolve
6. **Agent Assignment**: Bot → Live Agent based on complexity or bot limitation
7. **Case Resolution**: Issue resolution and case closure
8. **CSAT Survey**: Post-resolution satisfaction survey
9. **Session Cleanup**: Session data cleanup after completion

## Current Deployment & Infrastructure

### Environment Configuration
- **Development**: Local development with Docker/Vagrant
- **Staging**: Staging environment for testing
- **Production**: AWS/cloud deployment
- **Environment Files**: Multiple environment configurations

### Dependencies
- **PHP 8.0+**: Core runtime requirement
- **MySQL/PostgreSQL**: Primary database
- **Redis**: Caching and session storage
- **ClickHouse**: Analytics database
- **Node.js**: Frontend asset compilation
- **Composer**: PHP dependency management

### Deployment Process
- **Artisan Commands**: Database migrations, cache clearing
- **Queue Workers**: Background job processing
- **Web Server**: Nginx/Apache configuration
- **SSL/TLS**: HTTPS configuration for production

## Recommendations for Improvement

### 1. Code Quality
- **Refactoring**: Break down large controllers into smaller, focused classes
- **Testing**: Implement comprehensive unit and integration tests
- **Documentation**: Add comprehensive API documentation
- **Code Standards**: Enforce consistent coding standards

### 2. Performance Optimization
- **Database Optimization**: Implement proper indexing and query optimization
- **Caching Strategy**: Implement consistent caching across the application
- **Async Processing**: Move more operations to background jobs
- **CDN Integration**: Implement CDN for static assets

### 3. Scalability Enhancements
- **Database Sharding**: Implement database sharding for large datasets
- **Microservices**: Consider breaking into smaller, focused services
- **Load Balancing**: Implement proper load balancing
- **Auto-scaling**: Implement auto-scaling for queue workers

### 4. Security Improvements
- **API Security**: Implement comprehensive API security measures
- **Data Encryption**: Encrypt sensitive data at rest and in transit
- **Rate Limiting**: Implement proper rate limiting
- **Security Auditing**: Regular security audits and penetration testing

### 5. Monitoring & Observability
- **APM Integration**: Implement Application Performance Monitoring
- **Structured Logging**: Implement structured logging with correlation IDs
- **Metrics Collection**: Comprehensive metrics collection and alerting
- **Health Checks**: Implement comprehensive health check endpoints

## Conclusion

Mevrik Channels PHP (BlueBird) is a robust omnichannel customer experience platform with comprehensive functionality for multi-channel communication, AI-powered customer interactions, and analytics. The system's integration of Google Dialogflow for AI-powered bot services, combined with intelligent "Connect Agent" functionality, provides a seamless customer experience that automatically escalates to human agents when needed.

While the system provides extensive features and integrations, there are opportunities for improvement in code organization, performance optimization, scalability, and security. The modular package architecture provides a good foundation for future enhancements and maintenance.

The system successfully handles complex customer interactions across multiple channels while maintaining session state, providing AI-powered responses, and offering comprehensive reporting capabilities. With proper refactoring and optimization, the platform can scale to handle increased load and provide even better performance and reliability.
