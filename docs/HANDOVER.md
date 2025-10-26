# 🧾 Project Handover Document

**Project Name:** Mevrik Channels  
**Version:** 1.0  
**Date:** October 26, 2025  
**Prepared By:** Nazrul Islam  
**Handover To:** [New Developer Name]

---

## 📑 Table of Contents

1. [Project Overview](#1-project-overview)
2. [Repository Information](#2-repository-information)
3. [Environment Details](#3-environment-details)
4. [Deployment Process](#4-deployment-process)
5. [Database Information](#5-database-information)
6. [Queue and Jobs](#6-queue-and-jobs)
7. [Cron Jobs](#7-cron-jobs)
8. [API and Integrations](#8-api-and-integrations)
9. [Notifications and Communication](#9-notifications-and-communication)
10. [Authentication and Security](#10-authentication-and-security)
11. [Monitoring and Logs](#11-monitoring-and-logs)
12. [Known Issues and TODOs](#12-known-issues-and-todos)
13. [Documentation and References](#13-documentation-and-references)
14. [Contacts and Ownership](#14-contacts-and-ownership)
15. [Quick Start for New Developers](#15-quick-start-for-new-developers)

---

## 1. Project Overview

### Summary
Mevrik Channels is a multi-channel messaging platform built on Laravel that enables businesses to manage customer communications across various channels (Facebook Messenger, Facebook Comment, Website Widget, Email, Video Call etc.). The platform includes bot services, journey management, analytics, and reporting capabilities.

### Key Features
- Multi-channel messaging support (Facebook Messenger, Facebook Comment, Widget, Email etc.)
- Automated bot journeys and conversation flows
- Real-time message processing with queues
- Session management and state tracking
- Language detection
- Customer engagement analytics
- Report generation and exports

### Tech Stack
- **Framework:** Laravel 9 (PHP 8.2)
- **Database:** MySQL, Postgress, Clickhouse
- **Cache:** Redis
- **Web Server:** Nginx with PHP-FPM
- **Queue:** Laravel Horizon (Redis-based)
- **Process Manager:** Supervisor
- **Real-time Communication:** WebSocket (Laravel WebSockets) + Pusher
- **Package Manager:** Composer (PHP)

### Architecture
- **Action-based Architecture:** Uses Laravel Actions (Lorisleiva) pattern where single classes can act as Controllers, Jobs, Listeners, and Commands simultaneously
- **Event-driven Architecture:** Laravel Events & Listeners for decoupled components
- **Queue-based Asynchronous Processing:** Background job processing with Laravel Horizon
- **Modular Package Design:** Custom packages (Efemer, Genex) for domain-specific functionality (located in `packages/` directory)
- **Service-Oriented Design:** Business logic encapsulated in dedicated service classes

### Laravel Actions Pattern
The project extensively uses the `lorisleiva/laravel-actions` package, which allows a single class to serve multiple purposes:

```php
use Lorisleiva\Actions\Concerns\AsAction;

class MessageReply
{
    use AsAction;
    
    // Can be dispatched as a Job or controller
    public function handle($payload) { ... }
    
    
    public function asJob( $payload) { ... }
}
```

**Benefits:**
- Reduces code duplication
- Single class for multiple execution contexts
- Easier testing and maintenance
- Consistent business logic across different entry points

**Example Usage:**
```php
// As a Job
MessageReply::dispatch($payload);

// As a direct call
MessageReply::run($payload);
```

**Locations:** Primarily used in `packages/genex/laravel-reports/src/Actions/` and `packages/genex/laravel-mevrik/src/Actions/`

---

## 2. Repository Information

```
Repository: mevrik-channels-php
Location: https://github.com/genexgit/mevrik-channels-php
Branch Strategy:
  - channels_v4: Production
  - nazrul/golive: Staging environment (Current)
  - {developerName}/{branchName}: Developer-specific feature/fix branches (e.g., john/v4_report)
```

### Key Directories

**⚠️ Important:** All API endpoints, models, actions, controllers, and core business logic are organized within the `packages/` directory, not in the main `app/` directory.

```
packages/                                    # Main codebase location
├── efemer/
│   └── laravel-higg/                       # Efemer Higg package
│       ├── src/                            # Package source code
│       └── factory/
│           └── handlers/                   # Core handler classes
│               ├── FormSubmission handlers # Form data processing and validation
│               ├── ReportBrowser handlers  # Report viewing and filtering
│               └── Action handlers         # Generic action processing
│
└── genex/                                  # Genex packages (main business logic)
    ├── laravel-mevrik/                     # Core Mevrik functionality
    │   ├── src/Actions/                    # Laravel Actions (Jobs/Controllers/Listeners)
    │   ├── src/Models/                     # Eloquent models
    │   └── src/Services/                   # Business logic services
    │
    ├── laravel-reports/                    # Reporting functionality
    │   ├── src/Actions/                    # Report generation actions
    │   │   ├── Report/                     # Report generators
    │   │   └── Stats/                      # Statistics preparation
    │   └── src/Models/                     # Report models
    │
    ├── laravel-channels/                   # Channel management
    │   ├── src/Controllers/                # API controllers
    │   ├── src/Models/                     # Channel models
    │   └── src/Services/                   # Channel services
    │
    ├── laravel-chatbot/                    # Chatbot functionality
    │   ├── src/Actions/                    # Chatbot actions
    │   └── src/Models/                     # Chatbot models
    │
    ├── laravel-facebook/                   # Facebook integration
    │   ├── src/Controllers/                # Facebook controllers
    │   └── src/Services/                   # Facebook services
    │
    ├── laravel-email/                      # Email channel
    │   └── src/                            # Email functionality
    │
    └── laravel-grameenphone/               # Grameenphone specific integration
        └── src/                            # GP-specific code

app/                                        # Laravel base app
├── Console/Kernel.php                      # Scheduled tasks
├── Events/                                 # Global events
├── Listeners/                              # Global listeners
├── Http/
│   ├── Kernel.php                          # Middleware registration
│   └── Middleware/                         # Global middleware
├── Providers/                              # Service providers
└── Services/
    └── apiResponse/                        # Bot journey responses & API resources
        ├── roaming/                        # Roaming bot journey responses
        ├── [other_bot_journeys]/           # Other journey-specific folders
        └── apiResources.php                # GP (Grameenphone) API - single class for all GP APIs

config/                                     # Configuration files
database/migrations/                        # Database migrations
routes/
├── api.php                                 # Main route file (loads package routes)
└── web.php                                 # Web routes
```

### Bot Journeys & Google Dialogflow Integration
Bot journey responses and Google Dialogflow integration are managed in the `app/` directory:

**Location:** `app/Services/apiResponse/`

This directory contains **static bot journey response files** organized by journey type:

```
app/Services/apiResponse/
├── roaming/                    # Roaming bot journey responses
│   ├── greeting.php           # Static responses for greetings
│   ├── plans.php              # Roaming plan information
│   └── activation.php         # Activation flow responses
│
├── balance/                    # Balance inquiry journey
├── recharge/                   # Recharge journey
├── offers/                     # Offers and promotions journey
└── apiResources.php            # ⭐ GP (Grameenphone) API class
                                # Single class containing ALL GP API integrations
```

**Key Points:**

1. **Static Journey Files:** Each bot journey (roaming, balance, recharge, etc.) has its own folder with response files
2. **Customer-Specific:** Journey responses are tailored for specific customers and use cases
3. **Dialogflow Integration:** These files work with Google Dialogflow for natural language processing
4. **GP API Centralization:** All Grameenphone APIs are consolidated in `apiResources.php` for easier maintenance

**Example Usage:**
```php
// Bot journey response files return structured data
// app/Services/apiResponse/roaming/greeting.php
return [
    'type' => 'text',
    'message' => 'Welcome to Roaming Services! How can I help you today?',
    'quick_replies' => ['Check Plans', 'Activate Roaming', 'Get Help']
];

// GP API calls via apiResources.php
use App\Services\apiResponse\apiResources;

$gpApi = new apiResources();
$roamingPacksDetails = $gpApi->roamingPacksDetails($msisdn);
```

**Why in `app/` Directory?**
Bot journey responses are customer-specific static files that change frequently based on business requirements, so they're kept separate from the core package logic for easier updates.

### Efemer Higg Handlers
The `packages/efemer/laravel-higg/factory/handlers/` directory contains critical handler classes for processing various operations:

**Handler Types:**

1. **Form Submission Handlers**
   - Process form data submissions
   - Handle data validation and sanitization
   - Transform input data before storage
   - Manage form-specific business logic

2. **Report Browser Handlers**
   - Handle report viewing requests
   - Implement filtering and sorting logic
   - Format report data for display
   - Manage pagination for large reports

3. **Action Handlers**
   - Generic action processing framework
   - Handle custom actions and workflows
   - Process user interactions
   - Execute business operations

**Usage Pattern:**
```php
// Form Submission Handler Example
use Efemer\Higg\Factory\Handlers\FormHandler;

// FormHandler("{model_name}.{action_name}", config)
// Example: "message.register-form" where:
//   - "message" = model name
//   - "register-form" = action name
$form = new FormHandler("message.register-form", [
    'data'   => $data,      // Form data to process
    'query'  => $query,     // Additional query parameters
    'action' => 'submit'    // Action type (submit, update etc.)
]);

$form->handleSubmit();

if ($form->isSucceeded()) {
    // Form submission successful
    // Process others logic if any
}
```

**FormHandler Naming Convention:**
- Format: `"{model_name}.{action_name}"`
- **Model Name:** The entity being operated on (e.g., `message`, `case`, `session`)
- **Action Name:** The specific operation (e.g., `register-form`)
- Examples:
  - `"message.register-form"` - Register a new message
  - `"customer.update-details"` - Update customer details

**Location:** `packages/efemer/laravel-higg/factory/handlers/`

This handler-based architecture provides a flexible and extensible way to process different types of operations without coupling the code to specific implementations.

### Package Routes
All API routes are defined within their respective packages:
```
packages/genex/
├── laravel-mevrik/routes/                  # Mevrik routes
├── laravel-channels/routes/                # Channel routes
├── laravel-chatbot/routes/                 # Chatbot routes
├── laravel-facebook/routes/                # Facebook routes
├── laravel-email/routes/                   # Email routes
├── laravel-reports/routes/                 # Report routes
└── laravel-grameenphone/routes/            # Grameenphone routes
```

**Example:** To find Facebook-related endpoints, check: `packages/genex/laravel-facebook/routes/`

### Package Autoloading
```json
"autoload": {
    "psr-4": {
        "App\\": "app/",
        "Efemer\\Higg\\": "packages/efemer/laravel-higg/src/",
        "Genex\\Mevrik\\": "packages/genex/laravel-mevrik/src/",
        "Genex\\Channels\\": "packages/genex/laravel-channels/src/",
        "Genex\\Chatbot\\": "packages/genex/laravel-chatbot/src/",
        "Genex\\Grameenphone\\": "packages/genex/laravel-grameenphone/src/",
        "Genex\\Email\\": "packages/genex/laravel-email/src/",
        "Genex\\Facebook\\": "packages/genex/laravel-facebook/src/",
        "Genex\\Reports\\": "packages/genex/laravel-reports/src/"
    }
}
```

### Dependencies
```bash
# Install PHP dependencies
composer install

```
---

## 3. Environment Details

### Environment Variables
All environment configurations are stored in the root **`.env`** file.

**Location:** `/.env` (project root)

**Setup:**
```bash
# For new setup, copy from example
cp .env.example .env

# Edit with your environment-specific values
sudo vim .env
```

**Note:** The `.env` file is git-ignored. Never commit sensitive credentials to version control.

### Key Environment Variables

```env
# ==================== Application ====================
APP_NAME=Channels
APP_ENV=production
APP_KEY=
APP_DEBUG=false

# URLs
APP_URL=
WIDGET_URL=
WIDGET_STATIC_URL=
MEVRIK_API_URL=
MEVRIK_MAIL_URL=
MEVRIK_VIDEO_URL=
BASE_URL=
BASE_URL_API=

# ==================== Laravel Horizon ====================
HORIZON_PREFIX=production
HORIZON_MAX_PROCESS=20
HORIZON_MIN_PROCESS=5

# ==================== Logging ====================
LOG_CHANNEL=production
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

# GELF Host
GELF_HOST=
GELF_PORT=

# ==================== Database - MySQL ====================
DB_CONNECTION=mysql
DB_HOST=
DB_PORT=
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=

# Email Database
DB_DATABASE_EMAIL=
DB_HOST_EMAIL=
DB_HOST_EMAIL_READ=

# Social Report Database
DB_CONNECTION_SOCIAL=mysqlSocialReport
DB_DATABASE_SOCIAL=

# ==================== Database - PostgreSQL ====================
PG_DATABASE_URL="{DB_URL}"
PG_DB_HOST=
PG_DB_PORT=
PG_DB_DATABASE=
PG_DB_USERNAME=
PG_DB_PASSWORD=

# ==================== Database - Clickhouse (Analytics) ====================
CLICKHOUSE_HOST=
CLICKHOUSE_PORT=
CLICKHOUSE_USERNAME=
CLICKHOUSE_PASSWORD=
CLICKHOUSE_DATABASE=
CLICKHOUSE_TIMEOUT_CONNECT=2
CLICKHOUSE_TIMEOUT_QUERY=2

# ==================== Cache & Session ====================
BROADCAST_DRIVER=soketi
CACHE_DRIVER=file
FILESYSTEM_DRIVER=local
QUEUE_CONNECTION=redis
SESSION_DRIVER=file
SESSION_LIFETIME=120

# ==================== Redis ====================
REDIS_HOST=
REDIS_PASSWORD=
REDIS_PORT=
REDIS_DB=0
REDIS_CACHE_DB=1
REDIS_CACHE_EXPIRE=300
REDIS_HOST_DATA=

# ==================== Pusher/WebSocket (Real-time Communication) ====================
PUSHER_HOST=
PUSHER_PORT=
PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=

# Laravel WebSockets SSL
LARAVEL_WEBSOCKETS_SSL_LOCAL_CERT={ServerKeyLocation}
LARAVEL_WEBSOCKETS_SSL_LOCAL_PK={ServerKeyLocation}

# ==================== Facebook Integration ====================
FACEBOOK_PAGE_ID=
FACEBOOK_GRAPH_API=https://graph.facebook.com
FACEBOOK_GRAPH_VERSION=v20.0
FACEBOOK_VERIFY_TOKEN=
FACEBOOK_ACCESS_TOKEN=

# ==================== Google Dialogflow (Bot Journey/NLU) ====================
GOOGLE_PROJECT_ID=
GOOGLE_PROJECT_MEVRIK_ID=
GOOGLE_PROJECT_SMALLTALK_ID=
GOOGLE_PROJECT_MEVRIK=
GOOGLE_PROJECT_SMALLTALK=
DIALOGUE_DOMAIN_SCORE=0.6
DIALOGUE_SMALL_SCORE=0.5

# ==================== NLP/NLU ====================
WIT_URL=
WIT_APP_ACCESS_TOKEN=
WIT_APP_VERSION=
WIT_APP_ACCESS_TOKEN_NLU=
WIT_APP_VERSION_NLU=
NLP_URL={HOST_URL}

# ==================== JWT Authentication ====================
JWT_SECRET=

# ==================== AWS S3 (File Storage) ====================
FILESYSTEM_DRIVER=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=
AWS_BUCKET=
AWS_BUCKET_EMAIl=
AWS_CDN=
AWS_BUCKET_EMAIL_PREVIOUS=

# ==================== Error Tracking ====================
BUGSNAG_API_KEY=

# ==================== Slack Alerts ====================
SLACK_ALERT_WEBHOOK=
SLACK_ERROR_WEBHOOK=

# ==================== Webhooks ====================
WEBHOOK_CLIENT_SECRET=mevrik

# ==================== SFTP (daily DUMP Report Upload) ====================
SFTP_HOST=
SFTP_USERNAME=
SFTP_PASSWORD=

# ==================== Cron Monitoring ====================
CRONITOR_API_KEY=

# ==================== Laravel Octane ====================
OCTANE_SERVER=swoole

# ==================== Mevrik Instance Config ====================
MEVRIK_INSTANCE=worker
MEVRIK_CACHED_API=0
MEVRIK_SESSION_TTL=15
MEVRIK_SILENT=0
MEVRIK_DEFAULT_APP=1
MEVRIK_BOT_PERSONAL_URL={DEfault Avater CDN Link}

# ==================== GP (Grameenphone) API ====================
GP_API_ENV={APP_ENV}
```

**Environment-Specific Setup:**
For different environments (local, staging, production), update the values in the same `.env` file according to the environment where the application is deployed.

### Server Requirements
- **PHP:** >= 8.2
- **MySQL:** >= 5.7
- **Redis:** >= 6.0
- **Composer:** >= 2.x
- **Nginx:** Latest stable

### Nginx Configuration

The application runs on **Nginx with PHP-FPM** (PHP 8.2). There are two server blocks configured:

#### Server Block 1: Main Application (Port 4202 - SSL)
**Location:** `/etc/nginx/sites-available/mevrik-channels`

```nginx
server {
    listen 4202 ssl;
    server_name _;
    root /var/www/html/mevrik-channels-php/public;
    index index.php index.html index.htm index.nginx-debian.html;

    # Extended timeouts for long-running requests
    proxy_read_timeout 3000s;
    proxy_connect_timeout 3000s;
    proxy_send_timeout 3000s;
    fastcgi_read_timeout 3000s;

    # SSL Configuration
    ssl_certificate     {ENTER_CERTIFICATE_LOCATION};
    ssl_certificate_key {ENTER_CERTIFICATE_KEY_LOCATION};
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    location / {
        client_max_body_size 512M;  # Support large file uploads
        proxy_read_timeout 3000s;
        proxy_connect_timeout 3000s;
        proxy_send_timeout 3000s;
        fastcgi_read_timeout 3000s;
        try_files $uri $uri/ /index.php?$query_string;
    }

    # PHP-FPM Configuration
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    }

    # Deny access to .htaccess files
    location ~ /\.ht {
        deny all;
    }
}
```

#### Server Block 2: API/Health Check (Port 4203 - Non-SSL)
```nginx
server {
    listen 4203;
    server_name _;
    root /var/www/html/mevrik-channels-php/public;
    index index.php index.html index.htm index.nginx-debian.html;

    # Health check endpoints (status/ping)
    location ~ ^/(status|ping)$ {
        access_log off;
        include fastcgi_params;
        allow all;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;    
        fastcgi_pass 127.0.0.1:9001;  # PHP-FPM via TCP
    }

    location / {
        client_max_body_size 512M;
        try_files $uri $uri/ /index.php?$query_string;
    }

    # PHP-FPM Configuration
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    }

    # Deny access to .htaccess files
    location ~ /\.ht {
        deny all;
    }
}
```

#### Key Configuration Notes:

1. **Application Root:** `/var/www/html/mevrik-channels-php/public`

2. **Ports:**
   - `4202` - Main application with SSL
   - `4203` - API/health check endpoints (non-SSL, internal use)

3. **SSL Certificates:**
   - Wildcard certificate for `*.mevrik.com`
   - Location: `{Added Location}`

4. **PHP-FPM Socket:**
   - Unix socket: `/var/run/php/php8.2-fpm.sock`
   - TCP socket for health checks: `127.0.0.1:9001`

5. **Timeouts:**
   - All timeouts set to `3000s` (50 minutes) for long-running report generation and batch processing

6. **File Upload Limit:**
   - `client_max_body_size 512M` - Supports large file uploads (media, documents)

7. **Health Check Endpoints:**
   - `/status` - PHP-FPM status
   - `/ping` - PHP-FPM ping endpoint

#### Nginx Commands:
```bash
# Test configuration
sudo nginx -t

# Reload configuration
sudo nginx -s reload

# Restart nginx
sudo systemctl restart nginx

# Check status
sudo systemctl status nginx

# View error logs
sudo tail -f /var/log/nginx/error.log

# View access logs
sudo tail -f /var/log/nginx/access.log
```

### PHP-FPM Configuration
- **PHP Version:** 8.2
- **FPM Pool:** Custom pool configuration recommended
- **PHP Memory Limit:** 512M minimum
- **Max Execution Time:** 3000 seconds (set in nginx timeouts)
- **Socket:** Unix socket for better performance

### PHP.ini Configuration

**Location:** `/etc/php/8.2/fpm/php.ini` and `/etc/php/8.2/cli/php.ini`

**Required Changes:**

```ini
# Upload Settings
upload_max_filesize = 512M
post_max_size = 512M
max_file_uploads = 20

# Execution Settings
max_execution_time = 3000
max_input_time = 3000

# Memory Settings
memory_limit = 512M

# Timezone
date.timezone = Asia/Dhaka

# Error Reporting (Production)
display_errors = Off
display_startup_errors = Off
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
log_errors = On
error_log = /var/log/php8.2-fpm-error.log

# Session Settings
session.gc_maxlifetime = 7200
session.cookie_lifetime = 7200

# OPcache Settings (for better performance)
opcache.enable=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=10000
opcache.revalidate_freq=2
opcache.fast_shutdown=1
```

**Note:** The `upload_max_filesize` must be coordinated with nginx's `client_max_body_size` setting. Both are set to 512M to support large file uploads (images, videos, documents).

#### Apply PHP Configuration Changes:

```bash
# Edit php.ini
sudo vim /etc/php/8.2/fpm/php.ini
sudo vim /etc/php/8.2/cli/php.ini

# Verify PHP-FPM configuration
sudo php-fpm8.2 -t

# Restart PHP-FPM to apply changes
sudo systemctl restart php8.2-fpm

# Check PHP-FPM status
sudo systemctl status php8.2-fpm

# View PHP configuration
php -i | grep upload_max_filesize
php -i | grep post_max_size
php -i | grep memory_limit
```

---

## 4. Deployment Process

### Recommended Deployment (Using `mm` Alias)

The **`mm` command** is a custom alias that automates the entire deployment process:

```bash
# Navigate to project folder
cd /var/www/html/mevrik-channels-php

# Run the deployment alias (handles everything automatically)
mm
```

**What the `mm` command does:**
1. Pulls latest code from git
2. Installs/updates Composer dependencies
3. Clears and caches configurations (config, route, view)
4. Stops all supervisor services
5. Rereads supervisor configuration
6. Starts all supervisor services

This is the **recommended approach** for production deployments.

---

### Manual Deployment (Step by Step)

If you need to deploy manually or debug individual steps:

```bash
# 1. Pull latest code
git pull origin {branchName}

# 2. Install/update dependencies if any
composer install --no-dev --optimize-autoloader

# 3. Run migrations (if needed)
php artisan migrate --force

# 4. Clear and cache configurations
php artisan config:cache
php artisan route:cache
php artisan view:cache
# Or use single command
php artisan optimize:clear

# 5. Restart services (Best Practice)
sudo supervisorctl stop all
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start all
```

### Rollback Procedure
```bash
# Revert to previous commit
git reset --hard HEAD~1

# Rollback migrations (if needed)
php artisan migrate:rollback --step=1

# Clear caches and restart
php artisan config:clear
sudo supervisorctl restart all
```

### Deployment Checklist
- [ ] Test in staging environment
- [ ] Review migration files if any
- [ ] Check environment variables
- [ ] Verify queue workers are running via the horizon dashboard
- [ ] Monitor error logs post-deployment via the graylog

---

## 5. Database Information

### Connection Details
```
Host: [DB_HOST]
Port: 
Database: 
User: [DB_USERNAME]
Password: [DB_PASSWORD]
```

### Key Tables

| Table Name | Database Type | Database Name | Description |
|------------|---------------|---------------|-------------|
| `email_messages` | MySQL | `dev_channels_db` | Store email messages |
| `facebook_posts` | MySQL | `dev_channels_db` | Store Facebook customer comments |
| `messages` | MySQL | `dev_channels_db` | Message storage across all channels |
| `mevrik_cases` | MySQL | `dev_channels_db` | Conversation threads |
| `sessions` | MySQL | `dev_channels_db` | User sessions |
| `customers` | MySQL | `dev_channels_db` | Customer records |
| `csat_surveys` | MySQL | `dev_channels_db` | Customer feedback/satisfaction values |
| `flows` | MySQL | `dev_channels_db` | Store SmallTalk intent names |
| `report_master` | MySQL | `dev_channels_db` | Generated master report (all-in-one) |
| `webhook_event_logs` | MySQL | `dev_channels_db` | Webhook event logs |
| `gp_api_configurations` | MySQL | `dev_channels_db` | GP API host configuration |
| `gp_api_service_endpoints` | MySQL | `dev_channels_db` | GP API endpoint configuration |
| `gp_api_service_requests` | MySQL | `dev_channels_db` | GP API service call request logs |
| `page_posts` | MySQL | `dev_social_channels_report` | Store All Facebook Post Data |
| `report_social_customer_social_activities` | MySQL | `dev_social_channels_report` | Store All Close comment data with the maded actions |



### Migrations
- Location: `database/migrations/`
- Run migrations: `php artisan migrate`
- Rollback: `php artisan migrate:rollback`
- Fresh install: `php artisan migrate:fresh --seed` (⚠️ Destroys data)

### Seeders
- Location: `database/seeders/`
- Run all seeders: `php artisan db:seed`
- Specific seeder: `php artisan db:seed --class=UserSeeder`
---

## 6. Queue and Jobs

### Queue Configuration
The application uses **Laravel Horizon** for queue management with Redis as the backend.

### Horizon Dashboard
- **URL:** `https://channels.mevrik.com/h0r1z0n`

### Queue Management
```bash
# Start Horizon (managed by Supervisor)
php artisan horizon

# Terminate Horizon gracefully
php artisan horizon:terminate

# Pause all queues
php artisan horizon:pause

# Continue processing
php artisan horizon:continue

# Check queue status
php artisan horizon:status

# Manual queue worker (without Horizon) : not needed to run this
php artisan queue:work redis --queue=default,report,facebook
```

### *** Key Jobs
```php
MasterReportScheduler  - Generates LiveChat reports along with other necessary data reports
CustomerSocialActivityReportGenerator  - Generates customer social reports

```

### Supervisor Configuration
Location: `/etc/supervisor/conf.d/`

```ini
[program:horizon]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/html/mevrik-channels-php/artisan horizon
autostart=true
autorestart=true
#user=forge
numprocs=6
redirect_stderr=true
stdout_logfile=/var/www/html/mevrik-channels-php/storage/logs/horizon.log
stopwaitsecs=3600
```

**Supervisor Commands:**
```bash
# Restart Horizon workers
sudo supervisorctl restart horizon:*

# View status
sudo supervisorctl status

# Reload configuration
sudo supervisorctl stop horizon:*
sudo supervisorctl reread
sudo supervisorctl update
```

### Failed Jobs
```bash
# View failed jobs
php artisan queue:failed

# Retry specific failed job
php artisan queue:retry [job-id]

# Retry all failed jobs
php artisan queue:retry all

# Flush failed jobs
php artisan queue:flush
```

---

## 7. Cron Jobs

### Scheduled Tasks
Defined in: `app/Console/Kernel.php`

```php
// Example schedule
$schedule->call(function() use (&$cron){
            $cron->job( 'LiveChat:AgentHourlyPerformance', function() {
                return AgentHourlyPerformanceScheduler::run();
            } );
        })->hourly();
```

### Laravel Scheduler
The Laravel scheduler must be configured in system cron:

```bash
* * * * * cd /path/to/project && php artisan schedule:run >> /dev/null 2>&1
```

### Key Scheduled Tasks
- **Daily Report Generation** - `reports:generate` at 2:00 AM
- **Session Cleanup** - Hourly cleanup of expired sessions
- **Analytics Aggregation** - Daily at 2:00 AM
- **Database Backups** - Daily at 3:00 AM
- **Log Rotation** - Weekly

---

## 8. API and Integrations

### Internal APIs

#### Base URL
```
Production: 
Staging: https://channels-staging.mevrik.com
```

#### Authentication
```bash
# JWT Authentication
Authorization: Bearer {token}


#### Key API Endpoints

**Route File Location:** `packages/genex/laravel-channels/routes/channels-routes.php`  
**Controllers Location:** `packages/genex/laravel-channels/src/Controllers/`
```

| Method | Endpoint | Controller | Description | Authentication | Middleware |
|--------|----------|------------|-------------|----------------|------------|
| GET | `/init-by-trusted-origin` | `ChannelsController` | Get session token based on trusted origin | None | `api` |
| GET | `/i-was-here` | `ChannelsController` | Rehydrate session from cookie reference | None | `api` |
| POST | `/init-chat` | `ChannelsController` | Get session token by client id and client secret | None | `api` |
| GET | `/widget-persona` | `ChannelsController` | Get widget persona for anonymous load | None | `api` |
| GET | `/referrer_verification` | `ChannelsController` | Get client id and client secret by trusted origin | None | `api` |
| POST | `/pusher/auth` | `ChannelsController` | Pusher auth token for private channel | None | `api` |
| POST | `/pusher/event` | `ChannelsController` | Pusher event proxy | None | `api` |
| POST | `/api/v1/ping` | `ChannelsController` | Check token validity | Session Token | `api`, `channel.auth` |
| POST | `/api/v1/reset` | `ChannelsController` | Reset session | Session Token | `api`, `channel.auth` |
| POST | `/api/v1/session-data` | `ChannelsController` | Get user session data by token | Session Token | `api`, `channel.auth` |
| POST | `/api/v1/session-room` | `ChannelsController` | Video call details | Session Token | `api`, `channel.auth` |
| POST | `/api/v1/claim-session` | `ChannelsController` | Update session with user reference | Session Token | `api`, `channel.auth` |
| POST | `/api/v1/claim_guest_session` | `ChannelsController` | Save session for guest user | Session Token | `api`, `channel.auth` |
| POST | `/api/v1/tools/uploader` | `ChannelsController` | Session-only upload API | Session Token | `api`, `channel.auth` |
| POST | `/api/v1/messages` | `MessagesController` | Ingest messages from live chat widget | Session Token | `channel.auth`, `mevrik.action` |
| POST | `/api/v1/agent/messages` | `MessagesController` | Agent message endpoint | Agent Token | `channel.agent`, `mevrik.action` |
| POST | `/api/v1/agent/video` | `MevrikVideoController` | Control Mevrik video | Agent Token | `channel.agent`, `mevrik.action` |
| GET | `/api/v1/webhook/facebook` | `FacebookController` | Facebook webhook verification | None | None |
| POST | `/api/v1/webhook/facebook` | `FacebookController` | Facebook webhook receiver | None | None |
| GET | `/api/v1/webhook/facebook-social` | `FacebookController` | Facebook social webhook verification | None | None |
| POST | `/api/v1/webhook/facebook-social` | `FacebookController` | Facebook social webhook receiver | None | None |
| POST | `/api/v1/ingest/facebook` | `FacebookController` | Direct Facebook message ingestion | None | None |
| POST | `/api/v1/widget/{action?}` | `WidgetController` | Widget actions | None | None |
| POST | `/api/v1/messaging/remote` | `ShimApiChannelController` | API channel shimming | None | None |

**Middleware Definitions:**
- `api` - Basic API middleware
- `channel.auth` - Session token authentication for end users
- `channel.agent` - Agent authentication for internal users
- `mevrik.action` - Custom action handler middleware

### External Integrations

#### Facebook Messenger
- **Webhook URL:** `/api/webhooks/facebook`
- **Verify Token:** Stored in `.env` as `FACEBOOK_VERIFY_TOKEN`
- **App ID:** `FACEBOOK_APP_ID`
- **App Secret:** `FACEBOOK_APP_SECRET`
- **Documentation:** [Facebook Messenger Platform Docs](https://developers.facebook.com/docs/messenger-platform)


### Webhook Debugging
```bash
# View webhook logs
tail -f storage/logs/webhooks.log

# Test webhook locally with ngrok
./ngrok.sh  # Starts ngrok tunnel
```

---

## 9. Notifications

### Slack Alerts
- **Slack** - Configured in `config/slack-alerts.php`
```

```bash
# Test Slack notification
php artisan slack:test
```

### Broadcasting & Real-time Communication
- **Driver:** Pusher (for real-time events)
- **WebSocket Server:** Laravel WebSockets package (`beyondcode/laravel-websockets`)
- **Configuration:** `config/broadcasting.php`, `config/websockets.php`

#### Pusher Configuration
```env
PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=ap1
```

#### Broadcasting Events
The application uses Laravel's broadcasting feature for real-time updates:
- Message notifications
- Customer status changes
- Queue updates
- Live chat events

---

## 10. Authentication and Security

### Authentication Methods
- **JWT Tokens** - API authentication
- **Session-based** - Web authentication

### JWT Configuration
Config: `config/jwt.php`

```bash
# Generate JWT secret
php artisan jwt:secret
```

### Security Measures
- **CORS:** Configured in `config/cors.php`
- **Middleware:** Custom authentication middleware in `app/Http/Middleware/`
- **Rate Limiting:** API throttling enabled
- **CSRF Protection:** Enabled for web routes
- **XSS Protection:** Blade templating auto-escapes output

### Key Security Files
```
app/Http/Middleware/
├── Authenticate.php
├── CheckChannelAccess.php
├── ValidateWebhookSignature.php
└── [other security middleware]
```

### Secrets Management
- **Never commit** `.env` files
- Store production credentials in secure password manager
- Use environment-specific secrets

---

## 11. Monitoring and Logs

### Application Logs
```bash
# Main application log
storage/logs/laravel.log

# Horizon log
storage/logs/horizon.log

# Webhook logs
storage/logs/webhooks.log

# View logs in real-time
tail -f storage/logs/laravel.log

# View last 100 lines
tail -n 100 storage/logs/laravel.log
```

### Log Management
- **Configuration:** `config/logging.php`
- **Channels:** daily, stack, slack, stderr
- **Rotation:** Daily logs, kept for 14 days
- **Log Viewer:** Available at `/log-viewer` (config: `config/log-viewer.php`)

### Performance Monitoring & Log
```bash
# Logs Monitoring
http://channels.mevrik.com:9000/search

# Horizon monitoring
https://channels.mevrik.com/h0r1z0n

# Server metrics
htop
iotop
```



---

## 12. Known Issues and TODOs

### ⚠️ Pending Branch Merges (Ready for Final Test)

| Branch | Feature | Target Branch | Status | Action Required |
|--------|---------|---------------|--------|-----------------|
| `nazrul/gp-otp` | MyGP Push Notification | `channels_v4` | Pull request created | Final testing and merge |
| `nazrul/golive` | Soketi Channels Changes | `channels_v4` | Pull request created | Final testing and merge |

**Important:** Both branches have pull requests ready and need final testing before merging to production.

### TODOs

#### High Priority Tasks
- [ ] **Merge `nazrul/gp-otp` branch** - MyGP push notification (after final test)
- [ ] **Merge `nazrul/golive` branch** - Skitto channels changes (after final test)

#### Medium Priority Tasks
- [ ] Optimize database queries in analytics
- [ ] Document custom packages (Efemer, Genex)

### Technical Debt
- Some service classes exceed 500 lines - consider splitting
- Missing test coverage for several controllers
- Some legacy code patterns should be refactored to modern Laravel

---

## 13. Documentation and References

### Project Documentation
```
docs/
├── HANDOVER.md                          - This document
├── ONBOARDING.md                        - Developer onboarding guide
├── architecture-tmpl.md                 - Architecture template
├── messages-controller-prd.md           - Messages controller requirements
├── mevrik-channels-*-prd.md            - Various PRD documents
└── INTENT_MAPPING.md                   - GP APIs wise INTENT mapping documentation
```

### Code Documentation
- **PHPDoc:** Used throughout codebase
- **README:** See `repos/mevrik-channels-php/README.md`
- **Dev Notes:** See `repos/mevrik-channels-php/dev-notes.md`

### External Resources
- **Laravel Documentation:** https://laravel.com/docs
- **Facebook Messenger Platform:** https://developers.facebook.com/docs/messenger-platform


### Useful Commands Cheatsheet
```bash

# Debugging
php artisan tinker

# Check an Environment Variable
php artisan tinker
> env('APP_ENV');

# Check Redis Connection
php artisan tinker
> Redis::connection()->ping();

# Check Database Connection
php artisan tinker
> Genex\Channels\Factory\Models\Message::first();

```

---

## 14. Contacts and Ownership

### Project Ownership
- **Product Owner:** [Name, email@company.com]
- **Tech Lead:** [Name, email@company.com]
- **DevOps Lead:** [Name, email@company.com]

### Development Team
- **Backend Lead:** [Name, email@company.com]
- **Frontend Developer:** [Name, email@company.com]
- **QA Engineer:** [Name, email@company.com]

### Emergency Contacts
- **Production Issues:** [Emergency email/phone]
- **Security Issues:** [Security team contact]
- **Database Issues:** [DBA contact]

---


## 15. Quick Start for New Developers

### Day 1 Setup
```bash
# Clone repository
git clone [repository-url]
cd mevrik-channels-php

# Copy environment file
cp .env.example .env
# Edit .env with provided credentials

# Install dependencies
composer install

# Generate application key (If needed)
php artisan key:generate

# Run migrations
php artisan migrate


# Start local server
php artisan serve

# In another terminal
php artisan queue:work --queue=default --tries=3 --timeout=90
```

## 📞 Need Help?

If you encounter any issues or have questions:
1. Check this `docs/HANDOVER.MD` document
2. Review `docs/ONBOARDING.md`
3. Ask in `#devhub` Slack channel
5. Contact the previous developer: [`nazrulcst@gmail.com` | `01867536941`]
6. Escalate to Tech Lead: [email]

---

*Good luck with the project! 🎉*
