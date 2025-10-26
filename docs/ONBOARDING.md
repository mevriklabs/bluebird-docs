# 🚀 Project Onboarding Guide

Welcome aboard!  
This guide will help you set up the environment, understand the architecture, and start contributing to the project efficiently.

---

## 🧩 Tech Stack Overview

| Component | Description |
|------------|-------------|
| **Language** | PHP 8.2 (FPM) |
| **Framework** | Laravel Version-9
| **Core Package** | [Loris Leiva – Laravel Actions](https://github.com/lorisleiva/laravel-actions) |
| **Queue Manager** | Laravel Horizon |
| **Cache / Queue Driver** | Redis |
| **Process Manager** | Supervisor |
| **Databases** | MySQL (primary)(reports), PostgreSQL (queue,user management), ClickHouse (GP static bot package, kcp,kmp number matching) |
| **Web Server** | Nginx (Production)
| **OS** | Ubuntu Linux | (Production)

---

## ⚙️ Local Development Setup

### 1. Prerequisites
- PHP ≥ 8.2 with FPM
- Composer
- MySQL & PostgreSQL
- Redis
- ClickHouse (optional for reports)
- Git
- Nginx (or Valet/Sail for local serving)

---


### 2. Clone the Repository

```bash
git clone https://github.com/genexgit/mevrik-channels-php.git
cd mevrik-channels-php
```

---

### 3. Install Dependencies
```bash
cd mevrik-channels-php
composer install
```

---

### 4. Environment Setup
```bash
cp .env.example .env
```
Then update database, Redis, and app URLs as needed:
```bash
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=
DB_HOST_READ=
DB_HOST_BROWSER=
DB_DATABASE=
DB_PORT=3306
DB_USERNAME=
DB_PASSWORD=

# EMAIL
DB_DATABASE_EMAIL=
DB_HOST_EMAIL=
DB_HOST_EMAIL_READ=

# SOCIAL
DB_CONNECTION_SOCIAL=
DB_DATABASE_SOCIAL=

### POSTGRES DATABASE
PG_DATABASE_URL=
PG_DB_HOST=
PG_DB_DATABASE=
PG_DB_PORT=5432
PG_DB_USERNAME=
PG_DB_PASSWORD=

### CLICKHOUSE
CLICKHOUSE_HOST=
CLICKHOUSE_DATABASE=
CLICKHOUSE_PASSWORD=
CLICKHOUSE_PORT=
CLICKHOUSE_USERNAME=
CLICKHOUSE_TIMEOUT_CONNECT=2
CLICKHOUSE_TIMEOUT_QUERY=2

REDIS_HOST=
REDIS_PASSWORD=
REDIS_PORT=6379
REDIS_DB=
REDIS_CACHE_DB=
REDIS_CACHE_EXPIRE=
REDIS_HOST_DATA=

QUEUE_CONNECTION=redis

### Google Dialog flow Credentials
GOOGLE_PROJECT_ID=
GOOGLE_PROJECT_SMALLTALK_ID=
GOOGLE_PROJECT_MEVRIK=
GOOGLE_PROJECT_SMALLTALK=
DIALOGUE_DOMAIN_SCORE=
DIALOGUE_SMALL_SCORE=

### AWS
FILESYSTEM_DRIVER=s3
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=
AWS_BUCKET=bluebirdchannel
AWS_BUCKET_EMAIl=bluebirdemail
AWS_CDN=https://cdn.mevrik.com
AWS_BUCKET_EMAIL_PREVIOUS=bluebirdemailx

### SFTP Server
SFTP_HOST=
SFTP_USERNAME=
SFTP_PASSWORD=
```
---

### 5. Start the Application (Local Developemnt)
```bash
php artisan serve
```
---

### 6. Application Architecture

Action-Based Structure (Loris Leiva):
The application uses the Laravel Actions pattern instead of traditional controllers.
Each feature is represented by a dedicated Action class.

Example Action: 
```bash
use Lorisleiva\Actions\Concerns\AsAction;
class SendMessage
{
    use AsAction;

    public function handle(array $data)
    {
        //write your logic here.
    }

    function asJob(array $data) {
        $this->handle($data);
    }
}
```
Usage Example : 
```bash
// Run directly (like a controller or service)
SendMessage::run($data);
// Dispatch as a queued job
SendMessage::dispatch($data);
```

---

## Databases:

MySQL: Primary transactional data (core business logic) message, case, session, customer, email_messages, social feed (comments) and all reports management.

PostgreSQL: Core User, Queue, Channels management.

ClickHouse: GP Static offer, Channel TOken, KCP, KPM user management.


Queues & Jobs:
Managed by Laravel Horizon for background processing.
Use Redis as the queue driver.

Supervisor:
Used to keep Horizon and scheduled workers running.

---

## 🚀 Deployment ( via SSH)

Steps 1: SSH into the server:
  ```ssh user@server_ip```

---
Steps 2: Navigate to the project directory:
  ```cd /var/www/project```

---
Steps 3: Pull latest changes:
  ```git pull origin branch``` (check the currecnt branch)

---
Steps 4: Install new dependencies (if any):
  ```composer install --no-dev --optimize-autoloader```

---
Steps 5: Clear and optimize cache:
  ```php artisan optimize:clear```
  ```php artisan optimize```

---

Steps 6: Restart queue workers and Horizon:
  ```php artisan horizon:terminate```
  ```sudo supervisorctl restart all```

--- 

## 🧩 Best Practices

Follow PSR-12 coding standards.

Use feature branches (e.g., feature/add-reporting-endpoint).

Test before pushing.

Use Loris Leiva Actions for new features — avoid creating new controllers directly.

---
## 🔗 References

Laravel Actions: https://github.com/lorisleiva/laravel-actions

Laravel Horizon Docs: https://laravel.com/docs/horizon

Supervisor Docs: http://supervisord.org

Redis CLI: https://redis.io/docs

---

## ✅ Final Notes

Restart Horizon whenever code or env changes.

Monitor job queues from Horizon dashboard.

For production, make sure:

PHP-FPM and Redis are running.

Supervisor is monitoring Horizon.

---